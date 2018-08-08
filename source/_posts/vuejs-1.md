---
title: Vue.js学习 - 创建第一个应用
date: 2016-12-13 15:34:39
tags:
- Frameworks
- Vue.js
---

## 在github上创建一个项目

## 在本地创建对应的repro

### init git

```bash
echo "# test" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/BigYu/test.git
git push -u origin master
```

- 我创建了两个branch，master和develop。如果是一个人写的小项目，其实没有必要。管他呢，就当假装follow gitflow的一点执念吧，嘻嘻。

### init npm

  加上-y选项，即全部填写默认值。如果将来需要修改，在文件里修改吧，什么都没写的时候干这些意义不大

## 准备好第三方依赖

### 梳理一下我要用啥

- vue.js
- ES2015(6?) & babel
- webpack/webpack devserver
- sass

先暂时想到这些。以后想起来再加。

### 安装webpack

```bash
npm install --save-dev webpack
npm install webpack-dev-server -g
```

dev server用于本地调试，所以没有加入到packages.json里

### 使用webpack dev server

- webpack.config.js

```javascript
module.exports = {
  entry: './src/index.js',
  output: {
    path: './assets',
    filename: 'bundle.js'
  },
  devServer: {
    port: 99
  }
};
```
- src/index.js

```javascript
document.write('Hello vue!!! I am Yucong.');
```

- bash commands

```
webpack dev server
```

浏览器打开 http://localhost:99/webpack-dev-server/bundle 就能看到啦。自动刷新哦~

### webpack

- bash command

```bash
webpack
```

然后就看到bundle.js在assets目录下被创建了

再加入一个index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  <script src="assets/bundle.js"></script>
</body>
</html>
```

### 安装babel loader并调试

- install

```bash
npm install babel-loader babel-core babel-preset-es2015 --save-dev
```

- config

```javascript
module: {
  loaders: [
    {
      test: /\.js$/,
      exclude: /(node_modules|bower_components)/,
      loader: 'babel-loader',
      query: {
        presets: ['es2015']
      }
    }
  ]
}
```

- src/index.js

```javascript
import './test1';

document.write('Hello vue!!! I am Yucong.');
```

- src/test1.js

```javascript
import './test_module/test2';

window.console.log('Hello');
```

- src/test_module/test2.js

```javascript
import test3 from './test3';

const a = 2;

alert(a);
alert(test3.foo);
```

- src/test_module/test3.js

```javascript
export default {
  foo: 'bar',
};
```

** 这里还有一个问题，所有的import path都是相对路径，还不知道怎么做成绝对路径。。。 **

### html loader

```bash
npm install html-loader --save-dev
```

config

``` javascript

```

### 安装vue.js 写一个hello word

```bash
npm install vue --save
```

- index.js

```javascript
import Vue from 'vue';

new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!',
  }
});
```

** The runtime-only build does not include the template compiler, and does not support the template option. You can only use the render option when using the runtime-only build, but it works with single-file components, because single-file components’ templates are pre-compiled into render functions during the build step. The runtime-only build is roughly 30% lighter-weight than the standalone build, weighing only 17.60kb min+gzip. **

```javascript
resolve: {
  alias: {
    'vue$': 'vue/dist/vue.common.js'
  }
}
```

- index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  <div id="app">{{message}}</div>
  <script src="/bundle.js"></script>
</body>
</html>
```

