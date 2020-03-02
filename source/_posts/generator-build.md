---
title: "generator-react-lib: 我创建的用typescript写react库的脚手架"
date: 2020-03-01 14:39:49
tags:
---

# 关于Yeoman

简单的说，就是用来开发脚手架的脚手架。具体见：https://yeoman.io/

# 为什么需要脚手架

工作中处于各种各样的理由，我们需要把自己编写的代码打包成单独的npm包发布出去，再在相关的工程中使用，比如复用场景比较多的ui组件之类。因为一些原因，我们使用某个版本的typescript编写，但是使用者完全可能因为不同的原因，并没有使用typescript或者使用的不兼容的typescript版本。因此我们需要打包成npm包，暴露出umd来实现。这个generator，针对的就是使用typescript开发的react库。

# 为什么不用create-react-app

CRA是用来创建app的。有些创建库的时候需要的体验缺乏，比如说typescript definition的输出。又有很多创建app的时候需要的创建库的时候并不需要，比如说生产环境的部署。

# 这个generator做了什么事情

- 自动生成package.json,添加了编写一个基于typescript的react组件库一定会用到的依赖。
- 自动生成webpack配置，用于库的build
- 自动生成lint配置，内置了个人推荐的lint规则。使用者可以根据需要自行修改。
- 自动生成基于storybook的开发和测试环境，用于库的开发。
- 添加了用于vscode的开发配置。

之后会写更多的post来具体解释上面做的事情。

# 使用方法

直接贴Readme吧：

## Installation

Install Yeoman first:

```bash
npm install -g yo
```

Then you can install this generator

```bash
npm install -g generator-react-lib-ts
```

Done!

Of course you can also do it with Yarn or integrate it into your npm/yarn scripts

## Generating a new component

```bash
# make sure yo and generator-react-lib-ts is installed globally or let npm/scripts to do it.
yo react-lib-ts

# Then the generator will ask you to input the name of your library, the path for your lib folder,
# and let you choose the type for your library.

```
## Usage

The following commands are available in your project:

```bash
# build your library. The output is uglified umd bundle with source map files.
yarn build

# run tests. Powered by jest.
yarn test

# linting your code.
yarn lint

# start local storybook demos
yarn storybook

# build local static storybook output
yarn build:demo
```

