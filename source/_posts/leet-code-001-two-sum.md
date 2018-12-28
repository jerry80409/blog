---
title: leet-code-001-two-sum
cover: /img/home-bg.jpg
date: 2018-12-12 12:50:45
categories: ['misc']
tags: ['leetcode']
---
### Test
```java
public class TwoSumTest {

    final TwoSum twoSum = new TwoSum();

    @Test
    public void test1() {
        final int[] nums = new int[]{2, 7, 11, 15};
        final int target = 9;

        int[] act = twoSum.twoSum(nums, target);

        Assert.assertArrayEquals(new int[]{0, 1}, act);
    }

    @Test
    public void test2() {
        final int[] nums = new int[]{3, 3};
        final int target = 6;

        int[] act = twoSum.twoSum(nums, target);

        Assert.assertArrayEquals(new int[]{0, 1}, act);
    }

    @Test
    public void test3() {
        final int[] nums = new int[]{3, 2, 4};
        final int target = 6;

        int[] act = twoSum.twoSum(nums, target);

        Assert.assertArrayEquals(new int[]{1, 2}, act);
    }
}
```

### Solution
```java
/**
 * https://leetcode.com/problems/two-sum/
 * 
 * Given nums = [2, 7, 11, 15], target = 9,
 * Because nums[0] + nums[1] = 2 + 7 = 9,
 * return [0, 1].
 */
public class TwoSum {

    /**
     * 
     * @param nums
     * @param target
     * @return
     */
    public int[] twoSum(final int[] nums, final int target) {
        
        HashMap<Integer, Integer> cache = new HashMap<>();
        int[] result = new int[2];

        for (int i = 0; i < nums.length; i++) {
            
            if (cache.containsKey(nums[i])) {
                int index = cache.get(nums[i]);
                result[0] = index;
                result[1] = i;
                return result;

            } else {
                // put [diff, index]
                cache.put(target - nums[i], i);
            }
        }
        return result;
    }
}
```