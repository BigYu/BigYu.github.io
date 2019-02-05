---
title: javascript 原型链继承
date: 2019-02-01 19:50:56
tags:
---

## prototype

JavaScript用new关键字+构造函数来“模拟”类以实现面向对象编程。
所有实例对象需要共享的属性和方法，放在prototype对象中；不需要共享的属性和方法，放在构造函数中。

## 原型链

每一个对象都有指向prototype的链接，每个原型对象又有自己的原型，直到某个对象的原型为null。

## 继承

### 构造函数绑定

```javascript
function Animal() {
    this.species = "animal";
}

function Cat(name, color) {
    Animal.apply(this, arguments);
    this.name = name;
    this.color = color;
}

```

### prototype模式

```javascript
Cat.prototype = new Animal();
Cat.prototype.constructor = Cat;

```

注意一点，一旦替换了prototype对象，下一步必然是为新的prototype加上constructor属性，并将这个属性指向原来的构造函数。

### 直接继承prototype

```javascript
function Animal() {}

Animal.prototype.species = 'animal';

Cat.prototype = Animal.prototype;
Cat.prototype.constructor = Cat;
```

### 利用空对象作中介

```javascript
var F = function() {};

F.prototype = Animal.prototype;
Cat.prototype = new F();
Cat.prototype.constructor = Cat;
```