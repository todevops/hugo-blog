---
title: LeetCode每日一题（最长公共前缀）
date: 2019-12-19
lastmod: 2019-12-19
categories: ["go"]
tags: ["go"]
---

LeetCode每日一题（罗马数字转整数）[https://leetcode-cn.com/problems/longest-common-prefix/](https://leetcode-cn.com/problems/longest-common-prefix/)
<!--more-->

## 题目

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 `""`。

## 示例

```
输入: ["flower","flow","flight"]
输出: "fl"

输入: ["dog","racecar","car"]
输出: ""
```

## 题解

```go
func longestCommonPrefix(strs []string) string {
	if len(strs) == 0 {
		return ""
	}
	// 取strs第一个元素作为前缀
	prefix := strs[0]
	// 遍历数组的长度
	for i := 1; i < len(strs); i++ {
		// 从数组第二个元素开始，判断是否存在前缀prefix
		for !strings.HasPrefix(strs[i], prefix) {
			fmt.Println(i, len(prefix), prefix)
			// 如果不存在则将前缀prefix最后一个字符删除
			prefix = prefix[0 : len(prefix)-1]
			// 当前缀prefix=0时返回空字符串
			if len(prefix) == 0 {
				return ""
			}
		}
	}
	return prefix
}

```