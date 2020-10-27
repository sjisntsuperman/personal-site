---
title: "leetcode-1027 最长等差数列"
subtitle: ""
date: 2020-10-28T07:34:57+08:00
lastmod: 2020-10-28T07:34:57+08:00
author: ""
authorLink: ""
description: ""

tags: [
  "algo"
]
categories: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

<!--more-->
## 动态规划

说到动态规划，自然想到了动态规划模板。

```go
func solution(arr) {
    ans := 0
    n := len(arr)
    dp := []map[int]int{}
    for i:=0;i<n;i++{
        dp[i] = make(map[int]int)
    }

    for i:=0;i<n;i++{
        for j:=0;j<n;j++{
            // 状态转移方程
            dp[i][j] = dp[i][j-1] + 1
        }
    }
    return ans
}
```

### 分析重叠子问题

那么如何找到呢？条件太少了，

- 我们并不知道arr中哪个数是等差数列的开头和结尾
- 也并不知道他们之前是如何联系起来的

那么如何解决这些问题呢？

### 找出状态转移方程

陷入迷茫期，究竟选动态规划来解决这个问题对吗？想到这里可能会觉得动规很麻烦，坚持不下去，其实不然，不如我们换个思路。

### 为何会想到用动态规划呢

因为仔细分析题意，要找到最长的等差数列。意味着两点：

1. 找到arr中存在的等差数列数列
2. 比较它们的长度，且获得最长的长度

现在问题变为了，如何解决以上两个问题。

这里一个一个来解决。

### 如何找到等差数列

回想一下等差数列的性质，$An = a1 + n*d(d为差值)$

这是最基本的性质，除此之外，还有个，$2*An = An-1 + An; 2*cur = prev + next;$

到了这里回想一下动态规划的解题思路，$dp[i][j]$ 一般代表的是路径的长度。到这边结合一下，$dp[i][j]$代表arr长度。这样的话，我们可以$假设i作为等差数列中的一员，j为等差数列中的delta。dp[i][j]为计算出delta时所在i的arr长度值。$

这样的话，我们每次得出delta值的时候，为arr内每位成员添加 1个路径，然后到达最后一个等差数列的成员时，就会获得一个它所在数列中的最长长度。

这里我们并没有主动去找等差数列，而是用取巧的方式，把所有潜在的数列加上 1个路径值，如果这个可能性不存在的话，自然会比2小。说明无法成为一个等差数列（等差数列中至少有两委成员）。

### 如何取到最长的长度

这个问题其实上面有说到，最长的长度必定是$dp[i][j]$的值。如果不明白上面所说的话，可以自己用数学归纳法推导一下。

那么之前的问题就逐一解决了。

### 代码时间

让我们套用上面的模板。复制过来再加修改。

```go
func solution(arr) {
    ans := 0
    n := len(arr)
    dp := []map[int]int{}
    for i:=0;i<n;i++{
        dp[i] = make(map[int]int)
    }

    // 作了修改
    for i:=1;i<n;i++{
        for j:=0;j<i;j++{
            // 差值
            delta = A[j] - A[0]
            // 状态转移方程
            // dp[i][j] = dp[i][j-1] + 1
            if _,ok:=dp[j][delta];ok{
                dp[i][delta] = dp[j][delta] + 1
            } else {
                dp[j][delta] = 1
                dp[i][delta] = dp[j][delta] + 1
            }
        }
    }

    return ans
}
```

到这一步这题就解决了。

## 总结

动态规划比较难想到的是，状态转移方程。假设什么才能比较好把路径如何联系起来。其实多加练习之后，就会变得简单。
