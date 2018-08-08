---
title: "data-binding: dirty check"
date: 2017-10-11 15:31:04
tags:
- framework
- javascript
- data binding
---

见过的前端js框架，data-binding的实现方案大致有三种：

- 直接把数据的暴露成function,如knockout
- dirty check, 如angular
- 劫持getter,setter + 依赖收集,如vue

自己实现了一个非常简化的dirty-check

# $$watchers
scope是angular中最重要的一个概念，这里不再重复。用dirty check来实现data binding,scope中的$$watchers起到了核心的作用。$$watchers是一个数组，用于存放scope下所有的数据。每个数据有以下几个字段：

- name: 变量名，就是$scope.someName中的someName
- last: 变量上一次的值，这是一个值。
- newVal: 获取新值的函数，这是个函数。
- listener: 监听的回掉函数。

# $watch

将一个新的成员加入$$watchers

```javascript
  $watch(name, exp, listener = function () {}) {
    this.$$watchers.push({
      name,
      last: '',
      newVal: exp,
      listener,
    });
  }
```

有了$watch方法，我们就可以把scope中的所有值都加入$$watchers:

```javascript
  for (const key in $scope) {
    if (key != '$$watchers' && !_.isFunction($scope[key])) {
      $scope.$watch(key, () => $scope[key]);
    }
  }
```


# $digest

$digest的作用，就是遍历$$watchers里所有的值，比较新值和旧值，调用listener，并把新值赋值给last,直到所有的新值旧值都相等。

```javascript
  $digest() {
    let dirty = true;

    while (dirty) {
      dirty = false;

      _.each(this.$$watchers, (watcher) => {
        const newVal = watcher.newVal();
        const oldVal = watcher.last;

        if (newVal !== oldVal && !_.isNaN(newVal) && !_.isNaN(oldVal)) {
          dirty = true;
          watcher.listener(oldVal, newVal);
          watcher.last = newVal;
        }
      });
    }
  }
```

再加入更新dom元素的逻辑，就变成了

```javascript
$digest() {
    const bindList = document.querySelectorAll('[ng-bind]');
    let dirty = true;

    while (dirty) {
      dirty = false;

      _.each(this.$$watchers, (watcher) => {
        const newVal = watcher.newVal();
        const oldVal = watcher.last;

        if (newVal !== oldVal && !_.isNaN(newVal) && !_.isNaN(oldVal)) {
          dirty = true;
          watcher.listener(oldVal, newVal);
          watcher.last = newVal;

          _.each(bindList, (bind) => {
            const modelName = bind.getAttribute('ng-bind');

            if (modelName === watcher.name) {
              if (bind.tagName === 'INPUT') {
                bind.value = this[modelName];
              } else {
                bind.innerHTML = this[modelName];
              }
            }
          });
        }
      });
    }
  }
```
# 绑定到dom

剩余的工作就是把一些dom事件绑定一下，这里简单的绑定一下click button和inputbox的change

```javascript
  const clickBindList = document.querySelectorAll('[ng-click]');

  _.each(clickBindList, (clickBind) => {
    clickBind.onclick = () => {
      $scope[clickBind.getAttribute('ng-click')]();
      $scope.$digest();
    };
  });

  const inputList = document.querySelectorAll('input[ng-bind]');

  _.each(inputList, (input) => {
    input.addEventListener('input', () => {
      $scope[input.getAttribute('ng-bind')] = input.value;
      $scope.$digest();
    });
  });
```


# 完整代码

```javascript
import _ from 'underscore';

class Scope {
  constructor() {
    this.$$watchers = [];
  }

  $watch(name, exp, listener = function () {}) {
    this.$$watchers.push({
      name,
      last: '',
      newVal: exp,
      listener,
    });
  }

  $digest() {
    const bindList = document.querySelectorAll('[ng-bind]');
    let dirty = true;

    while (dirty) {
      dirty = false;

      _.each(this.$$watchers, (watcher) => {
        const newVal = watcher.newVal();
        const oldVal = watcher.last;

        if (newVal !== oldVal && !_.isNaN(newVal) && !_.isNaN(oldVal)) {
          dirty = true;
          watcher.listener(oldVal, newVal);
          watcher.last = newVal;

          _.each(bindList, (bind) => {
            const modelName = bind.getAttribute('ng-bind');

            if (modelName === watcher.name) {
              if (bind.tagName === 'INPUT') {
                bind.value = this[modelName];
              } else {
                bind.innerHTML = this[modelName];
              }
            }
          });
        }
      });
    }
  }
}

export function ngController(controller) {
  const $scope = new Scope();

  controller($scope);

  const clickBindList = document.querySelectorAll('[ng-click]');

  _.each(clickBindList, (clickBind) => {
    clickBind.onclick = () => {
      $scope[clickBind.getAttribute('ng-click')]();
      $scope.$digest();
    };
  });

  const inputList = document.querySelectorAll('input[ng-bind]');

  _.each(inputList, (input) => {
    input.addEventListener('input', () => {
      $scope[input.getAttribute('ng-bind')] = input.value;
      $scope.$digest();
    });
  });

  for (const key in $scope) {
    if (key != '$$watchers' && !_.isFunction($scope[key])) {
      $scope.$watch(key, () => $scope[key]);
    }
  }

  $scope.$digest();
};
```

## Demo

```javascript
ngController(($scope) => {
  $scope.count = 0;
  $scope.increment = () => {
    $scope.count++;
  };
});
```
