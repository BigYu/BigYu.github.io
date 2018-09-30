---
title: 为什么不能在proptypes的intanceof来检查React component类型
date: 2018-09-30 10:58:07
tags:
---

# 问题

```jsx
propTypes: {
  children: React.PropTypes.instanceOf(OtherComponent)
}
```

这样写是不起作用的。<OtherComponent> 在instanceOf的检查中是不通过的。

# 为什么？

看一下React源码：https://github.com/facebook/react/blob/master/packages/react/src/ReactElement.js#L171

可以看到<OtherComponent> 并不是单纯new出来的，当然不能用简单的instanceOf来检查了

# Children是什么

测试代码见： https://github.com/BigYu/react-perf-test/blob/940ab0dabd406af0a32dccb5c025833816c3bfab/instance-of-component/src/App.js#L12

结果：

```jsx
(2) [{…}, {…}]
0:
$$typeof: Symbol(react.element)
key: null
props: {children: " This is child Title"}
ref: null
type: "h1"
_owner: FiberNode {tag: 2, key: null, type: ƒ, stateNode: App, return: FiberNode, …}
_store: {validated: true}
_self: App {props: {…}, context: {…}, refs: {…}, updater: {…}, _reactInternalFiber: FiberNode, …}
_source: {fileName: "D:\CodeBase\react-perf-test\instance-of-component\src\App.js", lineNumber: 10}
__proto__: Object
1:
$$typeof: Symbol(react.element)
key: null
props: {}
ref: null
type: ƒ GrandChild()
_owner: FiberNode {tag: 2, key: null, type: ƒ, stateNode: App, return: FiberNode, …}
_store: {validated: true}
_self: App {props: {…}, context: {…}, refs: {…}, updater: {…}, _reactInternalFiber: FiberNode, …}
_source: {fileName: "D:\CodeBase\react-perf-test\instance-of-component\src\App.js", lineNumber: 11}
__proto__: Object
length: 2
__proto__: Array(0)
```

# 结论

- 可以用shape来进行精确的检查
  - 不要偷懒用object或者any，那样等于没有检查。
- 希望proptypes尽快支持原生检查
  - 本来还想自己contribute一下的，已经被别人先做了： https://github.com/facebook/prop-types/pull/211 不知道哪天能merge然后release
- 其实吧，personally, I prefer render function over elements.