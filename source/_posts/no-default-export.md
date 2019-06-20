---
title: "JavaScript Modules: stop default exporting"
date: 2019-06-20 13:58:47
tags:
- JavaScript
- JavaScript Modules
---

遇到了这样一条eslint规则：[no-default-export](https://github.com/benmosher/eslint-plugin-import/blob/master/docs/rules/no-default-export.md) 不允许使用default export,于是就好奇：1.为什么js modules里同时有named export和default export。 2. 到底应该用哪种。当然，还有一条完全相反的规则：[import/prefer-default-export](https://github.com/benmosher/eslint-plugin-import/blob/master/docs/rules/prefer-default-export.md)

搜了一圈，这篇文章是最能说服我的：https://humanwhocodes.com/blog/2019/01/stop-using-default-exports-javascript-module/ 这篇文章的作者是eslint的作者。

## 从CommonJS到ES6 modules

default export最初来源于CommonJS中：

```js
class LinkedList {}

module.exports = LinkedList;
```

这段简单的code export了LinkedList，但是别人在用的时候，并没有指定名字。比方说这个文件名是linked-list.js，你在另一个CommonJs Module中,可以这样引用：

```js
const LinkedList = require("./linked-list");
```

这里的require函数的返回值可以和我刚才eport的一致，也叫LinkedList，但是，我也可以把这个变量名叫别的：

```js
const Dog = require("./linked-list");
```

CommonJs标准十分流行，所以es6的module定义也支了这种pattern

> ES6 favors the single/default export style, and gives the sweetest syntax to importing the default.
> — David Herman June 19, 2014

```js
export default class LinkedList {}
```

```js
import LinkedList from "./linked-list.js"
```

也可以

```js
import Dog from "./linked-list.js"
```

## named export

CommonJs和JavaScript Modules都支持named export. 在named export
中，可以export function, class, 变量。

### named export in CommonJs

```js
exports.LinkedList = class LinkedList {};
```

```js
const LinkedList = require("./linked-list").LinkedList;
```

当然 这里你把前面的变量名写成Dog也是可以的。

### named export in JavaScript modules

```js
export class LinkedList {}
```

```js
import { LinkedList } from "./linked-list.js";
```

这里就不能乱起名字了。


## Personal preference from Nicholas

原文作者提到了他个人的peronal preference。这是几条普遍适用的准则，无论你在用什么编程语言

1· Explicit over implicit.

2. Names should be consistent throughout all files.

3. Throw errors early and often.

4. Fewer decisions mean faster development.

5. Side trips slow down development（Whenever you have to stop and look. something up in the middle of coding, I call that a side trip.）.

6. Cognitive overhead slows down development.

## default export的问题

### What is that thing?

还是这个例子：

```js
const list = require("./list");
```

在这个上下文中，如果你对list这个module不熟悉，一定会一脸懵逼：list是个啥？显然不太像一个primitive value，那这是一个function？class？或者是别的object类型？搞清楚这件事就是上面说的side trip。

- 如果你own list.js,就得去看看自己之前写的code，看看export的是啥。
- 如果这是别人写的module，那就得去查文档。

至于用IDE抬杠的。。作者在原文中解释的是很有道理的。我自己没怎么用过那些高级的ide，就不乱转了。


## Name matching problems

在上面的李子中，如果选择了named export,那么你只要全局搜LinkedList，就很容易找到所有用到了它的地方。但是如果使用default export话，就不一定了。由于default export没有限制使用时的名字，所以在用的时候，还需要花额外的精力来想一下命名。如果是多人合作的项目，为了保证大家同同样的名字描述同一件事，还需要额外的communication。

即使因为需求，需要给import的东西换个名字，也不影响上述结论：

```js
const MyList = require("./list").LinkedList;
```

```js
import { LinkedList as MyList } from "./list.js";
```

在一个项目中，整个codebase下保持名字一致很重要：

- 高效的全局搜索
- Refactor的时候轻松多了

## Importing the wrong thing

```js
import { LinkedList } from "./list.js"
```

如果说LinkedList不存在的话，使用named export可以更早的报出错误。

