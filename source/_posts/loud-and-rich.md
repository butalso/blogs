---
title: loud-and-rich
date: 2019-08-17 13:43:39
tags:
    - Depth-first-Search
categories:
    - leetcode
    - Medium
---

题目链接：[loud-and-rich](https://leetcode-cn.com/problems/house-robber-iii/)

# 题目描述
在一组 N 个人（编号为 0, 1, 2, ..., N-1）中，每个人都有不同数目的钱，以及不同程度的安静（quietness）。为了方便起见，我们将编号为 x 的人简称为 "person x "。如果能够肯定 person x 比 person y 更有钱的话，我们会说 richer[i] = [x, y] 。注意 richer 可能只是有效观察的一个子集。另外，如果 person x 的安静程度为 q ，我们会说 quiet[x] = q 。现在，返回答案 answer ，其中 answer[x] = y 的前提是，在所有拥有的钱不少于 person x 的人中，person y 是最安静的人（也就是安静值 quiet[y] 最小的人）

# 题目分析
1. 直观上看，对每个人，我们需要找出比他有钱或者和他一样有钱（他自己）的人，然后比较找出这些人（包括他自己）中最安静的一个作为结果。（解法1）
2. 事实上，在寻找结果过程中，我们没必要找出所有比他有钱的人，比如A比B有钱，C比A有钱，在查找过程中，对于B，我们没有必要同时比较A、B、C三个人的安静值，我们可以先比较A、C的安静值，将min(A, C)结果存起来，然后再比较min(min(A, C), B)的值，即得出比B有钱且最安静的人。（解法2）

# 算法
## 深度优先遍历（Depth-first-Search）
### 代码
``` java
import java.util.*;

class Solution {
    public int[] loudAndRich(int[][] richer, int[] quiet) {
        int N = quiet.length; 
        Map<Integer, Set<Integer>> map = new HashMap<>(); // value值存的是比key值更有钱的人
        for (int[] r : richer) {
            Set<Integer> tmp = map.getOrDefault(r[1], new HashSet<>());
            tmp.add(r[0]);
            map.put(r[1], tmp);
        }
        for (int i = 0; i < N; i++) {
            // 遍历深搜，对每个人，找出比他更有钱的人
            Set<Integer> tmp = new HashSet<>();
            dfs(map, tmp, i);
            map.put(i, tmp);
        }

        int[] result = new int[N];
        for (int i = 0; i < N; i++) {
            // 比较，对每个人，从比他更有钱的人找出最安静的人
            result[i] = i;
            for (int p : map.get(i)) {
                if (quiet[result[i]] > quiet[p])
                    result[i] = p;
            }
        }
        return result;
    }
    
    public void dfs(Map<Integer, Set<Integer>> map, Set<Integer> cur, int j) {
        Set<Integer> tmp = new HashSet<>(map.getOrDefault(j, new HashSet<>()));
        for (int richer : tmp) {
            if (!cur.contains(richer)) {
                cur.add(richer);
                dfs(map, cur, richer);                
            }
        }
    }    
}
```

## 深度优先遍历（优化）（Depth-first-Search）
### 代码
``` java
import java.util.*;

class Solution {
    List<Integer>[] graph; // 同解法1map,graph[i]表示比i更有钱的人（不完全集合）
    int[] result;
    int[] quiet;
    
    public int[] loudAndRich(int[][] richer, int[] quiet) {
        int N = quiet.length; 
        graph = new ArrayList[N];
        result = new int[N];
        this.quiet = quiet;
        
        for (int i = 0; i < N; i++) {
            graph[i] = new ArrayList<Integer>();
            result[i] = -1;
        }
        for (int[] edge : richer) {
            graph[edge[1]].add(edge[0]);
        }
        for (int node = 0; node < N; node++) {
            dfs(node);
        }
        return result;
    }
    
    // 返回比node更有钱或同样有钱（包括他自己）的人中最安静的人的编号
    public int dfs(int node) {
        if (result[node] == -1) {
            result[node] = node;
            for (int child : graph[node]) {
                int cand = dfs(child);
                if (quiet[cand] < quiet[result[node]]) {
                    result[node] = cand;
                }
            }
        }
        return result[node];
    }    
}
```
