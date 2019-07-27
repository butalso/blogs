---
title: minesweeper
date: 2019-07-27 12:27:36
tags:
    - Depth-first-Search
    - Breadth-first-Search
categories:
    - leetcode
    - Medium
---

题目链接：[minesweeper](https://leetcode-cn.com/problems/most-stones-removed-with-same-row-or-column/)

# 题目描述
让我们一起来玩扫雷游戏！
给定一个代表游戏板的二维字符矩阵。 'M' 代表一个未挖出的地雷，'E' 代表一个未挖出的空方块，'B' 代表没有相邻（上，下，左，右，和所有4个对角线）地雷的已挖出的空白方块，数字（'1' 到 '8'）表示有多少地雷与这块已挖出的方块相邻，'X' 则表示一个已挖出的地雷。
现在给出在所有未挖出的方块中（'M'或者'E'）的下一个点击位置（行和列索引），根据以下规则，返回相应位置被点击后对应的面板：
1. 如果一个地雷（'M'）被挖出，游戏就结束了- 把它改为 'X'。
2. 如果一个没有相邻地雷的空方块（'E'）被挖出，修改它为（'B'），并且所有和其相邻的方块都应该被递归地揭露。
3. 如果一个至少与一个地雷相邻的空方块（'E'）被挖出，修改它为数字（'1'到'8'），表示相邻地雷的数量。
4. 如果在此次点击中，若无更多方块可被揭露，则返回面板。

# 题目分析
1. 这里的相邻是上下左右和四个对角线，注意不是只有上下左右

# 算法
## 深度优先遍历（Depth-first-Search）
### 代码
``` java
class Solution {
    public char[][] updateBoard(char[][] board, int[] click) {
        int row = click[0];
        int col = click[1];
        if (board[row][col] == 'M') board[row][col] = 'X';  // 踩雷直接返回
        else dfs(board, row, col);
        return board;
    }
    
    private void dfs(char[][] board, int row, int col) {
        if (isOutOfBoundary(board, row, col) || board[row][col] != 'E') return; // 越界或已经遍历过
        int mines = getNumOfMinesAround(board, row, col);
        if (mines == 0) {
            // 周围地雷数量为0，才会递归遍历
            board[row][col] = 'B';
            for (int i = -1; i <= 1; i++) {
                for (int j = -1; j <= 1; j++) {
                    dfs(board, row + i, col + j);
                }
            }
        } else {
            board[row][col] = (char)('0' + mines);
        }
    }
    
    // 获取周围地雷数量
    public int getNumOfMinesAround(char[][] board, int row, int col) {
        int result = 0;
        for (int i = -1; i <= 1; i++) {
            for (int j = -1; j <= 1; j++) {
                if ((i == j && i == 0) || isOutOfBoundary(board, row + i, col + j)) continue;
                if (board[row+i][col+j] == 'M') result++;
            }
        }
        return result;
    }
    
    // 判断是否越界
    public boolean isOutOfBoundary(char[][] board, int row, int col) {
        return row < 0 || row >= board.length || col < 0 || col >= board[0].length;
    }
}
```

## 广度优先遍历（Breadth-first-Search）
### 代码
``` java
class Solution {
    public char[][] updateBoard(char[][] board, int[] click) {
        int row = click[0];
        int col = click[1]; 
        if (board[row][col] == 'M') {
            // 踩雷直接返回
            board[row][col] = 'X';
            return board;
        }

        Queue<int[]> queue = new LinkedList<>();
        queue.offer(click);
        while (!queue.isEmpty()) {
            click = queue.poll();
            row = click[0];
            col = click[1];
            if (isOutOfBoundary(board, row, col) || board[row][col] != 'E') continue;   // 越界或已经遍历过
            int mines = getNumOfMinesAround(board, row, col);
            if (mines == 0) {
                // 周围地雷数量为0，入队遍历
                board[row][col] = 'B';
                for (int i = -1; i <= 1; i++) {
                    for (int j = -1; j <= 1; j++) {
                        queue.offer(new int[]{row + i, col + j});
                    }
                }
            } else {
                board[row][col] = (char)('0' + mines);
            }    
        }
        return board;
    }
    
    // 获取周围地雷数量
    public int getNumOfMinesAround(char[][] board, int row, int col) {
        int result = 0;
        for (int i = -1; i <= 1; i++) {
            for (int j = -1; j <= 1; j++) {
                if ((i == j && i == 0) || isOutOfBoundary(board, row + i, col + j)) continue;
                if (board[row+i][col+j] == 'M') result++;
            }
        }
        return result;
    }
    
    // 判断是否越界
    public boolean isOutOfBoundary(char[][] board, int row, int col) {
        return row < 0 || row >= board.length || col < 0 || col >= board[0].length;
    }
}
```