# 滑动窗口

### 10.1 LeetCode3 [无重复元素最长子数组](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

Given a string s, find the length of the longest substring without repeating characters.

Example 1:

Input: s = "abcabcbb" Output: 3 Explanation: The answer is "abc", with the length of 3.

#### 分析

一般子数组、子集都要求是连续的，所以用滑动窗口比较合适。

滑动窗口出自计算机网络，维护窗口的左边界和右边界即可，转化成代码就是初始化一个left = 0， right = 0的数组界限。

窗口只能一直向前移动，当满足条件时，right向右扩张，窗口变大；当遇到特殊情况时，left向右锁，窗口变小。

本题：当right所指的元素从未出现过时，right++；

​ 当right所指的元素出现过时，left指到之前该元素之前出现位置的后一个；因此本题还需要用in\[26]的数组记录元素上次出现的位置，每次又出现后，覆盖即可。

【注意】“abba” , 为了防止left突然回缩，left取 **max(left, 上次出现的索引 + 1)**

#### 代码

```java
    public static int lengthOfLongestSubstring1(String s) {
        //用数组记录right处单词上次出现的index

        int[] lastOccurIndex = new int[26];

        int left = 0, right = 0, ans = 0;

        while(right < s.length()) {
            char c = s.charAt(right);
            left = Math.max(left, lastOccurIndex[c - 'a'] + 1);
            lastOccurIndex[c - 'a'] = right;
            ans = Math.max(ans, right - left + 1);
            right++;
        }

        return ans;
    }
```

哈希map版

```java
 //substing  subset都用滑动窗口解    abcdefb
    public static  int lengthOfLongestSubstring2(String s) {
        int left = 0, right = 0, ans = 0;
        Map<Character, Integer> map = new HashMap<>();
        while(right < s.length()) {
            char c = s.charAt(right);
            // right出现过
            if(map.containsKey(c)) {
                left = Math.max(left, map.get(c) + 1); //left缩到上次出现后的位置后面一个
            } 
            // 右边总要前进，还要记录
            map.put(c, right);
            ans = Math.max(ans, right - left + 1); //注意先计算再让right++！！！
            right++;
        }

        return ans;
    }
```

#### 复杂度

时间复杂度：O(n)， 一次遍历。

空间复杂度：O(1), 没有占用额外空间。

***

### 2. [LeetCode1052 爱生气的书店老板](https://leetcode-cn.com/problems/grumpy-bookstore-owner)

There is a bookstore owner that has a store open for n minutes. Every minute, some number of customers enter the store. You are given an integer array customers of length n where customers\[i] is the number of the customer that enters the store at the start of the ith minute and all those customers leave after the end of that minute.

On some minutes, the bookstore owner is grumpy. You are given a binary array grumpy where grumpy\[i] is 1 if the bookstore owner is grumpy during the ith minute, and is 0 otherwise.

When the bookstore owner is grumpy, the customers of that minute are not satisfied, otherwise, they are satisfied.

The bookstore owner knows a secret technique to keep themselves not grumpy for minutes consecutive minutes, but can only use it once.

Return the maximum number of customers that can be satisfied throughout the day.

Example 1:

Input: customers = \[1,0,1,2,1,1,7,5], grumpy = \[0,1,0,1,0,1,0,1], minutes = 3 Output: 16 Explanation: The bookstore owner keeps themselves not grumpy for the last 3 minutes. The maximum number of customers that can be satisfied = 1 + 1 + 1 + 1 + 7 + 5 = 16. Example 2:

Input: customers = \[1], grumpy = \[0], minutes = 1 Output: 1

#### 分析

数组，连续时间，想到用滑动窗口解题

#### 代码一（差）

以下代码不断享有推进固定大小的窗口，用unsatisfiedCustomer记录最大的不满意客人数量，但是没有利用滑动窗口的性质，每次都在while里面遍历窗口。时间复杂度为 O(mn), 其中m为数组长度，n为分钟数。

