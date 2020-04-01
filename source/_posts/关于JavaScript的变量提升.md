---
title: 关于JavaScript的变量提升
date: 2020-03-30 22:20:29
tags:
---

```
var v = 'hello world';
alert(v);
```

弹出 Hello World

```
var v = 'hello world';
(function() {
    alert(v);
    var v = 'I love you';
})();
```
也是弹出 hello world

```
var v = 'hello world';
(function() {
    alert(v);
    var v = 'i love you';
})();
```
弹出 undefined

这里隐藏了一个陷阱，就是JavaScript中的变量提升

它相当于

```
var v = 'hello world';
(function() {
    var v;
    alert(v);
    v = 'i love you';
})();
```

变量提升，简单的理解，就是把变量提升到函数的最顶端的地方。需要说明的是，变量提升只是提升变量的声明，并不是把赋值也提升上来，没有赋值的变量初始值是undefined。所以上面就出现了声明undefined的var，因为赋值在后面，声明提升在了前面。

```
function foo() {
    if (false) {
        var x = 1;
    }
    return;
    var y = 1;
}
function foo() {
    var x, y;
    if (false) {
        x = 1;
    }
    return;
    y = 1;
}
```

还有一点注意的是因为JavaScript是函数级作用域，只有函数才会创建新的作用域，而不像其他语言有块级作用域，例如块，就像if语句，在上面的例子中，不管会不会进入if代码块，函数声明都会提升到当前作用域的顶部，得到执行，在JavaScript中并不会创建一个新的作用域。
从这里我们应该体会到，当我们在写JavaScript code 的时候，我们需要把变量放到块级作用域的顶端，不然容易发生一些意想不到的错误。
注意：ES5只有全局作用域和函数作用域，没有块级作用域。

还有一种就是函数提升

```
function myTest() {
    foo();
    function foo() {
        alert('hello world');
    }
}
myTest();
```

弹出 'hello world' // 这里函数声明提升

```
function myTest() {
    foo();
    var foo = function() {
        alert('我不会被弹出');
    }
}
myTest();
```

报错：foo不是函数
相当于

```
function myTest() {
    var foo;
    foo();
    foo = function() {
        alert('我不会被弹出');
    }
}
myTest();
```
