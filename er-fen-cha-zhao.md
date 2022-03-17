# 二分查找

### LeetCode 153： 寻找旋转排序数组中的最小值

#### [题目描述](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)

!\[image-20210512163050220]\(/Users/neon/Library/Application Support/typora-user-images/image-20210512163050220.png)

#### 1.题目分析

数组可以按旋转点为分界线分为左右两部分，如：

\[4,5,6,0,1,2,3] = \[4,5,6] + \[0,1,2,3] 数组A：\[4,5,6]; 数组B：\[0,1,2,3] · 注意本题为各不相同 于是我们便可以分两种情况讨论：\*\* ①mid指在数组A，②mid指在数组B。\*\*

**1.1 情况①：**

```java
	       A                  B
		[4,     5,     6] + [0,1,2,3]
		 ↑      ↑                  ↑
		left   mid                right
```

mid在数组A，则nums\[mid] >= nums\[left]。出现这种情况，便让left = mid + 1，因为最小值一定在B中，这样可以缩小左边范围。

**1.2 情况②：**

```java
         A                B
		[4,5,6] + [0,   1,    2,   3]
		 ↑              ↑          ↑
		left           mid       right
```

mid在数组A，则nums\[mid] < nums\[left]。出现这种情况，便让right = mid, 之所以不是mid + 1, 是因为mid 可能就指向最小值，让right = mid就不会错过这种情况。

***

#### 2. 我的尝试

**2.1 错误尝试1 （边界溢出）：**

```java
class Solution {
    public int findMin(int[] nums) {
        int left = 0, right = nums.length - 1;
        while(left <= right) {
            int mid = left + (right - left) / 2;
            if(nums[mid] >= nums[left]) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return nums[left];   
    }
}
```

***

**2.1.1 报错信息：**

![截屏2021-05-12 下午5.41.32.png](https://pic.leetcode-cn.com/1620812747-XsEhsZ-%E6%88%AA%E5%B1%8F2021-05-12%20%E4%B8%8B%E5%8D%885.41.32.png)

**2.1.2 为什么会索引异常呢？**

答：代码中left = mid + 1, 导致right = mid后，继续left继续右移了一位，导致边界异常。

***

**2.2 错误尝试2 while (left < right)：**

```java
class Solution {
    public int findMin(int[] nums) {
        int left = 0, right = nums.length - 1;
        while(left < right) {
            int mid = left + (right - left) / 2;
            if(nums[mid] >= nums[left]) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return nums[left];   
    }
}
```

***

那我们不要让left = right后跳出循环 ，是否能解决边界溢出呢？

**2.2.2 结果错误** ![截屏2021-05-12 下午5.48.35.png](https://pic.leetcode-cn.com/1620812926-eJMkAk-%E6%88%AA%E5%B1%8F2021-05-12%20%E4%B8%8B%E5%8D%885.48.35.png)

**2.2.3 为什么还是不对呢？**

原来当left跑到B中后，我们的选择条件便不再成立， 如：

```java
              A                   B
		[4,     5,     6] + [0,   1,    2,    3]
		                     ↑    ↑           ↑
		                    left  mid        right
```

于是left会不断右移，导致最后输出的都是最右边的值。

**2.2.4 如何解决呢?**

出现这个问题的根本原因是nums\[left]跑到B中去了，导致一直都只能进入如下的选择语句

```java
	if (nums[mid] >= nums[left]) {
	                left = mid + 1;
	}
```

由于本题是为了找最小值，而最小值一定在B中，所以right指针一定不会跑到A里面去。如果拿nums\[right]作为比较依据，便可以避免这种情况。

***

#### 3. 代码实现

```java
class Solution {
    public int findMin(int[] nums) {
        int left = 0, right = nums.length -1;

        while(left < right) {
            int mid = left + (right - left) / 2;
            if(nums[mid] < nums[right]) {
                right = mid; // 不是mid - 1是防止 mid已经指向最小的 从而错过最小值
            } else{
                left = mid + 1;
            }
        }
        return nums[left];      
    }
}
```

***

#### 4. 思考

**为什么这里不是while（left <= right) 呢？**

答：因为我们返回的是nums\[left]。若循环条件为while（left <= right)，则跳出循环时，left = right = mid，都指向最小值，而这时候还得进行最后一次循环，使left = mid + 1。这时，left便跑到mid和right右边一位去了，因此若要用while(left <= right)条件必须返回 nums\[right] 或者 nums\[left - 1]。

### LeetCode33: 搜索旋转排序数组

#### 1. [题目描述](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

!\[image-20210512181930482]\(/Users/neon/Library/Application Support/typora-user-images/image-20210512181930482.png)

#### 2. 题目分析

