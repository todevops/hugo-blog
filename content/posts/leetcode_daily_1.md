---
title: LeetCode每日一题（两数之和）
date: 2019-12-16
categories: ["go"]
tags: ["go"]
---

LeetCode每日一题（两数之和）[https://leetcode-cn.com/problems/two-sum/](https://leetcode-cn.com/problems/two-sum/)
<!--more-->

## 题目
给定一个整数数组 `nums` 和一个目标值 `target`，请你在该数组中找出和为目标值的那 **两个** 整数，并返回他们的数组下标。
> 你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

### 示例

```
给定 nums = [2, 7, 11, 15], target = 9
因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

## 题解
### 方法一：暴力法

遍历每个元素 `x`，并查找是否存在一个值与 `target − x` 相等的目标元素。
- 时间复杂度：O(n^2)
> 对于每个元素，我们试图通过遍历数组的其余部分来寻找它所对应的目标元素，这将耗费 O(n) 的时间。因此时间复杂度为 O(n^2)。
- 空间复杂度：O(1)

```go
func twoSum(nums []int, target int) []int {
	ret := make([]int, 0, 2)

	for i := 0; i < len(nums); i++ {
		for j := i + 1; j < len(nums); j++ {
			if target-nums[i] == nums[j] {
				ret = append(ret, i, j)
				break
			}
		}
	}
	return ret
}
```

### 方法二：两遍哈希表

在第一次迭代中，我们将每个元素的值和它的索引添加到表中。然后，在第二次迭代中，我们将检查每个元素所对应的目标元素`（target - nums[i]）`是否存在于表中。注意，该目标元素不能是 `nums[i]` 本身！
- 时间复杂度：O(n)
> 我们把包含有 n 个元素的列表遍历两次。由于哈希表将查找时间缩短到 O(1) ，所以时间复杂度为 O(n)。
- 空间复杂度：O(n)
> 所需的额外空间取决于哈希表中存储的元素数量，该表中存储了 n 个元素。

```go
func twoSum(nums []int, target int) []int {
	numsMap := make(map[int]int)
	ret := make([]int, 2)

	// 遍历 nums 并将 nums 的数值和索引映射
	// k: nums []int 索引
	// v: nums []int 值
	for k, v := range nums {
		numsMap[v] = k
	}

	for i, v := range nums {
		// 取出一个数然后求出剩下一个数的值
		x := target - v
		// 判断剩下的数是否存在于 numsMap 中
		// 如果存在则返回两个数的索引 i, numsMap[x]
		if _, ok := numsMap[x]; ok {
			// 当 x(target-v) 值所处的索引 (numsMap[x]) 等于当前索引
			// 即为同一个数则直接返回空
			if i != numsMap[x] {
				ret = append(ret, i, numsMap[x])
				break
			}
		}
	}
	return ret
}
```

### 方法三：一遍哈希表

在进行迭代并将元素插入到表中的同时，我们还会回过头来检查表中是否已经存在当前元素所对应的目标元素。如果它存在，那我们已经找到了对应解，并立即将其返回。
- 时间复杂度：O(n)
> 我们只遍历了包含有 n 个元素的列表一次。在表中进行的每次查找只花费 O(1) 的时间。
- 空间复杂度：O(n)
> 所需的额外空间取决于哈希表中存储的元素数量，该表最多需要存储 n 个元素。

```go
func twoSum(nums []int, target int) []int {
	numsMap := make(map[int]int)
	ret := make([]int, 0, 2)

	for i, v := range nums {
		x := target - v

		if _, ok := numsMap[x]; ok {

			if i != numsMap[x] {
				ret = append(ret, numsMap[x], i)
				break
			}
		}

		numsMap[v] = i
	}
	return ret
}
```