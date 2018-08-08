---
title: What I learn after first glance on ReactJs
date: 2016-08-01 12:47:47
tags:
- Frameworks
keywords:
- React Js
- One-way data-binding
- UI
---

就不再这里罗列各种关于React的新特性或者feature了，意义不大，而且我也是初学者。
之前我个人也一直很喜欢Facebook的东西——小巧，简单，清晰，还有他们漂亮的文档实在是吸引人去看。

- f(Status) = UI

  这个公式可以概括React作为view层面的framework最精髓的地方。你的code可以根据逻辑操作status。status的变化反映在UI上的变化大部分工作室React帮我们做的，开发者只需要很少的精力去编写，或者说是配置status和UI之间的映射关系。
  UI是复杂的，我们现在处理五花八门的dom结构就已经十分头痛了，再加上种类越来越多的富文本元素，再加上，再加上越来越复杂的动态交互，使得现在UI的开发成本越来越高，代码越来越复杂，而且runtime的维护成本也越来越高。包括缓存在内的问题也越来越棘手。
  Status是简单的，清晰的（我觉得清晰比简单更加能吸引我。当然，让事情变简单可以让事情变清晰）。Status就是数据结构，对于UI来说，并不会十分复杂和庞大，一般只需要一些简单的array和object。处理他们是程序员们最熟悉的事情，而且可以想象代码可以像数据结构课本上那么清晰明了。另外，做UI的Cache和Delta的代价显然十分高昂，但是换成这么一堆初学者都能看懂的array啊object啊就完全不是问题了，这也是React给出来的魔术般的解决缓存和Performance的策略：页面的render是基于status update的delta来的。

- 单向数据流

  我们之前遇到的各种前端框架显然不满足

  ```
  f(Status) = UI
  ```

  通常来说，应该是

  ```
  f(Status, 用户交互，推送信息，其他UI联动变化，……) = UI
  ```

  那么React必然要做的事情就是实现一个

  ```
  g(上面其他因素) = status
  ```

  这就是需要单向数据流的原因。
  我个人的体会，因为可能已经习惯了诸如knockout、angularjs等成熟的框架提供的各种完善的two-way data-binding的机制，第一眼很难接受React的one-way data flow，而且写一些很小的component的时候（比如hello world之类的），会发现很简单的事情反而需要我们花一些额外的精力。
  比如一个最简单的input框，如果用ko，只需要把他和vm上一个observable绑定就好了，在ng中也是很简单的ng-model=xxx，但是突然React告诉你，不行，你要绕一圈，你的输入只能作用于status的update，绕一圈才能回到UI的变化，当然不习惯。
  Facebook的工程师们肯定不是吃饱了撑的才这样设计。我们可以想象一些稍微复杂的逻辑——比如，如果这个input框需要把用户的输入全部转成大写字母，或者拒绝掉一些非法字符。当然，现在的框架已经足够优秀，我们可以pass in各种custom filter。问题就在于当这些需求一个一个堆砌起来，逻辑就会逐渐变得混乱，filter，suscription，events等等聚集在一起，代码的可读性会严重降低且难以维护。当UI中的元素越来越多，业务越来越复杂，这些不便利之处会指数级爆炸。
  而单向的数据流就不存在这些问题了，flux中有一个很好的Dispature和Store的概念，就像现代物流的集散分拣中心一样的（此处谢绝抬杠），一件快递的时候看起来不必要，但是成千上万的数据涌来，他依然能把所有的东西处理得井井有条，所有的东西就像阅兵式上的方阵，清晰有条理。我们会被从filter，suscription，events中被拯救出来，只有两样简单的事情：status，UI。