这道题与那道[【LeetCode 153. 寻找旋转排序数组】](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/solution/zui-xiang-xi-de-ti-jie-you-wo-zi-ji-shua-ywlb/)中的最小值的区别在于，找最小值只可能在B数组中去寻找，right一定指向B中元素，而本体为搜索元素，right和left都有可能在A或者在B中，于是分类便复杂了很多。三个指针的相对关系如何确定呢，我们需要先定下一个，然后确定其他两个。 ![IMG\_5402B2D17DD1-1.jpeg](https://pic.leetcode-cn.com/1620821039-dhlQWe-IMG\_5402B2D17DD1-1.jpeg)

**2.1 情况①: left在数组A中时，即nums\[left] > nums\[right]**

* 情况2.1.1 nums\[left] <= target <= nums\[mid], 则 right = mid -1;
* 情况2.1.2 nums\[left] <= nums\[mid] && nums\[left] > target, 则left = mid + 1;
* 情况2.1.3 nums\[left] > nums \[mid] && nums\[mid] < target, 则left = mid + 1;
* 情况2.1.4 nums\[left] <= target && nums\[left] > nums\[mid]，则 right = mid -1;
* 情况2.1.5 nums\[left] > nums\[mid] && target <= nums\[mid], 则 right = mid -1;

**2.2 情况②: left在数组B中时，即nums\[left] < nums\[right]**

* 情况2.2.1 nums\[left] <= target <= nums\[mid], 则 right = mid - 1;
* 情况2.2.2 nums\[mid] <= target <= nums\[right], 则 left = mid + 1;

***

#### 代码实现

```java
class Solution {
    public int search(int[] nums, int target) {
        int left = 0, right = nums.length -1;

        while(left <= right) {
            int mid = left + ((right - left) >> 1) ;
     
            if(nums[mid] == target) {
                return mid;
            }

            if(nums[left] > nums[right]) {
              	// oh my gosh, what a lousy if conditional statement!
                if((nums[left] <= target && target <= nums[mid]) || (nums[left] <= target && nums[left] > nums[mid]) || nums[left] > nums[mid] && target <= nums[mid]) { 
                    right = mid - 1;            
                } else {
                    left = mid + 1;                
                }
            } else {
                if(nums[left] <= target && target <= nums[mid]) {
                    right = mid - 1;
         
                } else {
                    left = mid + 1;         
                }
            }
        }
        return -1;
    }
}
```

***

#### 4. 思考和优化

由于条件语句太复杂了，我遗漏了两种情况，才勉强做出来。虽然很冗长，但在这一分析过程中我也有所收获。

**Conclusion1:** target 一定在 \[left, right]区间内，并且是闭区间；

**Conclusion2:** 通过对上面分类的总结，得出了这样的结论：当target在 \[left, mid) 区间内时（左闭右开，因为如果mid若指向target便会返回），right = mid - 1; 当target在 (mid, right] 区间内时（左开右闭，同理因为如果mid若指向target便会返回），left = mid + 1;

通过Conclusion2，我们可以对if语句再次优化。将原本【把left定在一个数组内】改为 【把mid定在一个数组内】，这样可以构造出：target在 \[left, mid) 区间内 和 target在 (mid, right] 区间内。从而优化了条件判断，代码如下：

```java
class Solution {
    public int search(int[] nums, int target) {
        int left = 0, right = nums.length -1;

        while(left <= right) {
            int mid = left + ((right - left) >> 1) ;
     
            if(nums[mid] == target) {
                return mid;
            }

            /*优雅多了，以后只用记住：
            * mid 在左边时，用left和mid把target夹住，让right = mid - 1， 否则 left = mid + 1；
            * mid 在右边时，用mid和right把target夹住，让left = mid + 1， 否则 right = mid - 1；
            */
            if (nums[mid] >= nums[left]) {
                if (nums[left] <= target && target < nums[mid]) {
                    right = mid - 1;
                } else {
                    left = mid + 1;
                }
            } else {
                if (nums[mid] < target && target <= nums[right]) {
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
        }
        return -1;
    }
}
```

### LeetCode154: 寻找旋转排序数组中的最小值 II

#### 1 [题目描述](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/)

!\[image-20210512204910836]\(/Users/neon/Library/Application Support/typora-user-images/image-20210512204910836.png)

#### 2. 题目分析

同样还是分AB数组，此题和[【LeetCode 153. 寻找旋转排序数组】](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/solution/zui-xiang-xi-de-ti-jie-you-wo-zi-ji-shua-ywlb/)的区别在于有重复元素。因此当mid 指向元素和right都相同时，就让right一直左移，移到相同的元素全部不在\[left, right]窗口内为止，这时右边的数组中的重复元素就将消失。 可能会有疑问，若重复元素就是最小元素怎么办，当right左移，相应的mid也会左移，慢慢的mid将不再指向最小元素，right一定不会跳过最小的元素，如：

```java
[2,   2,   2,   0,    0,   0 ,   0]     (right--)    [2,   2,   2,   0,   0,   0 ,   0]   (left = mid + 1) 
 ↑              ↑               ↑        ——————>      ↑         ↑              ↑           --------------> 
left            mid             right                left       mid          right

[2,   2,   2,   0,    0,   0 ,   0]    (right--)    [2,   2,   2,   0,    0,   0 ,   0]   (right--) 
                ↑     ↑    ↑            ——————>                     ↑     ↑                ——————> 
               left  mid right                                  left   mid(right)
[2,   2,   2,   0,    0,   0 ,   0]
   							↑ 
   							left(mid、right)
```

#### 代码实现

```java
class Solution {
    public int findMin(int[] nums) {
        int left = 0, right = nums.length - 1;

        while(left < right) {
            int mid = left + ((right - left)>>1);
            
            if(nums[mid] > nums[right]) { // 和nums[right]比是因为right指针总会在右边数组B内
                left = mid + 1;
            } else if(nums[mid] < nums[right]) {
                right = mid; // 不减一的理由和LeetCode153相同，避免跳过了最小值
            } else {
                right--;  // 消除重复值，以便找到索引最小的最小值
            }
        }
        return nums[left]; // return nums[right]也okay的 因为跳出的时候left == right
    }
}
```
