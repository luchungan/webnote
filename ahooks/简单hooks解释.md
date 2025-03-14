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