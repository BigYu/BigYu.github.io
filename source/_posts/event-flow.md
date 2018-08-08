---
title: event flow
date: 2017-03-15 10:23:36
tags:
- Javascript
- Dom
---

### 流 Flow - 事件流 event flow

简单来说 流就是**具有方向的数据**
事件流所描述的就是从页面中接受事件的顺序。

### 事件冒泡

事件冒泡即事件开始时，由最具体的元素接收（也就是事件发生所在的节点），然后逐级传播到较为不具体的节点

### 事件捕获

当某个事件发生时，父元素应该更早接收到事件，具体元素则最后接收到事件

### addEventListener

```javascript
element.addEventListener(event, function, useCapture)
```

第一个参数是需要绑定的事件，第二个参数是触发事件后要执行的函数。而第三个参数默认值是false，表示在事件冒泡的阶段调用事件处理函数，如果参数为true，则表示在事件捕获阶段调用处理函数

### DOM 事件流

- 事件捕获阶段

- 处于目标阶段

- 事件冒泡阶段

- 阻止事件冒泡

```javascript
event.stopPropagation()
```
