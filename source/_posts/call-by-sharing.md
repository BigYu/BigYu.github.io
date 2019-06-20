---
title: Call by sharing in javascript
date: 2019-02-26 20:43:42
tags:
- javascript

---

javascript中的变量，是它的值的引用（有点绕口）。对于函数的参数，既不是值传递，也不是址传递，而是神奇的call by sharing：

```javascript
function doSomething(a) {
    // do something
}

var foo = {
    // something
}

doSomething(foo);

```

在这个例子中，进入到函数doSomething里面之后，a和foo是不同的两个变量，但是他们指向同一个值，而不是拷贝过去的值。这就是call by sharing。效果就是，如果你对foo做了一些mutation，比如```a.x = 'y'```,那么，foo也会受影响。但是，如果你将a重新赋值，```a = { x: 'y' }``` 那么，这个变量会指向别的值，foo不受影响。

```javascript
let setNewInt = function (i) {
    i = i + 33;
};

let setNewString = function (str) {
    str += "cool!";
};

let setNewArray = function (arr1) {
    var b = [1, 2];
    arr1 = b;
};

let setNewArrayElement = function (arr2) {
    arr2[0] = 105;
};
```

这些例子中，只有最后一个会对传进去的参数造成影响。
