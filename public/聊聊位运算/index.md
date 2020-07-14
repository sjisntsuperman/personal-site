# 聊聊「位运算」


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

## 参考

1. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators
2. https://leetcode-cn.com/problems/valid-sudoku/
3. https://www.w3school.com.cn/js/js_obj_number.asp


