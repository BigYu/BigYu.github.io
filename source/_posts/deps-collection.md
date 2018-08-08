---
title: "Data binding: hijack and dependencies collection"
date: 2017-10-18 10:22:55
tags:
- framework
- javascript
- data binding
---

见过的前端js框架，data-binding的实现方案大致有三种：

- 直接把数据的暴露成function,如knockout
- dirty check, 如angular
- 劫持getter,setter + 依赖收集,如vue

写下第三种：

[Vue 源码](https://github.com/vuejs/vue/blob/dev/src/core/observer/index.js)

我们的demo code是这样的：

```jade
form
  input(type="text", v-bind="count")
  button(type="button", v-click="increment") increment
div(v-bind="count")
```

```javascript
import { Observable, Watcher } from '@bingads-webui/simple-observable';

const person = new Observable({
  firstName: 'Michael',
  lastName: 'Jordan',
});

new Watcher(person, 'fullName', () => {
  return `${person.firstName} ${person.lastName}`;
}, (val) => {
  console.log(`The full name is ${val}`);
});

console.log(person.fullName);

person.lastName = 'Jackson';

window.person = person;
```

带dom操作的:

```javascript
import { App } from '@bingads-webui/simple-observable';
import template from './index.pug';

new App({
  el: document.body,
  template,
  data: {
    count: 3,
  },
  method: {
    increment: function() {
      this.data.count++;
    }
  },
});
```

# Observable

## 在属性被读写的时候主动通知：

```javascript
const person = {};

let firstName = 'Michael';

Object.defineProperty(person, 'firstName', {
  get() {
    console.log('Reading person.firstName');

    return 'Michael';
  },

  set(val) {
    console.log(`Setting person.firstName ${val}`);

    firstName = val;
  }
});
```

我们可以定义最基本的observable类和defineReactive方法：

```javascript
function defineReactive(obj, key, val) {
  Object.defineProperty(obj, key, {
    get() {
      // todo

      return val;
    },

    set(newVal) {
      // todo

      val = newVal;
    },
  });
}

function obervable(obj) {
  const keys = Object.keys(obj);

  keys.forEach((key) => {
    defineReactive(obj, key, obj[key]);
  });

  return obj;
}
```

Refactor一下，写成class

```javascript
class Observable {
  constructor(obj) {
    return this.walk(obj);
  }

  walk(obj) {
    const keys = Object.keys(obj);

    keys.forEach((key) => {
      this.defineReactive(obj, key, obj[key]);
    });

    return obj;
  }

  defineReactive(obj, key, val) {
    Object.defineProperty(obj, key, {
      get() {
        // todo

        return val;
      },

      set(newVal) {
        // todo

        val = newVal;
      },
    });
  }
}
```

# Watcher

现在来实现demo中的这个部分：

```javascript
new Watcher(person, 'fullName', () => {
  return `${person.firstName} ${person.lastName}`;
});
```

(有没有觉得和dirty check里面的$$watchers很像。。。后面还有更像的)

- watcher属性无法被赋值
- watcher的值由第三个参数决定

```javascript
function watcher(obj, key, getVal) {
  Object.defineProperty(obj, key, {
    get() {
      const val = getVal();

      return val;
    },
    set() {
      throw new Error('Cannot set values of wathers.');
    }
  })
}
```

改写成watcher类并加入listener

```javascript
class Watcher {
  constructor(obj, key, cb， onComputedUpdate) {
    this.obj = obj;
    this.key = key;
    this.cb = cb;
    this.onComputedUpdate = onComputedUpdate;

    return this.defineComputed();
  }

  defineComputed() {
    const self = this;

    const onDepUpdated = () => {
      const val = self.cb();

      this.onComputedUpdate(val);
    };

    Object.defineProperty(this.obj, this.key, {
      get() {
        const val = self.cb();
        this.onComputedUpdate(val);

        return val;
      },

      set() {
        throw new Error('Cannot set values of wathers.');
      },
    });
  }
}
```

这样observable和watcher就可以一起工作了：

```javascript
const person = new Observable({ firstName: 'Michael', lastName: 'Jordan' });

new Watcher(person, 'fullName', () => `${person.firstName} ${person.lastName}`, (val) => { console.log(`new value ${val}`) });
```

# Dependencies collection

这样有一个问题：我们是主动去取fullName的值的时候，它才会被计算。而我们期望的行为，是firstName或者lastName被修改的时候，主动通知fullName，并更新,触发onComputedUpdate。
可以设计这样一个dependency collector, 来收集所有的dependencies. 具体一点就是,让一个observable知道有哪些watcher依赖它,并且在自己被赋值的时候通知watcher.

注意到watcher的getter被call到的时候,它依赖哪些observable,就会call到这些observable,可以用这个时机来告诉observable,这个watcher依赖你了.然后在call到setter时,就可以把dependencies拿出来通知:

简单版本,拿一个全局变量存dependencies做依赖收集器

```javascript
const Dep = {
  target: null,
};

```

在watcher中:

```javascript
  defineComputed() {
    const self = this;

    Object.defineProperty(this.obj, this.key, {
      get() {
        Dep.target = () => {
          const val = self.cb();
          this.onComputedUpdate(val);
        }
        const val = self.cb();

        return val;
      },

      set() {
        throw new Error('Cannot set values of wathers.');
      },
    });
  }
```

在observable中

```javascript
  defineReactive(obj, key, val) {
    const deps = [];

    Object.defineProperty(obj, key, {
      get() {
        deps.push(Dep.target);

        return val;
      },

      set(newVal) {
        val = newVal;
        deps.forEach(dep => dep());
      },
    });
```

# 完整代码

```javascript
import _ from 'underscore';

class Dep {
  constructor() {
    this.deps = [];
  }

  depend() {
    if (Dep.target && this.deps.indexOf(Dep.target) === -1) {
      this.deps.push(Dep.target);
    }
  }

  notify() {
    this.deps.forEach((dep) => {
      dep();
    });
  }
}

Dep.target = null;

export class Observable {
  constructor(obj) {
    return this.walk(obj);
  }

  walk(obj) {
    const keys = Object.keys(obj);

    keys.forEach((key) => {
      this.defineReactive(obj, key, obj[key]);
    });

    return obj;
  }

  defineReactive(obj, key, val) {
    const dep = new Dep();

    Object.defineProperty(obj, key, {
      get() {
        dep.depend();

        return val;
      },

      set(newVal) {
        val = newVal;
        dep.notify();
      },
    });
  }
}

export class Watcher {
  constructor(obj, key, cb, onComputedUpdate) {
    this.obj = obj;
    this.key = key;
    this.cb = cb;
    this.onComputedUpdate = onComputedUpdate;

    return this.defineComputed();
  }

  defineComputed() {
    const self = this;

    const onDepUpdated = () => {
      const val = self.cb();

      this.onComputedUpdate(val);
    };

    Object.defineProperty(this.obj, this.key, {
      get() {
        Dep.target = onDepUpdated;

        const val = self.cb();

        Dep.target = null;

        return val;
      },

      set() {
        throw new Error('Cannot assign value computed!');
      },
    });
  }
}

export class App {
  constructor({ el, template, data, method }) {
    this.el = el;
    this.template = template;
    this.data = new Observable(data);
    this.method = method;
    this.render();
    this.applyBindings();
  }

  applyBindings() {
    const bindList = this.el.querySelectorAll('[v-bind]');

    _.each(bindList, (bind) => {
      const bindKey = bind.getAttribute('v-bind');

      if (bind.tagName === 'INPUT') {
        // bind.value = this.data[bindKey];

        new Watcher(this.data, `$${bindKey}Value`, () => {
          return this.data[bindKey];
        }, (newVal) => {
          bind.value = newVal;
        });

        bind.value = this.data[`$${bindKey}Value`];

        bind.addEventListener('input', (e) => {
          this.data[bindKey] = e.target.value;
        });
      } else {
        new Watcher(this.data, `$${bindKey}InnerHtml`, () => {
          return this.data[bindKey];
        }, (newVal) => {
          bind.innerHTML = newVal;
        });

        bind.innerHTML = this.data[`$${bindKey}InnerHtml`];
      }
    });

    const clickList = this.el.querySelectorAll('[v-click]');

    _.each(clickList, (click) => {
      const clickKey = click.getAttribute('v-click');
      const clickFunc = this.method[clickKey];

      click.onclick = clickFunc.bind(this);
    });
  }

  render() {
    this.el.innerHTML = this.template();
  }
}

```
