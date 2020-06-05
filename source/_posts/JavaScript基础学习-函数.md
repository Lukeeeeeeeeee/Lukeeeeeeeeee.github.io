---
title: JavaScript基础学习-函数
date: 2020-06-02 22:23:07
updated_at: 2020-06-05 22:18:11
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

```javascript

```