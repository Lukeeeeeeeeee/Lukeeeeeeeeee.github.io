---
title: 一道关于JavaScript综合面试题
date: 2020-03-30 20:29:04
tags: [面试题汇总]
---

>作者：Wscats
>https://github.com/Wscats/articles/issues/85

以下是我之前确实面试过的一道题，当时答的很不好，事后也想不起来了，正好前两天看到一个公众号发出来了，把思路也给捋清楚了，就记录到这里了。

#### 题目

```
function Foo() {
    getName = function() { alert(1); }
    return this;
}
Foo.getName = function() { alert(2); }
Foo.prototype.getName = function() { alert(3); }
var getName = function() { alert(4); }
function getName() { alert(5); }

// 请写出一下输出结果
Foo.getName();
getName();
Foo().getName();
getName();
new Foo.getName();
new Foo().getName();
new new Foo().getName();
```
#### Foo.getName();

让我们来分析下，
首先定义了一个 Foo 函数，
之后为Foo创建了一个叫 getName 的静态属性并赋值一个匿名函数，
之后为Foo的原型对象新创建了一个叫 getName 的匿名函数。
创建了一个名为 getName 函数表达式，
声明了一个 getName 函数。

第一问，Foo 函数也是对象，所以自然访问的就是函数上的静态属性，即`Foo.getName = function() { alert(2); }`;

以下例子用来加深理解

```
function User(name) {
    var name = name; // 私有属性
    this.name = name; // 公开属性
    function getName() { // 私有方法
        alert(1);
        console.log(this);
    }
}
User.prototype.getName = function() { // 公有方法
    alert(2);
}
User.name = 'user'; // 静态属性
User.getName = function() { // 静态方法
    alert(3);
}
```

#### getName();

第二问，直接调用 getName 函数。既然是直接调用，那么就是访问当前上下文作用域内的叫 getName 的函数，所有主要看 4 和 5。
这里有个坑，一是变量声明提升，二是函数表达式和函数声明的区别。

相关文档可参考 {% post_link 关于JavaScript的函数声明和函数表达式 %} 和 {% post_link 关于JavaScript的变量提升 %}

##### 函数声明

```
// 函数声明
function test() {
    alert(1);
}
```

##### 函数表达式

````
// 函数表达式
var test = function () {
    alert(2);
}
````
先看下面这个经典问题，在一个程序里面同时用函数声明和函数表达式定义一个名为 getName 的函数

````
getName(); // oaoafly
var getName = function() {
    console.log('wscat');
}
getName(); // wscat
function getName() {
    console.log('oaoafly');
}
getName(); // wscat
````

上面的代码看起来很类似，但实际上，JavaScript函数上的一个“缺陷”就体现在JavaScript两种类型的函数定义上。

- **JavaScript解释器中存在一种变量声明被提升的机制，也就是说函数声明会被提升到作用域的最前面，即使写代码的时候卸载最后面，也还是会被提升至最前面**
- **而用函数表达式创建的函数是在运行时进行赋值，且要等到表达式赋值完成后才调用**

```
var getName; // 变量被提升，此时为 undefined

getName(); // oaoafly 函数被提升  这里受函数声明的影响，虽然函数声明在最后，但是被提升到最前面来了
var getName = function() {
    console.log('wscat');
}
getName(); // wscat  函数表达式此时才被赋值并覆盖函数声明的定义
function getName() {
    console.log(oaoafly);
}
getName(); // wscat  这里就执行了函数表达式的值
```

所以可以分解为这两个简单的问题来看清楚区别的本质

```
var getName;
console.log(getName); // undefined
getName(); // Uncaught TypeError: getName is not a function
var getName = function() {
    console.log('wscat');
}
getName(); // wscat
```

```
var getName;
console.log(getName); // function getName() { console.log('oaoafly'); }
getName(); // oaoafly
function getName() {
    console.log('oaoafly');
}
```
这个区别看似微不足道，但在某些情况下确实是一个难以察觉并且“致命”的陷阱。出现这个陷阱的本质原因体现在这两种类型在函数提升和运行时机（解析时/运行时）上的差异。
当然我们给一个总结：JavaScript中**函数声明**和**函数表达式**是存在区别的，**函数声明**在JavaScript解析时，进行函数提升，因此在同一个作用域内，不管函数声明在哪里定义，该函数都可以进行调用。而**函数表达式**的值是在JavaScript运行时确定，并且在表达式赋值完成后，该函数才能调用。

所以第二问 `getName();` 答案为4，5的函数声明被 4 的函数表达式覆盖了。

#### Foo().getName();

`Foo().getName();` 先执行了 Foo 函数，然后调用 Foo 函数的返回值（this）对象的 getName 属性函数。
Foo 函数的第一句 `getName = function() { alert(1); };` 是一句函数赋值语句，注意它没有 var 声明，所以先向当前 Foo 函数作用域中寻找 getName 变量，没有。再向当前函数作用域上层，即外层作用域内寻找是否含有 getName 变量，找到了，也就是第二问 `var getName = function() { alert(4); };`，将此变量的值赋值为 `function() { alert(1); };`。
此处实际上是将外层作用域内的 getName 函数修改了。

>注意：此处若依然没有找到会一直向上查找到 window 对象，若 window 对象中也没有 getName 属性，就在 window 对象中创建一个 getName 变量并赋值 `function() { alert(1); };`。

之后 Foo 函数的返回值是 this。this 的指向是由所在函数的调用方式决定的。而此处是直接调用方式，this 指向 window 对象，相当于执行 window.getName()，而 window 中的 getName 已经被修改为 `function() { alert(1); };`，所以最终输出 1。
