---
title: 一道关于JavaScript综合面试题
date: 2020-03-30 20:29:04
tags: [面试题汇总]
---

> 作者：Wscats
> https://github.com/Wscats/articles/issues/85

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
之后为 Foo 创建了一个叫 getName 的静态属性并赋值一个匿名函数，
之后为 Foo 的原型对象新创建了一个叫 getName 的匿名函数。
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

```
// 函数表达式
var test = function () {
    alert(2);
}
```

先看下面这个经典问题，在一个程序里面同时用函数声明和函数表达式定义一个名为 getName 的函数

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

上面的代码看起来很类似，但实际上，JavaScript 函数上的一个“缺陷”就体现在 JavaScript 两种类型的函数定义上。

- **JavaScript 解释器中存在一种变量声明被提升的机制，也就是说函数声明会被提升到作用域的最前面，即使写代码的时候卸载最后面，也还是会被提升至最前面**
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
当然我们给一个总结：JavaScript 中**函数声明**和**函数表达式**是存在区别的，**函数声明**在 JavaScript 解析时，进行函数提升，因此在同一个作用域内，不管函数声明在哪里定义，该函数都可以进行调用。而**函数表达式**的值是在 JavaScript 运行时确定，并且在表达式赋值完成后，该函数才能调用。

所以第二问 `getName();` 答案为 4，5 的函数声明被 4 的函数表达式覆盖了。

#### Foo().getName();

`Foo().getName();` 先执行了 Foo 函数，然后调用 Foo 函数的返回值（this）对象的 getName 属性函数。
Foo 函数的第一句 `getName = function() { alert(1); };` 是一句函数赋值语句，注意它没有 var 声明，所以先向当前 Foo 函数作用域中寻找 getName 变量，没有。再向当前函数作用域上层，即外层作用域内寻找是否含有 getName 变量，找到了，也就是第二问 `var getName = function() { alert(4); };`，将此变量的值赋值为 `function() { alert(1); };`。
此处实际上是将外层作用域内的 getName 函数修改了。

> 注意：此处若依然没有找到会一直向上查找到 window 对象，若 window 对象中也没有 getName 属性，就在 window 对象中创建一个 getName 变量并赋值 `function() { alert(1); };`。

之后 Foo 函数的返回值是 this。this 的指向是由所在函数的调用方式决定的。而此处是直接调用方式，this 指向 window 对象，相当于执行 window.getName()，而 window 中的 getName 已经被修改为 `function() { alert(1); };`，所以最终输出 1。

此处考察了两个知识点，一个是**变量作用域问题**，一个是**this 指向问题**。我们利用下面的代码来回顾一下

```
var name = 'wscats'; // 全局变量
window.name = 'wscats'; // 全局变量
function getName() {
    name = 'oaoafly'; // 没有用 var 声明，所以是全局变量，当前作用域没有 name
    var privateName = 'stacsw';
    return function() {
        console.log(this); // window
        return privateName;
    }
}
var getPrivate = getName('Hello'); // 当前传参是局部变量，但函数中没有接受这个参数
console.log(name); // oaoafly
console.log(getPrivate()); // stacsw
```

因为 JavaScript 没有块级作用域，但是函数是能产生一个作用域的，函数内部不同定义值的方法会直接或者间接影响到全局或者局部变量，函数内部的私有变量可以用闭包获取。
而关于 this，this 的指向在函数定义的时候是确定不了的，只有函数执行的时候才能确定 this 到底指向谁，实际上 this 最终指向的是那个调用它的对象。
所以第三问 `Foo().getName();` 中实际就是 window 在调用 Foo() 函数，所以 this 的指向就是 window。

```
window.Foo().getName();
// -> window.getName();
```

#### getName();

直接调用 getName 函数，相当于 window.getName()，因为这个变量已经被 Foo 函数执行时修改了，所以结果和第三问 `Foo().getName();` 相同，为 1。也就是说 Foo 执行后把全局的 getName 函数给重写了一次。

#### new Foo.getName();

第五问 `new Foo.getName();` 此处考察的是 JavaScript 的运算符优先级问题，这个还是挺难的，我当时看到这都懵了，没见过这么写的啊...
下面是 JavaScript 运算符的优先级表格，从高(20)到低(1)排列。可参见[MDN 运算符优先级](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)

