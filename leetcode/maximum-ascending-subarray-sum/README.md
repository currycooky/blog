# Maximum Ascending Subarray Sum

## Overview
Given an array of positive integers nums, return the maximum possible sum of an ascending subarray in nums.

A subarray is defined as a contiguous sequence of numbers in an array.

A subarray [numsl, numsl+1, ..., numsr-1, numsr] is ascending if for all i where l <= i < r, numsi < numsi+1. Note that a subarray of size 1 is ascending.

> 来源：力扣（LeetCode）
> 
> 链接：https://leetcode.cn/problems/maximum-ascending-subarray-sum
> 
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## Solution
```java
class Solution {
    public int maxAscendingSum(int[] nums) {
        int sum = 0;
        int prev = 0;
        int res = 0;
        for (int num : nums) {
            if (num > prev) {
                sum += num;
            } else {
                res = Math.max(res, sum);
                sum = num;
            }
            prev = num;
        }
        return Math.max(res, sum);
    }
}
```