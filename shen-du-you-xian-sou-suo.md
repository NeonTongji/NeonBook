# 深度优先搜索

### LeetCode46 全排列

Given an array nums of **distinct** integers, return all the possible permutations. You can return the answer in any order.

```
Example 1:
Input: nums = [1,2,3]
Output: [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

```java
class Solution {
    List<List<Integer>> ans = new ArrayList<>();

    public List<List<Integer>> permute(int[] nums) {

        dfs(nums, 0, new ArrayList<>());

        return ans;
    }

    private  void dfs(int[] nums, int index, List<Integer> item) {
        if(nums == null || nums.length == 0) return;

        int n = nums.length;

        if(index == n) {  //遍历一次数组，让每个数能做一次第一个数字，当遍历完成，返回
            ans.add(new ArrayList<Integer>(item));
            return;
        }

        for(int i = 0; i < n; i++) {
            if(!item.contains(nums[i])) { //数组中的数字是独一无二的，只需要让同一个数不加到item两次即可
                item.add(nums[i]);
                dfs(nums, index + 1, item);
                item.remove(item.size() - 1); //list删除用remove(int index) 
            }

        }
    }

```

### LeetCode47： 全排列2

给定一个可包含重复数字的序列 `nums` ，**按任意顺序** 返回所有不重复的全排列。

```
输入：nums = [1,1,2]
输出：
[[1,1,2], [1,2,1],[2,1,1]]
```

#### 分析

和不包含重复元素的全排列相比，包含重复元素的全排列需要标记，是否使用过。不包含重复元素的需要知道是否已经添加进去了。

```java
class Solution {
    List<List<Integer>> ans = new ArrayList<>();
    public List<List<Integer>> permuteUnique(int[] nums) {
        boolean[] used = new boolean[nums.length]; //每次这里都敲错，注意一下
        Arrays.sort(nums); // 排序后容易对重复元素处理
        dfs(nums, 0, new ArrayList<>(), used);
        return ans;
    }

    private void dfs(int[] nums, int index, List<Integer> item, boolean[] used) {
        if(nums == null || nums.length == 0) return;

        int n = nums.length;
        if(index == n) {
          // item只是一个引用，需要重新开辟一块空间存入item的值，避免item后续过程引用改变导致错误
                ans.add(new ArrayList<Integer>(item)); //注意加入的时候一定要new一下	
            return;
        }

        for(int i = 0; i < nums.length; i++) { 
            if(i > 0 && nums[i] == nums[i - 1] && used[i - 1]) continue;
            if(!used[i])  {
                used[i] = true;
                item.add(nums[i]);
                dfs(nums, index + 1, item, used);
                item.remove(item.size() - 1);
                used[i] = false;
            }
        }
    }
}
```

\## 拓展

大家发现，去重最为关键的代码为：

```
if (i > 0 && nums[i] == nums[i - 1] && used[i - 1] == false) {
    continue;
}
```

**如果改成 `used[i - 1] == true`， 也是正确的!**，去重代码如下：

```
if (i > 0 && nums[i] == nums[i - 1] && used[i - 1] == true) {
    continue;
}
```

这是为什么呢，就是上面我刚说的，如果要对树层中前一位去重，就用`used[i - 1] == false`，如果要对树枝前一位去重用`used[i - 1] == true`。

**对于排列问题，树层上去重和树枝上去重，都是可以的，但是树层上去重效率更高！**

这么说是不是有点抽象？

来来来，我就用输入: \[1,1,1] 来举一个例子。

树层上去重(used\[i - 1] == false)，的树形结构如下：

![47.全排列II2](https://pic.leetcode-cn.com/1631608102-aDQDkD-file\_1631608102194)

树枝上去重（used\[i - 1] == true）的树型结构如下：

![47.全排列II3](https://pic.leetcode-cn.com/1631608102-rbEqiu-file\_1631608101900)

大家应该很清晰的看到，树层上对前一位去重非常彻底，效率很高，树枝上对前一位去重虽然最后可以得到答案，但是做了很多无用搜索。

\## 总结

这道题其实还是用了我们之前讲过的去重思路，但有意思的是，去重的代码中，这么写：

```
if (i > 0 && nums[i] == nums[i - 1] && used[i - 1] == false) {
    continue;
}
```

和这么写：

```
if (i > 0 && nums[i] == nums[i - 1] && used[i - 1] == true) {
    continue;
}
```

都是可以的，这也是很多同学做这道题目困惑的地方，知道`used[i - 1] == false`也行而`used[i - 1] == true`也行，但是就想不明白为啥。

所以我通过举\[1,1,1]的例子，把这两个去重的逻辑分别抽象成树形结构，大家可以一目了然：为什么两种写法都可以以及哪一种效率更高！

### LeetCode 93 : 复原IP地址

#### 题目描述

给定一个只包含数字的字符串，用以表示一个 IP 地址，返回所有可能从 s 获得的 有效 IP 地址 。你可以按任何顺序返回答案。

有效 IP 地址 正好由四个整数（每个整数位于 0 到 255 之间组成，且不能含有前导 0），整数之间用 '.' 分隔。

例如："0.1.2.201" 和 "192.168.1.1" 是 有效 IP 地址，但是 "0.011.255.245"、"192.168.1.312" 和 "192.168@1.1" 是 无效 IP 地址。

来源：力扣（LeetCode） 链接：https://leetcode-cn.com/problems/restore-ip-addresses 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

***

#### 题目分析

这种有多少种可能的题目会需要dfs并回溯，回溯是指当走到尽头时，发现这条路走不通，便回到上个路口，尝试走另外一条路，如果再发现该节点所有路都走不通，或者是所有路都尝试过了，便再回到上个节点。

用二叉树来描述便是，若走到了叶子节点，发现没有子节点了，便回溯到上个父节点，再从这个父节点往另一个叶子节点搜索，若另外一个叶子节点不存在，或是另一个叶子节点已经被搜索过了，则再回到上个父节点，以此类推，直到遍历完整颗树为止。

***

#### dfs框架

```java
public void dfs() {
// 截止条件, 注意下文的dfs入参继续搜索条件与这里的返回条件相同时，便会触发返回。
	if(返回条件) {
		// 打印结果，或者是把结果挨个记录到输出数组中
		return;
	}

// 选择条件
	for() {
		// 操作数据
		dfs(继续搜索条件)
		// 还原被操作的数据
	}
}
```

#### 代码描述

```java
class Solution {
    List<String> list = new ArrayList<>();
    Stack<String> stack = new Stack<>();

