---
title: 在二叉树中分配硬币
date: 2019-06-29 13:39:47
tags:
    - Tree
    - Depth-first-Search
categories: 
    - leetcode
    - Medium
thumbnail: /2019/06/29/distribute-coins-in-binary-tree/tree1.png
---

题目链接：[distribute-coins-in-binary-tree](https://leetcode-cn.com/problems/distribute-coins-in-binary-tree/)

# 题目描述
给定一个有 N 个结点的二叉树的根结点 root，树中的每个结点上都对应有 node.val 枚硬币，并且总共有 N 枚硬币。在一次移动中，我们可以选择两个相邻的结点，然后将一枚硬币从其中一个结点移动到另一个结点。(移动可以是从父结点到子结点，或者从子结点移动到父结点。)。返回使每个结点上只有一枚硬币所需的移动次数。

# 题目分析
1. 要使得硬币移动次数最少，我们应该把结点中多余的硬币移到离它最近且需要硬币的结点上。
2. 由于是在二叉树上移动，则硬币移动轨迹一定是从某结点移动到他的父节点或者子节点上。
3. 从需要硬币的结点到有多余硬币的结点的距离等于从有多余硬币结点的距离到需要硬币的结点的距离。
4. 一个结点可能有两个子节点，因此，如果该结点需要硬币或者有多余的硬币，我们无法一下子确定该硬币应该从左子节点还是右子节点获得或者移向左子节点还是右子节点；而一个结点一定只有一个父节点，因此，如果该结点需要硬币或者有多余的硬币，移动轨迹一定会经过他的父节点。

# 算法分析
采用深度搜索优先的思路，从树的叶子节点出发，如果该结点（设为cur）需要硬币，则从他的父节点获得，如果该结点有多余的硬币，则多余的硬币全部移动向他的父节点。即该结点的初始值为cur.val,从该结点递归到他的父节点，移动次数增加Math.abs(cur.val - 1)。该结点cur.val变为期待的1，如果该结点的值cur.val > 1，则他的父节点的的值cur.parent.val = cur.parent.val + cur.val - 1;如果该结点的值cur.val <= 1,则他的父节点的的值cur.parent.val = cur.parent.val - (-cur.val + 1)。逐步递归，直到遍历完整棵树，则可得出结果。


# 代码
## 解法一
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
    int moves = 0;      // 记录移动次数
    
    public int distributeCoins(TreeNode root) {
        dfs(root);
        return moves;
    }
    
    // 该方法返回值为移动完成后结点cur拥有多余的或者缺少的硬币数量，即cur.val - 1。
    public int dfs(TreeNode cur) {
        if (cur == null) return 0;
        int l = dfs(cur.left);
        int r = dfs(cur.right);
        moves += Math.abs(l) + Math.abs(r);         // 总结果加上左右子节点硬币流向父节点或从父节点移动过去需要的次数
        return cur.val + l + r - 1;         // 当前结点缺少的的或者多余的硬币数量
    }
    
}
```


## 解法二
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
    
    public int distributeCoins(TreeNode root) {
        return dfs(root, null);
    }
    
    // 该方法返回值为移动好当前结点（包括他的子节点，即当前结点表示的树）需要的移动次数
    public int dfs(TreeNode cur, TreeNode parent) {
        if (cur == null) return 0;
        int l = dfs(cur.left, cur);
        int r = dfs(cur.right, cur);
        
        int result = l + r;         // 移动好左右子树需要的移动次数
        // 修改当前结点父节点的硬币值
        if (cur.val < 1) {
            // 当前结点硬币值小于1，从父节点移动硬币到当前结点,父节点硬币数量减少
            result += (1 - cur.val);
            parent.val -= (1 - cur.val);
        } else if (cur.val > 1) {
            // 当前结点硬币值大于1，将多余的硬币值移动到父节点，父节点硬币数量增加
            result += (cur.val - 1);
            parent.val += (cur.val - 1);
        }
        return result;
    }
    
}
```