layout: blog
title: Javascript - try/catch/finally return statement
date: 2018-08-08 12:01:36
tags:
---

```js
(function example() {
    try {
        return true;
    }
    finally {
        return false;
    }
})()

// returns false
```

在try catch 语句中 finally block里的东西 永远都会被执行！