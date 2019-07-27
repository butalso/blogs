---
title: friend-circles
date: 2019-07-27 12:42:01
tags:
    - Depth-first-Search
    - Union-Find
categories:
    - leetcode
    - Medium
---

题目链接：[friend-circles](https://leetcode-cn.com/problems/friend-circles/)

# 题目描述
班上有 N 名学生。其中有些人是朋友，有些则不是。他们的友谊具有是传递性。如果已知 A 是 B 的朋友，B 是 C 的朋友，那么我们可以认为 A 也是 C 的朋友。所谓的朋友圈，是指所有朋友的集合。给定一个 N * N 的矩阵 M，表示班级中学生之间的朋友关系。如果M[i][j] = 1，表示已知第 i 个和 j 个学生互为朋友关系，否则为不知道。你必须输出所有学生中的已知的朋友圈总数。

# 题目分析
1. 要求返回的是朋友圈总数，即不相连的朋友圈数量，典型深搜和并查集题目。

# 算法
## 深度优先遍历（Depth-first-Search）
### 代码
``` java
class Solution {
    public int findCircleNum(int[][] M) {
        int result = 0;
        for (int row = 0; row < M.length; row++) {
            // 遍历每个学生
            result += dfs(M, row);
        }
        return result;
    }
    
    public int dfs(int[][] M, int row) {
        if (M[row][row] == 0) return 0; // 该学生已经被深搜遍历过
        for (int col = 0; col < M.length; col++) {
            if (M[row][col] == 1) {
                // row, col表示两个人，他们是朋友，从row遍历到col
                M[row][col] = 0;
                M[col][row] = 0;
                dfs(M, col);
            }
        }
        return 1;
    }
    
}
```

## 并查集（Union-Find）
### 代码
``` java
class Solution {
    public int findCircleNum(int[][] M) {
        UnionFind uf = new UnionFind(M.length);
        for (int row = 0; row < M.length; row++) {
            for (int col = 0; col < M.length; col++) {
                if (M[row][col] == 1) uf.union(row, col);   // 两人是朋友，合并朋友圈
            }
        }
        return uf.count;
    }
    
    class UnionFind {
        int[] parents;
        int count;  // 统计parents[id] == id的数量
        
        UnionFind(int n) {
            parents = new int[n];
            count = n;
            for (int i = 0; i < n; i++) {
                parents[i] = i;
            }
        }
        
        public int find(int p) {
            while (parents[p] != p) {
                parents[p] = parents[parents[p]];
                p = parents[p];
            }
            return p;
        }
        
        public void union(int p, int q) {
            if (find(p) != find(q)) {
                // 两者合并，parents[id] == id的数量减1
                count--;
                parents[find(p)] = find(q); 
            }
        }
    }
        
}
```