# 分治

### 排序链表

给你链表的头结点 head ，请将其按 升序 排列并返回 排序后的链表 。

进阶：

你可以在 O(n log n) 时间复杂度和常数级空间复杂度下，对链表进行排序吗？

示例 1：

输入：head = \[4,2,1,3] 输出：\[1,2,3,4] 示例 2：

输入：head = \[-1,5,3,4,0] 输出：\[-1,0,3,4,5] 示例 3：

输入：head = \[] 输出：\[]

提示：

链表中节点的数目在范围 \[0, 5 \* 104] 内 -105 <= Node.val <= 105

#### 1.递归

我首先想到，链表二叉树都是递归结构，于是想到用递归求解。递归中非常重要的两点：

1. 有递归终点；
2. 把递归函数当做已经实现了的函数来用

#### 代码一

```java
public ListNode sortList1(ListNode head) {
        if(head == null || head.next == null) return head; // 递归终点
        
        ListNode sorted = sortList(head.next);  // 架设首节点后面的都排序好了

        if(head.val <= sorted.val) {  // 如果首节点值 <= 后面的头结点值，直接连起来
            head.next = sorted;
            return head;
        } else {  // 如果首节点值比后面头结点值大，要分两种情况，一种是可以直接插入，另一种是插后面
            ListNode cur = sorted.next; // cur == null
            ListNode prev = sorted; // prev = 3;

            while(cur != null) {
                if(head.val < cur.val) { // 找到比head大的节点，插到他前面，这样就需要记录一个cur
                    prev.next = head;
                    head.next = cur;
                    break;
                }
                cur = cur.next;
                prev = prev.next;
            }

            if(cur == null) {  // 注意直接插在后面的
                prev.next = head;
                head.next = null;
            }
            return sorted;
        }
```

#### 复杂度

用到了递归，而且还有for循环在里面，时间复杂度为O(n^2);

#### 2. 分治（自顶向下）

