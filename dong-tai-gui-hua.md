# 动态规划

### 7.1 背包问题【01背包】

背包问题需要解决的问题主要有

* 内外循环谁先谁后，是外物品，内背包；还是外背包，内物品；
* 内层循环正序还是逆序，为什么二维背包可以正序，一维背包就要逆序。

#### 7.1.1 LeetCode 1049: 最后一块石头的重量

1. 题目描述
2.  题目分析

    可以把题目转化为尽可能地分为两堆重量相同的石头，剩下的最小重量应为 【大石堆重量 - 小石堆重量】。 从而可以把问题转化为求小石堆或者大石堆的重量。求小石堆的重量即【求容量为石堆总重一半的背包能装下最大的最大石头重量】，这是一个01背包问题，每块石头可以选择装或者不装。
3.  代码实现

    3.1 一维背包

    ```java
    class Solution {
        public int lastStoneWeightII(int[] stones) {
            // 把stones分成最接近的两半，返回大的减小的
            int sum = 0;
            for (int stone : stones) {
                sum += stone;
            }

            // 背包最大容量为half
            int half = (sum >> 1);
            int[] dp = new int[half + 1];
            for(int stone : stones) {
                for(int i = half; i >= 0; i--) {  // 0 1背包一维情况倒序
                    if (i >= stone)
                        dp[i] = Math.max(dp[i], dp[i - stone] + stone);
                }
            }

            return  sum - 2 * dp[half];

        }
    }
    ```

    3.1.1 思考

    **a) 循环为什么要先石头，后背包？可以先背包后石头吗？**

    ​ **要**想回答循环是谁先谁后的问题，可以比较下面两图，发现只有先把石头放在外层循环，内层循环放背包。例如第一块石头为a，则在第一次把背包容量循环完后，所有能装下石头a的背包，值都将为a。这样在外层循环为b时，重新开始遍历背包的时候，若剩下的空间能装下b，则背包的总重量将会为a+b, 后一轮循环的值会加载前一轮循环上。 但假如是背包容量在外层，则每次遍历背包都只能装内部循环的石头重量，后一次的重量无法累加到前一次的重量上面。

    **b) 内部循环为什么要逆序，可以正序吗？**

    ​ **通**过比较下面的图，可以发现，因为一维背包会重复利用前面的dp数组的值，在正序遍历的时候，如果前面的数组已经装了一次石头a，后面某个数组会再加上一次a。

    ​ 如dp\[2] = 2;

    ​ **dp\[4] = max(dp\[4], dp\[4 - 2] + 2) = max(0, 2 + 2) = 4; 这代表把重量为2的石头再加了一次，因为dp\[2]已经装过一次重量为2的石头了。**

