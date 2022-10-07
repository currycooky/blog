# 3Sum Closest

## Overview
Given an integer array nums of length n and an integer target, find three integers in nums such that the sum is closest to target.

Return the sum of the three integers.

You may assume that each input would have exactly one solution.

> 来源：力扣（LeetCode）
> 
> 链接：https://leetcode.cn/problems/3sum-closest
> 
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## Solution
```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        int res = 10000000;
        Arrays.sort(nums);
        int length = nums.length;
        for (int i = 0; i < length; i++) {
            int a = nums[i];
            int l = i + 1, r = length - 1;
            while (l < r) {
                int sum = a + nums[l] + nums[r];
                if (sum == target) {
                    return sum;
                }
                // find best, update res
                if (Math.abs(res - target) > Math.abs(sum - target)) {
                    res = sum;
                }
                // because sum grater than target, we need to decrease sum, and nums are sorted, we can only move left from right
                if (sum > target) {
                    r--;
                } else {
                    // on the contrary, we can only move right from left when we want to increase sum
                    l++;
                }
            }
        }
        return res;
    }
}
```