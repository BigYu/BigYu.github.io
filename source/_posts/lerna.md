---
title: lerna——多项目多模块的管理
date: 2018-05-23 10:45:18
tags:
- JavaScript
- project management
- package management
---

.Net项目中有solution和projects的概念（本人.Net大菜鸟。。）通常的WebUI项目是没有的。甚至连project这个概念都不太清晰，一些比较随意的项目，或者有些年头的项目，package.json到处都是是很常见的现象。甚至还有很多项目没有使用npm那一套进行管理。同时，前端组件经常出现跨平台跨项目复用的情况——比如说，整个公司所有产品网站的警告框，需要采用相同的设计和行为，我们不希望每一个前端团队都拿着设计稿重复一遍实现，那样有整体的rebranding的时候会是噩梦。
现在很常见的一种实践就是，把每一个组件都打成npm包发布出去，每个项目都根据npm规范来使用他们。这种实践下，我们一个“项目”里，就会出现多个npm package。一个常见的情况，我们会同时开发多个模块，这些模块之间互相有依赖，本地debug的时候可能有dependency并没有发布出去。如果我们大量使用npm link会让本地开发变得十分麻烦和混乱，自动化测试也变得很困难。这时，使用lerna管理我们的项目会是一个不错的主意。

https://github.com/lerna/lerna

```bash
npm install lerna -g
```

### 典型的lerna项目结构

```
my-lerna-repo/
  package.json
  packages/
    package-1/
      package.json
    package-2/
      package.json
```

这个项目的readme写得非常舒服，就不贴命令来凑篇幅了。

### Add

``` bash
lerna add xxx
lerna add xxx --scope=yyy
```

和npm install/yarn add不同的是，使用lerna add，可以一个lerna repo下的所有项目依赖相同的版本，避免不同的owner之间问“喂，你们用的啥版本jQuery？这个API我能用不？”。当然，会有情况下某些第三方的包/版本只适用于其中的某个/某些pacakge，可以用--scope来做到。另外，有些同时正在开发的package可能没有被发布，或者线上版本已经过时，lerna会把你本地的代码link在一起，让调试和自动化测试变得容易的多。lerna add package1 --scope=package2： 这样，你本地的package1会依赖于本地的package2，而不用担心package2没发布或者已发布的版本是过时的。

### Bootstrap

```bash
lerna bootstrap
```

读取每个package的package.json，安装依赖，并且把跨项目的依赖链接在一起。比如说package1的packag.json中有：

```json
{
  "dependencies": {
    "react": "a.b.c",
    "package2": "d.e.f"
  }
}
```

执行bootstrap之后，相当于用npm/yarn安装了react并link了package2：这在依赖关系复杂的时候，解决了很大的问题。

### publish

publish干了以下的事情：

1. 检查哪些项目应该被publish
2. 调整lerna.json中的版本号
3. 修改所有的package.json让它们只想正确的版本
4. 更新所有的依赖
5. 创建新的git commit和tag
6. publish

其他的publish参数可以参开人家的[Readme](https://github.com/lerna/lerna/blob/master/README.md)

### run

lerna run 就可以执行所有对应名字的npm scripts
