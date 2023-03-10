---
title: JavaScript 对象和函数基础
date: 2022-04-24T11:00:00
categories: Programming
tags: [JavaScript]
---

# 对象的属性名

什么时候必须使用`['属性名']`的方式调用对象的属性？

- 属性名包含特殊字符：`-` `空格`
- 属性名为一个变量（不是一个确定的字符串）

```js
var p = {};

p['content-type'] = 'text/json';

var propName = 'myAge';
var value = 18;
p[propName] = value;
```

# 立即调用函数IIEF

作用在于可以隐藏内部实现，和不会污染外部命名空间。

```js
;(function() {
  // 函数体
})()
```

> 为防止JS语句不加分号而引起的错误，可以在这里小括号开头的语句前补一个分号。

# 函数中this的指向问题（初步）

- 没有手动指定调用对象的函数中，this均默认为全局Window对象
- 手动指定调用对象的函数中，this为调用的对象
- 在`new`语句中，构造函数中的this为实例化出来的实例对象
- 在`call()`和`apply()`中，this为传入的对象

> ES6中的箭头函数特殊，此处略过。