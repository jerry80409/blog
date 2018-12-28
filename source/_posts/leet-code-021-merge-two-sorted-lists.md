---
title: leet-code-021-merge-two-sorted-lists
cover: /img/home-bg.jpg
date: 2018-12-12 12:51:36
categories: ['misc']
tags: ['leetcode']
---
### Test
```java
public class MergeTwoSortedListsTest {

    final MergeTwoSortedLists mergeTwoSortedLists = new MergeTwoSortedLists();

    /**
     * input: null [0]
     * expect: [0] 
     */
    @Test
    public void test1() {
        final ListNode n1 = null;
        final ListNode n2 = new ListNode(0);

        ListNode act = mergeTwoSortedLists.mergeTwoLists(n1, n2);

        Assert.assertEquals(0, act.val);
    }

    /**
     * input: [2] [1]
     * expect: [1, 2] 
     */
    @Test
    public void test2() {
        final ListNode n1 = new ListNode(2);
        final ListNode n2 = new ListNode(1);

        ListNode act = mergeTwoSortedLists.mergeTwoLists(n1, n2);

        Assert.assertEquals(1, act.val);
        Assert.assertEquals(2, act.next.val);
    }

    /**
     * input: [1] [2]
     * expect: [1, 2] 
     */
    @Test
    public void test3() {
        final ListNode n1 = new ListNode(1);
        final ListNode n2 = new ListNode(2);

        ListNode act = mergeTwoSortedLists.mergeTwoLists(n1, n2);

        Assert.assertEquals(1, act.val);
        Assert.assertEquals(2, act.next.val);
    }

    /**
     * input: [5] [1, 2, 4]
     * expect: [1, 2, 4, 5] 
     */
    @Test
    public void test4() {
        final ListNode n1 = new ListNode(5);
        final ListNode n2 = new ListNode(1);
        final ListNode n3 = new ListNode(2);
        final ListNode n4 = new ListNode(4);
        n2.next = n3;
        n3.next = n4;

        ListNode act = mergeTwoSortedLists.mergeTwoLists(n1, n2);
        Assert.assertEquals(1, act.val);
        Assert.assertEquals(2, act.next.val);
        Assert.assertEquals(4, act.next.next.val);
        Assert.assertEquals(5, act.next.next.next.val);
    }

    /**
     * input: [-9, 3] [5, 7]
     * expect: [-9, 3, 5, 7] 
     */
    @Test
    public void test5() {
        final ListNode n1 = new ListNode(-9);
        final ListNode n2 = new ListNode(3);
        n1.next = n2;

        final ListNode n3 = new ListNode(5);
        final ListNode n4 = new ListNode(7);
        n3.next = n4;

        ListNode act = mergeTwoSortedLists.mergeTwoLists(n1, n3);

        Assert.assertEquals(-9, act.val);
        Assert.assertEquals(3, act.next.val);
        Assert.assertEquals(5, act.next.next.val);
        Assert.assertEquals(7, act.next.next.next.val);
    }
}
```

### Solution
```java
/**
 * https://leetcode.com/problems/merge-two-sorted-lists/
 * 
 * Example:
 * Input: 1->2->4, 1->3->4
 * Output: 1->1->2->3->4->4
 */
public class MergeTwoSortedLists {

    /**
     * 
     * @param n1
     * @param n2
     * @return
     */
    public ListNode mergeTwoLists(ListNode n1, ListNode n2) {
        if (Objects.isNull(n1)) return n2;
        if (Objects.isNull(n2)) return n1;

        if (n1.val < n2.val) {
            n1.next = mergeTwoLists(n2, n1.next);
            return n1;

        } else {
            n2.next = mergeTwoLists(n1, n2.next);
            return n2;
        }
    }

    /**
     * Definition for singly-linked list.
     */
    static class ListNode {
        public int val;
        public ListNode next;

        public ListNode(int x) {
            val = x;
        }
    }
}

```