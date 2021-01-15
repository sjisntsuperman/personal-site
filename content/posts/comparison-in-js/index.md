---
title: "JS中的类型转换"
subtitle: ""
date: 2020-09-24T19:02:25+08:00
lastmod: 2020-09-24T19:02:25+08:00
# draft: true
author: ""
authorLink: ""
description: ""

tags: [
    “web
]
categories: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: "featured-image.png"
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

<!--more-->

## 前言

> 从零开始的类型转换？

其实大部分可以从ES6规范中看到。下面就直接举例子了。

## JS中的类型

### 基础类型

- Number
- String
- Boolean
- Undefined
- Null

### 引用类型

- Object（包含Function、Array）
- Symbol

## toString

是个内置方法，作用是转换为String类型。

栗子

```js
const a = [1]
const b = {0:1}
const c = 1
const d = false

a.toString() // '1'
b.toString() // '[object Object]'
c.toString() // '1'
d.toString() // 'false'
```

除此之外，toString还可以注入参数。

栗子

```js
const a = 3
const b = 10

a.toString(2) // 11
a.toString(8) // 3
b.toString(2) // 1010
b.toString(8) // 12
```

这里的参数ratix是数字，作用是将a转换为ratix进制的数字。如果ratix比本身的数要大的话，会返回本身。

也可以用`Object.prototype.toString.call(ctx)`来判断类型。

```js
Object.prototype.toString.call(1) // [object Number]
Object.prototype.toString.call([]) // [object Array]
```

一些不容易察觉的点

```js
1.toString() // Invalid or unexpected token
1..toString() // '1'
(1).toString() // '1'
(1.).toString() // '1'
```

## ValueOf

返回本身。

栗子

```js
const a = [1]
const b = {0:1}
const c = 1
const d = false

a.valueOf() // [1]
b.valueOf() // {0:1}
c.valueOf() // 1
d.valueOf() // false
```

以及修改对象的valueOf来达到某些目的

```js
const a = {
    value: 1,
    valueOf: function(){
        return this.value++
    }
}

a == 1 && a == 2 && a == 3 // true
```

## toPrimitive

返回原始值。

### 原始值

符合基础类型的被判定为原始类型。基于这些类型的值也被称为原始值。

对于引用类型而言，一般流程是：

1. 判断类型
2. valueOf
3. toString

## instanceof 和 typeof

- typeof 判断类型
- instanceof 判断继承关系

小技巧

- 在判断object类型的时候用typeof，注意把null排除掉

```js
typeof x === 'object' && x !== null
```

## 比较符

### ==

```js
'1' == 1
// '1'=>1 != 1 => true
[] == 0
// []=>''=>0=>true
{} == 0
// {}=>'[object Object]'=>NaN=>false
null == undefined
// true 
```

以下内容摘录自ES6规范：

The comparison x == y, where x and y are values, produces **true** or **false**. Such a comparison is performed as follows:

1. If [Type](#sec-8)(*x*) is the same as [Type](#sec-8)(*y*), then
    1.  If [Type](#sec-8)(*x*) is Undefined, return **true**.
    2.  If [Type](#sec-8)(*x*) is Null, return **true**.
    3.  If [Type](#sec-8)(*x*) is Number, then
        1.  If *x* is **NaN**, return **false**.
        2.  If *y* is **NaN**, return **false**.
        3.  If *x* is the same Number value as *y*, return **true**.
        4.  If *x* is **+0** and *y* is **−0**, return **true**.
        5.  If *x* is **−0** and *y* is **+0**, return **true**.
        6.  Return **false**.
    4.  If [Type](#sec-8)(*x*) is String, then return **true** if *x* and *y* are exactly the same sequence of characters (same length and same characters in corresponding positions). Otherwise, return **false**.
    5.  If [Type](#sec-8)(*x*) is Boolean, return **true** if *x* and *y* are both **true** or both **false**. Otherwise, return **false**.
    6.  Return **true** if *x* and *y* refer to the same object. Otherwise, return **false**.
2.  If *x* is **null** and *y* is **undefined**, return **true**.
3.  If *x* is **undefined** and *y* is **null**, return **true**.
4.  If [Type](#sec-8)(*x*) is Number and [Type](#sec-8)(*y*) is String,
    return the result of the comparison *x* == [ToNumber](#sec-9.3)(*y*).
5.  If [Type](#sec-8)(*x*) is String and [Type](#sec-8)(*y*) is Number,
    return the result of the comparison [ToNumber](#sec-9.3)(*x*) == *y*.
6.  If [Type](#sec-8)(*x*) is Boolean, return the result of the comparison [ToNumber](#sec-9.3)(*x*) == *y*.
7.  If [Type](#sec-8)(*y*) is Boolean, return the result of the comparison *x* == [ToNumber](#sec-9.3)(*y*).
8.  If [Type](#sec-8)(*x*) is either String or Number and [Type](#sec-8)(*y*) is Object,
    return the result of the comparison *x* == [ToPrimitive](#sec-9.1)(*y*).
9.  If [Type](#sec-8)(*x*) is Object and [Type](#sec-8)(*y*) is either String or Number,
    return the result of the comparison [ToPrimitive](#sec-9.1)(*x*) == *y*.
10.  Return **false**.

### ===

判断类型和值是否对等。不会进行类型转换。有一种情况除外，就是通过一些手段更改对象的`get`与`set`函数。

栗子

```js
var value = 1

Object.defineProperty(window, 'a', {
    get(val){
        return value++
    }
})

a === 1 && a === 2 && a === 3 // true
```

## NAN

JS中比较特殊的一个关键字，就是NAN。

那么`Number().isNAN()`与`window.isNAN()`又有什么区别呢？

看下面的几个栗子就懂了

```js
NAN === NAN // false

window.isNAN('String') // true
Number.isNAN('String') // false

window.isNAN({}) // true
Number.isNAN({}) // false

window.isNAN(false) // true
Number.isNAN(false) // true

0/0 // NAN
0 * Infinity // NAN
```

## 运算符号

### +

一般流程：

1. `left`进入运算
2. `left`进行getValue
3. `right`进入运算
4. `right`进行getValue
5. `left`求原始值
6. `right`求原始值
7. `left`或`right`为String类型，则两边转化为String类型再进行求值
8. `left`或`right`不为String类型，toNumber然后求值

### -、*、/

不像`+`这么多功能，它们只有数字运算的功能。遇到非Number类型的值是，会先toNumber再运算取值。

## 参考

1. <https://www.ecma-international.org/ecma-262/5.1/#sec-11.9.3>
2. <https://dmitripavlutin.com/nan-in-javascript/>
