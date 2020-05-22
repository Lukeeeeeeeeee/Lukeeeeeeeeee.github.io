---
title: JavaScript基础学习-不可变的原始值和可变的对象引用
date: 2020-05-21 22:34:18
tags: [JavaScript]
---

### JavaScript的原始值和对象(基本类型和引用类型)

JavaScript的基础类型(原始值)：undefined、null、string、number、boolean、symbol

#### 原始值和对象区别：

- 原始值是不可改变的，任何方法都不可改变一个原始值。

```javascript
var s = 'hello';
s.toUpperCase();    // 返回 HELLO，但是并没有改变 s 的值
s                   // => hello，原始值并未发生改变
```

#### 原始值和对象的比较

原始值的比较是值的比较。
对象的比较并非是值的比较：即使两个对象包含同样的属性及同样的值，也是不相等的。

对象值是引用，对象的比较是引用的比较：当且仅当它们指向同一个地址内存，它们才相等。

```javascript
var a = [];
var b = a;
b[0] = 1;
a[0];       // => 1
a === b;    // true
```

用一道题来结束：如何通过比较数组的每个元素判断两个数组是否相等？

```javascript
function equalArray(a, b) {
    if (a.length != b.length) return false; // 两个数组长度不同，肯定不相等
    for (let i = 0; i < a.length; i++) {
        if (a[i] !== b[i]) return false;    // 如果有任意元素不等，这两个数组就不想等
    }
    return true;
}
```