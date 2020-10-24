# LeetCode-1024 视频切片


你将会获得一系列视频片段，这些片段来自于一项持续时长为 T 秒的体育赛事。这些片段可能有所重叠，也可能长度不一。

视频片段 clips[i] 都用区间进行表示：开始于 clips[i][0] 并于 clips[i][1] 结束。我们甚至可以对这些片段自由地再剪辑，例如片段 [0, 7] 可以剪切成 [0, 1] + [1, 3] + [3, 7] 三部分。

我们需要将这些片段进行再剪辑，并将剪辑后的内容拼接成覆盖整个运动过程的片段（[0, T]）。返回所需片段的最小数目，如果无法完成该任务，则返回 -1 。

来源：力扣（LeetCode）
链接：[https://leetcode-cn.com/problems/video-stitching](https://leetcode-cn.com/problems/video-stitching)
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 动态规划

首先说说动态规划解决了哪些问题？ 解决了重叠子问题。那么此题目中，有哪些重叠子问题，以及如何解决重叠子问题？

### 发现重叠子问题

先说说我的思路，如果是人为计算的话，会先找出开头[0, x]的一个clip， 然后在找另一个clip，并且这个clip 的要求是这样的，`cur[0] > prev[0] && cur[1] > pre[1] &&cur[0] ≤ prev[0]` 。

后来我发现，这样解下去，只能是个暴力法。并不是最优解的解法。

仔细分析不难发现，视频切片是要填满 T 时间的。这是主线问题。

重叠子问题就是，如何找到对应的切片来填满这个T 时间。

我们可以用一个 dp 来模拟主问题。

然后每个子问题的答案都可以从dp中找到。

```jsx
// 主线问题
const dp = new Array(T).fill(Infinity);
```

### 计算 状态转移方程

dp 里的每个切片，应该是被包含在clips当中的。

这句话翻译成代码，就是 

```jsx
for(let i = 0; i <= T; i ++) {
	if( i > clip[0] && i < clip[1] ) {
		dp[i] = Math.min(dp[i], dp[clip[0]] + 1)
	}
}
```

这里为何要加 1 呢？加1代表，在此最优解方法上，会加 1步的 路径。换句话来说就是到达目的地的路径会由这些路径组成。那么到达T的时候，路径就会补全成为 答案所要求的路径数，也就是数组的长度。

得到这段代码后，其实状态转移方程也出来了。

就是 代码块中的`dp[i] = Math.min(dp[i], dp[clip[0]] + 1)`

### 得出答案前，先对base case进行查验

这里有哪些base case呢？

```jsx
dp[0] = 0
```

因为 dp 是模拟 时间段的，起点必须是0， 终点必须是 T。如果没到达终点，dp[T] 也就是 -1。

那么另外一个base case也出来了。

```jsx
return dp[T] == Infinity ? -1 : dp[T]
```

那么一道题就这样结束了。这里列下完整代码。（这里只列出主代码部分）

```jsx
const dp = new Array(T).fill(Infinity)
dp[0] = [0]
for(let i = 0; i <= T; i ++) {
	if( i > clip[0] && i < clip[1] ) {
		dp[i] = Math.min(dp[i], dp[clip[0]] + 1)
	}
}
return dp[T] == Infinity ? -1 : dp[T]
```

### 小结

这里用暴力法的话，其实也可以，不过时间复杂度和空间复杂度这块肯定会高上不少。

这里留下一个问题，就是这种解法，用时哪种数据结构来解决问题的呢？

## 贪心算法

前面说到了比较容易想到的动态规划的解法。那么下面来说说一个优化版的动规解法。

上面一个解法仍然可以优化，怎么优化呢？这个时候我们该想到用，状态压缩的方式。

也就是得出状态压缩方程。

### 优化

如何进行优化？

其实就是不必利用dp 来存储状态，而是只需要记录一个合适的状态即可。

因为 每个能到达终点T的最优解，要想尽早到达终点T， 那么每个切片的长度就需要足够长，或者更长，这就是贪心算法的核心。

因此得出这段代码。

```jsx
const maxn = new Array(T).fill(0)
for(const clip of clips){
	if(clip[0] < T) {
		maxn[clip[0]] = Math.max(maxn[clip[0], clip[1]])
	}
}
```

### 得出状态压缩方程

接下来的工作很简单，就是从之前得出的数组里取出一段最短到达终点T的路径。

这里需要用到一个last来存储当前片段的结尾 以及 prev 来记录当i到达结尾时last的状态，

res 来记录路径数。

因此得出这样一段代码

```jsx
let last = 0, prev = 0, res = 0
for(let i = 0; i <= T; i++) {
	last = Math.max(last, maxn[i])
	if(prev == i){
		res++
		prev = last
	}
}
```

### base case

考虑到可能会存在无法到达终点T的条件。就是当 last == Math.max(last, maxn[i])的时候，意味着last已经是最接近T的值了，但没到达。所以会返回 -1。用代码表示的话是这样，

```jsx
if( last == i) {
	return -1
}
```

### 完整代码

```jsx
const maxn = new Array(T).fill(0)
for(const clip of clips){
	if(clip[0] < T) {
		maxn[clip[0]] = Math.max(maxn[clip[0], clip[1]])
	}
}

let last = 0, prev = 0, res = 0
for(let i = 0; i <= T; i++) {
	last = Math.max(last, maxn[i])
	if( last == i) {
		return -1
	}
	if(prev == i){
		res++
		prev = last
	}
}
```

## 总结

其实读懂题目之后，解决问题并不困难。难是难在题没读懂就开始动手做题，结果用了暴力法，半天没想出答案。更加浪费精力和拖低效率。

上面的解法是参考 LeetCode上官方解析做出的 自己的理解。

可能存在理解错误的地方，欢迎指出。

