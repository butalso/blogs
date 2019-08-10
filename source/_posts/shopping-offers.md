---
title: shopping-offers
date: 2019-08-04 11:48:23
tags:
    - Dynamic-Programming
    - Depth-first-Search
categories:
    - leetcode
    - Medium
---

题目链接：[shopping-offers](https://leetcode-cn.com/problems/shopping-offers/)

# 题目描述
在LeetCode商店中， 有许多在售的物品。然而，也有一些大礼包，每个大礼包以优惠的价格捆绑销售一组物品。现给定每个物品的价格，每个大礼包包含物品的清单，以及待购物品清单。请输出确切完成待购清单的最低花费。每个大礼包的由一个数组中的一组数据描述，最后一个数字代表大礼包的价格，其他数字分别表示内含的其他种类物品的数量。任意大礼包可无限次购买。

说明:
1. 最多6种物品， 100种大礼包。
2. 每种物品，你最多只需要购买6个。
3. 你不可以购买超出待购清单的物品，即使更便宜。

# 题目分析
1. 首先，改题目明显具有最有子结构的特征，因此可以采用动态规划算法。
2. 最优值，如果采用深搜，明显具有可以剪枝特征。

# 算法
## 深度优先遍历（Depth-first-Search）
### 代码
``` java
public class Solution {
    public int shoppingOffers(List < Integer > price, List < List < Integer >> special, List < Integer > needs) {
        return dfs(price, special, needs, 0);
    }
    
    public int dfs(List < Integer > price, List < List < Integer >> special, List < Integer > needs, int offerIndex) {
        if (offerIndex == special.size()) {
            // 不采用套餐
            int cost = 0;
            for (int i = 0; i < needs.size(); i++) cost += price.get(i) * needs.get(i);
            return cost;
        }
        
        int cost = Integer.MAX_VALUE;
        // 采用当前套餐
        List<Integer> offer = special.get(offerIndex);
        if (check(offer, needs)) {
            for (int i = 0; i < needs.size(); i++) {
                needs.set(i, needs.get(i) - offer.get(i));
            }
            cost = Math.min(cost, 
                            offer.get(price.size()) + dfs(price, special, needs, offerIndex));     // 继续采用当前套餐
            cost = Math.min(cost,
                            offer.get(price.size()) + dfs(price, special, needs, offerIndex + 1)); // 采用下一个套餐
            // 将当前需要商品数还原
            for (int i = 0; i < needs.size(); i++) {
                needs.set(i, needs.get(i) + offer.get(i));
            }
        }
        // 不采用当前套餐
        cost = Math.min(cost,
                        dfs(price, special, needs, offerIndex + 1));
        return cost;
    }
    
    
    // 检验当前套餐是否可用
    private boolean check(List<Integer> offer, List<Integer> needs) {
        for (int i = 0; i < needs.size(); i++) {
            if (offer.get(i) > needs.get(i)) return false;
        }
        return true;
    }

}
```

## 深度优先遍历+剪枝（Depth-first-Search+Pruning）
### 代码
``` java
class Solution {
    int total = Integer.MAX_VALUE;  // 总花费
    
    public int shoppingOffers(List<Integer> price, List<List<Integer>> special, List<Integer> needs) {
        dfs(price, special, needs, 0, 0);
        return total;
    }
    
    private void dfs(List<Integer> price, List<List<Integer>> special, List<Integer> needs, int offerIndex, int cost) {
        if (cost >= total) return;  // 剪枝
        if (offerIndex == special.size()) {
            // 不采用套餐
            for (int i = 0; i < needs.size(); i++) cost += price.get(i) * needs.get(i);
            total = Math.min(total, cost);
            return;
        }

        // 采用当前套餐
        List<Integer> offer = special.get(offerIndex);
        if (check(offer, needs)) {
            for (int i = 0; i < needs.size(); i++) {
                needs.set(i, needs.get(i) - offer.get(i));
            }
            dfs(price, special, needs, offerIndex, cost + offer.get(price.size()));     // 继续采用当前套餐
            dfs(price, special, needs, offerIndex + 1, cost + offer.get(price.size())); // 采用下一个套餐
            for (int i = 0; i < needs.size(); i++) {
                needs.set(i, needs.get(i) + offer.get(i));
            }
        }

        // 不采用当前套餐
        dfs(price, special, needs, offerIndex + 1, cost);
    }
    
    // 检验当前套餐是否可用
    private boolean check(List<Integer> offer, List<Integer> needs) {
        for (int i = 0; i < needs.size(); i++) {
            if (offer.get(i) > needs.get(i)) return false;
        }
        return true;
    }
    
}
```

## 动态规划（Dynamic-Programming）
### 代码
``` java
class Solution {
    
    public int shoppingOffers(List<Integer> price, List<List<Integer>> special, List<Integer> needs) {
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, 0);
        dp(price, special, needs, map);
        int key = hashcode(needs);
        return map.get(key);
    }
    
    private void dp(List<Integer> price, List<List<Integer>> special, List<Integer> needs, Map<Integer, Integer> map) {
        int code = hashcode(needs);
        if (map.get(code) != null) return;  // 已被解决的子问题，剪枝
        
        int result = 0;
        // 不采用套餐
        for (int i = 0; i < needs.size(); i++) {
            result += price.get(i) * needs.get(i);
        }
        
        for (List<Integer> offer : special) {
            if (check(offer, needs)) {
                List<Integer> temp = new ArrayList<>();
                for (int i = 0; i < needs.size(); i++) temp.add(needs.get(i) - offer.get(i));
                dp(price, special, temp, map);
                result = Math.min(result, map.get(hashcode(temp)) + offer.get(price.size()));       // 此时，子问题一定已经被解决，直接从map中获取
            }
        }
        map.put(code, result);  // 将当前最优解放入map中
    }
    
    // 检验当前套餐是否可用
    private boolean check(List<Integer> offer, List<Integer> needs) {
        for (int i = 0; i < needs.size(); i++) {
            if (offer.get(i) > needs.get(i)) return false;
        }
        return true;
    }
    
    // 将当前需要转化成一个hash值，因为说明最多买6中商品
    private int hashcode(List<Integer> needs) {
        int result = 0;
        for (int need : needs) result = result * 6 + need;
        return result;
    }
}
```
