---
title: JavaScript基础学习-类型转换
date: 2020-05-21 22:58:48
tags: [JavaScript]
---

### 类型转换表

| 值                     | 转换为：字符串 | 数字     | 布尔值    | 对象                  |
| ---------------------- | -------------- | -------- | --------- | --------------------- |
| undefined              | 'undefined'    | NaN      | **false** | throw TypeError       |
| null                   | 'null'         | NaN      | **false** | throw TypeError       |
| true                   | 'true'         | 1        |           | throw TypeError       |
| false                  | 'false'        | 0        |           | throw TypeError       |
| ''(空字符串)           |                | 0        | **false** | new String('')        |
| '1.2'                  |                | 1.2      | true      | new Number(1.2)       |
| 'one'                  |                | NaN      | true      | new String('one')     |
| 0                      | '0'            | 0        | **false** | new Number(0)         |
| -0                     | '0'            | 0        | **false** | new Number(-0)        |
| NaN                    | 'NaN'          |          | **false** | new Number(NaN)       |
| Infinity               | 'Infinity'     |          | true      | new Number(Infinity)  |
| -Infinity              | '-Infinity'    |          | true      | new Nubmer(-Infinity) |
| 1(无穷大, 非零)        | '1'            | 1        | true      | new Nubmer(1)         |
| {}(任意对象)           | 注 1[^1]       | 注 2[^2] | true      |
| [](任意数组)           | ''             | 0        | true      |
| [9](1个数字元素)       | '9'            | 9        | true      |
| ['a'](其他数组)        | 注 3[^3]       | NaN      | true      |
| function(){}(任意函数) | 注 4[^4]       | NaN      | true      |

[^1]: 会先调用 toString()，如果没有，则调用 valueOf()，都没有则抛出类型错误异常
[^2]: 会先调用 valueOf()，如果没有，则调用 toString()，都没有则抛出类型错误异常
[^3]: 调用 join()，如['a'] => 'a', [1,2,3] => '1,2,3'
[^4]: 调用 toString()