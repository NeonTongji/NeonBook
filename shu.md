# 树

### LeetCode98: 验证二叉搜索树

#### 1[题目描述](https://leetcode-cn.com/problems/validate-binary-search-tree/)

给定一个二叉树，判断其是否是一个有效的二叉搜索树。

假设一个二叉搜索树具有如下特征：

* 节点的左子树只包含小于当前节点的数。
* 节点的右子树只包含大于当前节点的数。
* 所有左子树和右子树自身必须也是二叉搜索树。

```java
输入:
    2
   / \
  1   3
输出: true
```

#### 2 题目分析

树肯定是递归做（基础好用迭代）。~~什么是快乐星球？~~ 什么是二叉搜索树BST?

* null 是吗？ 是
* ~~\[1] 是吗？ 是~~ 不能用单节点来判断是因为 root.left和root.right会空指针异常！！！！
* root.left要是BST, root.right也要是BST

为了避免产生空指针异常，需要引入比最左叶子节点还要小的值，**Long.MIN\_VALUE**。还要引入比最右叶子结点还要大的值，**Long.MAX\_VALUE**。

***

#### 3. 代码实现

方法一：递归深搜

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
    public boolean isValidBST(TreeNode root) {
        long lower = Long.MIN_VALUE;
        long upper = Long.MAX_VALUE;
        return isValidBST(root, lower, upper);
    }

    boolean isValidBST(TreeNode root, long lower, long upper) {
				
      	/* Base case为什么是 root == null, 而不是 root.left == null && root.right == null 
      	* 因为这样会产生空指针异常
        */
      
        if(root == null) 
            return true;
				
				// 引入了lower和upper相当于引入了两个哨兵节点
        if(root.val <= lower || root.val >= upper ) {
            return false;
        }
		
				/* 对于左子树来说，最小边界仍为lower，即Long.MIN_VALUE, 而最大边界要更新为当前子树的根节点的值。
        * 对于右子树来说，最大边界仍为upper，即Long.MAX_VALUE, 而最小边界要更新为当前子树的根节点的值。
        */
        return isValidBST(root.left, lower, root.val) && isValidBST(root.right, root.val, upper);
    }

}
```

***

方法二：中序遍历

对于一颗BST，中序遍历数组一定是一个升序数组，则我们只需要维护前一个节点值，与后一个节点值比较。当遍历完整个树，如果一直是升序的，便返回true，中途一旦出现降序或相等，便返回false。

中序遍历的流程为：

1. 新建一个栈，将左子节点一次压入栈中，直到到达最左叶子节点为止；
2. 让元素依次出栈比较。首先是左子节点出栈和根节点比较，接着把根节点迭代为右子节点，对右子树中序遍历。

```java
class Solution {
    public boolean isValidBST(TreeNode root) {
      // 设置哨兵节点prev为Long.MIN_VALUE。因为中序遍历是从最小的节点值开始的，我们便可以前面加个虚的prev节点，令其值最小。
        return inorderTraversal(root, Long.MIN_VALUE);
    }

    boolean inorderTraversal(TreeNode root, long prev) {
        Stack<TreeNode> stack = new Stack<>();

        while(root != null || !stack.isEmpty()) {
          // 左子节点依次入栈
            while (root != null) {
                stack.push(root);
                root = root.left;
            }
    
            root = stack.pop();
          // 验证是否升序，不是升序则返回false
            if (root.val <= prev)
                return false;
          // 前一个节点更新为当前节点值
            prev = root.val;
          // 去右子树继续验证
            root = root.right;
        }
        return true;
    }

}
```

由于中序遍历是【左子树--->中--->右子树】这样的顺序遍历，因此只需要维护住前一个节点prev与当前节点值root.val进行比较。

### LeetCode144: 二叉树前序遍历

#### 方法一：递归

***

```java
```

### LeetCode589: N叉树前序遍历

### LeetCode94: 二叉树中序遍历

#### 方法一：递归

```java
private void inorder(TreeNode root, List<TreeNode> list) {
         
        if(root == null) return ;

        inorder(root.left, list);
        list.add(root);
        inorder(root.right, list);
 }
```

### LeetCode589: N叉树中序遍历

### LeetCode145: 二叉树后序遍历

### LeetCode590: N叉树后序遍历

### LeetCode102: 二叉树层序遍历

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {

        Queue<TreeNode> q = new ArrayDeque<>();
        if(root != null) q.add(root); //先将根节点入栈
        List<List<Integer>> lists = new ArrayList<>();

        while(!q.isEmpty()) {
            int n = q.size();
            List<Integer> level = new ArrayList<>();
            for(int i = 0; i < n; i++) { // 这一层节点数
                    TreeNode node = q.poll(); // 清掉这层的节点
                    level.add(node.val);   // 把这层的节点全部加进来
                    if(node.left != null) { 
                        q.add(node.left); // 下一层的左子节点入栈
                    }
                    if(node.right != null) { 
                        q.add(node.right); // 下一层的右子节点入栈
                    }
            }
            lists.add(level);
        }

        return lists;
    }
}
```

### LeetCode96 可能的二叉搜索树个数

Given an integer n, return the number of structurally unique BST's (binary search trees) which has exactly n nodes of unique values from 1 to n.

**Example 1:**

![img](https://assets.leetcode.com/uploads/2021/01/18/uniquebstn3.jpg)

```
Input: n = 3
Output: 5
```

**Example 2:**

```
Input: n = 1
Output: 1
```

我是猪

G(n): n个递增自然数可以组成的二叉树个数 【1，2，3，4，... , root, ... , n】

当 root= 1，可能性 G(0) \* G(n - 1);

当 root= 2，可能性 G(1) \* G(n - 2);

当 root= 3，可能性 G(2) \* G(n - 3);

……

当 root= n，可能性 G(n - 1) \* G(0);

将所有的可能性加起来：

G(n) = G(0) \* G(n - 1) + G(1) \* G(n - 2) + ... + G(n - 1) \* G(0);

```java
class Solution {

    public int numTrees(int n) {
        int[] G = new int[n + 1];
        G[0] = 1;
        G[1] = 1;

        for(int i = 2; i <= n; i ++) {
            for(int j = 0; j < i; j++) {
                G[i] += G[j] * G[i - j - 1];
            }
        }

        return G[n];
    }
}
```

LeetCode

##
