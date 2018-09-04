---
title: React什么时候会触发render
date: 2018-08-29 16:30:07
tags:
- react
- framework
- performance
---

我的测试代码（写得有点随意。。。）：https://github.com/BigYu/react-perf-test/tree/master/test-rerender

# 只要state变了，render就会被触发

实验了这样一个简单的component：

```jsx
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  componentDidMount() {
    setInterval(() => {
        this.setState(() => {
            console.log('setting state');
            return { unseen: "does not display" }
        });
    }, 1000);
}

  render() {
    console.log('render called');
    return (<div>Hello</div>);
  }
}

export default App;
```

打开debug tool 会发现，每一秒钟setting state和render called偶会被处罚

# 同时还会触发child的render

修改上面的实验，加一个child

```jsx
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';
import Child from './child';

class App extends Component {
  componentDidMount() {
    setInterval(() => {
        this.setState(() => {
            console.log('setting state');
            return { unseen: "does not display" }
        });
    }, 1000);
}

  render() {
    console.log('render called');
    return (
      <div>
        <Child />
        Hello
      </div>
    );
  }
}

export default App;
```

```jsx
import React from 'react';

export default class Child1 extends React.Component {
  render() {
    console.log('render child');

    return <div> This is child 1 </div>
  }
}
```

可以看到每一秒钟都输出了setting state, render called, render child

# PureComponent不会随着parent的render而重新render

给上面的例子加一个pure component的child

```jsx

import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';
import Child from './child';
import PureChild from './pure-child'

class App extends Component {
  componentDidMount() {
    setInterval(() => {
        this.setState(() => {
            console.log('setting state');
            return { unseen: "does not display" }
        });
    }, 1000);
}

  render() {
    console.log('render called');
    return (
      <div>
        <Child />
        <PureChild />
        Hello
      </div>
    );
  }
}

export default App;
```

```jsx
import React from 'react';

export default class PureChild extends React.PureComponent {
  render() {
    console.log('render pure child');

    return <div> This is child 1 </div>
  }
}
```

可以看到render pure child只会被触发一次

# Pure child的child不会被重复render

给上面例子的pure child再加上一个不pure的child 可以看到它也只被render了一次

```jsx
import React from 'react';
import Child from './child';

export default class PureChild extends React.PureComponent {
  render() {
    console.log('render pure child');

    return (
      <div>
        This is a pure child
        <Child />
      </div>
    );
  }
```

# 当props变化时，pure component会被重新render，包括他的children，哪怕它并没有用到这个变化的props

注意最后一句话！
注意最后一句话！
注意最后一句话！

```jsx
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';
import Child from './child';
import PureChild from './pure-child'

class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      count: 0,
    }
  }

  componentDidMount() {
    setInterval(() => {
        this.setState(() => {
            console.log('setting state');
            return { unseen: "does not display", count: this.state.count + 1 }
        });
    }, 1000);
}

  render() {
    console.log('render called');
    return (
      <div>
        <Child />
        <PureChild count={Math.floor(this.state.count / 10)}/>
        Hello
      </div>
    );
  }
}

export default App;

```

这样修改以后，可以看到，每10秒钟，pure component的render被触发了一次。

# 让shouldComponentUpdate 返回false，可以阻止component render

给出这样一个component

```jsx
import React from 'react';

export default class ChildCount extends React.Component {
  shouldComponentUpdate () {
    return false;
  }

  render() {
    console.log('render child count');

    return <div>{this.props.count}</div>
  }
}
```

```jsx
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';
import Child from './child';
import PureChild from './pure-child'
import ChildCount from './child-count';

class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      count: 0,
    }
  }

  componentDidMount() {
    setInterval(() => {
        this.setState(() => {
            console.log('setting state');
            return { unseen: "does not display", count: this.state.count + 1 }
        });
    }, 1000);
}

  render() {
    console.log('render called');
    return (
      <div>
        <Child />
        <PureChild count={Math.floor(this.state.count / 10)} />
        <ChildCount count={this.state.count} />
        Hello
      </div>
    );
  }
}

export default App;

```

会发现页面上一直显示0，也只render了一次