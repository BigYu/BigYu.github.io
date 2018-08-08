---
title: es6-generator
date: 2017-03-24 16:54:15
tags:
- Javascript
- ES6(ES2015)
---

看babel官网上提供的例子

```javascript
var fibonacci = {
  [Symbol.iterator]: function*() {
    var pre = 0, cur = 1;
    for (;;) {
      var temp = pre;
      pre = cur;
      cur += temp;
      yield cur;
    }
  }
}

for (var n of fibonacci) {
  // truncate the sequence at 1000
  if (n > 1000)
    break;
  console.log(n);
}
```

在for...of...循环中，会调用迭代器对象的迭代器方法。之前那篇迭代器有提到过，但是这里的function*和普通的函数function不一样。如果是普通的函数，不管什么时候执行，得到的结果总是一样的（不考虑全局变量之类的干扰），或者说，和上一次无关。



