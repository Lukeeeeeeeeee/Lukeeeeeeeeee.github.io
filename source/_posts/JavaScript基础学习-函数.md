---
title: JavaScript基础学习-函数
date: 2020-06-02 22:23:07
updated: 
tags: [JavaScript]
---

### 函数声明

当使用函数声明语句时，函数名称和函数体都被提升到顶部：所以可以在一个函数声明之前调用它。
但使用函数表达式则不可以：因为以表达式定义的函数，其变量的声明提前了，但是函数体没有被提前。

### 方法链

在 jQuery 中，我们竟然见到链式调用的方式，其原理就是当方法不需要返回值时，直接返回 this。

### 函数调用

关于 this：如果嵌套函数作为方法调用，其 this 值指向调用它的对象。如果嵌套函数作为函数调用，其 this 值不是全局对象就是 undefined。

### 闭包

不在它定义时的作用域链下被调用时，被当作返回值时。

```javascript
var scope = 'global';
function checkscope() {
    var scope = 'local';
    function f() { return scope; }
    return f();
}
checkscope(); // 会返回什么？
```

```javascript
var scope = 'global';
function checkscope() {
    var scope = 'local';
    function f() { return scope; }
    return f;
}
checkscope()(); // 会返回什么？
```

在这里，我们想一下此法作用域的基本规则：JavaScript 函数的执行用到了作用域链，这个作用域链是在定义函数时创建的。
也就是说，不管这个函数 f() 在哪里被执行，其变量 scope 一定是局部变量。

闭包其实就是捕捉到局部变量，并一直保存下来。

### call()、apply() 和 bind() 方法

在 ECMAScript5 的严格模式中，call() 和 apply() 的第一个参数都会变为 this 的值，哪怕传入的是原始值，甚至是 null 和 undefined。
在 ECMAScript3 和非严格模式中，传入 null 和 undefined 会被替换为全局对象，而其它原始值则会被相应的包装对象替代。

bind() 方法实现：

```javascript
function bind(f, o) {
    if (f.bind) return f.bind(o)
    else return function () {
        return f.apply(o, arguments);
    }
}
```

### ES5 实现 ES6 的 Set 对象

```javascript
function Set() {
    this.values = {};                   // 集合数据保存在对象的属性
    this.n = 0;                         // 集合中值的个数
    this.add.apply(this, arguments);    // 把所有参数都添加进这个集合
}

Set._v2s.next = 100;    // 设置初始 id 值

Set._v2s = function (val) {
    switch(val) {
        case undefined: return 'u';                 //
        case null: return 'n';                      //  特殊的原始值
        case true: return 't';                      //  返回其首字母
        case false: return 'f';                     //
        default: switch(typeof val) {               
            case 'number': return '#' + val;        // 数字带有 # 前缀
            case 'string': return '"' + val;        // 字符串带有 " 前缀
            default: return '@' + objectId(val);    // object 和 function 该有 @ 前缀
        }
    }
    /**
     * 对任意对象来说，都会返回一个字符串
     * 针对不同的对象，这个函数会返回不同的字符串
     * 对于同一个对象的多次调用，总是返回相同的字符串
     * hasOwnProperty() 方法只检查是否是自身属性
    */
    function objeceId(o) {
        var prop = '|**objectId**|';                // 私有属性，用以存放 id
        if (!o.hasOwnProperty(prop)) {              // 如果对象没有 id
            o[prop] = Set._v2s.next++;              // 将下一个值赋给它
        }
        return o[prop];                             // 返回这个 id
    }
}

Set.prototype.add = function () {
    for (var i = 0; i < arguments.length; i++) {
        var val = arguments[i];
        var str = Set._v2s(val);
        if (!this.values.hasOwnProperty(str)) {
            this.values[str] = val;
            this.n++;
        }
    }
    return this;
}

Set.prototype.remove = function () {
    for (var i = 0; i < arguments.length; i++) {
        var str = Set._v2s(arguments[i]);
        if (this.values.hasOwnPrototype(str)) {
            delete this.values[str];
            this.n--;
        }
    }
    return this;
}

/**
 * 判断集合是否包含这个值
*/
Set.prototype.contains = function (value) {
    return this.values.hasOwnProperty(Set._v2s(value));
}

Set.prototype.size = function () {
    return this.n;
}

Set.prototype.foreach = function (f, context) {
    for (var s in this.values) {
        if (this.values.hasOwnProperty(s)) {
            f.call(context, this.values[s]);
        }
    }
}
```

### ES5 实现枚举类型

```javascript
function inherit(p) {
    // p 是一个对象，而不能是 null
    if (p == null) throw TypeError();
    // 如果有 Object.create() 方法，直接使用
    if (Object.create) return Object.create(p);
    // 否则获取 p 的类型进行检测
    var t = typeof p;
    if (t !== 'object' && t !== 'function') throw TypeError();
    // 定义一个空构造函数
    function f() {};
    // 将它的原型属性设置为 p
    f.prototype = p;
    // 返回一个使用 f() 方法创建的 p 的对象
    return new f();
}

function enumeration(nameToValues) {
    var enumeration = function() { throw "Can't Instantiate Enumerations"; }
    var proto = enumeration.prototype = {
        constructor: enumeration,
        toString: function () { return this.name; },
        valueOf: function () { return this.value; },
        toJSON: function () { return this.name; }
    }

    // 用以存放枚举对象的数组
    enumeration.values = [];

    for (name in nameToValues) {
        // 首先返回一个以 proto 继承的对象
        var e = inherit(proto);
        e.name = name;
        e.value = nameToValues[name];
        enumeration[name] = e;
        enumeration.values.push(e);
    }

    enumeration.foreach = function (f, c) {
        for (var i = 0; i < this.values.length; i++) {
            f.call(c, this.value[i]);
        }
    }

    return enumeration;
}

var Coin = enumeration({Penny: 1, Nickel: 5, Dime: 10, Quarter: 25});
var c = Coin.Dime;
c instanceof Coin;                      // => true
c.constructor == Coin;                  // => true
Coin.Quarter + 3 * Coin.Nickel;         // => 40
Coin.Dime == 10;                        // => true
Coin.Dime > Coin.Nickel;                // => true
String(Coin.Dime) + ':' + Coin.Dime;    // => 'Dime: 10'

```

```javascript

```