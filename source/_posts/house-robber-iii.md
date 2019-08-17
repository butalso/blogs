---
title: house-robber-iii
date: 2019-08-17 13:30:47
tags:
    - Tree
    - Depth-first-Search
categories:
    - leetcode
    - Medium
---

题目链接：[house-robber-iii](https://leetcode-cn.com/problems/house-robber-iii/)

# 题目描述
在上次打劫完一条街道之后和一圈房屋后，小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为“根”。 除了“根”之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果两个直接相连的房子在同一天晚上被打劫，房屋将自动报警。计算在不触动警报的情况下，小偷一晚能够盗取的最高金额。

# 题目分析
不相邻的节点和最大值，让我们可以分成两种情况：
1. 该节点值 + 他的间接子树最大值的和
2. 他的直接子树最大值的和

# 算法
## 深度优先遍历（Depth-first-Search）
### 代码
``` java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int rob(TreeNode root) {
        return dfs(root);
    }
    
    public int dfs(TreeNode root) {
        if (root == null) return 0;
        int result = root.val;
        if (root.left != null) result += dfs(root.left.left) + dfs(root.left.right);    // 该节点左间接子树和最大值
        if (root.right != null) result += dfs(root.right.left) + dfs(root.right.right); // 该节点右间接子树和最大值
        result = Math.max(result, dfs(root.left) + dfs(root.right));    // 比较选择包含该节点还是不包含该节点的和最大值
        return result;
    }
}
```

## 深度优先遍历（优化）（Depth-first-Search）
在上一解法中，明显可以发现，我们在寻找当前树的最大值时，还需要寻找子树的子树最大值，而在寻找子树的最大值时，我们已经查找过了子树的子树的最大值，即有很多计算是重复的。因此，在深搜过程中，同时返回包含该节点和最大值和不包含该节点和最大值可以给我们运行时间上带来不少优化。
### 代码
``` java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int rob(TreeNode root) {
        int[] result = dfs(root);
        return Math.max(result[0], result[1]);
    }
    
    // [0] => 包含该节点的和最大值
    // [1] => 不包含该节点的和最大值
    public int[] dfs(TreeNode root) {
        if (root == null) return new int[]{0, 0};
        int result = root.val;
        int[] l = dfs(root.left);
        int[] r = dfs(root.right);
        return new int[] {
            root.val + l[1] + r[1], Math.max(l[0], l[1]) + Math.max(r[0], r[1])
            };
    }
}
```