    public List<String> restoreIpAddresses(String s) {
        dfs(s, 0);
        return list;
    }

    public void dfs(String s, int index) {
        // 截止条件
        if(index == s.length()|| stack.size() == 4) {
            if(index == s.length() && stack.size() == 4) {
                list.add(s.join(".", stack));
            }
            return;
        }

        // 选择条件
        for(int i = 1; i <= 3; i++) {
            if(index + i > s.length()) {
                break;
            } 
            String part = s.substring(index, index + i);
            if(Integer.parseInt(part) <= 255 && (part.equals("0") || part.charAt(0) != '0')) {
                stack.push(part);
                dfs(s, index + i);
                stack.pop();
            }
        }

    }
}
```

时间复杂度: O(logN);

***

### LeetCode337: 打家劫舍III

#### 题目描述

在上次打劫完一条街道之后和一圈房屋后，小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为“根”。 除了“根”之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果两个直接相连的房子在同一天晚上被打劫，房屋将自动报警。

计算在不触动警报的情况下，小偷一晚能够盗取的最高金额。

来源：力扣（LeetCode） 链接：https://leetcode-cn.com/problems/house-robber-iii 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

#### 题目分析

用dfs算出只从根节点开始抢的，和不从根节点开始抢的，比较大小即可。

```java
Max(dfs(root), dfs(root.left) + dfs(root.right))
```

注意DFS实质为递归，时间复杂度为递归的时间复杂度和递归的次数，上面的描述中递归次数太多了，容易导致超时，我们便可以把尝试过的节点缓存起来。因为如果抢根节点，则需要把孙子节点的最大返回值计算出来，当孙子节点为根节点时，这个值又需要再被计算一次，所以我们便可以把每个节点为根节点时，能抢到的最大钱存起来，就不需要再重复递归下去。建立一个HashMap\<TreeNode, Integer> memo, 键值对entity里面存放的是map\[当前节点：当前节点为根节点能抢到的最大钱数]。

***

#### DFS带备忘录框架

```java
public void dfs() {
// 截止条件, 注意下文的dfs入参继续搜索条件与这里的返回条件相同时，便会触发返回。
	if(返回条件) {
		// 打印结果，或者是把结果挨个记录到输出数组中
		return;
	}
	if(检查备忘录){
		输出备忘录的值
		return;
	}

// 选择条件
	for() {
		code1 // 操作数据
		dfs(继续搜索条件)
		put // 把上一轮dfs的结果存到备忘录里面
		code2 // 还原被操作的数据
	}
}
```

***

#### 代码实现

```java
class Solution {
    public int rob(TreeNode root) {
       return dfs(root, new HashMap<>());
    }

