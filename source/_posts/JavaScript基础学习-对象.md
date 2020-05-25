---
title: JavaScript基础学习-对象
date: 2020-05-24 22:28:45
tags: [JavaScript]
---

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

### 检测属性

常用的检测属性的方法有：in 运算符、hasOwnProperty()、属性查询
此外还有一个：propertyIsEnumerable()方法

in 运算符不在介绍

hasOwnProperty() 方法用来检测是否是自身属性，对于继承属性返回 false。

```javascript
var o = {x: 1};
o.hasOwnProperty('x'); // => true
o.hasOwnProperty('y'); // => false
o.hasOwnProperty('toString'); // => false: toString() 方法是继承属性
```

propertyIsEnumerable() 方法是 hasOwnProperty() 的增强版，只检测自身属性并且该属性是可枚举的才返回 true。

```javascript
var o = inherit({y: 2});
o.x = 1;
o.propertyIsEnumerable('x'); // => true
o.propertyIsEnumerable('y'); // => false：y 是继承来的
Object.prototype.propertyIsEnumerable('toString'); // => false: 不可枚举
```

属性查询，判断是不是 undefined

```javascript
var o = {x: 1};
o.x !== undefined; // => true
o.y !== undefined; // => false
o.toString !== undefined; // => true
```

写到这里有同学就会问如果 o 对象的 x 值，我显示给 undefined 上面的方法不就不能区分了嘛？没错，所以遇到这种情况，则只能使用 in 运算符。

```javascript
var o = {x: undefined};
o.x !== undefined; // false
o.y !== undefined; // false
// 上面不能区分哪个属性不存在 o 对象中
'x' in o; // true
'y' in o; // false
delete o.x;
'x' in o; // false
```

所以通常情况下，使用 in 运算符就可以了。

### 枚举属性

for/in 循环可以遍历对象的所有**可枚举属性**(包括自身属性和继承属性)。

除了 for/in 循环外，还有 Object.keys()，它返回一个数组，这个数组由对象中可枚举的自身属性的名称(key)组成。

Object.getOwnPropertyNames() 返回对象中的所有自身属性的名称，包括不可枚举属性。

```javascript

```