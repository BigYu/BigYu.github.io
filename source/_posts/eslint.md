---
title: Eslint
date: 2018-07-26 08:25:03
tags:
---

“进步”是一个很抽象，或者说，无法测量的概念，我们不妨用“突破上限”和“提高下限”来标记自己的成长。良好的编程习惯可以很好的提高自己的下限，既能在一些简单的代码中避免很多的stupic mistake，也能在处理复杂代码时理清自己的思路。在团队合作当中，良好的编程习惯可以显著的提高工作相率，降低独立开发者之间的摩擦力。
关于缩进，括号等等习惯的争执数不胜数。我的观点是，在一个团队内，这种公说公有理婆说婆有理的东西，统一的，规范的，就是正确的。比如说，我现在的团队的JavaScript代码采用的是[airbnb编程规范](https://github.com/airbnb/javascript)，其他团队可能有别的选择，包括自己的规范，让团队内的成员严格的规范统一，比无意义的嘴炮争吵更有意义。
在tool chain和infra的工作中，我们要不惮以最坏的恶意来揣测自己和团队：不管你有多少丰功伟绩，不管你平时的习惯多么好，都不能通过寄希望于“自觉”来保证代码质量，我们需要有相应的工具来检查，在CI的过程中拒收不符合标准的代码：

# Eslint: A fully pluggable tool for identifying and reporting on patterns in JavaScript

https://eslint.org

## 基本安装和用法

老习惯，不喜欢搬运文档来凑篇幅。所以贴链接来凑篇幅哈哈哈：https://github.com/eslint/eslint#local-installation-and-usage

注意：
1. 配合编辑器的正常工作，global installation应该会方便一点。
2. 写进脚本的时候，locally installation + 相对路径方便一些。
3. 这样我就知道为啥你编辑器的错误高亮和自动化脚本里的结果不一样了。

## 基本配置

还是贴链接： https://eslint.org/docs/user-guide/configuring

## customization & plugable

可以通过第三方的config或者plugin来配置自己的rule:

https://www.npmjs.com/package/eslint-plugin-lodash
https://www.npmjs.com/package/eslint-config-airbnb

也可以自己建一个npm包来作为自己的config：

```js
module.exports = {
  extends: [
    'airbnb',
    'plugin:react/recommended'
  ],
  parser: 'babel-eslint',
  parserOptions: {
    ecmaVersion: 2018
  },

  rules: {
    'linebreak-style': 'off',
    'react/jsx-filename-extension': [
      'error',
      {
        extensions: ['.js']
      }
    ]
  }
};
```