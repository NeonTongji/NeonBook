# 字符串



## 十三、字符串

### 字符串匹配算法

#### KMP算法

[https://zhuanlan.zhihu.com/p/83334559](https://zhuanlan.zhihu.com/p/83334559)

#### manacher算法

#### Rabin-Karp算法

### 回文

### 中心扩展法

给你一个字符串 `s` ，请你统计并返回这个字符串中 **回文子串** 的数目。

**示例 1：**

```
输入：s = "abc"
输出：3
解释：三个回文子串: "a", "b", "c"
```

```java
	int ans = 0;
    public int countSubstrings2(String s) {
        int n = s.length();
        for(int i = 0; i < n;  i++) {
            count(s,i,i);
            count(s,i,i+1);
        }
        return ans;
    }
		// 判断以i为中心的回文串个数           start<----i---->end
    private void count(String s, int start, int end) {
        while(start >= 0 && end < s.length()
                 && s.charAt(start) == s.charAt(end)) {
            ans++;
            start--;
            end++;
        }
    }
```