    public int dfs(TreeNode root, HashMap<TreeNode, Integer> memo) {
        // 1 截止条件
        // 1.1 为叶子节点，返回0
        if(root == null) {
            return 0;
        }
        // 1.2 为缓存节点，返回以该节点为根节点能偷到的钱
        if(memo.containsKey(root)) {
            return memo.get(root);
        }
		
		// 选择偷根节点
        int money = root.val;

        // 2 选择条件
        // 2.1 左子树不为空
        if(root.left != null) {
            money += dfs(root.left.left, memo) + dfs(root.left.right, memo);
        }
        // 2.2 右子树不为空
        if(root.right != null) {
            money += dfs(root.right.right, memo) + dfs(root.right.left, memo);
        }
        money = Math.max(money, dfs(root.right, memo) + dfs(root.left, memo));
        // 注意这里要存进去比较后的更大money
        memo.put(root, money);
        return money;
    }

}
```

### LeetCode 377: 组合总数I

Given an array of distinct integers candidates and a target integer target, return a list of all unique combinations of candidates where the chosen numbers sum to target. You may return the combinations in any order.

The same number may be chosen from candidates an unlimited number of times. Two combinations are unique if the frequency of at least one of the chosen numbers is different.

It is guaranteed that the number of unique combinations that sum up to target is less than 150 combinations for the given input.

```
Example 1:
Input: candidates = [2,3,6,7], target = 7
Output: [[2,2,3],[7]]
Explanation:
2 and 3 are candidates, and 2 + 2 + 3 = 7. Note that 2 can be used multiple times.
7 is a candidate, and 7 = 7.
These are the only two combinations.
```

#### 分析

注意截止条件：如果sum > target 返回，sum == target 加入后返回；

注意去重，先排序后回溯可以减少回溯次数，存放上次开始的索引，就不会每次都从0开始匹配，导致重复。ha

```java
class Solution {
    List<List<Integer>> ans = new ArrayList<>();
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        Arrays.sort(candidates);
        dfs(candidates, target, 0, new ArrayList(), 0);
        return ans;
    }

    private void dfs(int[] candidates, int target, int sum, List<Integer> item, int start) {
        if(candidates == null || candidates.length == 0) return;

        int n = candidates.length;
  
        if(sum == target) {
            ans.add(new ArrayList<Integer>(item));
            return;
        }

        if(sum > target) return; 

        for(int i = start; i < n && candidates[i] <= target ; i++) {
            item.add(candidates[i]);
            sum += candidates[i];
          // 把当前用完后的i，传到下次，避免重复计算，例如 2 3 3 = 8，下次就是 3 5 = 8， 3 不会再回去找2 
            dfs(candidates, target, sum, item, i);
            item.remove(item.size() - 1);
            sum -= candidates[i];
        }
    }
}
```

### LeetCode 377: 组合总数II

### LeetCode 377: 组合总数III

### LeetCode 377: 组合总数IV

#### 1. 题目描述

给你一个由 不同 整数组成的数组 nums ，和一个目标整数 target 。请你从 nums 中找出并返回总和为 target 的元素组合的个数。

题目数据保证答案符合 32 位整数范围。

来源：力扣（LeetCode） 链接：https://leetcode-cn.com/problems/combination-sum-iv 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

***

#### 2. 题目分析

**2.1 方法一：递归**

涉及到数组及其子集可以用递归来做，即包含或者不包含某元素。例如凑成target的方法种数为所有凑成target - num（num为nums中任意一元素）之和，即爬楼梯问题。如nums = \[1,2,3,4], 凑成target为差target - 1, target - 2, target -3, target -4的所有方法之和。由于是让target不断减小，则递归基应该是target有可能取得最小值，即target = 0。 又因为target最小就是0，则实参target - num > = 00。

***

**代码实现**

```java
class Solution {
    public int combinationSum4(int[] nums, int target) {
        int ans = 0;

        if(target == 0) {
            return 1;
        }

        for(int num : nums) {
            if(target >= num) {
                ans += combinationSum4(nums, target - num);
            }
        }
        return ans
    }
}
```

**2.2 方法二: 带备忘录的DFS**

```java
class Solution {
    /* 常见问题：
    * 1. memo放在何处: 为全局变量dfs才能识别，memo可用数组和HashMap，用数组则索引充当key；
    * 2. memo存放何值: 存放返回值，准确说是当前节点的返回值；
    * 3. dfs返回类型为什么比较好: 本题的返回值要做累加运算，dfs必须有个返回值；
    * 4. int ans放在 dfs()内，放进去也可以，dfs并不依赖于ans，则每次递归都应该是dfs(,,0),而不是dfs(,,ans);
    * 5. 为了解决4中的问题，可以直接把ans,在函数中声明。
    * 6. dfs回溯 有”回“就一定会有截止条件，只是有的可以把截止条件在初始化备忘录的时候就存放好。
    */
    
