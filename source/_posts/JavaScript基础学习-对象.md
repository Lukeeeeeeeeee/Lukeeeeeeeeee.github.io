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

### 属性的特性

数据属性的四个特性：值(value)、可写(writable)、可配置(configurable)、可枚举(enumerable)
存取器属性的四个特性：读取(get)、写入(set)、可配置、可枚举

Object.getOwnPropertyDescriptor() 可以获取某个对象自有属性的属性描述符：

```javascript
Object.getOwnPropertyDescriptor({x: 1}, 'x'); // {value: 1, writable: true, configurable: true, enumerable: true}

function random = {
   get octet() { return Math.floor(Math.random() * 256); }
}
Object.getOwnPropertyDescriptor(random, 'octet'); // {get: /* func */, set: undefined, configurable: true, enumerable: true}

// 对于不存在的属性和继承属性，返回 undefined
Object.getOwnPropertyDescriptor({}, 'x'); // undefined
Object.getOwnPropertyDescriptor({}, 'toString'); // undefined, 继承属性
```

如果要想获取继承属性的特性，需要遍历原型链，使用 Object.getPropertyOf()。

```javascript
Object.getPropertyOf({});
```

要想检测一个对象是否是另一个对象的原型(或处于原型链)，可以通过 Object.isPropertyOf() 方法。

```javascript
// 通过 p.isPropertyOf(o) 来检测 p 是否是 o 的原型
var p = {x: 1};
var o = Object.create(p);
p.isPropertyOf(o); // => true: o 继承 p
Object.prototype.isPropertyOf(o) // => true: p 继承自 Object.prototype
```


如果想要设置属性的特性，或者想让某个新建属性具有某种特性，可以通过调用 Object.defineProperty(要修改的对象, 要创建或者修改的属性名称，属性描述符对象)

注意：此方法不能修改继承属性

```javascript
var o = {};

Object.defineProperty({}, 'x', {
   value: 1,
   wriable: true,
   configurable: true,
   enumerable: false
})

//属性存在，但是不可以枚举
o.x; // => 1
Object.keys(o); // => []

// 现在对属性 x 做修改，让它变成只读
Object.defineProperty({}, 'x', { writable: false });

o.x = 2; // 无法改变且不会报错，在严格模式下会抛出类型错误异常
o.x = 1;

// 但属性却是可以配置的，因此通过下面这种方式，还是可以改变值的
Object.defineProperty({}, 'x', { value: 2 });
o.x; // => 2

// 现在将 x 从数据属性改为存取器属性
Object.defineProperty({}, 'x', { get: function() { return 0; } });
o.x; // => 0
```

如果通过 Object.defineProperty() 新创建了一个值，那么它的默认特性值是 undefined 或者 false

```javascript
var o = {};

Object.defineProperty(o, 'x', {});
Object.getOwnPropertyDescriptor(o, 'x'); // {value: undefined, writable: false, enumerable: false, configurable: false}
```

如果同时修改多个值，需要使用 Object.defineProperties(要修改的对象, 映射表)

```javascript
var p = Object.definedProperties({}, {
   x: {value: 1, writable: true, configurable: true, enumerable: true},
   y: {value: 1, writable: true, configurable: true, enumerable: true},
   r: {
      get: function() {
         return Math.sqrt(this.x * this.x + this.y * this.y)
      },
      configurable: true,
      enumerable: true
   }
})
```

关于 Object.defineProperty() 和 Object.defineProperties() 相关规则：

- 如果对象是不可扩展的，则可以编辑已有的自有属性，但不能给它添加新属性
- 如果属性是不可配置的，则不能修改它的可配置性和可枚举性
- 如果存取器属性是不可配置的，则不能修改其 getter 和 setter 方法，也不能将它转换成数据属性
- 如果数据属性是不可配置的，则不能将它转换成存取器属性，也不能将它的可写性从 false 修改为 true，但可以从 true 修改为 false
- 如果数据属性是不可配置且不可写的，则不能修改它的值。
- 如果数据属性是可配置但不可写，属性的值确实可以修改的(因为是可以配置的，所以先将它标记为可写的，然后修改它的值，最后在转换成不可写的)

简单概括：

1. 可写性控制着对值特性的修改
2. 可配置性控制着对其他特性(包括属性是否可以删除)的修改
3. 如果属性可配置的话，则可以修改不可写属性的值
4. 如果属性是不可配置的，则仍然可以将可写属性修改为不可写属性

根据这些了解的内容，我们创建一个可以复制属性的特性的函数：

```javascript
/**
 * 给 Object.prototype 添加一个不可枚举的 extend() 方法
 * 这个方法继承自调用它的对象，将作为参数传递的对象的属性一一复制
 * 除了值以外，也复制属性的所有特性，除非在目标对象中存在同名的属性，
 * 参数对象的所有自有对象(包括不可枚举的属性)也会一一复制
 */
Object.defineProperty(Object.prototype, 'extend', {
   writable: true,
   configurable: true,
   enumerable: false,
   value: function(o) {
      // 得到所有的自有属性，包括不可枚举属性
      var names = Object.getOwnPropertyNames();
      for (var i = 0; i < names.length; i++) {
         // 如果属性已经存在，则跳过
         if (names[i] in this) continue;
         // 获取 o 中的属性的描述符
         var desc = Object.getOwnPropertyDescriptor(o, names[i]);
         // 用它给 this 创建一个属性
         Object.defineProperty(this, names[i], desc);
      }
   }
})
```

### 类属性

