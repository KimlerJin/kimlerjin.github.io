---
layout: post
title: "动态规划算法详解与实践"
date: 2025-02-07
categories: blog
---

## 什么是动态规划？

动态规划（Dynamic Programming，简称DP）是一种通过把原问题分解为相对简单的子问题的方式求解复杂问题的方法。它的核心思想是：**若要解一个给定问题，我们需要解其不同部分（即子问题），再根据子问题的解以得出原问题的解**。

## 动态规划的特点

1. **最优子结构**：问题的最优解包含子问题的最优解
2. **重叠子问题**：在求解过程中，很多子问题会重复出现
3. **状态转移方程**：找到问题之间的转移关系

## 经典例题：0-1背包问题

通过一个简单的例子来理解动态规划：

``Question``
``给定n个物品，第i个物品的重量为 wgt[i-1] 价值为 val[i-1] ，和一个容量为 cap 
 的背包。每个物品只能选择一次，问在限定背包容量下能放入物品的最大价值。``
![img.png](/assets/images/dp/img.png)

我们可以将0-1背包问题看成由n轮决策的问题，该问题的目标求解是：“在限定的背包容量下放入物品的最大价值”，因此较大概率是一个动态规划问题

## 第一步：思考每轮的决策，定义状态，从而得到dp表
对于每个物品来说，不放背包，背包容量不变；放入背包，背包容量减少。由此可以得到状态定义：当前物品编号i和背包容量c，计为[i,c]
状态[i,c]对应的子问题为：当前i个物品在容量为c的背包下的最大价值。计为dp[i,c]
## 第二步，找出最优结构，进而推导出状态转移方程
当我们做出物品i的决策后，剩余的是前i-1个物品的决策的子问题，可以分为以下两种情况。
1. 不选择物品i：此时背包容量不变，价值为dp[i-1,c]
2. 选择物品i：此时背包容量减少wgt[i-1]，价值为dp[i-1,c-wgt[i-1]]+val[i-1]

上述分析可以得到状态转移方程：
```text
dp[i,c]=max(dp[i-1,c],dp[i-1,c-wgt[i-1]]+val[i-1])
```

## 第三步：确定边界条件和状态转移顺序
当无物品或背包容量为0的时候，最大价值为0，即dp[0,c]=dp[i,0]=0
当前状态[i,c]从上方[i-1,c]与左上方[i-1,c-wgt[i-1]]转移而来，因此通过两层循环编译整个dp表即可。

## 基于以上分析，我们要按顺序实现暴力搜索、记忆化搜索、动态规划解法。

## 1. 暴力搜索
```java
package com.ags.bussiness.trip.app;

public class Test {
    public static void main(String[] args) {
        Test test = new Test();
        int[] weight = {10, 20, 30};
        int[] value = {60, 110, 120};
        int result = test.dp(weight, value, 3, 50, "", true);
        System.out.println("最终结果: " + result);
    }

    public int dp(int[] weight, int[] value, int i, int c, String indent, boolean isLast) {
        // 打印当前递归调用的参数
        System.out.print(indent);
        if (!indent.isEmpty()) {
            System.out.print(isLast ? "└─ " : "├─ ");
        }
        System.out.println("dp(i=" + i + ", c=" + c + ")");

        if (i == 0 || c == 0) {
            System.out.println(indent + (isLast ? "   " : "│  ") + "基本情况：i == 0 或 c == 0，返回 0");
            return 0;
        }

        // 打印当前物品的重量和价值
        System.out.println(indent + (isLast ? "   " : "│  ") + "当前物品: 重量=" + weight[i - 1] + ", 价值=" + value[i - 1]);

        if (weight[i - 1] > c) {
            System.out.println(indent + (isLast ? "   " : "│  ") + "当前物品重量超过容量，跳过该物品");
            return dp(weight, value, i - 1, c, indent + (isLast ? "   " : "│  "), true);
        }

        // 递归计算放入当前物品的情况
        System.out.println(indent + (isLast ? "   " : "│  ") + "计算放入当前物品的情况");
        int input = dp(weight, value, i - 1, c - weight[i - 1], indent + (isLast ? "   " : "│  "), false) + value[i - 1];
        System.out.println(indent + (isLast ? "   " : "│  ") + "放入当前物品的结果: " + input);

        // 递归计算不放入当前物品的情况
        System.out.println(indent + (isLast ? "   " : "│  ") + "计算不放入当前物品的情况");
        int noPut = dp(weight, value, i - 1, c, indent + (isLast ? "   " : "│  "), true);
        System.out.println(indent + (isLast ? "   " : "│  ") + "不放入当前物品的结果: " + noPut);

        // 返回两种情况中的最大值
        int result = Math.max(noPut, input);
        System.out.println(indent + (isLast ? "   " : "│  ") + "返回最大值: " + result);
        return result;
    }
}
```

