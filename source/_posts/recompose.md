---
title: recompose
date: 2018-07-06 11:26:26
tags:
---

https://github.com/acdlite/recompose

# Background

## React不喜欢OO
[Composition vs. inheritance](https://reactjs.org/docs/composition-vs-inheritance.html#so-what-about-inheritance)

这里列出来结论就是，React官方旗帜鲜明的反对使用继承，并且在他们大量的实践中这个观点得到了证明。可以从官方文档和其他资料中学习到更多相关的讨论和practice。但是有一个事实是无法逃避的：如果在react工程中用OO的思想来组织代码，是会非常痛苦的。他们的绝大多数api和pattern都是非常鲜明的函数式编程的哲学。而全家桶当中其他一些优秀的library，以redux为首，更是非常纯洁的FP思维，用OO会寸步难行。

不过很搞笑的是，React提供的component api，包括各种lifecycle hooks啥的，真的很OO-like。

## Funcional programming:
http://ux.asia.microsoft.com/2017/12/25/the-functional-approach-to-frontend-development/

## React HOC:

> <i>Higher-Order Components (HOCs) are JavaScript functions which add functionality to existing component classes.</i>

比如说我们有一个基础的grid，可以单纯的画格子。

```jsx
function MyGrid(props) {
  return (
    <table>
      {/* ...code */}
    </table>
  );
}
```

可以有一个HOC，给这个grid加上pagination

```jsx
const PaginableGrid = withPaging(MyGrid);
```

还可以再套一层，让grid从不同的地方获取数据

```jsx
const PaginableGridWithOdataSource = withOdataSource(withPaging(MyGrid));
const PaginableGridWithMemorySource = withMemorySource(withPaging(MyGrid));
```

withPaging也可以用来wrap别的东西

```jsx
const PaginableList = withPaging(MyList);
```

错误的写法,也是习惯了OO思想的人最容易写的写法:

```jsx
class PaginableGrid extends MyGrid {
  getGridData(params) {
    const rawData = super.getGridData(params);

    return handlePagination(rawData, /* ...params */)
  }
}
```

甚至还会用factory pattern之类的方法让设计更优雅，结果却是南辕北辙。

不好的写法，也是我们的项目里最常见的写法是，把pagination的实现直接写到原来的component里头去，然后在装模做样的在props里加一个isPaginationEnabled的flag。等发现component变得很臃肿之后再想方设法的拆。

## Why I want to sell recompose to everyone?

- recompose更多的扮演了一个语法糖的角色：事实上它没有为react扩展任何的功能。甚至有些可以用react可以干的事它（暂时？）还干不了。
- 个人观点：不管是OO还是FP，都有很好的pattern来帮助我们写出很优雅高效的代码。然而，混起来用是非常恶心的，就像我们之前写react component一样。
- 使用recompose，对于绝大多数情况，尤其是我们处理各种特性和共性问题、扩展基础组件的需求，可以让我们写出非常纯净的函数式编程的代码，并且减少很多不必要的用jsx来描述的层次结构和逻辑嵌套。

### Example 1

setState不是纯函数。
Component中为了初始化state无法避免副作用。

```jsx
const enhance = withState('counter', 'setCounter', 0)
const Counter = enhance(({ counter, setCounter }) =>
  <div>
    Count: {counter}
    <button onClick={() => setCounter(n => n + 1)}>Increment</button>
    <button onClick={() => setCounter(n => n - 1)}>Decrement</button>
  </div>
)
```

### Example 2

使用HOC或者render props会产生大量嵌套

```jsx
import { fromRenderProps } from 'recompose';
const { Consumer: ThemeConsumer } = React.createContext({ theme: 'dark' });
const { Consumer: I18NConsumer } = React.createContext({ i18n: 'en' });
const RenderPropsComponent = ({ render, value }) => render({ value: 1 });

const EnhancedApp = compose(
  // Context (Function as Child Components)
  fromRenderProps(ThemeConsumer, ({ theme }) => ({ theme })),
  fromRenderProps(I18NConsumer, ({ i18n }) => ({ locale: i18n })),
  // Render props
  fromRenderProps(RenderPropsComponent, ({ value }) => ({ value }), 'render'),
)(App);

// Same as
const EnhancedApp = () => (
  <ThemeConsumer>
    {({ theme }) => (
      <I18NConsumer>
        {({ i18n }) => (
          <RenderPropsComponent
            render={({ value }) => (
              <App theme={theme} locale={i18n} value={value} />
            )}
          />
        )}
      </I18NConsumer>
    )}
  </ThemeConsumer>
)
```
# Recompose

## 现状

贴一下我再组内讨论中的原话吧：

I think recompose is not ready for production yet for the reasons: some apis are unstable. The official documentation lacks of examples and even contains some unreleased features. And it does not support some of the latest React features like new context apis. I have to search the pull request and patch the unreleased APIs myself.

I love it and I believe it's the correct direction. From my point of view, I prefer composing functions over JSX or hierarchies. I think either OO or functional programming is good if we follow correct patterns and I personally hate mix up of them. With the help of recompose, we can use pure FP and excape the fact that React/Redux team is proposing FP phylosophy but is providing OO-like component/factory APIs.

The debug experience is good to me. If you make each "step" a pure function you can set breakpoints for the entry and return value to solve most of the problems. One thing I feel unhappy is that if not careful, there will be a lot of "unknown" components when inspecting via React debug tool and no good way to know where it comes from at the first glance.

## 跳过API的部分，可以refer官方文档。

## Some good practice

### Easy debug

```js
const debug = withProps(console.log)

const enhance = compose(
  withState(/*...args*/),
  debug, // print out the props here
  mapProps(/*...args*/),
  pure
)
```

### Easy custom API

```js
const renderNothingIfNoSource = hasNoSource => branch(hasNoSource, renderNothing);

const enhance = compose(
  setPropTypes(/*...args*/),
  defaultProps(/*...args*/),
  renderNothingIfNoSource(({ source }) => _.isNil(source) || _.isEmpty(source)),
  withProps(/*...args*/),
);

export default enhance(SomeComponent);

```

```js
const withDataSourceOdata = ({/*...args*/}) => withProps({
  options: {
    data: await odataRequest(/*...args*/),
  }
})
```