---
title: LeetCode Weekly Contest 56
date: 2017-10-30 11:05:36
tags:
  - Algorithm
  - LeetCode
categories: Algorithm
---

# [LeetCode Weekly Contest 56](https://leetcode.com/contest/leetcode-weekly-contest-56/)

##Maximum Length of Repeated Subarray

Given two integer arrays `A` and `B`, return the maximum length of an subarray that appears in both arrays.

**Example 1:**

```
Input:
A: [1,2,3,2,1]
B: [3,2,1,4,7]
Output: 3
Explanation:
The repeated subarray with maximum length is [3, 2, 1].
```

<!--more-->

**Note:**

1. 1 <= len(A), len(B) <= 1000
2. 0 <= A[i], B[i] < 100

题意： 求最长连续子序列,类 LCS，转移条件换一下

```javascript
/**
 * @param {number[]} A
 * @param {number[]} B
 * @return {number}
 */
var findLength = function(A, B) {
  let maxL = 0
  let lenA = A.length,
    lenB = B.length
  let len = Math.max(lenA, lenB)
  let dp = new Array(len)
  for (let i = -1; i < len; i++) {
    dp[i] = new Array(len)
    for (let j = -1; j < len; j++) dp[i][j] = 0
  }
  for (let i = 0; i < lenA; i++) {
    for (let j = 0; j < lenB; j++) {
      if (A[i] == B[j]) {
        dp[i][j] = 1 + dp[i - 1][j - 1]
        maxL = Math.max(maxL, dp[i][j])
      }
    }
  }
  return maxL
}
```

## Find K-th Smallest Pair Distance

Given an integer array, return the k-th smallest **distance** among all the pairs. The distance of a pair (A, B) is defined as the absolute difference between A and B.

**Example 1:**

```
Input:
nums = [1,3,1]
k = 1
Output: 0
Explanation:
Here are all the pairs:
(1,3) -> 2
(1,1) -> 0
(3,1) -> 2
Then the 1st smallest distance pair is (1,1), and its distance is 0.
```

**Note:**

1. `2 <= len(nums) <= 10000`.
2. `0 <= nums[i] < 1000000`.
3. `1 <= k <= len(nums) * (len(nums) - 1) / 2`.

题意：任意两个数之间的差，求第 k 大的差是多少。

题解：原本以为利用排列组合知识可以解，后来发现太麻烦。因为求 k 大，感觉可以二分答案，于是二分答案然后 check 复杂度$O(log1000000\*len(Set(nums)))$

```javascript
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
var smallestDistancePair = function(nums, k) {
  let len = nums.length
  let pre = new Array(1000005).fill(0)
  nums = nums.map(item => item + 1)
  for (let i = 0; i < len; i++) pre[nums[i]]++
  let set = [...new Set(nums)]
  let count = pre.slice()
  for (let i = 1; i < 1000005; i++) pre[i] += pre[i - 1]
  const check = value => {
    let sum = 0
    set.forEach(item => {
      let count = pre[item] - pre[item - 1]
      if (count >= 2) sum += (count - 1) * count / 2
      const tem = item + value > 1000002 ? 1000002 : item + value
      let dis = pre[tem] - pre[item]
      sum += count * dis
    })
    return sum > k - 1
  }
  let l = 0,
    r = 1000000,
    mid = 0
  while (l < r) {
    mid = parseInt((l + r) / 2)
    if (check(mid)) r = mid
    else l = mid + 1
  }
  return l
}
```
