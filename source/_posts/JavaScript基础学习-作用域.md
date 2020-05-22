---
title: JavaScript基础学习-作用域
date: 2020-05-22 22:15:09
tags: [JavaScript]
---

### 变量作用域和函数作用域

全局变量的作用域就是全局，无论在哪里都可以使用。

函数内声明的变量，其作用域只在函数内(包括内嵌函数)可以使用。
别忘了，函数的参数，其作用域也是函数内部。
在函数内部，局部变量优先级高于同名的全局变量。如果一个函数的内部声明了一个局部变量或者函数带有的参数和全局变量同名，那么全局变量就会被局部变量或者参数覆盖。

```javascript
var scope = 'global';
function checkoutscope() {
    var scope = 'local';
    return scope;
}

checkoutscope(); // => 'local'
```

注意：全局变量可以不用 var 来声明，但是局部变量必须使用 var 来声明，否则会出现意想不到的bug。

```javascript
scope = 'global';
function checkscope2() {
    scope = 'local';
    myscope = 'local';
    return [scope, myscope];
}

checkscope2();  // => ['local', 'local']
scope // => 'local'
myscope // => 'local'
```

最后，让我们来看一看，函数嵌套的作用域

```javascript
var scope = 'global';
function checkoutscope() {
    var scope = 'local';
    function nested() {
        var scope = 'nested';
        return scope;
    }
    return nested();
}

checkoutscope(); // => 'nested'
```

### 声明提前

先看一段代码，猜猜它输出的结果是什么？

```javascript
var scope = 'global';
function checkoutscope() {
    console.log(scope);
    var scope = 'local';
    console.log(scope);
}
```

// => undefined
// => 'local'

为什么呢？全局变量不是可以在任何地方都可以使用嘛？其实是没错的

```javascript
var scope = 'global';
function checkoutscope() {
    console.log(scope); // => 'global'
}
```

但是，我们在上面还有一句话，**当函数内的局部变量与全局变量同名，局部变量优先级高于全局变量**。其实，一开始的那段代码与下面这段是等价的：

```javascript
var scope = 'global';
function checkoutscope() {
    var scope;
    console.log(scope); // => undefined
    scope = 'local';
    console.log(scope); // => 'local'
}
```

在 JavaScript 中通过 var 声明的变量会将其提升到最顶部，当运行到 var 语句时，才会被真正的赋值。

小知识：

当声明一个 JavaScript 全局变量时，实际上是给全局对象定义了一个属性：也就是在浏览器中，可以通过 window.[变量名] 的方式获取。
var 声明的变量是不可配置的，也就是说它是不可以被删除的，而直接给一个没有声明的变量赋值的话，JavaScript 会自动创建一个全局变量，但却可以删除。

```javascript
var test = 1;
test2 = 2;
this.test3 = 3;

delete test; // => false：变量并没有被删除
delete test2; // => true：被删除
delete this.test3; // => true：被删除
```