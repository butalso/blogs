---
title: pyramid-transition-matrix
date: 2019-08-03 12:27:19
tags:
    - Bit-Manipulation
    - Depth-first-Search
categories:
    - leetcode
    - Medium
---

题目链接：[pyramid-transition-matrix](https://leetcode-cn.com/problems/pyramid-transition-matrix/)

# 题目描述
现在，我们用一些方块来堆砌一个金字塔。 每个方块用仅包含一个字母的字符串表示，例如 “Z”。使用三元组表示金字塔的堆砌规则如下：(A, B, C) 表示，“C”为顶层方块，方块“A”、“B”分别作为方块“C”下一层的的左、右子块。当且仅当(A, B, C)是被允许的三元组，我们才可以将其堆砌上。初始时，给定金字塔的基层 bottom，用一个字符串表示。一个允许的三元组列表 allowed，每个三元组用一个长度为 3 的字符串表示。如果可以由基层一直堆到塔尖返回true，否则返回false。

# 题目分析
1. 首先，看完题目，我们可以很清楚的知道，我们需要将每层的各种可能性组合起来，然后不断往金字塔顶部迭代（递归），找到一种可能性。
2. 由于每层的方块数都不一致，因此，我们不能以层作为递归条件，而且，可能有好多层，for循环次数不一致。应该以每个块作为递归条件，在每次递归中，再加以判断是不是应该换层。

# 算法
## 深度优先遍历（Depth-first-Search）
### 代码
``` java
class Solution {
    public boolean pyramidTransition(String bottom, List<String> allowed) {
        Map<String, List<Character>> map = new HashMap<>(); // 可堆砌块的映射
        for (String s : allowed) {
            String key = s.substring(0, 2);
            List<Character> vals = map.getOrDefault(key, new ArrayList<Character>());
            vals.add(s.charAt(2));
            map.put(key, vals);
        }
        return dfs(bottom, map, "");
    }
    
    private boolean dfs(String bottom, Map<String, List<Character>> map, String path) {
        if (bottom.length() == 1) return true;  // 到达金字塔顶端，返回true
        if (bottom.length() == path.length() + 1) return dfs(path, map, "");    // 当前一个层完成，进入下一层
        String key = bottom.substring(path.length(), path.length() + 2);
        for (Character c : map.getOrDefault(key, new ArrayList<Character>())) {
            if (dfs(bottom, map, path + c)) return true; // 递归遍历，每个砖块
        }
        return false;
    }
    
}
```

## 深度优先遍历+位运算（Depth-first-Search + Bit-Manipulation）
### 代码
``` java
class Solution {
    int N = 7;  // 题目里只有7中方块
    public boolean pyramidTransition(String bottom, List<String> allowed) {
        int[][] blocks = new int[N][N];
        for (String s : allowed) {
            blocks[s.charAt(0) - 'A'][s.charAt(1) - 'A'] |= (1 << (s.charAt(2) - 'A')); // 类似上一解法中的map
        }
        int[][] pyramids = new int[bottom.length()][];
        for (int i = 0; i < bottom.length(); i++) {
            pyramids[i] = new int[bottom.length() - i]; // 第i层砖块长度
            pyramids[0][i] = bottom.charAt(i) - 'A'; // 初始层砖块
        }
        return dfs(pyramids, blocks, 1, 0);
    }
    
    private boolean dfs(int[][] pyramids, int[][] blocks, int level, int pos) {
        if (level == pyramids.length) return true;  // 金字塔构建完成
        if (pos == pyramids[level].length) return dfs(pyramids, blocks, level + 1, 0);  // 进入下一层
        
        int mask = blocks[pyramids[level - 1][pos]][pyramids[level - 1][pos + 1]];  // 当前两个砖块可以堆叠的全部可能性
        for (int i = 0; i < N; i++) {
            if ((mask & (1 << i)) != 0) {
                pyramids[level][pos] = i;
                if (dfs(pyramids, blocks, level, pos + 1)) return true;
            }
        }
        return false;
    }
}
```