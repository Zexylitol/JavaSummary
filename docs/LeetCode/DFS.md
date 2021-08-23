<!-- GFM-TOC -->

- [LeetCode 797. 所有可能的路径](#LeetCode-797-所有可能的路径)
- [LeetCode 22. 括号生成](#LeetCode-22-括号生成)
- [LeetCode 129. 求根节点到叶子节点数字之和](#LeetCode-129-求根节点到叶子节点数字之和)
- [LCP 07. 传递信息](#LCP-07-传递信息)
- [Offer 13. 机器人的运动范围](#Offer-13-机器人的运动范围)
- [面试题 0401. 节点间通路](#面试题-0401-节点间通路)

<!-- GFM-TOC -->

> DFS 算法就是回溯算法

# 模板

- https://mp.weixin.qq.com/s/Bh2huuuK2Z33oULZ7Qtt-Q

```java
Graph graph;
// 记录遍历过的节点，用于有环图
boolean[] visited;

/* 图遍历框架 */
void traverse(Graph graph, int s) {
    if (visited[s]) return;
    // 经过节点 s
    visited[s] = true;
    for (TreeNode neighbor : graph.neighbors(s)) {
        traverse(neighbor);
    }
    // 离开节点 s
    visited[s] = false;   
}
```

# LeetCode 797. 所有可能的路径

- 回溯

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
        dfs(0, graph, new ArrayList<>());
        return res;
    }

    public void dfs(int v, int[][] graph, List<Integer> path) {
        if (v == graph.length - 1) {
            path.add(v);
            res.add(new ArrayList<>(path));
            return ;
        }
        path.add(v);
        for (int s : graph[v]) {
            dfs(s, graph, path);
            path.remove(path.size() - 1);
        }
    }
}
```

- https://mp.weixin.qq.com/s/Bh2huuuK2Z33oULZ7Qtt-Q

# LeetCode 22. 括号生成

- https://leetcode-cn.com/problems/generate-parentheses/solution/hui-su-suan-fa-by-liweiwei1419/
  - 可以生成左括号的条件：左括号剩余数量（严格）大于0
  - 可以生成右括号的条件：左括号剩余数量（严格）小于 右括号剩余数量

```java
import java.util.ArrayList;
import java.util.List;

public class Solution {

    public List<String> generateParenthesis(int n) {
        List<String> res = new ArrayList<>();
        if (n == 0) {
            return res;
        }

        StringBuilder path = new StringBuilder();
        dfs(path, n, n, res);
        return res;
    }


    /**
     * @param path  从根结点到任意结点的路径，全程只使用一份
     * @param left  左括号还有几个可以使用
     * @param right 右括号还有几个可以使用
     * @param res
     */
    private void dfs(StringBuilder path, int left, int right, List<String> res) {
        if (left == 0 && right == 0) {
            // path.toString() 生成了一个新的字符串，相当于做了一次拷贝，这里的做法等同于「力扣」第 46 题、第 39 题
            res.add(path.toString());
            return;
        }

        // 剪枝（如图，左括号可以使用的个数严格大于右括号可以使用的个数，才剪枝，注意这个细节）
        if (left > right) {
            return;
        }

        if (left > 0) {
            path.append("(");
            dfs(path, left - 1, right, res);
            path.deleteCharAt(path.length() - 1);
        }

        if (right > 0) {
            path.append(")");
            dfs(path, left, right - 1, res);
            path.deleteCharAt(path.length() - 1);
        }
    }
}
```

# LeetCode 129. 求根节点到叶子节点数字之和

- 方法一：先求出所有根节点到叶子节点之间的路径，再求和

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
 List<List<Integer>> res = new ArrayList<>();
    /**
     *
     * @param root TreeNode类
     * @return int整型
     */
    public int sumNumbers (TreeNode root) {
        // write code here
        if (root == null) {
            return 0;
        }
        dfs(root, new ArrayList<Integer>());
        int ans = 0;
        for (int i = 0; i < res.size(); i++) {
            List<Integer> path = res.get(i);
            //path.forEach(System.out::print);
            //System.out.println();
            int sum = 0;
            for (Integer num : path) {
                sum *= 10;
                sum += num;
            }
            ans += sum;
        }
        return ans;
    }

    public void dfs(TreeNode root, List<Integer> path) {
        if (root == null) {
            return ;
        }
        if (root.left == null && root.right == null) {
            path.add(root.val);
            res.add(new ArrayList<>(path));
            path.remove(path.size() - 1);
            return ;
        }
        path.add(root.val);
        dfs(root.left, path);
        dfs(root.right, path);
        path.remove(path.size() - 1);
    }
}
```

- 方法二：前序遍历

```java
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
   
    public int sumNumbers(TreeNode root) {       
        return dfs(root, 0);
    }

    public int dfs(TreeNode root, int num) {
        if (root == null) return 0;
        num = num * 10 + root.val;
        if (root.left == null && root.right == null) {                    
            return num;
        }
        return dfs(root.left, num) + dfs(root.right, num);
    }
}
```

# LCP 07. 传递信息

- DFS

**解题思路**

\1. 每个小朋友可看成一个顶点；

\2. A、B小朋友之间可传递信息，可看成A、B之间有一条单向边

\3. 根据$relation$数组构造每个顶点的边集合

\4. 从0号小朋友开始遍历：

​    1）遍历结束条件：遍历了k轮且信息到达了n-1号小朋友，方案数+1

​    2）若遍历了k轮，信息没有到达n-1号小朋友，则返回

​    3）遍历当前小朋友可以将信息传递到的小朋友

```java
class Solution {
    int res = 0;
    public int numWays(int n, int[][] relation, int k) {
        List<List<Integer>> edges = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            edges.add(new ArrayList<>());
        }
        for (int[] r : relation) {
            int a = r[0];
            int b = r[1];
            // 小朋友a可以将信息传递到小朋友b
            edges.get(a).add(b);
        }
        // 从0号小朋友开始遍历，计算经过k轮可以将信息传递到n-1号小朋友的方案数
        dfs(edges, 0, k);
        return res;
    }

    public void dfs(List<List<Integer>> edges, int n,int k) {
        // 找到一种方案数
        if (n == edges.size() - 1 && k == 0) {
            res++;
            return ;
        }
        
        // 最多遍历k轮
        if (k <= 0) {
            return ;
        }

        // 遍历当前小朋友可以将信息传递到的顶点
        List<Integer> edge = edges.get(n);
        for (Integer e : edge) {
            dfs(edges, e, k - 1);
        }
    }
}
```



- 动态规划

```java
class Solution {
    public int numWays(int n, int[][] relation, int k) {
        // 动态规划
      	// dp[i][j] 为经过i轮传递到编号j的玩家的方案数
        int[][] dp = new int[k + 1][n];
        dp[0][0] = 1;
        for (int i = 0; i < k; i++) {
            for (int[] r : relation) {
                int src = r[0], dst = r[1];
                dp[i + 1][dst] += dp[i][src];
            }
        }
        return dp[k][n-1];
    }
}
```

# Offer 13. 机器人的运动范围

```java
class Solution {
    int res = 0;
    public int movingCount(int m, int n, int k) {
        if (k == 0) {
            return 1;
        }
        boolean[][] visited = new boolean[m][n];
        dfs(m, n, 0, 0, k, visited);
        return res;
    }

    public void dfs(int m, int n, int x, int y, int k, boolean[][] visited) {
        if (x < 0 || x >= m || y < 0 || y >= n || !isLessK(x, y, k) || visited[x][y]) {
            return ;
        }
        res++;
        visited[x][y] = true;
        // 上 右 下 左
        dfs(m, n, x - 1, y, k, visited);
        dfs(m, n, x, y + 1, k, visited);
        dfs(m, n, x + 1, y, k, visited);
        dfs(m, n, x, y - 1, k, visited);
    }

    public boolean isLessK(int m, int n, int k) {
        while (m != 0) {
            k = k - m%10;
            if (k < 0) {
                return false;
            }
            m = m / 10;
        }
        while (n != 0) {
            k = k - n%10;
            if (k < 0) {
                return false;
            }
            n = n / 10;
        }
        return true;
    }
}
```

# 面试题 0401. 节点间通路

节点间通路。给定有向图，设计一个算法，找出两个节点之间是否存在一条路径。

示例1:

> 输入：n = 3, graph = [[0, 1], [0, 2], [1, 2], [1, 2]], start = 0, target = 2
>  输出：true

**提示：**

1. 节点数量n在[0, 1e5]范围内。
2. 节点编号大于等于 0 小于 n。
3. 图中可能存在自环和平行边。

```java
class Solution {
    // 访问状态数组
    private boolean[] visited = null;
    public boolean findWhetherExistsPath(int n, int[][] graph, int start, int target) {
        // 创建访问状态数组
        this.visited = new boolean[graph.length];
        // DFS
        return helper(graph, start, target);
    }

    private boolean helper(int[][] graph, int start, int target) {
        // 深度优先搜索
        for (int i = 0; i < graph.length; ++i) {
            // 确保当前路径未被访问（该判断主要是为了防止图中自环出现死循环的情况）
            if (!visited[i]) {
                // 若当前路径起点与终点相符，则直接返回结果
                if (graph[i][0] == start && graph[i][1] == target) {
                    return true;
                }
                // 设置访问标志
                visited[i] = true;
                // DFS关键代码，思路：同时逐渐压缩搜索区间
                if (graph[i][1] == target && helper(graph, start, graph[i][0])) {
                    return true;
                }
                // 清除访问标志
                visited[i] = false;
            }
        }
        return false;
    }
}
```

