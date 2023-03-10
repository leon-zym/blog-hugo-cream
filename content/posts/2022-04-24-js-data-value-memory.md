---
title: JavaScript 数据 变量 内存
date: 2022-04-24T10:00:00
categories: Programming
tags: [JavaScript]
---

# 数据-变量-内存的联系

var a = xxx，变量a的内存中到底保存的是什么？

- 若xxx是一个基本数据类型，则保存的就是这个数据本身
- 若xxx是一个对象，则保存的是该对象的地址值
- 若xxx是一个变量，则保存的是xxx的内存中的内容，可能是基本的数据，也可能是地址值

# 引用变量的赋值问题

2个引用变量指向同一个对象，通过其中一个变量修改对象内部的数据，另一个变量看到的也是修改后的数据。

```js
var a = {name: 'Leon'};
var b = a;
b.age = 12;
console.log(a.age);
// 输出为 12

function fn(obj) {
  obj.name = 'Jack';
};
fn(a);
console.log(b.name);
// 输出为 Jack
```



2个引用变量指向同一个对象，让其中一个变量指向另一个对象，另一个变量仍会指向原对象。

```js
var a = {age: 12};
var b = a;
a = {name: 'Leon', age: 13};
console.log(b.age, a.name, a.age);
// 输出结果为 12 Leon 13

function fn(obj) {
  obj = {age: 15};
};
fn(a);
console.log(a.age);
// 输出为 13
// 函数执行完毕后内部的变量会被释放，{age: 15}这个对象会变成垃圾对象被回收
```

# 函数传递变量参数的问题

在JS调用函数传递变量参数时，是值传递还是引用传递？

- 理解一：都是值传递（基本值 / 地址值）
- 理解二：可能是值传递，也可能是引用传递（地址值）
- 本质：传递的是该变量的内存中的内容

```js
var a = 3;
function fn(a) {
  a = a + 1;
};
fn(a);
console.log(a);
// 输出为 3
```

# JS内存管理

JS引擎如何管理内存？

内存生命周期：

- 分配小内存空间，得到它的使用权
- 存储数据，可以反复进行操作
- 释放小内存空间

释放内存：

- 局部变量：函数执行完后自动释放
- 对象：先变成垃圾对象，后在某个时刻由JS的[垃圾回收机制](https://leonzym.com/posts/2022/04/22/JS%E5%9F%BA%E7%A1%80%E6%8B%BE%E9%81%97.html)回收