    int[] memo;
    public int combinationSum4(int[] nums, int target) {
        memo = new int[target + 1];
        Arrays.fill(memo, -1);
        // memo[target] = 1; 这里相当于【1.截止条件】，二选一就可以
        return dfs(nums, target);
    }

    private int dfs(int[] nums, int target) {
        // 1. 截止条件
        if(target == 0) 
            return 1;

        // 2. 判断是否缓存
        if(memo[target] != -1) 
            return memo[target];
        
        // 3. 递归
        int ans = 0;
        for(int num : nums) {
            if (target - num >= 0)
                ans += dfs(nums, target - num);
        }

        //4. 缓存
        memo[target] = ans;
        return ans;
    }

}
```

#### 方法三：动态规划

```java
class Solution {
    public int combinationSum4(int[] nums, int target) {
        // 1. 弄清楚dp[i]代表什么  ——————> dp[i]: 凑出金额为i的钱种数
        int[] dp = new int[target + 1];

        // 2. 边界条件: 注意虽然大多数题都是从dp[0],dp[1]开始的，但边界条件也可能是从dp[end]开始的
        dp[0] = 1;

        // 3. 转换方程: 背包问题是个双层循环，一层表示容量，一层表示物体大小。
        for(int i = 0; i <= target; i++) {
            for(int num : nums) {
                if (i - num >= 0)
                dp[i] += dp[i - num];
            }
        }
        // 4. 最终状态
        return dp[target];    
    }
}
```

### LeetCode 494: 目标和

#### 题目描述

给定一个非负整数数组，a1, a2, ..., an, 和一个目标数，S。现在你有两个符号 + 和 -。对于数组中的任意一个整数，你都可以从 + 或 -中选择一个符号添加在前面。

返回可以使最终数组和为目标数 S 的所有添加符号的方法数。

来源：力扣（LeetCode） 链接：https://leetcode-cn.com/problems/target-sum 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

![image-20210425212843391](C:%5CUsers%5CNeon%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20210425212843391.png)

#### 题目分析

其实自己也想到了思路，这次的转化条件挺好想的，dp\[i] = dp\[i - nums\[j]] + dp\[i + nums\[j]]。由于涉及了i和j，因此虽然只有一个数组，但还是可以用二维数组dp\[i] \[j]。

dp\[i] \[j]: 用nums索引为前i个元素组合出j的可能种数。

!\[img]\(file:///C:\Users\Neon\Documents\Tencent Files\791745910\Image\C2C\043648CF7D48197640B67B0384DE3295.png)

#### 代码实现

```java
class Solution {
    public int findTargetSumWays(int[] nums, int target) {
        
        int sum = 0;
        for(int num : nums) {
            sum += Math.abs(num);
        }

        if(Math.abs(target) > sum) return 0;

        int cols = 2 * sum + 1; // 11列：[-sum, sum]
        int rows = nums.length; // 5 列:[nums[0], nums[n]]
  
        // dp[i][j] 用前i个元素合成j的可能种数
        int[][] dp = new int[rows][cols]; // 5 * 11列

        for(int i = 0; i < rows; i++) {
            dp[i][0] = 0; // 0列全为0
        }
        for(int j = 0; j < cols; j++) {
            dp[0][j] = 0; // 0行全为0
        }

        dp[0][sum - nums[0]] = 1;
        dp[0][sum + nums[0]] = nums[0] == 0 ? 2 :1; // 如果为 0 的话 则有两种

        return dp[rows - 1][sum + target];
    }
}
```

#### 复杂度分析

时间复杂度：O(MN)。M为数组元素个数，N为元素绝对值总和。
