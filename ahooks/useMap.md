### useMap

```typescript
function useMap<K, T>(initialValue?: Iterable<readonly [K, T]>) {
  const getInitValue = () => new Map(initialValue); // 初始化值
  const [map, setMap] = useState<Map<K, T>>(getInitValue);

  const set = (key: K, entry: T) => { // 设置值
    setMap((prev) => {
      const temp = new Map(prev);
      temp.set(key, entry);
      return temp;
    });
  };

  const setAll = (newMap: Iterable<readonly [K, T]>) => { //重新设置map
    setMap(new Map(newMap));
  };

  const remove = (key: K) => { // 删除值
    setMap((prev) => {
      const temp = new Map(prev);
      temp.delete(key);
      return temp;
    });
  };

  const reset = () => setMap(getInitValue()); // 重置map

  const get = (key: K) => map.get(key); // 获取值

  return [
    map,
    {
      set: useMemoizedFn(set),
      setAll: useMemoizedFn(setAll),
      remove: useMemoizedFn(remove),
      reset: useMemoizedFn(reset),
      get: useMemoizedFn(get),
    },
  ] as const;
}
```