对象的类属性是一个字符串，用以表示对象的类型信息。格式：[object class]。以下函数就可以返回传递给它的任意对象的类：

```javascript
function classof(o) {
   if (o === null) return 'Null';
   if (o === undefined) return 'Undefined';
   return Object.prototype.toString.call(o).slice(8, -1);
}

// 通过对象直接量和 Object.create() 方法创建的对象的类属性是 'Object'，同时那些自定义的构造函数创建的对象也是 'Object'，因此对于自定义的类，没办法通过类属性区分对象的类：
classof(null);          // => 'Null'
classof(undefined);     // => 'Undefined'
classof(1);             // => 'Number'
classof('');            // => 'String'
classof(false);         // => 'Boolean'
classof({});            // => 'Object'
classof([]);            // => 'Array'
classof(/./);           // => 'Regexp'
classof(new Date());    // => 'Date'
classof(window);        // => 'Window'

function f() {};
classof(new f());       // => 'Object'

```

注意：除了数字，字符串、布尔值和对象是可以直接调用 toString() 方法的。

```javascript
1.toString(); // 报错，因为 '.' 被当作小数点

var a = 1;
a.toString(); // => '1'

''.toString(); // => ""
true.toString(); // => 'true'
var o = Object.seal(
   Object.create(
      Object.freeze({x: 1}),
      { y: { value: 2, writable: true } }
   )
)
```

### 可扩展性

通过 Object.isExtensible() 方法判断该对象是否是可扩展的。

如果想将对象转换为不可扩展的，可以调用 Object.preventExtensions() 方法，将待转换的对象作为参数传递进去。
注意，一旦将对象转换为不可扩展的，就无法再将其转换成可扩展的了。
一般来说，不可扩展对象的属性仍然是可被删除的。
同样需要注意的是，Object.preventExtensions() 影响只是对象本身的可扩展性。如果给一个不可扩展的对象的原型添加属性，不可扩展的对象还是会继承这些新属性。

```javascript
const object1 = {x: 1};
Object.preventExtensions(object1);
try {
   Object.defineProperty(object1, 'property1', {
      value: 42
   })
} catch(e) {
   console.log(e); // TypeError: Cannot define property property1, object is not extensible
}

delete object1.x; // => true

object1.x; // => undefined
```

通过 Object.isSealed() 来检测对象是否是封闭的。

Object.seal() 和 Object.preventExtensions() 相似，但是 Object.seal() 还将对象设置为不可配置的。
也就是说不能给这个对象添加新属性，同时它已有的属性也不能删除和配置，不过它已有的属性是可写的，则仍然可以设置。

```javascript
const object1 = {x: 1};
var o = Object.seal(object1);
object1.x = 33;
console.log(object1.x); // => 33

delete object1.x;
console.log(object.x); // => 33
```

通过 Object.isFrozen() 来检测对象是否冻结。

Object.freeze() 方法可以冻结一个对象。一个被冻结的对象再也不能被修改；不能像这个对象添加属性、不能删除已有的属性，不能修改该对象已有的属性的可配置性、可扩展性、可枚举性以及不可修改已有属性的值。
此外，冻结一个对象后该对象的原型也不能被修改。(如果对象的存取器属性具有 setter 方法，存取器则不受影响，仍然可以给属性赋值调用它们)。

```javascript
var obj = {x: 1};

Object.freeze(obj);

obj.x = 33; // Throws an error in strict mode

console.log(obj.x); // => 1
```

### 对象的其他方法

Object.assign() 方法会浅拷贝一个对象

Object.entries() 方法会返回一个给定对象自身可枚举属性的键值对数组，与 for...in 的区别是不会枚举原型中的属性：

```javascript
var o = {
   x: 1,
   y: 2
}

console.log(Object.entries(o)) // [ [ 'x', 1 ], [ 'y', 2 ] ]

for (let [key, value] of (Object.entries(o))) {
   console.log(`${key}, ${value}`)
}
// x, 1
// y, 2
```

Object.toString() 方法返回一个该对象的字符串

Object.toLocaleString() 方法返回一个表示这个对象的本地化字符串

Object.is() 方法判断两个值是否是相同的值。

判断依据：

- 两个值都是 undefined
- 两个值都是 null
- 两个值都是 true 或 false
- 两个值是由相同个数的字符按照相同的顺序组成的字符串
- 两个值指向同一个对象
- 两个值都是数字并且
   - 都是正零 +0
   - 都是负零 -0
   - 都是 NaN
   - 都是除零和 NaN 外的其它同一个数字

```javascript
Object.is('foo', 'foo');   // => true
Object.is(window, window); // => true

Object.is('foo', 'bar');   // => false
Object.is([], []);         // => false

var foo = {a: 1};
var bar = {a: 1};
Object.is(foo, foo);       // => true
Object.is(foo, bar);       // => false

Object.is(NaN, NaN);       // => true
NaN == NaN;                // => false
NaN === NaN;               // => false

// 特例
Object.is(0, -0);          // => false
Object.is(0, +0);          // => true
Object.is(-0, -0);         // => true
Object.is(NaN, 0/0);       // => true
0 == -0;                   // => true
0 === -0;                  // => true
```

Object.values() 方法返回一个给定对象自身的所有可枚举属性值的数组，不可枚举属性不会获取到。

```javascript
var obj = { foo: 'bar', baz: 42 };
console.log(Object.values(obj));    // ['bar', 42]

console.log(Object.values('foo'));  // ['f', 'o', 'o']
```