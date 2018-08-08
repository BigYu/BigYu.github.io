---
title: underscore源码阅读笔记（4）： eq(cont.)
date: 2016-09-07 11:01:11
tags:
- javascript
---

## javascript判断变量类型的方法

> javascript中 一切皆对象

```javascript
console.log(toString.call([123]));//[object Array]
console.log(toString.call('123'));//[object String]
console.log(toString.call({a: '123'}));//[object Object]
console.log(toString.call(/123/));//[object RegExp]
console.log(toString.call(123));//[object Number]
console.log(toString.call(undefined));//[object Undefined]
console.log(toString.call(null));//[object Null]
```

## eq函数中不同类型的“相等”判断

### RegExp和string

```javascript
// Strings, numbers, regular expressions, dates, and booleans are compared by value.
case '[object RegExp]':
// RegExps are coerced to strings for comparison (Note: '' + /a/i === '/a/i')
case '[object String]':
  // Primitives and their corresponding object wrappers are equivalent; thus, `"5"` is
  // equivalent to `new String("5")`.
  return '' + a === '' + b;
·```

** 以下“相等”均表示eq返回true **

+ 一个正则表达式 加上'' 仍让与原来相等
+ "abc"和new String('abc')是相等的


### Number

·```javascript
case '[object Number]':
        // `NaN`s are equivalent, but non-reflexive.
        // Object(NaN) is equivalent to NaN
        if (+a !== +a) return +b !== +b;
        // An `egal` comparison is performed for other numeric values.
        return +a === 0 ? 1 / +a === 1 / b : +a === +b;
```

+ js中 +NaN === +NaN 为false
+ Object(NaN)和NaN相等

### Date/Boolean

```javascrip
case '[object Date]':
case '[object Boolean]':
  // Coerce dates and booleans to numeric primitive values. Dates are compared by their
  // millisecond representations. Note that invalid dates with millisecond representations
  // of `NaN` are not equivalent.
  return +a === +b;
```

To be continued