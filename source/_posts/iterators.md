---
title: iterators & for...of... es6迭代器
date: 2017-03-16 22:01:56
tags:
- Javascript
- ES6
---

### 用for...in...一定要小心！！

```javascript
var myArray = [1, 2, 3, 4, 5];

for (var index in myArray) { // never do this!!
  console.log(myArray[index]);
}
```

- index不是number而是string，说不定就让自己掉进 '2' + 1 == ‘21’ 这样的bug里了
- for-in还会作用于自定义属性（myArray.name这种）
- 有些时候顺序是随机的

** 简而言之，for-in是为普通对象设计的，你可以遍历得到字符串类型的键，因此不适用于数组遍历 **

### .forEach的局限

```javascript
myArray.forEach(funciton(val) {
  console.log(val);
})
```

不能使用break语句中断循环，也不能使用return语句返回到外层函数

### for...of in es6

- 正确响应break, continue
- 避免for...in的缺点
- 除了Array, 还可以遍历String, map, set

### 迭代器对象
for-of循环语句通过方法调用来遍历各种集合。数组、Maps对象、Sets对象以及其它在我们讨论的对象有一个共同点，它们都有一个迭代器方法。

你可以给任意类型的对象添加迭代器方法。

当你为对象添加myObject.toString()方法后，就可以将对象转化为字符串，同样地，当你向任意对象添加myObject[Symbol.iterator]()方法，就可以遍历这个对象了

for-of循环首先调用集合的[Symbol.iterator]()方法，紧接着返回一个新的迭代器对象。迭代器对象可以是任意具有.next()方法的对象；for-of循环将重复调用这个方法，每次循环调用一次
