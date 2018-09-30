---
title: debounce
date: 2018-09-11 19:08:36
tags:
---

自己实现一个debounce,es5版本

```javascript
function debounce(func, delay) {
  var timer;

  return function() {
    var context = this;
    var args = arguments;

    clearTimeout(timer);

    timer = setTimeout(function() {
      func.apply(context, args);
    }, delay);
  }
}
```

es6简单一些

```javascript
function debounce(func, delay) {
  let timer;

  return (...args) => {
    clearTimeout(timer);

    timer = setTimeout(() => {
      func(...args);
    });
  }
}

```

underscore源码

```javascript
  _.debounce = function(func, wait, immediate) {
    var timeout, result;

    var later = function(context, args) {
      timeout = null;
      if (args) result = func.apply(context, args);
    };

    var debounced = restArguments(function(args) {
      if (timeout) clearTimeout(timeout);
      if (immediate) {
        var callNow = !timeout;
        timeout = setTimeout(later, wait);
        if (callNow) result = func.apply(this, args);
      } else {
        timeout = _.delay(later, wait, this, args);
      }

      return result;
    });

    debounced.cancel = function() {
      clearTimeout(timeout);
      timeout = null;
    };

    return debounced;
  };
```