| 优先级 | 运算类型             | 关联性      | 运算符               |
| ------ | -------------------- | ----------- | -------------------- |
| 20     | 圆括号               | n/a(不相关) | `( ... )`            |
| 19     | 成员访问             | 从左到右    | `... . ...`          |
|        | 需要计算的成员访问   | 从左到右    | `... [ ... ]`        |
|        | new(带参数列表)      | n/a         | `new ... ( ... )`    |
|        | 函数调用             | 从左到右    | `... ( ... )`        |
|        | 可选链               | 从左到右    | `?.`                 |
| 18     | new(无参数列表)      | 从右到左    | `new ...`            |
| 17     | 后置递增(运算符在后) | n/a         | `... ++`             |
|        | 后置递减(运算符在后) |             | `... --`             |
| 16     | 逻辑非               | 从右到左    | `! ...`              |
|        | 按位非               |             | `~ ...`              |
|        | 一元加法             |             | `+ ...`              |
|        | 一元减法             |             | `- ...`              |
|        | 前置递增             |             | `++ ...`             |
|        | 前置递减             |             | `-- ...`             |
|        | typeof               |             | `typeof ...`         |
|        | void                 |             | `void ...`           |
|        | delete               |             | `delete ...`         |
|        | await                |             | `await ...`          |
| 15     | 幂                   | 从右到左    | `... ** ...`         |
| 14     | 乘法                 | 从左到右    | `... * ...`          |
|        | 除法                 |             | `... / ...`          |
|        | 取模                 |             | `... % ...`          |
| 13     | 加法                 | 从左到右    | `... + ...`          |
|        | 减法                 |             | `... - ...`          |
| 12     | 按位左移             | 从左到右    | `... << ...`         |
|        | 按位右移             |             | `... >> ...`         |
|        | 无符号右移           |             | `... >>> ...`        |
| 11     | 小于                 | 从左到右    | `... < ...`          |
|        | 小于等于             |             | `... <= ...`         |
|        | 大于                 |             | `... > ...`          |
|        | 大于等于             |             | `... >= ...`         |
|        | in                   |             | `... in ...`         |
|        | instanceof           |             | `... instanceof ...` |
| 10     | 等号                 | 从左到右    | `... == ...`         |
|        | 非等号               |             | `... != ...`         |
|        | 全等号               |             | `... === ...`        |
|        | 非全等号             |             | `... !== ...`        |
| 9      | 按位与               | 从左到右    | `... & ...`          |
| 8      | 按位异或             | 从左到右    | `... ^ ...`          |
| 7      | 按位或               | 从左到右    | `... \| ...`         |
| 6      | 逻辑与               | 从左到右    | `... && ...`         |
| 5      | 逻辑或               | 从左到右    | `... \|\| ...`       |
| 4      | 条件运算符           | 从右到左    | `... ? ... : ...`    |
| 3      | 赋值                 | 从右到左    | `... = ...`          |
|        |                      |             | `... += ...`         |
|        |                      |             | `... -= ...`         |
|        |                      |             | `... *= ...`         |
|        |                      |             | `... /= ...`         |
|        |                      |             | `... %= ...`         |
|        |                      |             | `... <<= ...`        |
|        |                      |             | `... >>= ...`        |
|        |                      |             | `... >>>= ...`       |
|        |                      |             | `... &= ...`         |
|        |                      |             | `... ^= ...`         |
|        |                      |             | `... \|= ...`        |
| 2      | yield                | 从右到左    | `yield ...`          |
|        | yield\*              |             | `yield* ...`         |
| 1      | 展开运算符           | n/a         | `...` ...            |
| 0      | 逗号                 | 从左到右    | `... , ...`          |

从上面优先级表中第 19 和第 18 中可以看出关于 new 的优先级，`.成员访问` 和 `new(带参数列表)` 和 `函数调用` 同级，比 `new(无参数列表)` 高。

```
new Foo.getName();
// -> new (Foo.getName)();
```

- 点的优先级(19)比 `new(无参数列表)` (18) 优先级高
- 当点运算完后又因为有个括号 `()`，此时就是变成 `new有参数列表` (19)，所以直接执行 `new`，当然也可能有朋友会有疑问为什么遇到 `()` 不函数调用再 `new` 呢，那是因为 `函数调用` (18)比 `new有参数列表` (19)优先级低。

> `.成员访问`(19) > `函数调用`(19) > `new(无参数列表)`(18)

综上所述，第五问 `new Foo.getName();` 答案为 2。

#### new Foo().getName();

