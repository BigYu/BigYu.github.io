---
title: css中的定位
date: 2016-09-02 20:30:06
tags:
- css
---

css的position属性定义元素的定位类型，Position属性有四个值：static、fixed、absolute和relative。主流的浏览器都是支持所有值的。

** 著名坑爹货：任何的版本的 Internet Explorer （包括 IE8）都不支持属性值 "inherit"。 **

## static
默认值 表示一个元素不会被positioned 仍然放在正常文档流中（“正常”是什么东西见上一篇关于浮动的）

## relative
基本的表现和static一样 不同的是可以用top bottom left right属性来控制元素位置的偏移

## fixed
相对于视窗定位 就是说 即使页面滚动 元素的位置仍然不会改变

## absulote
absolute 与 fixed 的表现类似，除了它不是相对于视窗而是相对于最近的“positioned”祖先元素。如果绝对定位（position属性的值为absolute）的元素没有“positioned”祖先元素，那么它是相对于文档的 body 元素，并且它会随着页面滚动而移动。

> 一个“positioned”元素是指position 值不是 static 的元素。