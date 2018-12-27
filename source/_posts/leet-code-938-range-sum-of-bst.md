---
title: leet-code-938-range-sum-of-bst
cover: /img/home-bg.jpg
date: 2018-12-26 15:50:33
categories: ['leetcode']
tags: ['leetcode']
---
### Pazzle
Given the root node of a binary search tree, return the sum of values of all nodes with value between L and R (inclusive).

The binary search tree is guaranteed to have unique values.

Example:
Input: root = [10,5,15,3,7,null,18], L = 7, R = 15
Output: 32

Input: root = [10,5,15,3,7,13,18,1,null,6], L = 6, R = 10
Output: 23

### Test
花了一點時間, 才看懂 L, R 的用意, 一直以為是左葉, 右葉, 我在想啥 XD.
若單純用陣列看, L(7), R(5) 就是天花板跟地板,
只要取出陣列中 `5 <= n <=7` 的元素就好了, 
但題目限定要用 `TreeNode`, 一樣先搞定 test 再解題, 
先用測試把問題簡單化...

```java
/**
 * input: root = [10,5,15,3,7,null,18], L = 7, R = 15
 * expect: 23
 */
@Test
public void test_case_1() {
    // leafs
    RangeSumBst.TreeNode node3 = new RangeSumBst.TreeNode(3);
    RangeSumBst.TreeNode node7 = new RangeSumBst.TreeNode(7);
    RangeSumBst.TreeNode node18 = new RangeSumBst.TreeNode(18);

    // level_2
    RangeSumBst.TreeNode node5 = new RangeSumBst.TreeNode(5, node3, node7);
    RangeSumBst.TreeNode node15 = new RangeSumBst.TreeNode(15, null, node18);

    // level_1
    RangeSumBst.TreeNode root = new RangeSumBst.TreeNode(10, node5, node15);

    int l = 7;
    int r = 15;
    int act = bst.rangeSumBST(root, l, r);

    assertEquals(32, act);
}

/**
 * input: root = [5, 3, 7], L = 7, R = 15
 * expect: 23
 */
@Test
public void test_case_2() {
    // leafs
    RangeSumBst.TreeNode node3 = new RangeSumBst.TreeNode(3);
    RangeSumBst.TreeNode node7 = new RangeSumBst.TreeNode(7);

    // level_1
    RangeSumBst.TreeNode root = new RangeSumBst.TreeNode(5, node3, node7);

    int l = 7;
    int r = 15;
    int act = bst.rangeSumBST(root, l, r);

    assertEquals(7, act);
}

/**
 * input: root = [3, null, null], L = 7, R = 15
 * expect: 23
 */
@Test
public void test_case_3() {
    // level_1
    RangeSumBst.TreeNode root = new RangeSumBst.TreeNode(3, null, null);

    int l = 7;
    int r = 15;
    int act = bst.rangeSumBST(root, l, r);

    assertEquals(0, act);
}

/**
 * input: root = [7, null, null], L = 7, R = 15
 * expect: 23
 */
@Test
public void test_case_4() {
    // level_1
    RangeSumBst.TreeNode root = new RangeSumBst.TreeNode(7, null, null);

    int l = 7;
    int r = 15;
    int act = bst.rangeSumBST(root, l, r);

    assertEquals(7, act);
}
```

### Solution
解題的思路大概就是 binary-tree 走訪, node 大於 L 小於 R 就加總,
binary-tree 特性是, left tree 任何 node 遠永小於 root, right tree 任何 node 永遠大於 root;
這邊寫得比較清楚, 有複習有保庇 - [http://www.csie.ntnu.edu.tw/~u91029/BinaryTree.html](http://www.csie.ntnu.edu.tw/~u91029/BinaryTree.html)

```java
int rangeSumBST(TreeNode root, int l, int r) {

    if (Objects.isNull(root)) {
        return 0;
    }

    if (root.val < l) {
        return rangeSumBST(root.right, l, r);
    }

    if (root.val > r) {
        return rangeSumBST(root.left, l, r);
    }

    return root.val + rangeSumBST(root.left, l, r) + rangeSumBST(root.right, l, r);
}
```
