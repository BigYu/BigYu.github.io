---
title: Backbone model- on vs listenTo
date: 2017-07-10 17:06:50
tags:
- Framework
- Backbone
- Events
---

发现好久没有写blog了 要捡回来这个好习惯：）

今天遇到了一个bug，给一个全局的（不要吐槽,legacy code留下来的全局的东西没那么好消灭干净）添加了一个事件的callback，大致代码是这样

```javascript
aGlobalModel.on('a-event', () => {
  someView.doSomething();
});
```

原因是，someView被remove的时候，忘了call gGlabal.off. Ok, 加上off逻辑当然不难，但是：
1. 直接call off会清理掉所有的handler。这坑爹货是全局的。。。。
2. 可以指定off哪一个callback，所以这个callback会被传来传去。。比方说有很多处会remove的情况。on 并不是在someView里定义的。

事实上，listenTo是一个更好的选择。

```javascript
someView.listenTo(aGlobalModel, 'a-event', () => {
  // ...code
})
```

当someView的remove被call到的时候 会自动的执行stopListening.

### 看一下Backbone源码

- on

```javascript
// Bind an event to a `callback` function. Passing `"all"` will bind
// the callback to all events fired.
Events.on = function(name, callback, context) {
  this._events = eventsApi(onApi, this._events || {}, name, callback, {
    context: context,
    ctx: this,
    listening: _listening
  });

  if (_listening) {
    var listeners = this._listeners || (this._listeners = {});
    listeners[_listening.id] = _listening;
    // Allow the listening to use a counter, instead of tracking
    // callbacks for library interop
    _listening.interop = false;
  }

  return this;
};
```

- listenTo

```javascript
// Inversion-of-control versions of `on`. Tell *this* object to listen to
// an event in another object... keeping track of what it's listening to
// for easier unbinding later.
Events.listenTo = function(obj, name, callback) {
  if (!obj) return this;
  var id = obj._listenId || (obj._listenId = _.uniqueId('l'));
  var listeningTo = this._listeningTo || (this._listeningTo = {});
  var listening = _listening = listeningTo[id];

  // This object is not listening to any other events on `obj` yet.
  // Setup the necessary references to track the listening callbacks.
  if (!listening) {
    this._listenId || (this._listenId = _.uniqueId('l'));
    listening = _listening = listeningTo[id] = new Listening(this, obj);
  }

  // Bind callbacks on obj.
  var error = tryCatchOn(obj, name, callback, this);
  _listening = void 0;

  if (error) throw error;
  // If the target obj is not Backbone.Events, track events manually.
  if (listening.interop) listening.on(name, callback);

  return this;
};
```

上述代码中用到的eventsApi,是标准的事件，回调迭代器：

```javascript
// Iterates over the standard `event, callback` (as well as the fancy multiple
// space-separated events `"change blur", callback` and jQuery-style event
// maps `{event: callback}`).
var eventsApi = function(iteratee, events, name, callback, opts) {
  var i = 0, names;
  if (name && typeof name === 'object') {
    // Handle event maps.
    if (callback !== void 0 && 'context' in opts && opts.context === void 0) opts.context = callback;
    for (names = _.keys(name); i < names.length ; i++) {
      events = eventsApi(iteratee, events, names[i], name[names[i]], opts);
    }
  } else if (name && eventSplitter.test(name)) {
    // Handle space-separated event names by delegating them individually.
    for (names = name.split(eventSplitter); i < names.length; i++) {
      events = iteratee(events, names[i], callback, opts);
    }
  } else {
    // Finally, standard events.
    events = iteratee(events, name, callback, opts);
  }
  return events;
};
```
