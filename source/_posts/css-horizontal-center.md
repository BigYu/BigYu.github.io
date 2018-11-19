---
title: css垂直居中
date: 2018-11-18 21:51:21
tags:
---

## display: table

```html
    <div class="wrapper1">
      <div class="cell1">
        <div class="content1">
          content 1
        </div>
      </div>
    <div>
```

```css
.wrapper1 {
  display: table;
}

.cell1 {
  display: table-cell;
  vertical-align: middle;
  height: 100px;
  background-color: aqua;
}

.content1 {
  background-color: blueviolet;
}
```

- content可以动态改变高度
- wrapper没有足够空间时，content不会被截断
- 低版本IE不支持

## 绝对定位

```html
<div class="content2">
  content 2
</div>
```

```css
.content2 {
  position: absolute;
  top: 50%;
  height: 240px;
  margin-top: -120px;
  background-color: blueviolet;
}
```

## 插入floater

```html
<div class="floater3">
  <div class="content3">
    content 3
  </div>
</div>
```

```css
.floater3 {
  float: left;
  height: 50px;
  margin-bottom: -120px;
}

.content3 {
  clear: both;
  height: 240px;
  position: relative;
}
```

## 单行文本： height = line-height
