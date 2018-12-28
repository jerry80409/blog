---
title: codesignal-almost-increasing-sequence
cover: /img/home-bg.jpg
date: 2018-12-26 11:41:07
categories: ['misc']
tags: ['codesignal']
---
### Pazzle
Given a sequence of integers as an array, determine whether it is possible to obtain a strictly increasing sequence by removing no more than one element from the array.

Example:
For sequence = [1, 3, 2, 1], the output should be
almostIncreasingSequence(sequence) = false.

For sequence = [1, 3, 2], the output should be
almostIncreasingSequence(sequence) = true

Ref - [https://app.codesignal.com/arcade/intro/level-2/2mxbGwLzvkTCKAJMG](https://app.codesignal.com/arcade/intro/level-2/2mxbGwLzvkTCKAJMG)

### Test
```java
@Test
public void test_case_1() {
    boolean act = almostIncreasingSequence(new int[]{1, 3, 2, 1});
    assertFalse(act);
}

@Test
public void test_case_2() {
    boolean act = almostIncreasingSequence(new int[]{1, 3, 2});
    assertTrue(act);
}

@Test
public void test_case_3() {
    boolean act = almostIncreasingSequence(new int[]{10, 1, 2, 3, 4, 5});
    assertTrue(act);
}
    
@Test
public void test_case_4() {
    boolean act = almostIncreasingSequence(new int[]{1, 2, 1, 2});
    assertFalse(act);
}

@Test
public void test_case_5() {
    boolean act = almostIncreasingSequence(new int[]{1, 3});
    assertTrue(act);
}
```

### Solution
一開始的想法直覺是想到排序後做比較, diff > 2 以上 return false, 但不太對, 
像 `[10, 1, 2, 3, 4, 5]` 排序後, diff 絕對大於 2, 且也是屬於 almost incrasing seq,
最後回歸基本迴圈 foreach 比較 
```java
boolean almostIncreasingSequence(int[] seq) {
    int countNoSeq = 0;
    // compare with next element 
    for (int i = 0; i < seq.length - 1; i++) {
        if (seq[i] >= seq[i + 1]) {
            countNoSeq++;
        }
    }
    // compare with after next element, because removed an element
    for (int i = 0; i < seq.length - 2; i++) {
        if (seq[i] >= seq[i + 2]) {
            countNoSeq++;
        }
    }

    if (countNoSeq > 2) {
        return false;
    }

    return true;
}
```
