---
title: leet-code-807-max-increase-to-keep-city-skyline
cover: /img/home-bg.jpg
date: 2018-12-27 20:08:59
categories: ['misc']
tags: ['leetcode']
---
### Puzzle
[https://leetcode.com/problems/max-increase-to-keep-city-skyline/](https://leetcode.com/problems/max-increase-to-keep-city-skyline/)
給一個 2 dim 矩陣, 試調整矩陣, 讓每個行列元素皆不能大於最大的元素, 將計算所得的 2 dim 矩陣元素做加總。

Example:
Input: grid = [[3,0,8,4],[2,4,5,7],[9,2,6,3],[0,3,1,0]]
Output: 35

Explanation: The grid is:
[ [3, 0, 8, 4], 
  [2, 4, 5, 7],
  [9, 2, 6, 3],
  [0, 3, 1, 0] ]

圖解大概是這樣, 最後做矩陣相減, 求 sum
```
    9, 4, 8, 7
   ------------ 
8 | 3, 0, 8, 4
7 | 2, 4, 5, 7
9 | 9, 2, 6, 3
3 | 0, 3, 1, 0
    
    8, 4, 8, 7
    7, 4, 7, 7
    9, 4, 8, 7
    3, 3, 3, 3
    
    5, 4, 0, 3
    5, 0, 2, 0 
    0, 2, 2, 4
    3, 0, 2, 3
```

### Test
先縮小矩陣做思考, 先從 2*2 開始... 然後我還是卡超久 Orz,
```
    3, 4
  - - - -
3 | 3, 0 
4 | 2, 4 

    3, 3
    3, 4
  
    0, 3
    1, 0

    4
```

```java
@Test
public void test_case_1() {
    int[][] grid = new int[][] {
        {3, 0},
        {2, 4}
    };
    int act = skyline.maxIncreaseKeepingSkyline(grid);

    assertEquals(4, act);
}

@Test
public void test_case_2() {
    int[][] grid = new int[][]{
        {3, 0, 8, 4},
        {2, 4, 5, 7},
        {9, 2, 6, 3},
        {0, 3, 1, 0}
    };
    int act = skyline.maxIncreaseKeepingSkyline(grid);

    assertEquals(35, act);
}
```

### Solution
```java
int maxIncreaseKeepingSkyline(int[][] grid) {
    int[] rowMax = new int[grid.length];
    int[] colMax = new int[grid[0].length];
    int sum = 0;

    // 先掃過一次 Matrix, 取得 rowMax, colMax 
    for (int i = 0; i < grid.length; i++) {
        for (int j = 0; j < grid[i].length; j++) {
            if (grid[i][j] > rowMax[i]) {
                rowMax[i] = grid[i][j];
            }
            if (grid[i][j] > colMax[j]) {
                colMax[j] = grid[i][j];
            }
        }
    }

    // 取得可以蓋的最高高度, 做加總 
    for (int i = 0; i < rowMax.length; i++) {
        for (int j = 0; j < colMax.length; j++) {
            int increment = Math.min(rowMax[i], colMax[j]) - grid[i][j];
            sum += increment;
        }
    }

    return sum;
}
```

明明解法差不多, 但語法效能就是比別人差 XD