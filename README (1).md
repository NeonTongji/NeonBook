# 经典模拟

### LeetCode23 下一个排列

Implement next permutation, which rearranges numbers into the lexicographically next greater permutation of numbers.

If such an arrangement is not possible, it must rearrange it as the lowest possible order (i.e., sorted in ascending order).

The replacement must be in place and use only constant extra memory.

```
Example 1:

Input: nums = [1,2,3]
Output: [1,3,2]
Example 2:

Input: nums = [3,2,1]
Output: [1,2,3]
Example 3:

Input: nums = [1,1,5]
Output: [1,5,1]
Example 4:

Input: nums = [1]
Output: [1]
```

#### 分析

思路感觉不太好想，逆序，两两比较，right为left右边一位数，如果right比left的值小，则 在【right，数组末尾】里找出一个最小的能比left大的值，和left交换，初始默认是right，但有可能还有比right小，但比left大的，找到它，和left交换。交换后，对【right，数组末尾】进行排序。

例如 【1 3 4 7 8 9 5 7 3】-----swap------>【1 3 5 7 8 9 4 7 3】----sort(right, end) ---->【1 3 5 3 4 7 7 8 9】---->bingo ↑ ↑ 👆🏻 ↑ ↑ left right target left right

【坑1】： 要求下一次排序，因此找到后立刻break；

【坑2】：如果是最大的【3 2 1】，证明之前算法被穿透，用flag记录，若被穿透，排序后返回最小

#### 新API

部分数组排序: Arrays.sort(nums, fromIndex, to Index)

#### 代码

```java
class Solution {
    public void nextPermutation(int[] nums) {
       // [1 2 3 4] ->[1 2 4 3] ->[1 3 2 4] ->[1 3 4 2] ->[1 4 2 3] -> [1 4 3 2] ->[2 1 3 4]

       // 维护两个指针 ptr1 ptr2 = ptr1 - 1
        int n = nums.length;
        boolean flag = false;
       for(int i = n - 1; i > 0; i--) {
           int right = i, left = right - 1;
           if(nums[left] < nums[right]) {
               // nums[left] 和 [right, n]比 nums[left]大的最小数交换
               flag = true; 
               int target = searchGreaterMin(nums, right, left);
               swap(nums, left, target);
                Arrays.sort(nums, right, n);
                break;
           }
       }

       if(!flag) {
           Arrays.sort(nums);
           return;
       }
    }

    private void swap(int[] a, int x, int y) {
        int tmp = a[x];
        a[x] = a[y];
        a[y] = tmp;
   }

    private int searchGreaterMin(int[] a, int index, int x) {
        int ans = index;
        for(int i = index; i < a.length; i++) {
            if(a[i] < a[ans] && a[i] > a[x]) {
                ans = i;
            }
        }

        return ans;
    }
}
```