简易版如下：
```java
    public static void main(String[] args) {
        Test test = new Test();
        int[] weight = {10,20,30};
        int[] value = {60,110,120};
        test.dp(weight,value,3,30);

    }

    public int dp(int[] weight, int[] value, int i, int c) {
        if (i == 0 || c == 0) {
            return 0;
        }
        if (weight[i - 1] > c) {
            return dp(weight, value, i - 1, c);
        }
        int input = dp(weight, value, i - 1, c - weight[i - 1]) + value[i - 1];
        int noPut = dp(weight, value, i - 1, c);
        return Math.max(noPut, input);
    }
```
递归树如下：观察可以发现dp[1,10] 多次使用，重复计算了，因此可以引入记忆
![img_1.png](/assets/images/dp/img_1.png)
## 2：记忆化搜索
```java
public class Test {

    public static void main(String[] args) {
        Test test = new Test();
        int[] weight = {10, 20, 30};
        int[] value = {60, 110, 120};
        int[][] mem = new int[4][31];
        System.out.println(test.dp(weight, value, 3, 30, mem));
        System.out.println("success");

    }

    public int dp(int[] weight, int[] value, int i, int c, int[][] mem) {
        if (i == 0 || c == 0) {
            return 0;
        }
        if (weight[i - 1] > c) {
            return dp(weight, value, i - 1, c, mem);
        }
        if (mem[i][c] > 0) {
            return mem[i][c];
        }
        int input = dp(weight, value, i - 1, c - weight[i - 1], mem) + value[i - 1];
        int noPut = dp(weight, value, i - 1, c, mem);
        mem[i][c] = Math.max(noPut, input);
        return mem[i][c];
    }

}
```
![img_2.png](/assets/images/dp/img_2.png)

## 3:动态规划
```java
   public int dp2(int[] weight, int[] value, int i, int c) {
        int dp[][] = new int[i + 1][c + 1];

        for (int k = 1; k < i + 1; k++) {
            for (int j = 1; j < c + 1; j++) {
                if (weight[k - 1] > j) {
                    dp[k][j] = dp[k - 1][j];
                } else {
                    dp[k][j] = Math.max(dp[k - 1][j], dp[k - 1][j - weight[k - 1]] + value[k - 1]);
                }
            }
        }
        return dp[i][c];

    }
```
![img_3.png](/assets/images/dp/img_3.png)

## 4.空间优化
![img_4.png](/assets/images/dp/img_4.png)
```java
/* 0-1 背包：空间优化后的动态规划 */
int knapsackDPComp(int[] wgt, int[] val, int cap) {
    int n = wgt.length;
    // 初始化 dp 表
    int[] dp = new int[cap + 1];
    // 状态转移
    for (int i = 1; i <= n; i++) {
        // 倒序遍历
        for (int c = cap; c >= 1; c--) { //可以优化为 c>=wgt[i-1]
            if (wgt[i - 1] <= c) {
                // 不选和选物品 i 这两种方案的较大值
                dp[c] = Math.max(dp[c], dp[c - wgt[i - 1]] + val[i - 1]);
            }
        }
    }
    return dp[cap];
}
```


# 完全背包问题
完全背包问题与 0-1 背包的区别在于，每个物品可以选取 无限次，而 0-1 背包每个物品只能选一次。
- 在0-1背包问题中，每种物品只有一个，因此将物品i放入背包后，只能用i-1个物品中选择。
- 在完全背包问题中，每种物品的数量都是无限的，因此将物品i放入背包后，仍可以从前i个物品中选择。
- 不放入物品i：dp[i,c] = dp[i-1,c]
- 放入物品i ：dp[i,c] = dp[i,c-wgt[i-1]] + val[i-1]
 
## 状态转移方程 dp[i,c] = max(dp[i-1,c],dp[i,c-wgt[i-1]] + val[i-1])
代码与上述0-1背包问题完全一样，除了状态转移从i-1变为了i

# 零钱兑换问题
给定n种硬币，第i钟硬币的面值为coins[i-1],目标金额为amt，每种硬币可以重复选取，问能够凑出目标金额的最少硬币数量。
如果无法凑出目标金额，则返回-1。 