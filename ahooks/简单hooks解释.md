# 简单hooks解释

```typescript
// 传一个初始值和反转值
function useToggle<D, R>(defaultValue: D = false as unknown as D, reverseValue?: R) { //
  const [state, setState] = useState<D | R>(defaultValue);

  const actions = useMemo(() => { // action 是个固定值，不会随着组件的状态更新而变化
    const reverseValueOrigin = (reverseValue === undefined ? !defaultValue : reverseValue) as D | R;
    // 返回四个方法设置state值
    const toggle = () => setState((s) => (s === defaultValue ? reverseValueOrigin : defaultValue));
    const set = (value: D | R) => setState(value);
    const setLeft = () => setState(defaultValue);
    const setRight = () => setState(reverseValueOrigin);

    return {
      toggle,
      set,
      setLeft,
      setRight,
    };
    // useToggle ignore value change
    // }, [defaultValue, reverseValue]);
  }, []);

  return [state, actions]; // 返回 state和actions
}
```

``` typescript
  //对useToggle参数的限制 ，限制为boolean类型
  export default function useBoolean(defaultValue = false): [boolean, Actions] {
  const [state, { toggle, set }] = useToggle(!!defaultValue);

  const actions: Actions = useMemo(() => {
    const setTrue = () => set(true);
    const setFalse = () => set(false);
    return {
      toggle,
      set: (v) => set(!!v),
      setTrue,
      setFalse,
    };
  }, []);

  return [state, actions];
}
```

```typescript
// 每次取最新值
function useLatest<T>(value: T) {
  const ref = useRef(value);
  ref.current = value;

  return ref;
}

```

```typescript
// 用于强制组件重新渲染
//在 JavaScript 中，两个空对象 {} 不相等是因为对象是引用类型。每次创建一个对象时，都会在内存中分配一个新的地址，即使两个对象的内容完全相同，它们的引用地址也是不同的。因此，比较两个对象时，比较的是它们的引用地址，而不是它们的内容。
const useUpdate = () => {
  const [, setState] = useState({});

  return useCallback(() => setState({}), []);// 前后值不相等，触发更新
};
```

```typescript
// 只有依赖变化和初始化的时候构造一次factory
function useCreation<T>(factory: () => T, deps: DependencyList) {
  const { current } = useRef({
    deps,
    obj: undefined as undefined | T,
    initialized: false,
  });
  if (current.initialized === false || !depsAreSame(current.deps, deps)) {// 初始化和依赖变化时构造
    current.deps = deps;
    current.obj = factory();
    current.initialized = true;
  }
  return current.obj as T;
}
```

```typescript
// 组件加载的时候执行一次fn
const useMount = (fn: () => void) => {
  useEffect(() => {
    fn?.();
  }, []);
};

```

```typescript
// 组件加载的时候执行
const useUnmount = (fn: () => void) => {
  const fnRef = useLatest(fn); // 执行fn的最新值
  useEffect(
    () => () => {
      fnRef.current();
    },
    [],
  );
}
//组件卸载的时候执行
const useUnmount = (fn: () => void) => {
  const fnRef = useLatest(fn);
  useEffect(
    () => () => { // 返回destroy函数，组件卸载的时候会执行
      fnRef.current();
    },
    [],
  );
};

```

```typescript
// 传入的fn是最新的，会更新
// 返回的fn不会更新，是最初的一个
function useMemoizedFn<T extends noop>(fn: T) {
  const fnRef = useRef<T>(fn);
  // why not write `fnRef.current = fn`?
  // https://github.com/alibaba/hooks/issues/728
  // const cachedValue = useMemo(calculateValue, dependencies) deps变化重新计算
  fnRef.current = useMemo<T>(() => fn, [fn]); // 保持fn为最新值
  const memoizedFn = useRef<PickFunction<T>>();
  if (!memoizedFn.current) {// 只会赋值一次
    memoizedFn.current = function (this, ...args) {
      return fnRef.current.apply(this, args);
    };
  }

  return memoizedFn.current as T;
}
```

```typescript
// 和useEffect类似 ，第一次不执行effect
 useUpdateEffect =  createUpdateEffect(useEffect);

 const createUpdateEffect: (hook: EffectHookType) => EffectHookType =
  (hook) => (effect, deps) => {
    const isMounted = useRef(false);

    // for react-refresh
    hook(() => { // 组件卸载触发 重新计算是否是第一次mounted
      return () => {
        isMounted.current = false;
      };
    }, []);

    hook(() => {
      if (!isMounted.current) { // 第一次efffect不执行
        isMounted.current = true;
      } else {
        return effect(); //执行deps更新后的effect
      }
    }, deps);
  };


```

