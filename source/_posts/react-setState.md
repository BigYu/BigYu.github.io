---
title: react setState 是同步还是异步
date: 2018-09-03 14:09:40
tags:
- react
- framework
---

之前被坑过，所以记得了setState是异步的，在setState的下一行取最新的state是取不到的。

真的一直是这样吗？实验一下：代码见：https://github.com/BigYu/react-perf-test/tree/master/test-set-state

分以下几种情况实验：

### 在lifecycle hooks中 - 异步

```jsx
  componentDidMount() {
    this.setState({
      count: this.state.count + 1,
    }, () => {
      console.log(`callback of setState inside handle click handler in componentDidMount: ${this.state.count}`);
    });

    console.log(`next line of handle click event in componentDidMount: ${this.state.count}`);

    document.getElementsByClassName('testDomButton')[0].addEventListener('click', this.handleClickDom.bind(this), false);
  }
```

测试后发现，第一次输出0，第二次输出1。在componentDidMount以及其他的lifecycle hooks中，setState是异步的。


### 在event callback 中 - 异步

这是平时最常见的一种情况，setState是异步的结论多数也是从这里来的，没什么疑问：

```jsx
<button onClick={this.handleClick}>Test react event</button>
```

```jsx
  handleClick() {
    this.setState({
      count: this.state.count + 1,
    }, () => {
      console.log(`callback of setState inside handle click handler: ${this.state.count}`);
    });

    console.log(`next line of handle click event: ${this.state.count}`);
  }

```

和之前类似，每次打印两个值，不一样。


### 在自己强行加上的dom even handler中 - 同步

这种写法实际估计没什么人这样干。。可能算是bug吧：

```jsx
<button onClick={this.handleClickSetTimeout}>Test react event with setTimeout</button>
```

```jsx
  componentDidMount() {
    console.log(`next line of handle click event in componentDidMount: ${this.state.count}`);

    document.getElementsByClassName('testDomButton')[0].addEventListener('click', this.handleClickDom.bind(this), false);
  }

  handleClickDom() {
    this.setState({
      count: this.state.count + 1,
    }, () => {
      console.log(`callback of setState inside handle click handler with dom addEventListener: ${this.state.count}`);
    });

    console.log(`next line of handle click event with dom addEventListener: ${this.state.count}`);
  }
```

神奇的发现，每次两个打印值是一样的。。。

### 在自己强行加上的异步逻辑setTimeout中 - 同步

比上面一种稍微正常一丢丢

```jsx
<button onClick={this.handleClickSetTimeout}>Test react event with setTimeout</button>
```

```jsx
  handleClickSetTimeout() {
    setTimeout(() => {
      this.setState({
        count: this.state.count + 1,
      }, () => {
        console.log(`callback of setState inside handle click handler with setTimeout: ${this.state.count}`);
      });

      console.log(`next line of handle click event with setTimeout: ${this.state.count}`);
    }, 100);
  }
```

发现这样每次打印的两个值也是一样的。。。