![IMG\_0395](https://tva1.sinaimg.cn/large/008i3skNly1guvgax81kqj60vb0u00vr02.jpg)

![IMG\_0396](https://tva1.sinaimg.cn/large/008i3skNly1guvgawj691j60u80u0ac102.jpg)

3.1.2 总结

​ a) 先物品、后背包（逆序）：**完美解决问题。因为先物品，使物品的重量可以在背包内累加；逆序使得同一个物品不会累加两次**

【因此一维01背包必须逆序】

![image-20210707115050361](https://tva1.sinaimg.cn/large/008i3skNly1gxi1sqc97mj31ae0h4753.jpg)

​ b) 先物品、后背包（正序）：**结果偏大**。因为先物品使重量可以累加，但正序导致的向后依赖，使同一个物品会被累加两次；【完全背包内部才正序】

![image-20210707114949467](https://tva1.sinaimg.cn/large/008i3skNly1gxi1sn37rbj31aq0k03zo.jpg)

​ c）先背包（正序）、后物品：**结果偏大**。先背包使得每次只能入一个物品，但正序的向后依赖，使同一个物品还是会被累加两次；

![image-20210707115556675](https://tva1.sinaimg.cn/large/008i3skNly1gxi1siv3udj31a80ia3zd.jpg)

​ d) 先背包（逆序）、后物品: **结果偏小**。先背包使得每次只能入一个物品，逆序又不能累加前一轮的，使每轮都只能取得最大的那个石头的重量。

![image-20210707115325912](https://tva1.sinaimg.cn/large/008i3skNly1gxi1shl277j31aa0icjs9.jpg)

### 7.1 背包问题【完全背包】

#### LeetCode 零钱兑换2

You are given an integer array coins representing coins of different denominations and an integer amount representing a total amount of money.

Return the number of combinations that make up that amount. If that amount of money cannot be made up by any combination of the coins, return 0.

You may assume that you have an infinite number of each kind of coin.

The answer is guaranteed to fit into a signed 32-bit integer.

```bash
Example 1:
Input: amount = 5, coins = [1,2,5]
Output: 4
Explanation: there are four ways to make up the amount:
5=5
5=2+2+1
5=2+1+1+1
5=1+1+1+1+1
```

#### 分析

每个数字可以被用无数次，因此这是一个完全背包问题。完全背包注意，先物品，后背包，均为正序。

定义：dp\[i], 兑换i，可能的组合数。

初始化：dp\[0]， 兑换0元，默认一种

状态转移方程：

​ 对于coins = {coin1, coin2, coin3}

​ dp\[i] = dp\[i - coin1] + dp\[i - coin2] + dp\[i - coin3]

​ 因此 dp\[i] (新) = dp\[i] (旧) + dp\[i - coin]

```java
class Solution {
    public int change(int amount, int[] coins) {

        // dp[i]: i元钱可以被兑换的种类

        int[] dp = new int[amount + 1];
        dp[0] = 1;

        for(int coin : coins) {
            for (int i = coin; i <= amount; i++) {
                dp[i] += dp[i - coin]; 
            }
        }

        return dp[amount];

    }
}
```

### 7.2 数组

#### LeetCode300: 最长上升子序列

```
Given an integer array nums, return the length of the longest strictly increasing subsequence.

A subsequence is a sequence that can be derived from an array by deleting some or no elements without changing the order of the remaining elements. For example, [3,6,2,7] is a subsequence of the array [0,3,1,6,2,2,7].

 

Example 1:

Input: nums = [10,9,2,5,3,7,101,18]
Output: 4
Explanation: The longest increasing subsequence is [2,3,7,101], therefore the length is 4.
Example 2:

Input: nums = [0,1,0,3,2,3]
Output: 4
```

**分析**

新的数加入，会扰动原有的结果，而且扰动不仅只依赖最后加入的这个数

例如【2，3，1，2，3】。

【2】------> ans = 1 【】

【2，3】------> ans = 2 【2，3】

【2, 3，1】------> ans = 2 【2，3】

\+++++++++++++++++这里就开始棘手了++++++++++++++++++++++++

【2，3，1，2】------> ans = 3 【1，2】

【2，3，1，2，3】------> ans = 3 【1，2，3】

用dp\[i] 表示，以nums\[i]结尾的最长上升子序列长度， 则针对上面例子

【2】------> ans = 1 【2】 dp\[0] = 1 【2】

【2，3】------> ans = 2 【2，3】 dp\[1] = 2 【2，3】

【2, 3，1】------> ans = 2 【2，3】 dp\[2] = 1 【1】 注意这里

\+++++++++++++++++这里就开始棘手了++++++++++++++++++++++++

【2，3，1，2】------> ans = 3 【1，2】 dp\[3] = 2 【1，2】

【2，3，1，2，3】------> ans = 3 【1，2，3】 dp\[4] = 3 【1，2，3】

因此可以用 j in \[0，i) 这段长度内的结果dp\[j] 和 当前dp\[i] 进行比较。

如果nums\[i] > nums\[j] 则 dp\[i] = Math.max(dp\[i], dp\[j] + 1);

注意要用res维护住最大的LIS

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        // dp[i] 的值代表 nums 以 nums[i] 结尾的最长子序列长度

        int n = nums.length;
        int[] dp = new int[n];
        Arrays.fill(dp, 1);
        int res = 1;

        for(int i = 0; i < n; i++) {
            for(int j = 0; j < i; j++) {
                if(nums[i] > nums[j]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                } 
            }
            res = Math.max(res, dp[i]);
        }

        return res;

    }
}
```

#### LeetCode473 最长上升子序列的数量

**分析**

基于上一题LIS, 只需要新加一个combinations数组，更新每次找到的子序列数量。

* 当dp\[j] + 1 > dp\[i], 说明最长长度要被更新，因此 combinations\[i] = combinations\[j];
*   当dp\[j] + 1 = dp\[i], 说明dp\[j] 的长度加 nums\[i] 可达dp\[i], 为方案一种，因此 combinations\[i] += combinations\[j];

    意思是，对于【1，6，5】+【7】 好几种dp\[j]都可以到达dp\[i], 因此需要把这些方案数加起来

```java
class Solution {
    public int findNumberOfLIS(int[] nums) {
        // dp[i] 以nums[i] 结尾的LIS个数

        int n = nums.length;
        int[] dp = new int[n];
        int[] combinations = new int[n];
        Arrays.fill(dp, 1);
        Arrays.fill(combinations, 1);

        int ans = 0, res = 1;

        for(int i = 0; i < n; i++) {
            for(int j = 0; j < i; j++) {
                if(nums[i] > nums[j]) {
                    if(dp[j] + 1 > dp[i]) { //找到更长的，被更新
                        dp[i] = dp[j] + 1;
                        combinations[i] = combinations[j]; // 长度更新为dp[j]对应长度
                    } else if(dp[j] + 1 == dp[i]) { 
                        combinations[i] += combinations[j]; // 找到一种方案，加进来
                    }
                }            
            }
            res = Math.max(res, dp[i]);
        }

      
        for(int i = 0; i < n; i++) {
            if(dp[i] == res) ans += combinations[i];
        }

        return ans;
    }
}
```

#### LeetCode152 乘积最大子数组

1. [Maximum Product Subarray](https://leetcode-cn.com/problems/maximum-product-subarray/) 难度中等

Given an integer array `nums`, find a contiguous non-empty subarray within the array that has the largest product, and return _the product_.

The test cases are generated so that the answer will fit in a **32-bit** integer.

A **subarray** is a contiguous subsequence of the array.

**分析**

此类动态规划，每个状态都是基于第i个元素的而存在。

dp\[i]: 以第i个元素结尾的最大乘积子数组的最大乘积，也就是说，这个乘积必定基于，nums\[i]。

由于nums\[i] 之前是正是负未知，因此

max\[i] = max{ max\[i - 1] \* nums\[i], min\[i - 1] \* nums\[i], nums\[i] }

min\[i] = min{ max\[i - 1] \* nums\[i], min\[i - 1] \* nums\[i], nums\[i] }

【踩坑】 注意初始化，不要因为for循环从1就开始，导致遗漏了结果就在0的情况。

**Example 1:**

```
Input: nums = [2,3,-2,4]
Output: 6
Explanation: [2,3] has the largest product 6.
```

**Example 2:**

```
Input: nums = [-2,0,-1]
Output: 0
Explanation: The result cannot be 2, because [-2,-1] is not a subarray.
```

```java
class Solution {
    public int maxProduct(int[] nums) {
        

        int n = nums.length;

        if(n < 2) return nums[0];

        int[] max = new int[n];
        int[] min = new int[n];
        max[0] = nums[0]; min[0] = nums[0];
        int ans = max[0];

        for(int i = 1; i < n; i++) {
            max[i] = Math.max(max[i - 1] * nums[i], Math.max(min[i - 1] * nums[i], nums[i]));
            min[i] = Math.min(max[i - 1] * nums[i], Math.min(min[i - 1] * nums[i], nums[i]));
            
            ans = Math.max(ans, max[i]);
        }

        return ans;

    }
}
```

**优化**

```java
class Solution {
    public int maxProduct(int[] nums) {
        int max = Integer.MIN_VALUE, imax = 1, imin = 1;
        for(int num : nums) {
            if(num < 0) {int tmp = imax; imax = imin; imin = tmp;}
            imax = Math.max(imax * num, num);
            imin = Math.min(imin * num, num);

            max = Math.max(max, imax);
        }

        return max;

    }
}
```

#### LeetCode918 环形子数组的最大和

Given a circular integer array nums of length n, return the maximum possible sum of a non-empty subarray of nums.

A circular array means the end of the array connects to the beginning of the array. Formally, the next element of nums\[i] is nums\[(i + 1) % n] and the previous element of nums\[i] is nums\[(i - 1 + n) % n].

A subarray may only include each element of the fixed buffer nums at most once. Formally, for a subarray nums\[i], nums\[i + 1], ..., nums\[j], there does not exist i <= k1, k2 <= j with k1 % n == k2 % n.

```
Example 1:
Input: nums = [1,-2,3,-2]
Output: 3
Explanation: Subarray [3] has maximum sum 3.

Example 2:
Input: nums = [5,-3,5]
Output: 10
Explanation: Subarray [5,5] has maximum sum 5 + 5 = 10.
```

**分析**

当答案不需要越界就可以找到时，就是最大子序和；当答案需要越界才能找到时，答案转化为 【数组元素之和 - 最小子序和】

不确定答案是在越界找到还是不在越界找到：所以答案为 max{ sum - min, max}

注意，当max 都 小于0时， sum - min = 0, 答案应该返回 max;

**代码**

```java
class Solution {
    public int maxSubarraySumCircular(int[] nums) {
        if(nums == null || nums.length < 1) return 0;
    
        int n = nums.length;
        int curMax, max, curMin, min, sum;
        curMax = max = curMin = min = sum = nums[0];
        
        for(int i = 1; i < n; i++) {
            curMax = Math.max(nums[i], curMax + nums[i]);
            curMin = Math.min(nums[i], curMin + nums[i]);
            max = Math.max(max, curMax);
            min = Math.min(min, curMin);
            sum += nums[i];
        }
        
        return max > 0 ? Math.max(sum - min, max) : max; 
    }

}
```

#### LeetCode 子矩阵最大和

#### 解题思路

和最长子序和一样，都是求以当前元素 matrix\[ i ]\[ j ]结尾的最大和，想要求某子矩阵的和，根据经验用【前缀和】矩阵，是比较方便的。 再通过限制住矩阵的边界，不断寻找最大子序和，用global维护全局的最大值。具体过程如下图：

![245841639904580\_.pic](https://tva1.sinaimg.cn/large/008i3skNly1gxjbnr3uqlj30xt0u0diy.jpg) ![245901639905438\_.pic\_hd](https://tva1.sinaimg.cn/large/008i3skNly1gxjbongzsfj312m0u078n.jpg) ![245871639904867\_.pic](https://tva1.sinaimg.cn/large/008i3skNly1gxjbq2xb5lj31420u0adx.jpg) ![245881639904947\_.pic](https://tva1.sinaimg.cn/large/008i3skNly1gxjbsianb5j31560u00wu.jpg) ![245891639904995\_.pic](https://tva1.sinaimg.cn/large/008i3skNly1gxjbsj3bkaj31380u00wm.jpg)

【坑】：前缀和计算时，记得perfix的长度比数组大1，可以优雅的初始化。

**代码**

```java
class Solution {
    public int[] getMaxMatrix(int[][] matrix) {
        // 求前缀和 prefixSum

        int m = matrix.length, n = matrix[0].length;
        int[][] prefix = new int[m + 1][n + 1];

        for(int i = 1; i < m + 1; i++) {
            for(int j = 1; j < n + 1; j++) {
                prefix[i][j] = matrix[i - 1][j - 1]
                             + prefix[i][j - 1] 
                             + prefix[i - 1][j]
                             - prefix[i - 1][j - 1];
            }
        }

        int[] ans = new int[4];

        // 固定边界，求最大子序和
        int global = Integer.MIN_VALUE;

        for(int top = 0; top < m; top++) {
            for(int bottom = top; bottom < m; bottom++) {
                int left = 0;
                for(int right = 0; right < n; right++) {
                    int newSum = prefix[bottom + 1][right + 1] 
                            - prefix[bottom + 1][left] 
                            - prefix[top][right + 1]
                            + prefix[top][left];

                    if(newSum > global) {
                        global = newSum;
                        ans[0] = top;
                        ans[1] = left;
                        ans[2] = bottom;
                        ans[3] = right;
                    }

                    if(newSum < 0) {
                        left = right + 1;
                    }
                }
            }
        }

        return ans;
    }
}
```

#### LeetCode 最短路径和

```
Given a m x n grid filled with non-negative numbers, find a path from top left to bottom right, which minimizes the sum of all numbers along its path.

只能向下和向右走
```

![image-20211221161819928](https://tva1.sinaimg.cn/large/008i3skNly1gxlj1l4hz7j31960p2ta1.jpg)

```java
public int minPathSum(int[][] grid) {
        // dp[i][j] 只可能是左、右一个
        int m = grid.length, n = grid[0].length;
        // 初始化，第一行和第一列只可能是前缀和
        for(int i = 1; i < m; i++) grid[i][0] += grid[i - 1][0];
        for(int i = 1; i < n; i++) grid[0][i] += grid[0][i - 1];
        for(int i = 1; i < m; i++) {
            for(int j = 1; j < n; j++) {
                grid[i][j] += Math.min(grid[i - 1][j], grid[i][j - 1]);
            }
        }

        return grid[m - 1][n - 1];

    }
```

#### LeetCode 编辑距离

Given two strings word1 and word2, return the minimum number of operations required to convert word1 to word2.

You have the following three operations permitted on a word:

Insert a character Delete a character Replace a character

Example 1:

Input: word1 = "horse", word2 = "ros" Output: 3 Explanation: horse -> rorse (replace 'h' with 'r') rorse -> rose (remove 'r') rose -> ros (remove 'e')

来源：力扣（LeetCode） 链接：https://leetcode-cn.com/problems/edit-distance

**分析**

怎么都没想到要用动态规划来做

```
最后一步：增！
word1.substring(0, i) ---> word2.substring(0, j - 1), 最后增一个word2.charAt(j);

最后一步：删！
word1.substring(0, i - 1) ---> word2.substring(0, j), 最后删除word1.charAt(i);

最后一步：换！
word1.substring(0, i - 1) ---> word2.substring(0, j - 1), 最后word1.charAt(i)换word1.charAt(j);
```

定义： dp【i】【j】 : 将 word1.substring(0, i) ---> word2.substring(0, j)需要的步数

\*\* 为了dp\[i - 1]能优雅的初始化，则需要让dp长度为【m + 1】【n + 1】\*\*

```java
class Solution {
    // dp[i][j]: 将  word1.substring(0, i) ---> word2.substring(0, j)需要的步数
       
    public int minDistance(String word1, String word2) {
        int m = word1.length(), n = word2.length();

        int[][] dp = new int[m + 1][n + 1];

        for(int i = 0; i <= m; i++) dp[i][0] = i; 
        for(int i = 0; i <= n; i++) dp[0][i] = i;
        for(int i = 1; i <= m; i++) {
            for(int j = 1; j <= n; j++) {
                int add = dp[i][j - 1], del = dp[i - 1][j], rep = dp[i - 1][j - 1];
                if(word1.charAt(i - 1) == word2.charAt(j - 1)) {
                    dp[i][j] = rep;
                } else { 
                    dp[i][j] = 1 + Math.min(rep, Math.min(add, del)); 
                }
            }
        } 

        return dp[m][n];
        
    }
}
```

##
