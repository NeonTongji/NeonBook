# 贪心算法

### LeetCode1705 吃苹果的最大数量

There is a special kind of apple tree that grows apples every day for n days. On the ith day, the tree grows apples\[i] apples that will rot after days\[i] days, that is on day i + days\[i] the apples will be rotten and cannot be eaten. On some days, the apple tree does not grow any apples, which are denoted by apples\[i] == 0 and days\[i] == 0.

You decided to eat at most one apple a day (to keep the doctors away). Note that you can keep eating after the first n days.

Given two integer arrays days and apples of length n, return the maximum number of apples you can eat.

```
Example 1:

Input: apples = [1,2,3,5,2], days = [3,2,1,4,2]
Output: 7
Explanation: You can eat 7 apples:

- On the first day, you eat an apple that grew on the first day.
- On the second day, you eat an apple that grew on the second day.
- On the third day, you eat an apple that grew on the second day. After this day, the apples that grew on the third day rot.
- On the fourth to the seventh days, you eat apples that grew on the fourth day.
```

#### 分析

贪心：尽早吃掉容易坏的水果——————>容易坏，即临期时间近，临期时间为 i + days\[i], 因此需要把苹果根据临期时间用优先队列排序。使用优先队列。

pq中存 int【今天采摘的苹果数量，这些苹果的临期时间】

*   首先，如果日期还在苹果的收获期内（i < n） ，采摘苹果，存入库存pq;

    ```java
    if(i < n) {
    		pq.offer(new int[]{apples[i], i + days[i]}); //把当天的apple存进去
    } 
    ```
*   检查库存，最临期的这批苹果有没有坏了的，有的话就扔了;

    ```java
     while(!pq.isEmpty() && pq.peek()[1] <= i) { 
                    pq.poll();  //挖掉所有过期的
     }
    ```
*   扔了烂苹果后，全部都是新鲜的啦，可以取出一包苹果（int\[苹果数量，日期]），吃掉一个，如果没吃完（item\[0] - 1 > 0）,就还放回去，下次吃。

    ```java
    if(!pq.isEmpty()) { // 吃光苹果库存
         int[] item = pq.poll(); //取出最临期的
         item[0] = item[0] - 1; // 吃掉一个
         ans++; // 吃掉一个
         if(item[0] > 0) { // 还有剩的放冰箱继续吃
             pq.offer(new int[]{item[0], item[1]});  
         }
    }  
    ```

#### 代码

```java
class Solution {
// 先吃最临期的苹果，统一临期时间：i + days[i] 天过期， 因此要用最小堆

    public int eatenApples(int[] apples, int[] days) {

        PriorityQueue<int[]> pq = new PriorityQueue<int[]>((a,b) -> a[1] - b[1]); // 按过期时间升序

        int ans = 0, i = 0, n = apples.length;
        
        while(i < n || !pq.isEmpty()) { // i < n的时候可以存苹果， pq不为空的时候可以吃库存
// ++++++++++++++++++++++++++++存苹果+++++++++++++++++++++++++++++
            if(i < n) {
                pq.offer(new int[]{apples[i], i + days[i]}); //把当天的apple存进去
            } 
// ++++++++++++++++++++++++++++扔苹果+++++++++++++++++++++++++++++++
            while(!pq.isEmpty() && pq.peek()[1] <= i) { //挖掉所有过期的
                pq.poll();
            }
// ++++++++++++++++++++++++++++吃苹果+++++++++++++++++++++++++++++++
            if(!pq.isEmpty()) { // 吃光苹果库存
                int[] item = pq.poll(); //取出最临期的
                item[0] = item[0] - 1; // 吃掉一个
                ans++; // 吃掉一个
                if(item[0] > 0) { // 还有剩的放冰箱继续吃
                    pq.offer(new int[]{item[0], item[1]});  
                }
            }  
            i++; // 又过了一天
        }
        return ans;
    }
}
```

***
