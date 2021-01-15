---
title: "JS中的正则表达式（强大的正则表达式）"
subtitle: ""
date: 2020-11-03T11:00:56+08:00
lastmod: 2020-11-03T11:00:56+08:00
draft: false
author: "steinw"
authorLink: "steinw.cc"
description: ""

tags: [
  "web"
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

> 最近，回顾了下JS中正则表达式的相关知识。借此机会重新梳理一下知识点，顺便整理成文章。

## JS中与正则有关的API

RegExp对象。

- String.prototype.match(reg)
- RegExp.prototype.test(str)
- String.prototype.replace(reg,str)
- RegExp.prototype.exec(str)

    ```javascript
  RegExp.exec(str) 相当于 str.match(regExp)
    ```

- RegExp.prototype.toString()

### test

> reg.test(str)

@Return Boolean。表示正则能否匹配上。

### match

> str.match(reg)

@Return Array。表示匹配到的字符次数。后续可以使用RegExp.$1或RegExp.$&等字符来获取匹配到的第几次的结果。

### replace

> str.replace(reg, str2 || callback($0,$1,$2){})

@Void。用于替换掉匹配到的结果。

### search

> str.search(reg)

@Return $0, $1, $2...。返回被匹配的部分。

## 正则中的小技巧

- %s 表示string类型的打印
- %d 表示number类型的打印
- %o 表示object类型的打印
- %j 表示json类型的打印，部分浏览器不支持
- \$1,\$2... 表示match后的匹配，\$1代表第二个匹配值，\$2代表第三个，于此类推
- regExp.lastMatch，相当于$&（非标准），表示最后一个匹配到的字符
- regExp.leftContent，相当于$`（非标准），表示匹配到的左部分

## 一些规则

> 注意：`^reg`只有在`[]`里面才会起到`非`的作用。

|  匹配规则 | 代表 |
|:--:|:--:|
| \|  |  或 |
| ()  |  优先匹配 |
| []  |  匹配一次 |
|[^reg]\|! | 非 |
|/g| 全局匹配模式|
|/i|not ignore mode|
|/m|多行模式|
|[a-zA-Z0-9]|匹配大小写和数字|

## 查找

> 注意：这里的前向指的是，从右到左，后向指的是，从左到右。这里的查找，只有在`()`里面才会起到查找的作用。

栗子：

```javascript
var str='great';
var str1='greater;
var reg=/\w(?!er)/g;
str.replace(reg, '');
console.log(str);
// le
```

|  匹配规则 | 代表 |
| :--: | :------: |
|  (?=)  | 前向查找 |
|  (?!)  | 前向负查找 |
|  (?<!)  | 后向负查找         |
|  (?<=)   |  后向正查找        |

它们之间有什么区别呢？举个例子。

这个有个字符串, "appleBanana"，使用 (?<!apple)banana 匹配时得到的结果是，apple。

当使用的是/(?!apple)banana/g 匹配到的是，banana。

同理，用 /apple(?=banana)/g 匹配时，匹配到的结果是，apple，

而使用 /apple(?<=banana)/g 时， 匹配到的是， banana。

是不这样就清晰一点了呢？

## 临界

什么是单词边界？举个栗子把。

```javascript
var str='apple banana';
var reg=/\b/g;
str.replace(reg, ',');
console.log(str);
// apple,banana
```

|  匹配规则 | 代表 |
| :--: | :------: |
|  ^   | 开头 |
|  $   |    结尾      |
|  \b  |    单词边界      |
|  \B  |    非单词边界      |
|  \w  |   匹配单词       |
|  \W  |   匹配非单词       |
|  \d  |     匹配数字     |
|  \D  |     匹配非数字     |
|  \s  |     匹配空白字符     |
|  \S  |     匹配非空白字符     |

## 匹配次数

|  匹配规则 | 代表 |
| :-------: | :--: |
|     ?     |    0或无数次  |
|     *     |   多次   |
|     +     |   1或无数次   |
| {min,max} |  min-max次    |
|     .     |   任何字符   |

## 常见面试题

1. 千位转换

    > 栗子1: 10000=>10,000
    > 栗子2: 12345.123=>12,345.123

    ```javascript
    function foo(str){
        return str.replace(/\d(?=(\d{3})+(\.|$))/g,'$&,');
    }
    ```

2. 匹配URL上的参数

    其实不太推荐用正则，还是建议直接从str上indexof比较好。这里只是示例一下正则的强大实力。

    ```js
    // https://www.baidu.com/search?test=123&&location=China
    function match(url){
      return url.match(/(?<=[\?&])([^=&#]+)=([^&#]*)/g)
    }
    ```