第六问 `new Foo().getName();`，比第五问 `new Foo.getName();` 出多了一个 `()`，优先级也就发生了变化

> new Foo().getName();
> // -> (new Foo()).getName();

根据优先级表，首先是 `new(有参数列表)`(19)跟 `点` 和 `函数调用` 的优先级是同级，同级按照从左向右的执行顺序，所以先执行 `new(有参数列表)`(19)再执行 `点` 的优先级(19)，最后再 `函数调用` (19)。

> `new(有参数列表)`(19) > `.成员访问`(19) > `函数调用`(19)

这里还有个小知识点，Foo 作为构造函数有返回值，这里说下 JavaScript 中构造函数返回值的问题。

##### 构造函数的返回值

在传统语言中，构造函数不应该有返回值，实际执行的返回值就是此构造函数的实例化对象。
而在 JavaScript 中的构造函数可以有返回值也可以没有。

1. 没有返回值则按照其他语言一样返回实例化对象。

```
function Foo(name) {
    this.name = name
}
console.log(new Foo('wscat'));
```

![](https://pic.downk.cc/item/5e832ef7504f4bcb0433479c.png)

2. 若有返回值，则检查其返回值是否为引用类型。如果是非引用类型，如基本类型(String, Number, Boolean, Undefined, Null, Symbol)则与无返回值相同，返回其实例化对象。

```
function Foo(name) {
    this.name = name;
    return 200;
}
console.log(new Foo('wscat'));
```

![](https://pic.downk.cc/item/5e832ff1504f4bcb043409d6.png)

3. 若返回值是引用类型，则返回这个引用类型。

```
function Foo(name) {
    this.name = name;
    return {
        age: 16
    }
}
console.log(new Foo('wscat'));
```

![](https://pic.downk.cc/item/5e83305f504f4bcb04346114.png)

原题中，由于返回的是 this，而 this 在构造函数中本来就代表当前实例化对象，所以 Foo 函数返回实例化对象。
之后调用实例化对象的 getName 函数，因为在 Foo 构造函数中没有为实例化对象添加任何属性，所以在当前对象的原型对象中寻找 getName 函数。
当然这里再拓展个题外话，如果构造函数和原型链都有相同的方法，如下面的代码，那么默认拿构造函数的公有方法而不是原型链上的方法。

```
function Foo(name) {
    this.name = name;
    this.getName = function() {
        return this.name;
    }
}
Foo.prototype.name = 'oaoafly';
Foo.prototype.getName = function() {
    return 'oaoafly';
}
console.log((new Foo('wscat')).name); // wscat
console.log((new Foo('wscat')).getName()); // wscat
```

#### new new Foo().getName();

第七问 `new new Foo().getName();` 同样是运算符优先级问题。

```
new new Foo().getName();
// -> new ((new Foo()).getName)();
```

> `new(有参数列表)`(19) > `.成员访问`(19) > `new(有参数列表)`(19)

先初始化 Foo 的实例化对象，然后将其原型上的 getName 函数作为构造函数再次 new，所以最终结果为 3。

#### 答案

```
function Foo() {
    getName = function () { alert (1); };
    return this;
}
Foo.getName = function () { alert (2);};
Foo.prototype.getName = function () { alert (3);};
var getName = function () { alert (4);};
function getName() { alert (5);}

//答案：
Foo.getName(); // 2
getName(); // 4
Foo().getName(); // 1
getName(); // 1
new Foo.getName(); // 2
new Foo().getName(); // 3
new new Foo().getName(); // 3
```

#### 后续

增加一下难度，在 Foo 函数里面多加一个公有方法 getName。

```
function Foo() {
    this.getName = function() {
        console.log(3);
        return {
            getName: getName //这个就是第六问中涉及的构造函数的返回值问题
        }
    }; 
    //这个就是第六问中涉及到的，JS构造函数公有方法和原型链方法的优先级
    getName = function() {
        console.log(1);
    };
    return this
}
Foo.getName = function() {
    console.log(2);
};
Foo.prototype.getName = function() {
    console.log(6);
};
var getName = function() {
    console.log(4);
};
function getName() {
    console.log(5);
}

//答案：
Foo.getName(); // 2
getName(); // 4
console.log(Foo()) // window
Foo().getName(); // 1
getName(); // 1
new Foo.getName(); // 2
new Foo().getName(); // 3
//多了一问
new Foo().getName().getName(); // 3 1
```
