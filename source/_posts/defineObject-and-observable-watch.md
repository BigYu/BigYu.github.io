---
title: Object.defineProperty
date: 2017-09-27 20:26:26
tags:
- javascript
---

# Object.defineProperty

在js中我们用可以用这样几种方法定义属性：

```javascript
foo.bar = 'abc';
```

```javascript
foo['bar'] = 'abc';
```

```javascript
Object.defineProperty(foo, 'bar', {
  value: 'abc',
});
```

defineProperty([MDN链接](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty))这种方法，最麻烦，最强大.

## 语法

```javascript
Object.defineProperty(obj, prop, descriptor)
```

## 参数

- obj: 需要被操作的目标对象
- prop: 目标对象需要定义或修改的属性的名称
- descriptor: 将被定义或修改的属性的描述符。

## 返回值

被传递给函数的对象

## descriptor

之前列出来的比较简单的两种写法，是通过赋值来创建并显示在属性枚举中（```for...in```或```Object.keys```），可以被修改，也可以被删除。使用definePropery,可以控制这些：

- configurable: 这个值为true时，descriptor才可以被修改，同时这个property也能从obj上被删除。默认为false。
- enumerable: 这个值为true时，才可以出现在```for...in```或者```Object.keys```中
- value: property的值。默认undefined。
- writable: 这个值为true时，propery才能被赋值运算符改变。默认false。
- get/set: 就跟其他的getter/setter差不多，包括es6的getter/setter




