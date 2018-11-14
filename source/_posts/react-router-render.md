---
title: react-router的配置中，render和component有什么不同
date: 2018-11-14 18:57:00
tags:
- React
- Framework
- Javascript
---

## react中的mount就行是什么意思。

在react中经常出现这个词，尤其是很多lifecycle方法中。究竟是什么意思呢？

词典中的mount，比较接近这个语境的解释有

- to get on a horse, bicycle, etc.. in order to ride
- to organize and begin an activity or event
- fix something to a wall, in a frame, etc., so that it can be looked at or used

react最主要的工作，当然还是操作DOM（所以这事儿得让它干，React不喜欢让人才参与这项他最重要的工作），具体来说，就是：

- mounting: 把新的节点加到DOM上
- unmounting： 在DOM上移除结点
- updating： 修改DOM上已经存在的节点

**This process of creating instances and DOM nodes corresponding to React components, and inserting them into the DOM, is called mounting.**

## react-router中的component和render

用一句话解释就是：component就是mount上去的，render就是字面意思上的render

- render只是生命周期中的一个步骤，用render的话，其他的，比如shouldComponentUpdate这些就不会被执行到
- 有些时候，render会省一些runtime的时间；然而，对于pureComponent或者其他lifecycle方法有作用的时候，就惨了。。

同样的道理，可以解释
{Child}和{Child()}的不同。