---
title: 尝试react-hooks过程中踩过的坑
date: 2018-11-13 11:41:36
tags:
- React
- Functional Programming
- javascript
- framework
---

### 报错“TypeError: dispatcher.useState is not a function”

- 除了react版本要升级到16.7.0-alpha.0之外，react-dom也要升级到相应的版本。

```json
  "dependencies": {
    "react": "16.7.0-alpha.0",
    "react-dom": "16.7.0-alpha.0",
    "react-scripts": "2.1.1"
  },
```

### useEffect和直接写有什么不同

- useEffect相当于componentDidMount and componentDidUpdate，发生在render之后。时序不一样
- In React class components, the render method itself shouldn’t cause side effects. It would be too early — we typically want to perform our effects after React has updated the DOM. This is why in React classes, we put side effects into componentDidMount and componentDidUpdate.

### 如何避免不必要的effect对performance的负面影响

```javascript
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // Only re-run the effect if count changes
```

### 如何在不同的hooks之间共享数据

```javascript
const [recipientID, setRecipientID] = useState(1);
const isRecipientOnline = useFriendStatus(recipientID);
```

### 用function component怎样实现PureComponent？

```javascript
const Button = React.memo((props) => {
  // your component
});
```

useMemo：

```javascript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);

function Parent({ a, b }) {
  // Only re-rendered if `a` changes:
  const child1 = useMemo(() => <Child1 a={a} />, [a]);
  // Only re-rendered if `b` changes:
  const child2 = useMemo(() => <Child2 b={b} />, [b]);
  return (
    <>
      {child1}
      {child2}
    </>
  )
}
```