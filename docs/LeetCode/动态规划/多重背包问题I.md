# 题目
[多重背包问题I](https://www.acwing.com/problem/content/4/)

有 $N$ 种物品和一个容量是 $V$ 的背包。

第 $i$ 种物品最多有 $s_{i}$ 件，每件体积是 $v_{i}$，价值是 $w_{i}$。

求解将哪些物品装入背包，可使物品体积总和不超过背包容量，且价值总和最大。
输出最大价值。

**输入格式**
第一行两个整数，$N$，$V$，用空格隔开，分别表示物品种数和背包容积。

接下来有 $N$ 行，每行三个整数 $v_{i}$,$w_{i}$,$s_{i}$，用空格隔开，分别表示第 $i$ 种物品的体积、价值和数量。

**输出格式**
输出一个整数，表示最大价值。

**数据范围**
$0<  N,V\le 100$
$0< v_{i},w_{i},s_{i}\le 100$

**输入样例**

> 4 5
> 1 2 3
> 2 4 1
> 3 4 3
> 4 5 2

**输出样例**

> 10

# 解题思路
<font color=#D2691E >**多重背包问题**：每个物品有$s_{i}$件</font>
<font color=#FF7F50 >**完全背包问题**：每个物品可以无限使用</font>

因此，对于当前物品，仍然考虑**取$0$件**、**取$1$件**、...、**取$k$件**，但除了考虑**背包容量限制**之外还需考虑**物品数量限制**，即：$k\ast v[i] \le j$ && $k\le s[i]$

1. **状态定义**
$dp[i][j]$ : 前$i$个物品，在容量为$j$的背包下，最大的价值

2. **状态转移**
$dp[i][j]=max\left ( dp[i][j], dp[i-1][j-k\ast v[i]]+k\ast w[i]  \right )$
$0\le k\le \left \lfloor \frac{j}{v[i]}  \right \rfloor$ && $k\le s[i]$

```java
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        int n = sc.nextInt();               // 物品种数
        int m = sc.nextInt();               // 背包容积
        
        int[] v = new int[n+1];
        int[] w = new int[n+1];
        int[] s = new int[n+1];
        
        int[][] dp = new int[n+1][m+1];
        
        for (int i = 1; i <= n; i++) {
            v[i] = sc.nextInt();
            w[i] = sc.nextInt();
            s[i] = sc.nextInt();
        }
        
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                for (int k = 0; k <= s[i] && (k * v[i] <= j); k++) {
                    dp[i][j] = Math.max(dp[i][j], dp[i-1][j-k*v[i]]+k*w[i]);     //k是从0开始更新的。因此当k=0时，实际上就是dp[i-1][j]
                }
            }
        }
        System.out.println(dp[n][m]);
    }
}

```

**时间复杂度**：$O\left ( n\ast m\ast s  \right )$
**空间复杂度**：$O\left ( n\ast m  \right )$