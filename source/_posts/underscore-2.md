---
title: underscore源码阅读笔记（2）： reduce
date: 2016-08-17 10:01:07
tags:
- Javascript
---

+ reduce和reduceRight的实现：

```javascript
  // **Reduce** builds up a single result from a list of values, aka `inject`,
  // or `foldl`.
  _.reduce = _.foldl = _.inject = createReduce(1);

  // The right-associative version of reduce, also known as `foldr`.
  _.reduceRight = _.foldr = createReduce(-1);
```

+ 核心：createReduce - 这是一个function decorator:

```javascript
  // Create a reducing function iterating left or right.
  function createReduce(dir) {
    // Optimized iterator function as using arguments.length
    // in the main function will deoptimize the, see #1991.
    function iterator(obj, iteratee, memo, keys, index, length) {
      for (; index >= 0 && index < length; index += dir) {
        var currentKey = keys ? keys[index] : index;
        memo = iteratee(memo, obj[currentKey], currentKey, obj);
      }
      return memo;
    }

    return function(obj, iteratee, memo, context) {
      iteratee = optimizeCb(iteratee, context, 4);
      var keys = !isArrayLike(obj) && _.keys(obj),
          length = (keys || obj).length,
          index = dir > 0 ? 0 : length - 1;
      // Determine the initial value if none is provided.
      if (arguments.length < 3) {
        memo = obj[keys ? keys[index] : index];
        index += dir;
      }
      return iterator(obj, iteratee, memo, keys, index, length);
    };
  }
```

> Optimized iterator function as using arguments.length in the main function will deoptimize the, see #1991.

换我这种人来写，肯定就是用arguments.length来取一下，然后开开心心的开始for循环了，嘿嘿嘿

### 195行到198行在干嘛？
如果没有传初始值，则把第一个值作为初始值，同时移动一下初始迭代位置

### 191行
同时支持数组和集合。可以借鉴：）