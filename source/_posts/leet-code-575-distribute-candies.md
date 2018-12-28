---
title: leet-code-575-distribute-candies
cover: /img/home-bg.jpg
date: 2018-11-27 14:49:18
categories: ['misc']
tags: ['leetcode']
---
### Quests
Q: [https://leetcode.com/problems/distribute-candies/](https://leetcode.com/problems/distribute-candies/)

> Given an integer array with even length, where different numbers in this array represent different kinds of candies. Each number means one candy of the corresponding kind. You need to distribute these candies equally in number to brother and sister. Return the maximum number of kinds of candies the sister could gain.

* 給予 **偶數** 長度的 int 陣列
* 一個 int value 代表一種糖果
* 需要平均分配糖果給 2 人 （sister, brother）
* 回傳 sister 可以拿到最多幾種糖果

### Limit
* The length of the given array is in range [2, 10,000], and will be even.
* The number in given array is in range [-100,000, 100,000].

### Cases
```
Input: candies = [1,1,2,2,3,3]
output: 3
6 顆糖果, 1 人最多 3 顆, 最大可以拿 3 種 => 3

Input: candies = [1,1,2,3]
Output: 2
4 顆糖果, 1 人最多 2 顆, 最大可以拿 2 種 => 2, [1,2] 或 [1,3] 或 [2,3]

Input: candies = [1,2,3,4]
Output: 2
4 顆糖果, 1 人最多 2 顆, 最大可以拿 2 種 => 2, [1,2] 或 [1,3] 或 [1,4] 或 [2,3] ...
```

### Solution
```java
public int distributeCandies(int[] candies) {
  // 如果只有 2 顆, 一人一顆, 最多一種
  if (candies.length == 2) {
    return 1;
  }
  
  // 先算出, 糖果有幾種
  HashSet<Integer> candiesType = new HashSet<>();
  Arrays.stream(candies).boxed().forEach(candiesType::add);
  int typeSize = candiesType.size();
  
  // 一人, 拿到最多糖果的可能
  int maxType = candies.length / 2;
  
  // 最多拿 maxType, 或者 typeSize
  return typeSize <= maxType ? typeSize : maxType;
}
```

可以優化為
```java
public int distributeCandies(int[] candies) {
  // 如果只有 2 顆, 一人一顆, 最多一種
  if (candies.length == 2) {
    return 1;
  }
  
  // 先算出, 糖果有幾種
  HashSet<Integer> candiesType = new HashSet<>();
  Arrays.stream(candies).boxed().forEach(candiesType::add);
  
  return Math.min(candiesType.size(), candies.length / 2);
}
```