---
title: 移除最多的同行或同列石头
date: 2019-07-13 12:15:51
tags:
    - Depth-first-Search
    - Union-Find
categories: 
    - leetcode
    - Medium
---


题目链接：[most-stones-removed-with-same-row-or-column](https://leetcode-cn.com/problems/most-stones-removed-with-same-row-or-column/)

# 题目描述
在二维平面上，我们将石头放置在一些整数坐标点上。每个坐标点上最多只能有一块石头。现在，move 操作将会移除与网格上的某一块石头共享一列或一行的一块石头。我们最多能执行多少次 move 操作？

# 题目分析
1. 刚开始看到这道题目的时候，一脸蒙蔽，不是很明白最多move操作的意思，然后把示例画成二维图，试了一下，终于明白，这道题目的意思是，让最终每行每列上最多只能有一块石头，最少剩下多少块石头，然后返回原石头数量-剩余石头数量。
2. 很显然，这道题目可以使用深度优先搜索和并查集的方法。我们可以将每行每列定义为一个component，将有石头相连的component合并成一个（并查集的方法），最后在统计有多少个独立的component，即剩下的石头数量；也可以对每块石头所在的component执行一个深度优先遍历，最后统计有从入口执行了多少次，即剩下的石头数量。

# 算法
## 深度优先遍历（Depth-first-Search,time: O(N<sup>2</sup>),memory：O(N）
### 代码
``` java
import java.util.*;

class Solution {
    
    public int removeStones(int[][] stones) {
        int N = stones.length;
        
        // 这里无所谓行和列的概念，我们将行和列都当做一个component，
        // graph[i][0]记录了第i块石头的邻居数量。
        int[][] graph = new int[N][N];
        for (int i = 0; i < N; i++) {
            for (int j = i + 1; j < N; j++) {
                // 两块石头在同一行或同一列，将他们记录为邻居
                if (stones[i][0] == stones[j][0] || stones[i][1] == stones[j][1]) {
                    graph[i][++graph[i][0]] = j;
                    graph[j][++graph[j][0]] = i;
                }
            }
        }
        

        boolean[] seen = new boolean[N];    // 深搜遍历过的标志，避免死循环
        int ans = 0;
        for (int i = 0; i < N; i++) {
            if (!seen[i]) {
                seen[i] = true;
                Stack<Integer> stack = new Stack<>();
                ans--;
                stack.push(i);
                while (!stack.isEmpty()) {
                    int cur = stack.pop();
                    ans++;
                    // 将该块石头的所有邻居推入栈
                    for (int j = 1; j <= graph[cur][0]; j++) {
                        if (!seen[graph[cur][j]]) {
                            seen[graph[cur][j]] = true;
                            stack.push(graph[cur][j]);
                        }
                    }
                }
            }
        }
        return ans;
    }
        
}
```

## 并查集（Union-Find,time: O(NlogN),memory：O(N）
### 代码
``` java
import java.util.*;

class Solution {
    
    public int removeStones(int[][] stones) {
        int N = stones.length;
        DSU dsu = new DSU(20000);
        // 将这块石头所在行和列合并成一个component
        // 最终，能合并在一起的石头会拥有同样的parent
        for (int i = 0; i < N; i++) dsu.union(stones[i][0], stones[i][1] + 10000);
        
        HashSet<Integer> seen = new HashSet<>();
        // 遍历所有的石头，找出它的父节点，即最后剩余的最少石头数量。
        for (int i = 0; i < N; i++) seen.add(dsu.find(stones[i][0]));
        return N - seen.size();
    }
    
    class DSU {
        int[] parent;
        
        public DSU(int n) {
            parent = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }
        
        public int find(int p) {
            if (parent[p] != p) parent[p] = find(parent[p]);
            return parent[p];
        }
        
        public void union(int p, int q) {
            parent[find(p)] = find(q);
        }
    }
        
}
```