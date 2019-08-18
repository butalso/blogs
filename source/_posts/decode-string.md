---
title: decode-string
date: 2019-08-18 11:27:37
tags:
    - Stack
    - Depth-first-Search
categories:
    - leetcode
    - Medium
---

题目链接：[decode-string](https://leetcode-cn.com/problems/decode-string/)

# 题目描述
给定一个经过编码的字符串，返回它解码后的字符串。编码规则为: k[encoded_string]，表示其中方括号内部的 encoded_string 正好重复 k 次。注意 k 保证为正整数。你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 k ，例如不会出现像 3a 或 2[4] 的输入。

# 题目分析
基本上，我们可以理解为只用三种情况
1. 字母字符串，直接返回
2. 数字字符串，后边跟了各'[‘，将他的下层返回的字符字符串加倍存储。
3. ']'一层结束的标志，返回字符字符串。

# 算法
## 栈（Stack）
### 代码
``` java
import java.util.*;

class Solution {
    public String decodeString(String s) {
        StringBuilder result = new StringBuilder();
        LinkedList<String> stack = new LinkedList<String>();
        int index = 0;
        while (index < s.length()) {
            if (Character.isLetter(s.charAt(index))) {
                // 字母字符串
                StringBuilder cur = new StringBuilder();
                while(index < s.length() && Character.isLetter(s.charAt(index))) {
                    cur.append(s.charAt(index));
                    index++;
                }
                stack.push(cur.toString());
            } else if (Character.isDigit(s.charAt(index))) {
                // 数字字符串
                int j = index;
                while (Character.isDigit(s.charAt(j))) j++;
                stack.push(s.substring(index, j));
                index = j + 1;
            } else {
                while (index < s.length() && s.charAt(index) == ']') {
                    // 一层结束
                    StringBuilder tmp = new StringBuilder();
                    String s2 = stack.pop();
                    String s1 = stack.pop();
                    if (Character.isDigit(s1.charAt(0))) {
                        // 将栈里最后两个字符串合并（数字 * 字母）
                        int k = Integer.parseInt(s1);
                        while (k-- > 0) {
                            tmp.append(s2);
                        }
                        index++;
                    } else {
                        // 有可能同一层里套了多个子层，所以，有可能是两个字符串，需要先合并如 "2[3[a]2[b]]"， 我们
                        // 在完成2[b]合并的同时，栈里是[2, aa, bb],我们需要将aa也合并，直到栈里的前一个字符串是数字。
                        tmp.append(s1);
                        tmp.append(s2);
                    }
                    stack.push(tmp.toString());
                }
            }
        }
        // 最外一层合并，如例子 "3[a]2[bc]"
        while (!stack.isEmpty()) {
            result.append(stack.removeLast());
        }
        return result.toString();
    }
}
```

## 深度优先遍历（Depth-first-Search）
事实上，感觉这题不是经典的深搜，更加符合stack的标签，不过理解为深搜也没毛病。
### 代码
``` java
class Solution {
    public String decodeString(String s) {
        StringBuilder result = new StringBuilder();
        dfs(s.toCharArray(), 0, result);
        return result.toString();
    }
    
    public int dfs(char[] cc, int start, StringBuilder str) {
        while (start < cc.length) {
            if (Character.isDigit(cc[start])) {
                int count = 0;
                while (Character.isDigit(cc[start])) {
                    count = count * 10 + cc[start] - '0';
                    start++;
                }
                StringBuilder inner = new StringBuilder();      // 将下一层解析的字符串结果带回来
                int end = dfs(cc, start + 1, inner);
                while (count-- > 0) str.append(inner.toString());   // 合并 数字 * 字符串
                start = end;
            } else if (Character.isLetter(cc[start])) {
                // 存储连续的字符串
                while (start < cc.length && Character.isLetter(cc[start])) {    
                    str.append(cc[start]);
                    start++;
                }
            } else {
                // ‘]'字符，返回这层结束后的下一个index
                return start + 1;
            }
        }
        return start;
    }
}
```

