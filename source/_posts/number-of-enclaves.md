---
title: 飞地的数量
date: 2019-07-06 16:04:13
tags: Depth-first-Search
categories:
    - leetcode
    - Medium
---


题目链接：[number-of-enclaves](https://leetcode-cn.com/problems/number-of-enclaves/)

# 题目描述
给出一个二维数组 A，每个单元格为 0（代表海）或 1（代表陆地）。移动是指在陆地上从一个地方走到另一个地方（朝四个方向之一）或离开网格的边界。返回网格中无法在任意次数的移动中离开网格边界的陆地单元格的数量。


# 题目分析
1. 这道题其实和leetcode上岛屿的数量那道题目很类似，不同点在于连接到边界的岛屿不计入数量，并且这里要计算的是岛屿的面积（1的数量），不是岛屿的面积。
2. 我们应该从边界开始遍历，要把能连接到边界的陆地1转化成用-1表示（像是地理上的半岛），最后，在重新遍历地图，数出1的数量。

# 算法
## 深度优先遍历（Depth-first-Search,time: O(M * N),memory：O(M*N）
### 代码
``` java
class Solution {
    int[][] dirs = { {-1, 0}, {1, 0}, {0, -1}, {0, 1} };    // 四个方向，上下左右
    
    public int numEnclaves(int[][] A) {
        int ans = 0;

        for (int row = 0; row < A.length; row++) {
            for (int col = 0; col < A[0].length; col++) {
                // 当在边界且边界这里是陆地（即1）的时候，往下深搜，把连接到这块陆地的所有陆地转化为半岛（-1）
                if (isBoundary(A, row, col) && A[row][col] == 1) {
                    dfs(A, row, col);
                }
            }
        }
        
        // 计算岛屿的面积
        for (int row = 0; row < A.length; row++)
            for (int col = 0; col < A[0].length; col++)
                if(A[row][col] == 1) ans++;
        return ans;
    }
    
    
    private void dfs(int[][] A, int row, int col) {
        // 超出地图边界或者非陆地或者已经遍历过的陆地，直接返回
        if (outOfBoundary(A, row, col) || A[row][col] == 0 || A[row][col] == -1) return;
        // 将该陆地标记为岛屿，并遍历
        A[row][col] = -1;
        for (int[] dir : dirs) {
            dfs(A, row + dir[0], col + dir[1]);
        }
    }
    
    // 判断地图边界
    private boolean isBoundary(int[][] A, int row, int col) {
        return row == 0 || row == A.length - 1 || col == 0 || col == A[0].length - 1;
    }
    
    // 判断是否超出地图边界
    private boolean outOfBoundary(int[][] A, int row, int col) {
        return row < 0 || row > A.length - 1 || col < 0 || col > A[0].length - 1;
    }
    
}
```
