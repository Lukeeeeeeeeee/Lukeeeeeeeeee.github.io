---
title: JavaScript基础学习-关系表达式
date: 2020-05-24 15:03:10
tags: [JavaScript]
---

这里不具体介绍所有的，主要介绍下 instanceof 运算符的运行原理。

关系表达式有：==、 ===、 !=、 !==、 <、 >、 <=、 >=、 in、 instanceof

### in 运算发

其中 in 运算符是用来检测对象中是否存在某个属性

```javascript
var point = { x: 1, y: 1};
'x' in point; // => true
'z' in point; // => false
'toString' in point // => true: 对象继承了 toString() 方法

var data = [7, 8, 9];
'0' in data; // => true: 0 在数组的索引中
1 in data; // => true: 数字转换成字符串
3 in data; // => false: 没有索引为 3 的元素
```

### instanceof 运算符

instanceof 运行原理：为了计算表达式 o instanceof f，JavaScript 首先计算 f.prototype，然后在原型链中查找 o，如果找到，那么 o 是 f 的一个实例，返回 true。
如果 f.prototype 不在 o 的原型链中的话，那么 o 不是 f 的实例，返回 false。

