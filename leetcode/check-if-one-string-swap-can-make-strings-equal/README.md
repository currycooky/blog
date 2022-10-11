# Check if One String Swap Can Make Strings Equal

## Overview
You are given two strings s1 and s2 of equal length. A string swap is an operation where you choose two indices in a string (not necessarily different) and swap the characters at these indices.

Return true if it is possible to make both strings equal by performing at most one string swap on exactly one of the strings. Otherwise, return false.

> 来源：力扣（LeetCode）
> 
> 链接：https://leetcode.cn/problems/check-if-one-string-swap-can-make-strings-equal
> 
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## Solution
```java
class Solution {
    public boolean areAlmostEqual(String s1, String s2) {
        if (s1.equals(s2)) {
            return true;
        }
        int length = s1.length();
        char a1 = ' ', b1 = ' ', a2 = ' ', b2 = ' ';
        for (int i = 0; i < length; i++) {
            char c1 = s1.charAt(i);
            char c2 = s2.charAt(i);
            if (c1 != c2) {
                // found first different
                if (a1 == ' ') {
                    a1 = c1;
                    a2 = c2;
                } else if (b1 == ' ') {
                    // found second different
                    b1 = c1;
                    b2 = c2;
                } else {
                    // found other different, no solution
                    return false;
                }
            }
        }
        return a1 == b2 && a2 == b1;
    }
}
```