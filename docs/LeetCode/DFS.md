<!-- GFM-TOC -->

- [LeetCode 22. 括号生成](#LeetCode-22-括号生成)
- [LeetCode 129. 求根节点到叶子节点数字之和](#LeetCode-129-求根节点到叶子节点数字之和)

<!-- GFM-TOC -->

> DFS 算法就是回溯算法

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




