---
title: JavaScript基础学习-对象
date: 2020-05-24 22:28:45
tags: [JavaScript]
---

```javascript

```
### 原型

每一个 JavaScript 对象(除了 null)都和另一个对象相关联。“另一个”对象就是我们所熟知的原型，每一个对象都从原型继承属性。

所有通过对象直接量创建的对象都具有同一个原型对象，并且可以通过 JavaScript 代码 Object.prototype 获取对原型对象的引用。
通过 new 和构造函数调用创建的对象的原型就是构造函数的 prototype 属性的值。

```javascript
var obj = {};
var newObj = new Object();

obj.__proto__ === Object.prototype // => true
newObj.__proto__ === Object.prototype // => true

var arr = [];
var newArr = new Array();

arr.__proto__ === Array.prototype // => true
newArr.__proto__ === Array.prototype // => true
```

注意：没有原型的对象不多，Object.prototype 就是其中之一。它不继承任何属性。

另外，还可以通过 Objec.create() 方法基于一个对象的原型，创建一个新对象。

```javascript
var o1 = Object.create({x: 1, y: 2}); // o1 继承了属性 x 和 y

var o2 = Object.create(null); // 通过传入 null，来创建一个不继承任何属性和方法的对象

var o3 = Object.create(Object.prototype); // o3 和 {}、new Object() 一样，创建了一个空对象

/**
 * inherit() 返回一个继承继承自对象 p 的属性的新对象
 */
 function inherit(p) {
    if (p == null) throw TypeError();

    if (Object.create) return Object.create(p);

    var t = typeof p;
    if(t !== 'object' && t !== 'function') throw TypeError();
    function f() {}
    f.prototype = p;
    return new f();
 }
```