```java
	public int maxSatisfied2(int[] customers, int[] grumpy, int minutes) {
	
		int n = customers.length; // 找出minutes长度窗口内 最多 不满意顾客	
		
    // corner case: 从不生气
		if(minutes >= n) {
			int sum = 0;
			for(int customer : customers) {
				sum += customer;
			}
			return sum;
		}

		int left = 0, right = minutes - 1, max = 0;

    //寻找窗口内生气最多的
		while(right < n) {
			int unsatisfiedCustomers = 0;
			for(int i = left; i <= right; i++) {
				if(grumpy[i] == 1) {
					unsatisfiedCustomers += customers[i];
				}
			}
			max = Math.max(max, unsatisfiedCustomers);
			right++;
			left++;
		} 

		int ans = 0;

		for(int i = 0; i < n; i++) {
				ans += (grumpy[i] == 1) ? 0 : customers[i];
		} 

		return ans + max;
	}
```

#### 代码二 （优化）

利用滑动窗口的性质，在固定大小的窗口向右推进的过程中，下一个窗口sum = 上个窗口sum + num\[right] - num\[left];

注意这样的窗口是左闭右开的。

**window: \[ left, right )**

![image-20211216160757189](https://tva1.sinaimg.cn/large/008i3skNgy1gxfqmalkkuj317o0qadnl.jpg)

```java
public static int maxSatisfied(int[] customers, int[] grumpy, int minutes) {
		
		int n = customers.length; 

	

		int left = 0, right = minutes, init = 0; // 开区间


		for(int i = 0; i < right; i++) {        // 初始化窗口值
			init += grumpy[i] * customers[i];
		}

		int max = init;

		while(right < n) {
			
			int rightNum = grumpy[right] * customers[right]; //right所指为生气顾客
			int leftNum = grumpy[left] * customers[left]; //left所指为生气顾客
			init = init + rightNum - leftNum;
			max = Math.max(max, unsatisfiedCustomers);
			right++;
			left++;	
		} 

		int ans = 0;

		for(int i = 0; i < n; i++) {
			ans += (grumpy[i] == 1) ? 0 : customers[i];

		} 

		return ans + max;

	}
```

### 3. LeetCode1423 可能获得最大点数

There are several cards arranged in a row, and each card has an associated number of points. The points are given in the integer array cardPoints.

In one step, you can take one card from the beginning or from the end of the row. You have to take exactly k cards.

Your score is the sum of the points of the cards you have taken.

Given the integer array cardPoints and the integer k, return the maximum score you can obtain.

Example 1:

Input: cardPoints = \[1,2,3,4,5,6,1], k = 3 Output: 12 Explanation: After the first step, your score will always be 1. However, choosing the rightmost card first will maximize your total score. The optimal strategy is to take the three cards on the right, giving a final score of 1 + 6 + 5 = 12.

来源：力扣（LeetCode） 链接：https://leetcode-cn.com/problems/maximum-points-you-can-obtain-from-cards 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

#### 方法一

![image-20211216170938539](https://tva1.sinaimg.cn/large/008i3skNgy1gxfsu0mfyzj318i0oi79v.jpg)

```java
public static int maxScore1(int[] cardPoints, int k) {

		// duplicate cardPoints
		int[] doubleCards = new int[2 * cardPoints.length];
		int n = cardPoints.length;
		for(int i = 0; i < n; i++) {
			doubleCards[i] = cardPoints[i];
			doubleCards[i + n] = cardPoints[i];
		}

		// obtain the max value in a sliding window of size k;

		int left = n - k, right = n, sum = 0;
		for(int i = left; i < right; i++) {
			sum += doubleCards[i];
		}

		// System.out.println(sum);

		int ans = sum;
		while(right < cardPoints.length + k) {
			System.out.println(doubleCards[right] + " " + doubleCards[left]);
			sum = sum + doubleCards[right] - doubleCards[left];
			ans = Math.max(ans, sum);
			right++;
			left++;
		} 

		return ans;
    }
```

#### 方法二（优化方法一）

不用复制数组

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxfsu0dar5j315w0ny0ui.jpg)

```java
public static int maxScore2(int[] cardPoints, int k) {

		// duplicate cardPoints
	
		int n = cardPoints.length;
	
		// obtain the max value in a sliding window of size k;

		int left = n - k, right = n, sum = 0;
		for(int i = left; i < right; i++) {
			sum += cardPoints[i];
		}

		right = 0;

		int ans = sum;
		while(right < k) {
			
			sum = sum + cardPoints[right] - cardPoints[left];
			ans = Math.max(ans, sum);
			right++;
			left++;
			if(left == n) left = 0;
		} 

		return ans;
    }
```

#### 方法三 （优化方法二）

即全数组和 - 尺寸为length - k的窗口内，最小元素之和

```java
class Solution {
    public int maxScore(int[] cardPoints, int k) {
        int n = cardPoints.length;
        // 滑动窗口大小为 n-k
        int windowSize = n - k;
        // 选前 n-k 个作为初始值
        int sum = 0;
        for (int i = 0; i < windowSize; ++i) {
            sum += cardPoints[i];
        }
        int minSum = sum;
        for (int i = windowSize; i < n; ++i) {
            // 滑动窗口每向右移动一格，增加从右侧进入窗口的元素值，并减少从左侧离开窗口的元素值
            sum += cardPoints[i] - cardPoints[i - windowSize];
            minSum = Math.min(minSum, sum);
        }
        return Arrays.stream(cardPoints).sum() - minSum;
    }
}
```

***

### 4. LeetCode 76 最小覆盖子字符串

```
Given two strings s and t of lengths m and n respectively, return the minimum window substring of s such that every character in t (including duplicates) is included in the window. If there is no such substring, return the empty string "".

The testcases will be generated such that the answer is unique.

A substring is a contiguous sequence of characters within the string.

Example 1:

Input: s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"
Explanation: The minimum window substring "BANC" includes 'A', 'B', and 'C' from string t.
Example 2:

Input: s = "a", t = "a"
Output: "a"
Explanation: The entire string s is the minimum window.


来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/minimum-window-substring
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

#### 分析

窗口不断滑动，每次找到满足要求的子字符串，就用len维护最小的长度，用start记住最小那次索引。

【注意事项】：

* len的初始化：若一开始就满足要求，len初始化为最初的值，不满足要求初始化为unreachable
* start: 要用start维护好出现结果那次位置在哪儿
* substring（）

#### 代码

```java
public String minWindow(String s, String t) {
        int m = s.length(), n = t.length();
        String res = "";
        if(m < n ) return res;
     
        //初始长度为n的窗口，检查有没有包含t，如果没有
        int left = 0, right = n;
        int[] recordS = new int[100];
        int[] recordT = new int[100];
        for(int i = 0; i < n; i++) {
            recordS[s.charAt(i) - 'A']++;  //ADO
            recordT[t.charAt(i) - 'A']++; // ABC
        }

        int len = check(recordS, recordT) ? n : m + 1; //初始化长度也很重要

        int start = 0;
        while(right < m) {
            if(right < m) {
                recordS[s.charAt(right++) - 'A']++;
            }

            //右移, 当前字符无效，或是被右边取代，无效什么都不做，已经被右边取代，则扣除后右移
            while(left < m && ( recordT[s.charAt(left) - 'A'] == 0 
                            || recordS[s.charAt(left) - 'A'] > recordT[s.charAt(left) - 'A'])) {
                if(recordS[s.charAt(left) - 'A'] > recordT[s.charAt(left) - 'A']) {
                    recordS[s.charAt(left) - 'A']--;
                }
                left++;
            }
            
            if(check(recordS, recordT)) {
                int tmp = len;
               len = Math.min(len, right - left);
               start = len < tmp ? left : start;
            }
           
        }

        if(len == m + 1) return res;    

        return s.substring(start, start + len);
     
    }

    // 检查等长数组a是否包含b
    private boolean check(int[] a, int[] b) {
        for(int i = 0; i < a.length; i++) {
            if(a[i] < b[i]) return false;
        }
        return true;
    }
```

#### 复杂度

时间复杂度：O（m）

空间复杂度：O（n）
