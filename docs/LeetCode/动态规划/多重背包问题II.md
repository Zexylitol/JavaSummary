# 题目
[多重背包问题II](https://www.acwing.com/problem/content/5/)
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
$0<  N\le 1000$
$0<  V\le 2000$
$0< v_{i},w_{i},s_{i}\le 2000$

**输入样例**

> 4 5
> 1 2 3
> 2 4 1
> 3 4 3
> 4 5 2

**输出样例**

> 10



# 解题思路
相比[多重背包问题I](https://blog.csdn.net/xylitolz/article/details/109146126)，本题**数据范围变大，需优化时间复杂度**。

不能用[完全背包问题](https://blog.csdn.net/xylitolz/article/details/109060118)中**抛弃k循环**的思路来优化这个问题，**因为每个物品的件数不同，不能像完全背包问题那样优化状态转移方程**。

## 多重背包的二进制优化
考虑**将多重背包问题转化为01背包问题**，考虑**把第$i$种物品换成若干件物品**，使得原问题中第$i$种物品<font color=#FF7F50>可取的每种策略(取$0$件、取$1$件、...、取$k$件)，均能等价于取若干件转换以后的物品，取超过$k$件的策略必不能出现</font>。

若直接把第$i$件物品换成$s[i]$件01背包中的物品，时间复杂度不变。

**考虑二进制的思想**，将$k$件物品拆成$\log_{2}{k}$件新的物品，$k=1+2+2^{2}+...+2^{t}+c\left ( c\le 2^{t+1}   \right )$，<font color=#FF7F50>用这$\log_{2}{k}$个新数，可以凑出$0 \sim k$中的任何一个数</font>。这$\log_{2}{k}$个数作为新物品的系数，新物品的体积和价值就是原物品的费用和价值乘以这个系数。

eg: $13=1+2+4+6$，最多取13件的物品被分成系数分别为$1,2,4,6$的四件物品，原先要枚举14次，拆分之后只需枚举5次；这种优化对于大数尤其明显，例如最多取1024件的物品，在正常情况下要枚举1025次 ， 二进制思想下转化成01背包只需要枚举11次。

```java
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        int n = sc.nextInt();               // 物品种数
        int m = sc.nextInt();               // 背包容积
        
        int[] v = new int[n];
        int[] w = new int[n];
        int[] s = new int[n];
        
        
        for (int i = 0; i < n; i++) {
            v[i] = sc.nextInt();
            w[i] = sc.nextInt();
            s[i] = sc.nextInt();
        }
        
        // 二进制拆分
        int[] newV = new int[n*11];    // 2^11 = 2048,也就是说s最多可以拆成11个，故数组容量乘以11
        int[] newW = new int[n*11];
        int newN = 0;                   // 新的物品种数
        for (int i = 0; i < n; i++) {
            for (int j = 1; j <= s[i]; j *= 2) {
                newV[newN] = j * v[i];   // 体积
                newW[newN] = j * w[i];   // 价值
                s[i] -= j;
                newN++;
            }
            if (s[i] > 0) {
                newV[newN] = s[i] * v[i];
                newW[newN] = s[i] * w[i];
                newN++;
            }
        }
        
        // 01背包问题
        int[] dp = new int[m+1];        
        for (int i = 0; i < newN; i++) {
            for (int j = m; j >= newV[i]; j--) {
                dp[j] = Math.max(dp[j], dp[j-newV[i]]+newW[i]);
            }
        }
        System.out.println(dp[m]);
    }
}

```



# Reference
1. [背包问题九讲](https://github.com/tianyicui/pack)
2. [多重背包的二进制拆分代码](https://www.acwing.com/solution/content/1024/)
3. [多重背包问题的优化思路](https://www.acwing.com/solution/content/5527/)
4. [多重背包问题的优化思路](https://www.acwing.com/solution/content/9434/)