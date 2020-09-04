---
title: "聊聊「位运算」"
date: 2020-07-08T12:00:41+08:00
tags: ["web"]
description: "十分钟带你了解JS中的位运算。"
summary: "十分钟带你了解JS中的位运算。"
---

> 十分钟带你了解JS中的位运算。

## 在JS中的表示

- 二进制0b开头
- 八进制0开头
- 十六进制0x开头

```js
var a = 0b111;
var b = 017;
var c = 0xF;

console.log(a,b,c);  // 7, 15, 15
```

## 进制转换

- 十进制转二进制

```js
console.log(Number(3).toString(2)); // 11
```

- 二进制转十进制

```js
// parseInt(数值, 进制数)
console.log(parseInt(3,2)); // 11
```

- 小数转二进制（乘2取整）

  ```js
  // 0.1
  0.1 * 2 = 0.2 // 0
  0.2 * 2 = 0.4 // 0
  0.4 * 2 = 0.8 // 0
  0.8 * 2 = 1.6 // 1
  0.6 * 2 = 1.2 // 1
  0.2 * 2 = 0.4 // 0
  // ...
  ```

- 二进制转整数（除2取余）

  ```js
  // 11
  11 / 2 = 5 // 1
  5 / 2 = 2 // 1
  2 / 2 = 1.25 // 1 
  // 1 + 1 + 1 = 3
  ```

## 按位操作符

|    别称    | 运算符 | 示例  |                             描述                             |
| :--------: | :----: | :---: | :----------------------------------------------------------: |
|   按位与   |   &    |  a&b  | 对于每个比特位， 只有两个操作数相应的比特位都是1时，结果才为1，否则为0。 |
|   按位或   |   \|   | a\|b  | 对于每个比特位， 当两个操作数相应的比特位至少有一个1时，结果为1，否则为0。 |
|  按位异或  |   ^    |  a^b  | 对于每个比特位， 当两个操作数相应的比特位有且只有一个1时，结果为1，否则为0。 |
|    左移    |   <<   | a<<b  |              左移b (< 32) 比特位，右边用0代替。              |
| 有符号右移 |   >>   | a>>b  |            右移b (< 32) 比特位，丢弃被移出的位。             |
| 无符号右移 |  >>>   | a>>>b | 右移` b `(< 32) 比特位，丢弃被移出的位，并使用 0 在左侧填充。 |
|   按位非   |   ~    |  ~a   |    对于每个比特位，反转操作数的比特位，即0变成1，1变成0。    |

## 一些在JS中的技巧

- 取奇偶

```js
console.log(7&1); // 1
console.log(4&1); // 0
```

- 取整

```js
console.log(~~7.77);    // 7
console.log(7.77<<0); // 7
console.log(7.77>>0); // 7
```

## 运用

判断一个 9x9 的数独是否有效。只需要**根据以下规则**，验证已经填入的数字是否有效即可。

1. 数字 `1-9` 在每一行只能出现一次。
2. 数字 `1-9` 在每一列只能出现一次。
3. 数字 `1-9` 在每一个以粗实线分隔的 `3x3` 宫内只能出现一次。

![valid-sudoku](/images/bit/valid-sudoku.png)

```js
var isValidSudoku = function(board) {
  const bitMap = [0,0,0,0,0,0,0,0,0];
  for (let row = 0; row < 9; row++) {
    for (let col = 0; col < 9; col++) {
      if (board[row][col] === '.') continue;
      const moveBit = Number(board[row][col]) - 1;
      const rowFlag = 1 << moveBit;
      const colFlag = 1 << 9 << moveBit;
      const subFlag = 1 << 9 << 9 << moveBit;
      const subIndex = parseInt(row / 3, 10) * 3 + parseInt(col / 3, 10);
      if (((bitMap[row] & rowFlag) === rowFlag)
        || ((bitMap[col] & colFlag) === colFlag)
        || ((bitMap[subIndex] & subFlag) === subFlag)
      ) {
        return false;
      }
      bitMap[row] |= rowFlag;
      bitMap[col] |= colFlag;
      bitMap[subIndex] |= subFlag;
    }
  }
  return true;
};
```

这里来解释一下上面这个算法。

首先这里的bitmap代表的是什么呢？这也是我一开始看不懂这个答案的主要点。

它指的是，这个二维数组的地图。也就是说，第一列和第一行的所有数都会设置到bitmap地图的第一个数中。第二列和第二行的所有数都会设置到bitmap的第二个数中。于此类推，第九列和第九行的所有数都会设置到bitmap的第九个数中。

从代码块来看一下，设置数的时候分别干了什么？

```js
const moveBit = Number(board[row][col]) - 1;
      const rowFlag = 1 << moveBit;
      const colFlag = 1 << 9 << moveBit;
      const subFlag = 1 << 9 << 9 << moveBit;
      const subIndex = parseInt(row / 3, 10) * 3 + parseInt(col / 3, 10);
      if (((bitMap[row] & rowFlag) === rowFlag)
        || ((bitMap[col] & colFlag) === colFlag)
        || ((bitMap[subIndex] & subFlag) === subFlag)
      ) {
        return false;
      }
      bitMap[row] |= rowFlag;
      bitMap[col] |= colFlag;
      bitMap[subIndex] |= subFlag;
```

这里是什么意思呢？

我们都知道0-9，是9个不同的数字，分别对应bitmap

`000000000|00000000|000000000`

中的每位数。先忽略最左边部分的情况下（这部分另外提出讨论），比如我dp(1,1)为8，那么bitmap就会变成

`000000000|000000010|000000010`

同理，如果dp(1,2)为8的话，那么就会在设置bitmap的时候发现同样位置上已经有个`1`占位了。那么就会执行判断条件

```js
if (((bitMap[row] & rowFlag) === rowFlag)
  || ((bitMap[col] & colFlag) === colFlag)
  || ((bitMap[subIndex] & subFlag) === subFlag)
) {
  return false;
}
```

返回`false`。那么接下来，我们看看第一部分。九宫格的数字怎么单独隔离出来呢？看下代码。

```js
const subIndex = parseInt(row / 3, 10) * 3 + parseInt(col / 3, 10);
```

我们先想象一下，一个9X9的数组是不应该有9个9宫格。那么就很清晰了。首先，第一个九宫格的区域范围可以表示为在这个区间范围内`row:[0-2],col:[0-2]`；也就是说，这个范围内的所有数都应该在bitmap中的第一个数中。

所以上面的公式就这样的出来了。我们验证一下这个公式。

```js
const subIndex = parseInt(row / 3, 10) * 3 + parseInt(col / 3, 10);

//输入dp(1,1) 返回 0
// dp(2,0) returns 0
// dp(7,8) returns 9
// dp(8,8) returns 9
```

好的，那么这个bitmap算法就完成拉。

感谢看到这里的朋友。希望看完这篇文章能给大家一点收获。

## 参考

1. <https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators>
2. <https://leetcode-cn.com/problems/valid-sudoku/>
3. <https://www.w3school.com.cn/js/js_obj_number.asp>
