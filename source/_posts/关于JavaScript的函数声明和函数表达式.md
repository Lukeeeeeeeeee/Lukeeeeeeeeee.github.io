---
title: 关于JavaScript的函数声明和函数表达式
date: 2020-03-30 22:03:15
tags: [JavaScript]
---

JavaScript定义函数有两种类型

#### 函数声明

```
function wscat(type) {
    console.log(type);
}
```

#### 函数表达式

```
var oaoafly = function(type) {
    console.log(type);
}
```

先看下面这个经典的问题，在一个程序里面同时用函数声明和函数表达式定义一个名为 getName 的函数

```
getName(); // oaoafly
var getName = function() {
    console.log('wscat');
}
getName(); // wscat
function getName() {
    console.log('oaoafly');
}
getName(); // wscat
```

上面的代码看起来很类似，感觉也什么太大差别。但实际上，JavaScript函数上的一个“陷阱”就体现在JavaScript两种类型的函数定义上。

- JavaScript解释器中存在一种变量声明被提升的机制，也就是说函数声明会被提升到作用域的最前面，即使写代码的时候写在最后面，也还是会被提升至最前面。
- 用函数表达式创建的函数是在运行时进行赋值，且要等到表达式赋值完成后才能调用。

```
var getName; // 变量被提升，此时为undefined
getName(); // oaoafly 函数声明被提升
var getName = function() {
    console.log('wscat');
}
getName(); // wscat 函数表达式此时才覆盖函数声明的定义
function getName() {
    console.log('oaoafly');
}
getName(); // wscat 这里执行的是函数表达式
```

所以可以分解为这两个简单的问题来看清楚区别的本质

```
var getName;
console.log(getName) // undefined
getName(); // Uncaught TypeError: getName is not a function
var getName = function() {
    console.log('wscat');
}
```

```
var getName;
console.log(getName); // function getName() { console.log('oaoafly'); }
getName(); // oaoafly
function getName() {
    console.log('oaoafly');
}
```

这个区别看似微不足道，但在某些情况下确实是一个难以察觉并且“致命”的陷阱。出现这个陷阱的本质原因体现在这两种类型在函数提升和运行时机(解析时/运行时)上的差异。

总结：JavaScript中的函数声明和函数表达式是存在区别的，函数声明在JavaScript解析时进行函数提升，因此在同一个作用域内，不管函数声明在哪里定义，该函数都可以进行调用。而函数表达式的值是在JavaScript运行时确定，并且在表达式赋值完成后，该函数才能调用。