![image-20211215105344980](https://tva1.sinaimg.cn/large/008i3skNgy1gxebx5a3qdj30zw0r841a.jpg)

1个链表可以拆成2个链表，2个可以拆成4个链表，直到链表为单个，然后依次合并

通过递归实现链表归并排序，有以下两个环节：

1. **分割 cut 环节**：找到当前链表中点，并从中点将链表断开（以便在下次递归 cut 时，链表片段拥有正确边界）； 找到链表中点方法：**快慢双指针法**，奇数个节点找到中点，偶数个节点找到中心左边的节点。找到中点 slow 后，执行 slow.next = Null 将链表切断。 递归分割时，输入当前链表左端点 head 和中心节点 slow 的下一个节点 tmp(因为链表是从 slow 切断的)。 cut 递归终止条件： 当**head.next == Null**时，说明只有一个节点了，直接返回此节点。
2.  **合并 merge 环节**： 将两个排序链表合并，转化为一个排序链表。 双指针法合并，建立**辅助ListNode Sentinel 作为头部**。 设置两指针 left, right 分别指向两链表头部，比较两指针处节点值大小，由小到大加入合并链表头部，指针交替前进，直至添加完两个链表。 返回辅助ListNode h 作为头部的下个节点 h.next。 时间复杂度 O(l + r)，l, r 分别代表两个链表长度。 当题目输入的 head == Null 时，直接返回Null。

    **两个易错重点**

    ① 递归终点为，节点成了单节点，因此重点为 head.next == null, return head

    ② 拆分不要忘，让prev = null。记住快慢指针找中点，slow只会在中间或者中间偏后，因此需要记录前面的prev

#### 代码

```java
public ListNode sortList(ListNode head) {
        return head == null ? head : mergeSort(head);
    }

    private ListNode mergeSort(ListNode head) {
      
        if(head.next == null) return head; // 递归终点，单节点 【重点1】
      
        ListNode ptr1 = head, ptr2 = head, prev = null;
        // 找到中间节点
        while(ptr2 != null && ptr2.next != null) { //记住快慢指针找中点的条件
            prev = ptr1; //这样prev就在前面一个
            ptr1 = ptr1.next;
            ptr2 = ptr2.next.next;
        }

        prev.next = null; //拆分成两个链表    【重点2】
				
        ListNode left = mergeSort(head);   //排序好head链表    
        ListNode right = mergeSort(ptr1);  // 排序好ptr1链表
      // 合并两个链表
        return mergeLists(left, right);
    }

    private ListNode mergeLists(ListNode l1, ListNode l2) {
        ListNode ptr1 = l1, ptr2 = l2;
        ListNode sentinel = new ListNode(-1), ptr = sentinel;
        while(ptr1 != null && ptr2 != null) {
            if(ptr1.val < ptr2.val) {
                ptr.next = ptr1;
                ptr1 = ptr1.next;
            } else {
                ptr.next = ptr2;
                ptr2 = ptr2.next;
            }
            ptr = ptr.next;
        }

        ptr.next = (ptr1 != null) ? ptr1 : ptr2;

        return sentinel.next;
    }
```

#### 复杂度（递归实现）

时间复杂度为O（nlogn）: logn层，每层操作时间为O（n）

空间复杂度O（logn）递归调用的栈空间

#### 3. 分治（自底向上）

![image-20211215140418162](https://tva1.sinaimg.cn/large/008i3skNgy1gxehf74xuij314c0muwgt.jpg)

bottom-to-up 的归并思路是这样的：先两个两个的 merge，完成一趟后，再 4 个4个的 merge，直到结束。举个简单的例子：`[4,3,1,7,8,9,2,11,5,6]`.

```
step=1: (3->4)->(1->7)->(8->9)->(2->11)->(5->6)
step=2: (1->3->4->7)->(2->8->9->11)->(5->6)
step=4: (1->2->3->4->7->8->9->11)->(5->6)
step=8: (1->2->3->4->5->6->7->8->9->11)
```

链表里操作最难掌握的应该就是各种断链啊，然后再挂接啊。在这里，我们主要用到链表操作的两个技术：

* `merge(l1, l2)`，双路归并，我相信这个操作大家已经非常熟练的，就不做介绍了。
* `cut(l, n)`，可能有些同学没有听说过，它其实就是一种 split 操作，即断链操作。不过我感觉使用 cut 更准确一些，它表示，将链表 `l` 切掉前 n 个节点，并返回后半部分的链表头。
* 额外再补充一个 dummyHead 大法，已经讲过无数次了，仔细体会吧。

\---

希望同学们能把双路归并和 cut 断链的代码烂记于心，以后看到类似的题目能够刷到手软。

掌握了这三大神器后，我们的 bottom-to-up 算法伪代码就十分清晰了：

```
current = dummy.next;
tail = dummy;
for (step = 1; step < length; step *= 2) {
	while (current) {
		// left->@->@->@->@->@->@->null
		left = current;

		// left->@->@->null   right->@->@->@->@->null
		right = cut(current, step); // 将 current 切掉前 step 个头切下来。

		// left->@->@->null   right->@->@->null   current->@->@->null
		current = cut(right, step); // 将 right 切掉前 step 个头切下来。
		
		// dummy.next -> @->@->@->@->null，最后一个节点是 tail，始终记录
		//                        ^
		//                        tail
		tail.next = merge(left, right);
		while (tail->next) tail = tail->next; // 保持 tail 为尾部
	}
}
```

#### 代码

```java
private static ListNode sortList(ListNode head) {

		int len = 0;
		for(ListNode p = head; p != null; p = p.next) {
			len++;
		}

		ListNode sentinel = new ListNode(0, head);
		for(int step = 1; step < len; step *= 2) { //代表层数，需要logn 层合并, 1 合 2， 2 合 4， 4 合 8 

				ListNode cur = sentinel.next; // 从 具体元素开始

				ListNode tail = sentinel; // 每层 cur、tail都从头上开始

				while(cur != null) { // 代表遍历每一层，进行mergeList操作

					ListNode left = cur;  // 1 -> 2 -> 3 -> 4 -> 5
					ListNode right = cut(left, step); // 1 -> 2 (left) 3 -> 4 ->5 (right)
					cur = cut(right, step);	// 1 -> 2 (left) 3 ->4 (right) 5 (cur)

					tail.next = mergeLists(left, right); // 用tail 连接起每个链表碎片, tail 位于 1 前

					while(tail.next != null) {  // tail 置于已经连接的链表碎片尾巴上
						tail = tail.next; // tail 爬到 4
					}
				}

		}

		return sentinel.next;
	
	}

	
	public static ListNode cut(ListNode head, int step) {
		while(step > 1 && head != null) {  // 注意，要切断，必须把返回节点前一个节点与要返回的关系斩断，因此step > 1，使head位于前一个
			head = head.next;       // 1 -> 2 -> 3 -> 4 -> null  , 要返回 3->4，则把head放到2，把2->null
			step--;            
		}                      

		if(head != null) {
			ListNode res = head.next;
			head.next = null;
			return res;
		}
		return null;
	}


	// 合并两个有序链表
	private static ListNode mergeLists(ListNode l1, ListNode l2) {
        ListNode ptr1 = l1, ptr2 = l2;
        ListNode sentinel = new ListNode(-1), ptr = sentinel;
        while(ptr1 != null && ptr2 != null) {
            if(ptr1.val < ptr2.val) {
                ptr.next = new ListNode(ptr1.val);
                ptr1 = ptr1.next;
            } else {
                ptr.next = new ListNode(ptr2.val);
                ptr2 = ptr2.next;
            }
            ptr = ptr.next;
        }

        ptr.next = (ptr1 != null) ? ptr1 : ptr2;

        return sentinel.next;
    }
```

##
