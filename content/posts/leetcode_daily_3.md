---
title: LeetCode每日一题（罗马数字转整数）
date: 2019-12-18
lastmod: 2019-12-18
categories: ["go"]
tags: ["go"]
---

LeetCode每日一题（罗马数字转整数）[https://leetcode-cn.com/problems/roman-to-integer/](https://leetcode-cn.com/problems/roman-to-integer/)
<!--more-->

## 题目

罗马数字包含以下七种字符: I， V， X， L，C，D 和 M。

字符 | 数值
---|---
I|1
V|5
X|10
L|50
C|100
D|500
M|1000

- I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
- X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。 
- C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900。

## 示例

```
输入: "III"
输出: 3

输入: "LVIII"
输出: 58
```

## 题解
```go
func romanToInt(s string) int {
	r1 := map[string]int{
		"I": 1,
		"V": 5,
		"X": 10,
		"L": 50,
		"C": 100,
		"D": 500,
		"M": 1000,
	}
	r2 := map[string]int{
		"IV": 4,
		"IX": 9,
		"XL": 40,
		"XC": 90,
		"CD": 400,
		"CM": 900,
	}
	// ret 用来接收计算结果
	ret := 0
	// 判断字符串长度是否为0
	for len(s) != 0 {
		if len(s) > 1 {
			// s长度大于1，先判断前两位是否在r中
			chars := s[:2]
			if v, ok := r2[chars]; ok {
				ret += v
				// 删掉前两个字符
				s = s[2:]
				// 判断一个字符是否在r中
			} else {
				ret += r1[string(s[0])]
				// 删除前一个字符
				s = s[1:]
			}

			// 长度为1时
		} else {
			ret += r1[string(s[0])]
			// 将s置空长度为0
			s = s[1:]
		}
	}
	return ret
}

```