
### useLocalStorageState

```typescript
  ///将状态存储在 localStorage 中的 Hook 
  // createUseStorageState 函数 相当于给 内部函数传递一个localstorage 全局参数
  const useLocalStorageState = createUseStorageState(() => (isBrowser ? localStorage : undefined));

  useSessionStorageState = createUseStorageState(() => isBrowser ? sessionStorage : undefined);
```

```typescript
export function createUseStorageState(getStorage: () => Storage | undefined) {
  function useStorageState<T>(key: string, options: Options<T> = {}) {
    let storage: Storage | undefined;
    const {
      listenStorageChange = false,
      onError = (e) => {
        console.error(e);
      },
    } = options;

    // https://github.com/alibaba/hooks/issues/800
    try {
      storage = getStorage();
    } catch (err) {
      onError(err);
    }

    const serializer = (value: T) => { // 序列化 默认JSON.stringify
      if (options.serializer) {
        return options.serializer(value);
      }
      return JSON.stringify(value);
    };

    const deserializer = (value: string): T => { // 反序列化 默认JSON.parse
      if (options.deserializer) {
        return options.deserializer(value);
      }
      return JSON.parse(value);
    };

    function getStoredValue() { // 获取存储的值
      try {
        const raw = storage?.getItem(key);
        if (raw) {
          return deserializer(raw); // 反序列化
        }
      } catch (e) {
        onError(e);
      }
      if (isFunction(options.defaultValue)) { //storage 没有获取到值
        return options.defaultValue();
      }
      return options.defaultValue;
    }

    const [state, setState] = useState(getStoredValue); // 获取值

    useUpdateEffect(() => {
      setState(getStoredValue());
    }, [key]); // key变化 重新获取

    const updateState = (value?: SetState<T>) => { // 更新值
      const currentState = isFunction(value) ? value(state) : value;

      if (!listenStorageChange) { // 不监听storage变化
        setState(currentState);
      }

      try {
        let newValue: string | null;
        const oldValue = storage?.getItem(key);

        if (isUndef(currentState)) { // 更新值是undefined ，清除storage
          newValue = null;
          storage?.removeItem(key);
        } else {
          newValue = serializer(currentState);
          storage?.setItem(key, newValue); // 更新storage 设置序列化后的值
        }
        // 触发一个自定义事件
        dispatchEvent(
          // send custom event to communicate within same page
          // importantly this should not be a StorageEvent since those cannot
          // be constructed with a non-built-in storage area
          new CustomEvent(SYNC_STORAGE_EVENT_NAME, {
            detail: {
              key,
              newValue,
              oldValue,
              storageArea: storage,
            },
          }),
        );
      } catch (e) {
        onError(e);
      }
    };

    const syncState = (event: StorageEvent) => { // 同步storage方法
      if (event.key !== key || event.storageArea !== storage) {
        return;
      }

      setState(getStoredValue());
    };

    const syncStateFromCustomEvent = (event: CustomEvent<StorageEvent>) => { // 通过自定义事件同步storage
      syncState(event.detail);
    };

    // from another document // 不同tab之前的通信 通过storage事件 同一域名下都可以触发localstorage事件
    useEventListener('storage', syncState, {
      enable: listenStorageChange,
    });

    // from the same document but different hooks
    useEventListener(SYNC_STORAGE_EVENT_NAME, syncStateFromCustomEvent, {
      enable: listenStorageChange,
    });

    return [state, useMemoizedFn(updateState)] as const;
  }

  return useStorageState;
}
```