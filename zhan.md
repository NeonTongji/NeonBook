# 栈

### LeetCode 简化路径

示例 1：

输入：path = "/home/" 输出："/home" 解释：注意，最后一个目录名后面没有斜杠。 示例 2：

输入：path = "/../" 输出："/" 解释：从根目录向上一级是不可行的，因为根目录是你可以到达的最高级。 示例 3：

输入：path = "/home//foo/" 输出："/home/foo" 解释：在规范路径中，多个连续斜杠需要用一个斜杠替换。

来源：力扣（LeetCode） 链接：https://leetcode-cn.com/problems/simplify-path 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

### 分析

前两次分别用数组和List做了，数组移除元素不方便，碰到连续两个".." ".." 更是不好处理，List移除元素虽然方便，但是一旦移除了元素，list的指针就倒退了，非常麻烦。

此外，还有一个坑，用Arrays.asList转的list，实际是一个Abstract List，没有remove方法可以用。以后再也不要转list了做题！！！！！，要转就一个个赋值。

```java
class Solution {
    public String simplifyPath(String path) {
        String[] strs = path.split("/"); // 路径碎片
        int n = strs.length;
        
        Deque<String> stack = new LinkedList<>();
        for(String str : strs) {
            if(str.equals(".") || str.equals("")) continue;
            if(str.equals("..")) {
                if(!stack.isEmpty()) stack.pop();
            } else {
                stack.push(str);
            }
        }

        StringBuffer ans = new StringBuffer();
        if (stack.isEmpty()) {
            ans.append('/');
        } else {
            while (!stack.isEmpty()) {
                ans.append('/');
                ans.append(stack.pollLast());
            }
        }
        return ans.toString();

    }
}
```

### LeetCode 更高的温度

请根据每日 `气温` 列表 `temperatures` ，请计算在每一天需要等几天才会有更高的温度。如果气温在这之后都不会升高，请在该位置用 `0` 来代替。

示例 1:

输入: temperatures = \[73,74,75,71,69,72,76,73] 输出: \[1,1,4,2,1,1,0,0] 示例 2:

输入: temperatures = \[30,40,50,60] 输出: \[1,1,1,0]

### 分析

又是一个消消乐游戏，后碰到的要来消灭以前的

```java
public int[] dailyTemperatures(int[] temperatures) {
        Deque<Integer> stack = new LinkedList<Integer>();

        int n = temperatures.length;
        int[] ans = new int[n];

        for(int i = 0; i < n; i++) {
            int cur = temperatures[i];
            // 消消乐
            while(!stack.isEmpty() && cur > temperatures[stack.peek()]) {
                int prev = stack.pop();
                ans[prev] = i - prev;
            }
            stack.push(i);
        }

        return ans;
    }
```
