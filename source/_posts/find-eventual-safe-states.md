---
title: find-eventual-safe-states
date: 2019-08-24 12:32:48
tags:
    - Depth-first-Search
    - Graph
categories:
    - leetcode
    - Medium
---

题目链接：[find-eventual-safe-states](https://leetcode-cn.com/problems/find-eventual-safe-states/)

# 题目描述
在有向图中, 我们从某个节点和每个转向处开始, 沿着图的有向边走。 如果我们到达的节点是终点 (即它没有连出的有向边), 我们停止。现在, 如果我们最后能走到终点，那么我们的起始节点是最终安全的。 更具体地说, 存在一个自然数 K,  无论选择从哪里开始行走, 我们走了不到 K 步后必能停止在一个终点。哪些节点最终是安全的？ 结果返回一个有序的数组。该有向图有 N 个节点，标签为 0, 1, ..., N-1, 其中 N 是 graph 的节点数.  图以以下的形式给出: graph[i] 是节点 j 的一个列表，满足 (i, j) 是图的一条有向边。

# 题目分析
题目意思即找出非闭环的所有节点
1. 我们可以从终点开始，即先找出那些没有出去的边的节点，即出度为0，这些节点一定符合条件，然后，反向搜索，找出他的上游，如果他的上游的所有出去的边都是在这些节点当中，则他的上游节点也是安全的。
2. 我们还可以采用深搜的方式，对每个节点的所有出去的边进行搜索，如果他的所有下游节点都是安全的，则他也是安全的。

# 算法
## 反向边（Graph)
### 代码
``` java
class Solution {
    
    public List<Integer> eventualSafeNodes(int[][] G) {
        List<Set<Integer>> graph = new ArrayList<>();       // 正向边图
        List<Set<Integer>> rgraph = new ArrayList<>();      // 反向边图
        Queue<Integer> queue = new LinkedList<>();          // 安全节点队列
        List<Integer> result = new ArrayList<>();           // 返回结果集
        boolean[] safe = new boolean[G.length];             // 标记安全节点（因为返回结果要有序的，这里用空间换时间）
        
        // 图的初始化
        for (int i = 0; i < G.length; i++) {
            graph.add(new HashSet<Integer>());
            rgraph.add(new HashSet<Integer>());
        }
        for (int i = 0; i < G.length; i++) {
            for (int j : G[i]) {
                graph.get(i).add(j);
                rgraph.get(j).add(i);
            }
            if (G[i].length == 0) queue.offer(i);
        }

        while (!queue.isEmpty()) {
            // 安全节点集非空，出队并标记
            int i = queue.poll();
            safe[i] = true;
            for (int j : rgraph.get(i)) {
                // 移除上游节点到安全节点的边，如果上游节点的所有边都是到安全节点（即graph.get(j).isEmpty()），则入队
                graph.get(j).remove(i);
                if (graph.get(j).isEmpty()) queue.offer(j);
            }
        }
        // 读取标记，返回结果
        for (int i = 0; i < G.length; i++) {
            if (safe[i]) result.add(i);
        }
        return result;
    }
}
```

## 深度优先搜索（Depth-first-Search）
### 代码
``` java
import java.util.*;

class Solution {
    // white表示未搜索，
    // black表示安全节点，
    // grey表示搜索中的节点或不安全节点
    enum Color {
        WHITE,
        BLACK,
        GREY,
    }
    
    public List<Integer> eventualSafeNodes(int[][] graph) {
        List<Integer> result = new ArrayList<>();
        if (graph.length == 0) return result;
        Color[] color = new Color[graph.length];
        Arrays.fill(color, Color.WHITE);
        for (int i = 0; i < graph.length; i++) {
            if (dfs(graph, i, color)) result.add(i);
        }
        return result;
    }
    
    public boolean dfs(int[][] graph, int i, Color[] color) {
        if (color[i] != Color.WHITE) return color[i] == Color.BLACK;        // 节点i被搜索过，剪支，快速返回
        color[i] = Color.GREY;          // 标记节点i在搜索中
        for (int child : graph[i]) {
            if (!dfs(graph, child, color)) return false;       
        }
        color[i] = Color.BLACK;         // 标记节点i安全
        return true;
    }
}
```
