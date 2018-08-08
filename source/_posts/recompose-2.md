---
title: recompose - 从一个小例子说起
date: 2018-07-13 14:12:04
tags:
- React
- recompose
- Design pattern
- Functional programming
---

https://github.com/acdlite/recompose

# Recompose是用来干嘛的？

Recompose是用来refactor react code的神器。我个人还是比较功利的，代码让人feel painful才需要refactor，而不是盲目的追求fashion之类的。

# 我们的代码如何变坏？

从一个小例子说起。我们画一个Card，里面有一个grid用来展示data。

```jsx
class MyCard extends React.Component {
  // ...lifecycle hooks
  render() {
    const { data } = this.props;

    return (
      <Grid data={data}/>
    );
  }
}

```

现在我们有一个很正常的需求：当我们没有数据的时候，就啥的不显示，以免grid
component报错。

我们每个人都sign off过这样的代码：

```jsx
class MyCard extends React.Component {
  // ...lifecycle hooks
  render() {
    const { data } = this.props;
      if (_.isEmpty(data)) {
      return null;
    }

    return (
      <Grid data={data}/>
    );
  }
}

```

这其实也无伤大雅。但是，很多时候，我们又得到了更多的需求，比如说，如果有error，我们就画一张哭脸。

sign off过这样代码的都需要反省一下，比如说我自己。。。

```jsx
class MyCard extends React.Component {
  // ...lifecycle hooks

  render() {
    const { data } = this.props;
    if (_.isEmpty(data)) {
      return null;
    }

    if (data.status === ‘error’) {
      return <CryingFace />;
    }

    return (
      <Grid data={data}/>
    );
  }
}

```

这种行为本身，就违背了对修改关闭，对扩展开放的open close原则。之后顺着这条道路走下去，我们讲得到爆炸式的一大串if或者switch。比如说我们的判断条件会变的越来越复杂，我们处理异常的方式也会不停的改变。

当然，我们可以这样解决：

```jsx
class MyCard extends React.Component {
  constructor(props) {
    super(props);
    const {
      dataValidator = _.constant(true),
      conditionalRenderer = () => null,
    } = props;
    this.dataValidator = dataValidator;
    this.conditionalRenderer = conditionalRenderer;
  }

  render() {
    // ...
  }
}

```

（当然我们也可以用一些design pattern，比如说这里可以考虑抽象工厂或者模板方法模式，让这段code更加优雅可扩展。这里不讨论。）

这样做解决了上面提到的一些问题，带来的新问题是：

1. 文档不够清楚的话，这个shared component share出去，基本上是没人会用的。。。
2. 很多个页面里，我们确实只需要一个简单的，没有data的时候显示一个default message的component，我们可能要把这个重复的dataValidator和conditionalRender传很多遍。

在React里，这样写是犯法的：

```jsx
class CardWithEmptyDataHandler extends Mycard {
  constructor(props) {
    super(_.defaults({}, {
      dataValidator: ({ data }) => {
        if (_.isEmpty(data)) {
          return 'empty';
        }
        return true;
      },
      conditionalRenderer: (result, defaultRenderer) => {
        switch(result) {
          case 'empty':
            return null;
          default:
            return defaultRenderer();
        }
      },
    }, props));
  }
}


```

## React: Composition vs Inheritance
- Use composition instead of inheritance to reuse code between components.
- E.g. Containment and Specialization
- At Facebook, we use React in thousands of components, and we haven’t found any use cases where we would recommend creating component inheritance hierarchies.

## React: Use HOCs For Cross-Cutting Concerns

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
# Recompose - Make HOCs easy to comprehend and act as the basic “bricks” to build apps.

用Recompose，我们可以这样写：

```jsx
const renderNothingIfNoData = hasNoData => branch(hasNoData, renderNothing);
const renderCryingFaceIfBadData = isBadData => branch(isBadData, renderCryingFace);

const enhance = compose(
  /* ...more wrappers */
  renderNothingIfNoData(({ data }) => _.isEmpty(data)),
  renderCryingFaceIfBadData(customeValidationHandler)
  /* ...more wrappers */
);
export default ComponentWithConditionalRendering = enhance(SomeComponent);

```

# Recompose - Pure functional programming and  helper functions to...

## lift state into functional wrappers.

下面是一个简单的计数器的例子，在react里非常常见：

```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { counter: 0 };
    this.handleDecrement = this.handleDecrement.bind(this);
    this.handleIncrement = this.handleIncrement.bind(this);
  }
  handleIncrement() {
    this.setState({ counter: this.state.counter + 1 });
  }
  handleDecrement() {
    this.setState({ counter: this.state.counter – 1 });
  }
  render() {
    return (
     <div>
       Count: {this.state.counter}
       <button onClick={this.handleIncrement}>Increment</button>
       <button onClick={this.handleDecrement}>Decrement</button>
     </div>
    );
  }
}

```

