---
title: all-nodes-distance-k-in-binary-tree
date: 2019-08-10 11:36:22
tags:
    - Tree
    - Depth-first-Search
    - Breadth-first-Search
categories:
    - leetcode
    - Medium
---

题目链接：[all-nodes-distance-k-in-binary-tree](https://leetcode-cn.com/problems/all-nodes-distance-k-in-binary-tree/)

# 题目描述
给定一个二叉树（具有根结点 root）， 一个目标结点 target ，和一个整数值 K 。返回到目标结点 target 距离为 K 的所有结点的值的列表。 答案可以以任何顺序返回。

# 题目分析
1. 距离给定节点距离为K的节点有两种情况：第一种是他的子节点，这种情况比较容易通过遍历得到。第二种是他的祖先节点的其他子树，对于这种情况，我们需要能找到节点的祖先节点的方法。
2. 找到某节点父节点的方法可有两种，一种是深搜回退到他的父节点，一种是建立子节点到父节点的映射。

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
    List<Integer> result;
    int k;

    public List<Integer> distanceK(TreeNode root, TreeNode target, int K) {
        result = new ArrayList<>();
        k = K;
        dfs(root, target);
        return result;
    }

    public int dfs(TreeNode cur, TreeNode target) {
        if (cur == null) {
            return -1;            // 返回-1表示在当前子树没有找到目标节点
        }
        if (cur == target) {
            subtreeAdd(cur, 0); // 找到目标节点了，对目标节点子树进行查找
            return 1; // 返回他的父节点到目标节点距离
        } else {
            int d = dfs(cur.left, target);
            if (d != -1) {
                // 目标节点在左子树
                if (d == k) {
                    // 当前节点符合到目标节点距离，添加到结果集
                    result.add(cur.val);                    
                }
                subtreeAdd(cur.right, d + 1);  // 寻找当前节点右子树符合条件的节点
                return d + 1; // 返回当前节点父节点到目标节点的距离
            } else {
                d = dfs(cur.right, target);
                if (d != -1) {
                    // 目标节点在右子树
                    if (d == k) {
                        // 当前节点符合到目标节点距离，添加到结果集
                        result.add(cur.val);    
                    }
                    subtreeAdd(cur.left, d + 1);    // 寻找当前节点左子树符合条件的节点
                    return d + 1;   // 返回当前节点父节点到目标节点的距离
                }
            }
            return -1;  // 目标节点在当前子树不存在
        }
    }

    // 查找当前子树符合到目标节点距离的值，保存结果
    public void subtreeAdd(TreeNode cur, int d) {
        if (cur == null || d > k) {
            return;
        }
        if (k == d) {
            result.add(cur.val);            
        }
        subtreeAdd(cur.left, d + 1);
        subtreeAdd(cur.right, d + 1);
    }
}
```

## 宽度优先搜索（Breadth-first-Search）
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

    public List<Integer> distanceK(TreeNode root, TreeNode target, int K) {
        Map<TreeNode, TreeNode> child2parent = new HashMap<>();
        dfs(root, null, child2parent);  // 建立子节点到父节点的映射

        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(target);
        queue.offer(null);  // 以null为到目标节点距离相同的节点集分割点

        HashSet<TreeNode> seen = new HashSet<>();   // 标记查找过的节点，保证入队节点距离目标节点的距离是逐渐增加的
        seen.add(target);
        seen.add(null);

        List<Integer> result = new ArrayList<>();   // 保存结果
        while (!queue.isEmpty()) {
            TreeNode tmp;
            if (K == 0) {
                // 到目标节点距离为K，将当前队列中所有节点集保存到结果，null节点是分割点
                while ((tmp = queue.poll()) != null) {
                    result.add(tmp.val);
                }
            } else {
                while ((tmp = queue.poll()) != null) {
                    // 遍历当前距离目标节点距离相等的所有节点，分别将他的父节点、左子节点、右子节点
                    // 入队（如果这些节点没有被标记过，即他们离目标节点的距离都是+1）
                    if (!seen.contains(tmp.left)) {
                        seen.add(tmp.left);
                        queue.offer(tmp.left);
                    }
                    if (!seen.contains(tmp.right)) {
                        seen.add(tmp.right);
                        queue.offer(tmp.right);
                    }
                    if (!seen.contains(child2parent.get(tmp))) {
                        seen.add(child2parent.get(tmp));
                        queue.offer(child2parent.get(tmp));
                    }
                }
                queue.offer(null); // null入队分割下次循环
                K--; // 目标距离-1
            }
        }
        return result;
    }
    
    public void dfs(TreeNode cur, TreeNode parent, Map<TreeNode, TreeNode> child2parent) {
        if (cur != null) {
            child2parent.put(cur, parent);
            dfs(cur.left, cur, child2parent);
            dfs(cur.right, cur, child2parent);
        }
    }
}
```