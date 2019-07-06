---
title: 由斜杠划分区域
date: 2019-07-01 11:20:37
tags:
    - Depth-first-Search
    - Union-Find
    - Graph
categories:
    - leetcode
    - Medium
thumbnail: /2019/07/01/regions-cut-by-slashes/region.png
---

题目链接：[regions-cut-by-slashes](https://leetcode-cn.com/problems/regions-cut-by-slashes/)

# 题目描述
在由 1 x 1 方格组成的 N x N 网格 grid 中，每个 1 x 1 方块由 /、\ 或空格构成。这些字符会将方块划分为一些共边的区域。（请注意，反斜杠字符是转义的，因此 \ 用 "\\" 表示。）。返回区域的数目。

# 题目分析
1. 该题目用用图来表示的话，一目了然。可是表示成了字符串之后，就感觉有点不知从何处下手。
2. 想办法将图形表示转化成直观的数组表示法。

# 算法
## 并查集（Union Find）
<!-- ![区域划分编号](/region2.png) -->
{% asset_img region2.png 区域划分编号 %}
由于每个小区域只填充'/'或者'\'，因此，小区域划分最多有三种情况（编号如上图所示）：
1. 编号0和编号1在同一个块，编号2和编号3在同一个块。
2. 编号0和编号2在同一个块，编号1和编号3在同一个块。
3. 编号0， 1， 2， 3都在同一个块。

相邻区域间，他们的编号小区域存在一定会在同一个块的情况。其中：
1. 编号0和上一个区域编号3一定在同一个块。
2. 编号1和左边区域的编号2一定在同一个块。
3. 编号2和右边区域的编号1一定在同一个块。
4. 编号3和下边区域的编号0一定在同一个块。

我们开始给每个小区域编一个唯一的块号，通过遍历区域，达到数出全部唯一块的数目。

### 代码
``` java
class Solution {
    private int[] grids;            // 用数组来表示块编号
    private int num;                // 统计唯一块的数目
    
    public int regionsBySlashes(String[] grid) {
        int N = grid.length;
        buildUnionFind(N);
        
        // 遍历合并所有的区域
        for (int r = 0; r < grid.length; r++)
            for (int c = 0; c < grid[r].length(); c++)
                process(r, c, N, grid[r].charAt(c));
        return num;
    }
    
    // 构建并查集，初始每个块有一个唯一编号
    private void buildUnionFind(int N) {
        grids = new int[N * N * 4];
        num = N * N * 4;
        for (int i = 0; i < grids.length; i++) grids[i] = i;
    }
    
    // 根据行和列的序号，得到块在数组中的位置
    private int getSubSquare(int r, int c, int N, int ith) {
        return r * N * 4 + c * 4 + ith;
    }
    
    // 寻找块的根编号
    private int find(int p) {
        while (grids[p] != p) {
            grids[p] = grids[grids[p]];
            p = grids[p];
        }
        return p;
    }
    
    // 合并两个块
    private void union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        
        if (rootP != rootQ) num--;      // 如果两个块拥有不同的编号，合并后，唯一块的数目减1
        grids[rootP] = rootQ;
    }
    

    private void process(int r, int c, int N, char square) {
        // 获取区域中四个小区域的块编号
        int s0 = getSubSquare(r, c, N, 0);
        int s1 = getSubSquare(r, c, N, 1);
        int s2 = getSubSquare(r, c, N, 2);
        int s3 = getSubSquare(r, c, N, 3);
        
        // 同一个区域的小区域合并
        switch(square) {
            case '\\':
                union(s0, s2);
                union(s1, s3);
                break;
            case '/':
                union(s0, s1);
                union(s2, s3);
                break;
            default:
                union(s0, s1);
                union(s1, s2);
                union(s2, s3);
        }

        // 相邻区域的合并
        if (r > 0) {
            union(s0, getSubSquare(r - 1, c, N, 3));
        }
        if (r < N - 1) {
            union(s3, getSubSquare(r + 1, c, N, 0));
        }
        if (c > 0) {
            union(s1, getSubSquare(r, c - 1, N, 2));
        }
        if (c < N - 1) {
            union(s2, getSubSquare(r, c + 1, N, 1));
        }
    }
}
```


## 深度优先搜索（Depth first Search）
我们很难从原图中，将该图转化成一个用数字表示的数组。但是，当我们将原图表示成一个规模*2的数组时，他将可以表示如下：
```
                        0 1 0 1
["//", "/ "]    =>      1 0 1 0
                        0 1 0 0
                        1 0 0 0
```
我们将数字1当做一道墙，数字0当做一个空白块，则没有墙相隔的块，他们属于同一个唯一块。从上数字中，我们可以直观的看出，一共存在3个唯一块。但是这时，属于同一个唯一空白块的小空白块他们可能还不相邻（即在斜对角线上）。我们再把原图表示成一个规模*3数组，表示如下：
```
                        0 0 1 0 0 1
                        0 1 0 0 1 0
["//", "/ "]    =>      1 0 0 1 0 0
                        0 0 1 0 0 0
                        0 1 0 0 0 0
                        1 0 0 0 0 0
```
这时，可以清楚的看出，一共存在3个唯一空白块，且在同一个唯一空白块的所有小空白块，都是相邻的了。这个时候，我们就可以采用深度优先搜索，从数组的边界开始，遍历存在多少个不相邻的0，即可得到答案。
### 代码
``` java
class Solution {
    int[][] grids;          // 表示原图的数组
    
    public int regionsBySlashes(String[] grid) {
        buildGrids(grid);
        int count = 0;
        // 深搜遍历
        for (int r = 0; r < grids.length; r++) {
            for (int c = 0; c < grids[r].length; c++) {
                count += dfs(r, c);
            }
        }
        return count;
    }
    
    // 将原图表示成规模*3的数组
    private void buildGrids(String[] grid) {
        int N = grid.length;
        grids = new int[N * 3][N * 3];
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[i].length(); j++) {
                if (grid[i].charAt(j) == '/') {
                    grids[i * 3 + 2][j * 3] = 1;
                    grids[i * 3 + 1][j * 3 + 1] = 1;
                    grids[i * 3][j * 3 + 2] = 1;
                } else if (grid[i].charAt(j) == '\\') {
                    grids[i * 3][j * 3] = 1;
                    grids[i * 3 + 1][j * 3 + 1] = 1;
                    grids[i * 3 + 2][j * 3 + 2] = 1;
                }
            }
        }
    }
    
    private int dfs(int r, int c) {
        // 越界返回0
        if (r < 0 || r >= grids.length || c < 0 || c >= grids[r].length) return 0;
        // 遇到墙（数字1）返回0
        if (grids[r][c] == 1) return 0;
        
        // 将空白块改为1，并向他四周遍历，结果返回1
        grids[r][c] = 1;
        dfs(r - 1, c);
        dfs(r + 1, c);
        dfs(r, c - 1);
        dfs(r, c + 1);
        return 1;
    }
}
```
