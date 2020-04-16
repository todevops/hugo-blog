---
title: LeetCode每日一题（整数反转）
date: 2019-12-17
lastmod: 2019-12-18
categories: ["go"]
tags: ["go"]
---

LeetCode每日一题（整数反转）[https://leetcode-cn.com/problems/reverse-integer/](https://leetcode-cn.com/problems/reverse-integer/)
<!--more-->

## 题目

给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。
> 假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 `[−2^31, 2^31 − 1]`
。请根据这个假设，如果反转后整数溢出那么就返回 0。

## 示例

```
输入: 123
输出: 321

输入: -123
输出: -321

输入: 120
输出: 21
```

## 题解
### 弹出和推入数字 & 溢出前进行检查
```
// 弹出操作:
pop = x % 10;
x /= 10;

// 弹出操作:
temp = rev * 10 + pop;
rev = temp;
```
当 `temp = rev * 10 + pop` 时会导致溢出，所以事先检查这个语句是否会导致溢出。
1. 如果 `temp = rev * 10 + pop` 导致溢出，那么一定有 `rev >= INTMAX/10`
2. 如果 `rev > INTMAX/10`，那么 `temp = rev * 10 + pop` 一定会溢出
3. 如果 `rev == INTMAX/10`，那么只要 `pop > 7` ，`temp = rev * 10 + pop` 就会溢出

- 时间复杂度：O(log(x))，x 中大约有 log10(x) 位数字
- 空间复杂度：O(1)

```go
func reverse(x int) int {
	rev := 0
	for x != 0 {
		pop := x % 10
		x /= 10
		if rev > math.MaxInt32/10 || (rev == math.MaxInt32/10 && pop > 7) {
			return 0
		}
		if rev < math.MinInt32/10 || (rev == math.MinInt32/10 && pop < -8) {
			return 0
		}
		rev = rev*10 + pop
	}
	return rev
}
```
