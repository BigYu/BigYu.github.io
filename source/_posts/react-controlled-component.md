---
title: React.js Forms - Controlled Components
date: 2017-12-29 14:53:39
tags:
- React
- framework
---

(自行脑补评书起手式)话说上回咱们说到，UI = f(r(state, action))。这回咱们把这个r按下不表，单讲这个f...（评书听的少编不下去了=。=||）

不卖关子了，一个很很受欢迎和好评的f自然是react.js，其中一个很能体现f(state) = UI的概念就是controlled components。汉化过后好像叫受控组件，挺绕口的就不管了。道理都讲过，但是具体怎么写呢？有哪些坑？写一篇整理一下。

[快速通往code和demo的传送门](https://github.com/lorenseanstewart/react-controlled-form-components)
[官方文档](https://reactjs.org/docs/forms.html#controlled-components)

# 从 UI = f(state) 说起

之前的那篇长文，很详细的阐述了单项数据流和UI as function if states的原则。然而，理想很丰满，现实很骨感。HTML中的表单元素，input,textarea,select都不是stateless的。相反，他们很典型的维护了自己的inner state。比如说一个checkbox,它的checked就是一个内部状态。用户的点击会修改它，UI的状态，也就是有没有打上那个小勾，受内部状态的影响。怎么样体现UI = f(state)呢，就是这里要说的UI = f(state)

做法其实很简单：

UI不应该受内部状态影响。所以说，我们可以强行指定一个input的value，这样，不管你如何点击输入，都会被忽略。因为value被指定了。或者说，it is controlled. It is a controlled component.

```javascript
<input type="text" value={this.state.value} />
```

那功能呢？总不能说用户输入啥都没反应吧。很简单啊，构造一个action发给相应的r。本次话题与r无关，这里不做展开，我们就用一个简单的handleChange表示

```javascript
<input type="text" value={this.state.value} onChange={this.handleChange} />
```

最最简单的情况（也是我们没有引入例如redux，mobx之类很fancy的r最常见的情况）

```javascript
handleChange(event) {
  this.setState({value: event.target.value.toUpperCase()});
}
```

# 具体的几种form component

## checkbox

我们要控制的，不是value，用checked就行了。

```javascript
<input type="checkbox" checked={this.state.isChecked} onChange={this.handleChange} />

handleChange(event) {
  this.setState({ isChecked: !this.state.isChecked} );
}
```

也可以用value实现，但是不要问怎么做，不知道对你比较好。

## textarea

在HTML中，textarea的text使用children来定义的。而React提供了一个小小的语法糖，统一成了value这个prop来保持接口一致

```javascript
<textarea value={this.state.value} onChange={this.handleChange} />
```

## select

其实直接贴例子就够了

```javascript
<select value={this.state.value} onChange={this.handleChange}>
  <option value="grapefruit">Grapefruit</option>
  <option value="lime">Lime</option>
  <option value="coconut">Coconut</option>
  <option value="mango">Mango</option>
</select>
```
# Multiple Inputs

通常在表单中都有一堆form components，当然你可以用上面的做法一个一个实现，通常也不麻烦。这里介绍一种避免大量重复代码的做法。

我们用到了ES6中的computed property names这个语法(其实用es5也没有麻烦到哪里去)

```javascript
this.setState({
  [name]: value
});
```

然后给form components一个那么属性就行了

```javascript
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }

  render() {
    return (
      <form>
        <label>
          Is going:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange} />
        </label>
        <br />
        <label>
          Number of guests:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```

# 有个坑

如果你忘了初始化value（checked）,那样react就会认为收到一个undefined。或者说，它会认为这是一个uncontrolled component

比如说你收到了这样的error

| Warning: Input is changing a controlled input of type text to be uncontrolled. Input elements should not switch from controlled to uncontrolled (or vice versa). Decide between using a controlled or uncontrolled input element for the lifetime of the component

那大概率就是这个问题。