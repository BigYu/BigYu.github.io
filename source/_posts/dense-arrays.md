---
title: Javascript中的稀疏数组 sparse arrays vs. 密集数组 dense arrays
date: 2017-03-21 14:49:41
tags:
- javascript
---

大多数情况下，javascript中的数组是稀疏数组（其实就是k-v pairs），也就是说a[0], a[100]存在，但是a[1]到a[99]可能都不存在。

不存在，不是说存在但是是undefined,就是真正的不存在:如果用foreach访问并console.log，会发现根本没有这么一项，而不是输出一个undefined

实际上,JavaScript并没有常规的数组,所有的数组其实就是个对象,只不过会自动管理一些"数字"属性和length属性罢了.说的更直接一点,JavaScript中的数组根本没有索引,因为索引应该是数字,而JavaScript中数组的索引其实是字符串.arr[1]其实就是arr["1"],给arr["1000"] = 1,arr.length也会自动变为1001.这些表现的根本原因就是,JavaScript中的对象就是字符串到任意值的键值对.注意键只能是字符串

### sparse arrays

- 得到一个稀疏数组

```javascript
var sparseArr1 = new Array(3);
```

```javascript
var sparseArr2 = [];

sparseArr2[0] = 1;
sparseArr2[100] = 100;
```

- 遍历稀疏数组，所有的“空隙”会被跳过
```javascript
var sparseArr2 = [];

sparseArr2[0] = 1;
sparseArr2[100] = 100;

sparseArr2.forEach(function(index, val) { console.log(index); console.log(val) }) // output: 1 0 100 100
```

### dense arrays

- 得到一个密集数组

```javascript
Array(undefined, undefined, undefined)
```

```javascript
var denseArr1 = Array.apply(null, Array(3))；
```

```javascript
var denseArr2 = Array.apply(null, Array(3)).map(Function.prototype.call.bind(Number))
```

