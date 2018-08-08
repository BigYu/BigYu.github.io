---
title: jquery 为动态添加的元素绑定事件
date: 2016-08-01 15:45:46
tags:
- Frameworks
---

**live函数已经被废弃了！！**

如果直接写click函数的话，只能把事件绑定在已经存在的元素上，不能绑定在动态添加的元素上

尝试过重新调用，结果是重复触发。。。

###可以用delegate来实现

```javascript
.delegate( selector, eventType, handler )
```

例如：

```javascript
$('someUlSelector').delegate('someLiSelector', 'click', function() {
    //codes...
    //$(this) for the current jquery instance of the element
});
```

