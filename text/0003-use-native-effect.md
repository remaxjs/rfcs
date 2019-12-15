- 日期: 2019-12-15
- RFC PR:
- Issue:

# 总结

添加一个 effect hook，每当 setData callback 执行时，触发 hook。

# 基本例子

```tsx
import { View, useNativeEffect } from 'remax';

function Page() {
  const [state, setState] = useState(1);
  useNativeEffect(() => {
    // 当原生 setData 执行后，并且 state 发生变化，触发这个回调
    // do sth
  }, [state]);

  return <View>page</View>;
}
```

# 背景

由于 Remax 通过 `setData` 触发小程序更新，因此 React 的 `useEffect` `useLayoutEffect` hooks 仅仅对应于 Remax 虚拟 dom 的更新操作。为了让开发者可以在 `setData` 行为真正执行后执行逻辑，我们需要暴露 `setData callback` 给开发者。

# 详细设计

```ts
/**
  * @param callback setData 执行后的回调函数
  * @param ids 唯一标识，只有 ids 发生变化才会执行 callback
  */
type useNativeEffect = (callback: () => void, ids: any[]);
```

```tsx
import { View, useNativeEffect } from 'remax';

function Page() {
  const [state, setState] = useState(1);
  useNativeEffect(() => {
    // ids 为空 setData 触发都会执行
  }, []);

  return <View>page</View>;
}
```

```tsx
import { View, useNativeEffect } from 'remax';

function Page() {
  const [state, setState] = useState(1);
  useNativeEffect(() => {
    // 没有传 ids， setData 触发都会执行
  });

  return <View>page</View>;
}
```

```tsx
import { View, useNativeEffect } from 'remax';

function Page() {
  const [state, setState] = useState(1);
  useNativeEffect(() => {
    // 当原生 setData 执行后，并且 state 发生变化，触发这个回调
    // do sth
  }, [state]);

  return <View>page</View>;
}
```

# 缺点

无

# 其他可选方案

无

# 其他问题

无
