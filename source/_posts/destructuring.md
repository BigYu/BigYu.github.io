---
title: ES6 - destructuring 整体析构赋值
date: 2017-03-20 21:18:57
tags:
- Javascript
- ES6(ES2015)
---

这个汉化——“ 整体析构赋值”——是抄的，我真的翻译不来。。。

### Array
// list matching

```javascript
const [a, ,b] = [1,2,3];

a === 1;
b === 3;
```

### Object

```javascript
const obj = { a: 1, b: 2, c: 3};
const {a, c} = obj;

a === 1;
c === 3;
```

### function parameters

- Can be used in parameter position

```javascript
function g({name: x}) {
  console.log(x);
}
g({name: 5})
```

- Work with default args

``` javascript
function r({x, y, w = 10, h = 10}) {
  return x + y + w + h;
}
r({x:1, y:2}) === 23
```