这是一段比较规范的react代码，肯定不能说有问题。但是有几个地方老会让人觉得不爽：

- 太长了。。。
- 像super(props); this.handleDecrement = this.handleDecrement.bind(this); 这样没有营养的代码，我们是不是写得有点多了。
- this.state = { counter: 0 }; 这是一个mutable的表达，从哲学上来说和react一直提倡的immutable是相违背的。
  - 副作用就是：如果我们认真学习了thinkings in React,然后想规范自己的代码，引入了eslint-plugin-immutable，然后就发现，lint挂了。
- this.setState({ counter: this.state.counter + 1 }); 这不是纯函数。

用recompose:

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

Or with a Redux-style reducer:

```jsx
const counterReducer = (count, action) => {
  switch (action.type) {
    case INCREMENT:
      return count + 1
    case DECREMENT:
      return count - 1
    default:
      return count
  }
}
const enhance = withReducer('counter', 'dispatch', counterReducer, 0)
const Counter = enhance(({ counter, dispatch }) =>
  <div>
    Count: {counter}
    <button onClick={() => dispatch({ type: INCREMENT })}>Increment</button>
    <button onClick={() => dispatch({ type: DECREMENT })}>Decrement</button>
  </div>
)


```

# Recompose - perform the most common React patterns

每一个学习React的码农都会接触flux，其中的大多数会称赞它的优美，严谨，完备，然而很少人会真的在产品中使用，因为复杂，晦涩。

用recompose来写这些common pattern,效果就不一样了。

```jsx
const provide = store => withContext(
  { store: PropTypes.object },
  () => ({ store })
)
// Apply to base component
// Descendants of App have access to context.store
const AppWithContext = provide(store)(App)


```

# Recompose - optimize rendering performance

```jsx
// A component that is expensive to render
const ExpensiveComponent = ({ propA, propB }) => {...}
// Optimized version of same component, using shallow comparison of props
// Same effect as extending React.PureComponent
const OptimizedComponent = pure(ExpensiveComponent)
// Even more optimized: only updates if specific prop keys have changed
const HyperOptimizedComponent = onlyUpdateForKeys(['propA', 'propB'])(ExpensiveComponent)

```

我们在优化react component的performance的时候，经常使用pureComponent，也就是说props变了才会重新render。Recompose的pure可以做这件事。

我们经常可以做更进一步的优化，比如说有一个东西，只有data变化的时候才重新render，其他的变化不应该触发render，就可以用onlyUpdateForKeys来做。

# Recompose - build your own libraries

比如说我们刚才一不小心就写出来两个自己的util接口:

- renderNothingIfNoData
- renderCryingFaceIfBadData

再来一个：

```jsx
const withCustomerId = (customerId) => withProps(props => ({
  currentCustomer : { Id: customerId, ...props.currentCustomer }
}));

```

# Recompose - Flattening HOCs hierarchies into pipelines.

Less JSX. Even less code.

优化前：

```jsx
const { Consumer: ThemeConsumer } = React.createContext({ theme: 'dark' });
const { Consumer: I18NConsumer } = React.createContext({ i18n: 'en' });
const RenderPropsComponent = ({ render, value }) => render({ value: 1 });
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

优化后：

```jsx
const { Consumer: ThemeConsumer } = React.createContext({ theme: 'dark' });
const { Consumer: I18NConsumer } = React.createContext({ i18n: 'en' });
const RenderPropsComponent = ({ render, value }) => render({ value: 1 });
const EnhancedApp = compose(
  // Context (Function as Child Components)
  fromRenderProps(ThemeConsumer, ({ theme }) => ({ theme })),
  fromRenderProps(I18NConsumer, ({ i18n }) => ({ locale: i18n })),
  // Render props
  fromRenderProps(RenderPropsComponent, ({ value }) => ({ value }),
    'render'),
)(App);

```

# Recompose - unit testing

- Writing pure functions is helpful for unit testing: I/O tests and no side effects.
  - https://github.com/jfmengels/eslint-plugin-fp
  - https://github.com/jhusain/eslint-plugin-immutable
  - No let; No this; No mutable.
- Easy snapshot test for the very basic component.

# Recompose - debug

这是目前recompose比较让人头疼的地方。详见这个issue：https://github.com/acdlite/recompose/issues/97

但是有两个小tips可以帮助debug

- Use “withDisplayName”
- withProps(console.log)

