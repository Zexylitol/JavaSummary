<!-- GFM-TOC -->

- [BFS与DFS区别](#BFS与DFS区别)
- [框架](#框架)
- [LeetCode 111 二叉树的最小深度](#LeetCode-111-二叉树的最小深度)
- [LeetCode 752 打开转盘锁](#LeetCode-752-打开转盘锁)
- [检测循环依赖](#检测循环依赖)
- [LeetCode 207. 课程表](#LeetCode-207-课程表)
- [LeetCode 210. 课程表II](#LeetCode-210-课程表II)

<!-- GFM-TOC -->

# BFS与DFS区别

- <font color="red">BFS 相对 DFS 的最主要的区别是：BFS 找到的路径一定是最短的，但代价就是空间复杂度比 DFS 大很多</font>

  - **为什么 BFS 可以找到最短距离，DFS 不行吗**？

    - DFS 不能找最短路径吗？其实也是可以的，但是时间复杂度相对高很多。你想啊，DFS 实际上是靠递归的堆栈记录走过的路径，你要找到最短路径，肯定得把二叉树中所有树杈都探索完才能对比出最短的路径有多长对不对？**而 BFS 借助队列做到一次一步「齐头并进」，是可以在不遍历完整棵树的条件下找到最短距离的**。
    - 形象点说，**DFS 是线，BFS 是面；DFS 是单打独斗，BFS 是集体行动**

  - **既然 BFS 那么好，为啥 DFS 还要存在**？

    - **BFS 可以找到最短距离，但是空间复杂度高，而 DFS 的空间复杂度较低**。

      还是拿刚才我们处理二叉树问题的例子，假设给你的这个二叉树是**满二叉树，**节点数为 `N`，**对于 DFS 算法来说，空间复杂度无非就是递归堆栈，最坏情况下顶多就是树的高度，也就是 `O(logN)`。**

      但是你想想 BFS 算法，队列中每次都会储存着二叉树一层的节点，这样的话最坏情况下空间复杂度应该是树的最底层节点的数量，**也就是 `N/2`，用 Big O 表示的话也就是 `O(N)`。**

      由此观之，**BFS 还是有代价的，一般来说在找最短路径的时候使用 BFS，其他时候还是 DFS 使用得多一些（主要是递归代码好写）。**

- 场景：**就是让你在一幅「图」中找到从起点 `start` 到终点 `target` 的最近距离，这个例子听起来很枯燥，但是 BFS 算法问题其实都是在干这个事儿**

  - 比如走迷宫，有的格子是围墙不能走，从起点到终点的最短距离是多少？如果这个迷宫带「传送门」可以瞬间传送呢？
  - 比如说两个单词，要求你通过某些替换，把其中一个变成另一个，每次只能替换一个字符，最少要替换几次？
  - 比如说连连看游戏，两个方块消除的条件不仅仅是图案相同，还得保证两个方块之间的最短连线不能多于两个拐点。你玩连连看，点击两个坐标，游戏是如何判断它俩的最短连线有几个拐点的？
  - **本质上就是一幅「图」，让你从一个起点，走到终点，问最短路径**

- 搜索之BFS提高课：https://www.acwing.com/blog/content/5845/

# 框架

```java
// 计算从起点 start 到终点 target 的最近距离
int BFS(Node start, Node target) {
    Queue<Node> q; // 核心数据结构
    Set<Node> visited; // 避免走回头路

    q.offer(start); // 将起点加入队列
    visited.add(start);
    int step = 0; // 记录扩散的步数

    while (q not empty) {
        int sz = q.size();
        /* 将当前队列中的所有节点向四周扩散 */
        for (int i = 0; i < sz; i++) {
            Node cur = q.poll();
            /* 划重点：这里判断是否到达终点 */
            if (cur is target)
                return step;
            /* 将 cur 的相邻节点加入队列 */
            for (Node x : cur.adj())
                if (x not in visited) {
                    q.offer(x);
                    visited.add(x);
                }
        }
        /* 划重点：更新步数在这里 */
        step++;
    }
}
```

- 队列 `q` 就不说了，BFS 的核心数据结构；`cur.adj()` 泛指 `cur` 相邻的节点，比如说二维数组中，`cur` 上下左右四面的位置就是相邻节点；`visited` 的主要作用是防止走回头路，大部分时候都是必须的，但是像一般的二叉树结构，没有子节点到父节点的指针，不会走回头路就不需要 `visited`

# LeetCode 111 二叉树的最小深度

```java
class Solution {
    public int minDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        int res = 0;
        while (!queue.isEmpty()) {
            res++;
            int n = queue.size();
            while (n > 0) {
                TreeNode node = queue.poll();
                if (node.left == null && node.right == null) {
                    return res;
                }
                if (node.left != null) {
                    queue.add(node.left);
                }
                if (node.right != null) {
                    queue.add(node.right);
                }
                n--;
            }
        }
        return res;
    }
}
```

# LeetCode 199 二叉树的右视图

对二叉树进行层序遍历，右视图即为每一层的最后一个遍历到的节点

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
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if (root == null) {
            return res;
        }
        Deque<TreeNode> queue = new LinkedList<>();
        queue.offerLast(root);
        while (!queue.isEmpty()) {
            int size = queue.size();
            while (size != 0) {
                TreeNode node = queue.removeFirst();
                if (node.left != null) queue.offerLast(node.left);
                if (node.right != null) queue.offerLast(node.right);
                if (size == 1) {
                    res.add(node.val);
                }
                size--;
            }
        }
        return res;
    }
}
```



# LeetCode 752 打开转盘锁

```java
class Solution {
    public int openLock(String[] deadends, String target) {
        // 记录需要跳过的死亡密码以及已经穷举过的密码
        Set<String> deads = new HashSet<>();
        for(String dead : deadends) {
            deads.add(dead);
        }
        Set<String> visited = new HashSet<>();
        Queue<String> queue = new LinkedList<>();
        queue.offer("0000");
        visited.add("0000");
        int res = 0;
        while (!queue.isEmpty()) {
            int size = queue.size();
            for (int i = 0; i < size; i++) {
                String cur = queue.poll();
                if (deads.contains(cur)) {
                    continue;
                }
                if (cur.equals(target)) {
                    return res;
                }

                // 枚举cur的相邻节点
                for (int j = 0; j < 4; j++) {
                    String up = upOne(cur, j);
                    if (!visited.contains(up)) {
                        queue.offer(up);
                        visited.add(up);
                    }

                    String down = downOne(cur, j);
                    if (!visited.contains(down)) {
                        queue.offer(down);
                        visited.add(down);
                    }
                }
            }
            res++;
        }
        return -1;
    }

    // 将s[i]向上拨动
    public String upOne(String s, int i) {
        char[] chars = s.toCharArray();
        if (chars[i] == '9') {
            chars[i] = '0';
        } else {
            chars[i]++;
        }
        return new String(chars);
    }

    // 将s[i向下拨动
    public String downOne(String s, int i) {
        char[] chars = s.toCharArray();
        if (chars[i] == '0') {
            chars[i] = '9';
        } else {
            chars[i]--;
        }
        return new String(chars);
    }
}
```

# 检测循环依赖

- https://mp.weixin.qq.com/s/q6AhBt6MX2RL_HNZc8cYKQ

<center><img src="https://i.loli.net/2021/04/29/fBOSbLACc1JXkDK.png"/></center>

<center><img src="https://i.loli.net/2021/04/29/j9IEoLSHYRNWhdw.png"/></center>

<center><img src="https://i.loli.net/2021/04/29/BORck4JQ7dbzi5X.png"/></center>

<center><img src="https://i.loli.net/2021/04/29/H5tBW4UKA9mSjEn.png"/></center>

<center><img src="https://i.loli.net/2021/04/29/mjhndtqJCEcgAxB.png"/></center>

<center><img src="https://i.loli.net/2021/04/29/RnN6FQlzbWVM4mY.png"/></center>

<center><img src="https://i.loli.net/2021/04/29/ImKkhawAvF2dLfy.png"/></center>

<center><img src="https://i.loli.net/2021/04/29/72i3Trw18FLyJVC.png"/></center>

<center><img src="https://i.loli.net/2021/04/29/Llg2yEesOQ8dNo4.png"/></center>

![image-20210429201641080](https://i.loli.net/2021/04/29/gBNYFqyJX3inrL9.png)

<center><img src="https://i.loli.net/2021/04/29/4tFg3cn7WO8HDEv.png"/></center>

<center><img src="https://i.loli.net/2021/04/29/Z6vd49nf3zAmpWV.png"/></center>

![image-20210429201730416](https://i.loli.net/2021/04/29/GYOBtUlPApfKk4i.png)

<center><img src="https://i.loli.net/2021/04/29/2HePo3yUM8v7aiV.png"/></center>

```java
import java.util.ArrayList;
import java.util.Deque;
import java.util.LinkedList;
import java.util.List;

public class CyclicDependency {
    List<Integer> haveCircularDependency(int n, int[][] prerequisites) {
        // 邻接表，存储图结构
        List<List<Integer>> g = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            g.add(new ArrayList<>());
        }
        // 每个点的入度
        int[] indeg = new int[n];
        // 结果序列
        List<Integer> res = new ArrayList<>();
        for (int i = 0; i < prerequisites.length; i++) {
            int a = prerequisites[i][0];
            int b = prerequisites[i][1]; // b 依赖于 a    a -> b
            g.get(a).add(b);
            indeg[b]++;
        }
        Deque<Integer> queue = new LinkedList<>();
        // 将入度为0的点全部入队
        for (int i = 0; i < n; i++) {
            if (indeg[i] == 0) {
                queue.addLast(i);
            }
        }
        while (!queue.isEmpty()) {
            int t = queue.removeFirst();
            res.add(t);
            // 删除边时，将终点的入度减一，若入度为0，则入队
            for (int i = 0; i < g.get(t).size(); i++) {
                int j = g.get(t).get(i);
                indeg[j]--;
                if (indeg[j] == 0) {
                    queue.addLast(j);
                }
            }
        }
        if (res.size() == n) {
            return res;
        }
        return new ArrayList<Integer>();
    }

    public static void main(String[] args) {
        CyclicDependency cyclicDependency = new CyclicDependency();
        int[][] pre1 = {{0,2}, {1,2}, {2,3}, {2,4}};
        int n = 5;
        List<Integer> list = cyclicDependency.haveCircularDependency(n, pre1);
        System.out.println(list);

        int[][] pre2 = {{0,1}, {1,2}, {2,1}};
        int m = 3;
        List<Integer> list1 = cyclicDependency.haveCircularDependency(m, pre2);
        System.out.println(list1);
    }
}
```

# LeetCode 207. 课程表

- https://mp.weixin.qq.com/s/7nP92FhCTpTKIAplj_xWpA
- 判断优有向图是否存在环

```java
// 建图
List<Integer>[] buildGraph(int numCourses, int[][] prerequisites) {
    // 图中共有 numCourses 个节点
    // 邻接表
    List<Integer>[] graph = new LinkedList[numCourses];
    for (int i = 0; i < numCourses; i++) {
        graph[i] = new LinkedList<>();
    }
    for (int[] edge : prerequisites) {
        int from = edge[1];
        int to = edge[0];
        // 修完课程 from 才能修课程 to
        // 在图中添加一条从 from 指向 to 的有向边
        graph[from].add(to);
    }
    return graph;
}
// 记录一次 traverse 递归经过的节点
boolean[] onPath;
// 记录遍历过的节点，防止重复遍历同一个节点
boolean[] visited;
// 记录图中是否有环
boolean hasCycle = false;

boolean canFinish(int numCourses, int[][] prerequisites) {
    List<Integer>[] graph = buildGraph(numCourses, prerequisites);

    visited = new boolean[numCourses];
    onPath = new boolean[numCourses];

    for (int i = 0; i < numCourses; i++) {
        // 遍历图中的所有节点
        traverse(graph, i);
    }
    // 只要没有循环依赖可以完成所有课程
    return !hasCycle;
}

void traverse(List<Integer>[] graph, int s) {
    if (onPath[s]) {
        // 出现环
        hasCycle = true;
    }

    if (visited[s] || hasCycle) {
        // 如果已经找到了环，也不用再遍历了
        return;
    }
    // 前序遍历代码位置
    visited[s] = true;
    onPath[s] = true;
    for (int t : graph[s]) {
        traverse(graph, t);
    }
    // 后序遍历代码位置
    onPath[s] = false;
}
```



# LeetCode 210. 课程表II

- 拓扑排序（Topological Sorting）

<center><img src="https://bkimg.cdn.bcebos.com/pic/cefc1e178a82b9010d8c4f1b708da9773912eff1?x-bce-process=image/watermark,image_d2F0ZXIvYmFpa2U4MA==,g_7,xp_5,yp_5/format,f_auto"/></center>

> **直观地说就是，让你把一幅图「拉平」，而且这个「拉平」的图里面，所有箭头方向都是一致的**，比如上图所有箭头都是朝右的
>
> 很显然，如果一幅有向图中存在环，是无法进行拓扑排序的，因为肯定做不到所有箭头方向一致；反过来，如果一幅图是「有向无环图」，那么一定可以进行拓扑排序。

给定一个包含 $n$ 个节点的有向图 $G$，我们给出它的节点编号的一种排列，如果满足：

> 对于图 $G$ 中的任意一条有向边 $(u, v)$，$u$在排列中都出现在 $v$的前面。

那么称该排列是图 $G$的「拓扑排序」。根据上述的定义，可以得出两个结论：

- 如果图 $G$ 中存在环（即图 $G$ 不是「有向无环图」），那么图 $G$ 不存在拓扑排序。
  - 这是因为假设图中存在环 $x_1, x_2, \cdots, x_n, x_1$，那么 $x_1$ 在排列中必须出现在 $x_n$ 的前面，但 $x_n$ 同时也必须出现在 $x_1$  的前面，因此不存在一个满足要求的排列，也就不存在拓扑排序；

- 如果图 $G$是有向无环图，那么它的拓扑排序可能不止一种.
  - 最极端的例子，如果图 $G$ 值包含 $n$个节点却没有任何边，那么任意一种编号的排列都可以作为拓扑排序

可以将本题建模成一个求拓扑排序的问题：

- 将每一门课看成一个节点；

- 如果想要学习课程 A 之前必须完成课程 B，那么从 B 到 A 连接一条有向边。这样，在拓扑排序中，B 一定出现在 A 的前面。 

求出该图的拓扑排序，就可以得到一种符合要求的课程学习顺序

- 方法一：BFS
  - 时间复杂度: $O(n+m)$，其中 $n$ 为课程数，$m$ 为先修课程的要求数。这其实就是对图进行广度优先搜索的时间复杂度。
  - 空间复杂度: $O(n+m)$。题目中是以列表形式给出的先修课程关系，为了对图进行广度优先搜索，我们需要存储成邻接表的形式，空间复杂度为 $O(n+m)$。在广度优先搜索的过程中，我们需要最多 $O(n)$ 的队列空间（迭代）进行广度优先搜索，并且还需要若干个 $O(n)$的空间存储节点入度、最终答案等


```java
class Solution {
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        List<List<Integer>> g = new ArrayList<>();   // 邻接表 
        for (int i = 0; i < numCourses; i++) {
            g.add(new ArrayList<>());
        }
        int[] indeg = new int[numCourses];           // 入度
        List<Integer> res = new ArrayList<>();

        for (int i = 0; i < prerequisites.length; i++) {
            int a = prerequisites[i][0];
            int b = prerequisites[i][1];   // a 依赖于 b   b -> a
            g.get(b).add(a);
            indeg[a]++;
        }

        Deque<Integer> queue = new LinkedList<>();
        // 所有入度为0的点入队
        for (int i = 0; i < numCourses; i++) {
            if (indeg[i] == 0) {
                queue.addLast(i);
            }
        }

        while (!queue.isEmpty()) {
            int t = queue.removeFirst();
            res.add(t);
            for (int i = 0; i < g.get(t).size(); i++) {
                int j = g.get(t).get(i);
                indeg[j]--;
                if (indeg[j] == 0) {
                    queue.addLast(j);
                }
            }
        }
        if (res.size() == numCourses) {
            int[] ans = new int[numCourses];
            int idx = 0;
            for (int i = 0; i < numCourses; i++) {
                ans[idx++] = res.get(i);
            }
            return ans;
        } 
        return new int[0];
    }
}
```

- 方法二：拓扑排序就是后序遍历反转之后的结果

> https://mp.weixin.qq.com/s/7nP92FhCTpTKIAplj_xWpA

> **将后序遍历的结果进行反转，就是拓扑排序的结果**
>
> 二叉树的后序遍历是什么时候？遍历完左右子树之后才会执行后序遍历位置的代码。换句话说，当左右子树的节点都被装到结果列表里面了，根节点才会被装进去。
>
> **后序遍历的这一特点很重要，之所以拓扑排序的基础是后序遍历，是因为一个任务必须在等到所有的依赖任务都完成之后才能开始开始执行**。
>
> 总之，你记住拓扑排序就是后序遍历反转之后的结果，且拓扑排序只能针对有向无环图，进行拓扑排序之前要进行环检测

```java
class Solution {
    boolean[] visited;
    boolean[] onPath;
    boolean hasCycle;
    List<Integer> postOrder;
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        visited = new boolean[numCourses];
        onPath = new boolean[numCourses];
        postOrder = new ArrayList<>();
        hasCycle = false;
        List<Integer>[] graph = buildGraph(numCourses, prerequisites);
        for (int i = 0; i < numCourses; i++) {
            traverse(graph, i);
        }
        if (hasCycle) {
            return new int[0];
        }

        // 将后序遍历结果反转，转化成 int[] 类型
        Collections.reverse(postOrder);
        int[] res = new int[numCourses];
        for (int i = 0; i < postOrder.size(); i++) {
            res[i] = postOrder.get(i);
        }
        return res;
    }

    public void traverse(List<Integer>[] graph, int s) {
        if (onPath[s]) {
            hasCycle = true;
            return ;
        }

        if (visited[s] || hasCycle) {
            return ;
        }

        visited[s] = true;
        onPath[s] = true;
        for (int t : graph[s]) {
            traverse(graph, t);
        }
        // 后序遍历位置
        onPath[s] = false;
        postOrder.add(s);
    }

    // 建图
    List<Integer>[] buildGraph(int numCourses, int[][] prerequisites) {
        List<Integer>[] graph = new LinkedList[numCourses];
        for (int i = 0; i < numCourses; i++) {
            graph[i] = new LinkedList<>();
        }
        for (int[] edge : prerequisites) {
            int from = edge[1];
            int to = edge[0];
            graph[from].add(to);
        }
        return graph;
    }
}
```

