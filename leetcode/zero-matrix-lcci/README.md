# Zero Matrix LCCI

## Overview
Write an algorithm such that if an element in an MxN matrix is 0, its entire row and column are set to 0.

Example 1:
```
Input: 
[
  [1,1,1],
  [1,0,1],
  [1,1,1]
]
Output: 
[
  [1,0,1],
  [0,0,0],
  [1,0,1]
]
```
Example 2:
```
Input: 
[
  [0,1,2,0],
  [3,4,5,2],
  [1,3,1,5]
]
Output: 
[
  [0,0,0,0],
  [0,4,5,0],
  [0,3,1,0]
]
```

> 来源：力扣（LeetCode）
> 
> 链接：https://leetcode.cn/problems/zero-matrix-lcci
> 
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## Solution
```java
class Solution {
    public void setZeroes(int[][] matrix) {
        final int rows = matrix.length;
        final int cols = matrix[0].length;
        boolean[] m = new boolean[rows];
        boolean[] c = new boolean[cols];

        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (matrix[i][j] == 0) {
                    m[i] = true;
                    c[j] = true;
                }
            }
        }
        // 重新遍历
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (m[i] || c[j]) {
                    matrix[i][j] = 0;
                }
            }
        }
    }
}
```