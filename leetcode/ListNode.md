# 1、反转链表

#### [206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

## 1.1、简单双指针



```java
class Solution 
{
    public ListNode reverseList(ListNode head) 
    {
      ListNode pre = null;
      ListNode cur = head;
      while(cur != null)
      {
        ListNode temp = cur.next;
        cur.next = pre;
        pre = cur;
        cur = temp;
      }
      return cur;
    }
}
```

无需判断传入值head是否为null



## 1.2、复杂双指针

```java
class Solution 
{
    public ListNode reverseList(ListNode head) 
    {
        if(head == null)
            return null;
        ListNode cur = head;
        while(head.next != null)
        {
            ListNode temp = head.next.next;
            head.next.next = cur;
            cur = head.next;
            
            //head始终不动，仅将head.next不断后移
            head.next = t;
        }
        return cur;
    }
}
        
```

需要判断head是否为null



# 2、相交链表

#### [160. 相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/)

给你两个单链表的头节点 `headA` 和 `headB` ，请你找出并返回两个单链表相交的起始节点。如果两个链表没有交点，返回 `null`

```java
public class Solution 
{
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) 
    {
      if(headA == null || headB == null)
        return null;
      ListNode ta = headA;
      ListNode tb = headB;
      while(ta != tb)
      {
        if(ta != null)
          ta = ta.next;
        else
          ta = headB;
        if(tb != null)
          tb = tb.next;
        else
          tb = headA;
      }
      return ta;
    }
} 
```

ta指向headA，tb指向headB，ta遍历完A链表后，开始遍历B链表；tb遍历完成B链表后，开始遍历A链表

若A和B相交，A=a+c，B=b+c，ta 和 tb 走过a+b+c后必然指向同一个节点，此时返回ta即可；

若A和B不相交，ta 和 tb遍历完2个链表后均为 null，此时可返回 null；

# 3、回文链表

#### [234. 回文链表](https://leetcode-cn.com/problems/palindrome-linked-list/)

给你一个单链表的头节点 `head` ，请你判断该链表是否为回文链表。如果是，返回 `true` ；否则，返回 `false`



用快慢指针找中点，将后链表反转与前链表比较

```java
class Solution {
    public boolean isPalindrome(ListNode head) {
      
      ListNode midnode = getMidNode(head);
      ListNode sechalf = reverseList(midnode.next);
      boolean flag = true;
      ListNode t1 = head;
      ListNode t2 = sechalf;
      while(t2 != null)
      {
        if(t1.val != t2.val)
          flag = flase;
      }
			//恢复原链表结构
      midnode.next = reverseList(sechalf);
      return flag;
      
      
      public ListNode getMidNode(ListNode head)
      {
        ListNode slow = head;
        ListNode fast = head;
        while(fast.next != null && fast.next.next != null)
        //实际判断fast.next.next,但是不加第一个判断会报空指针异常
        {
          slow = slow.next;
          fast = fast.next.next;
        }
        return slow;
      }
      
      
      public ListNode reverseList(ListNode head) 
      {
      		ListNode pre = null;
      		ListNode cur = head;
      		while(cur != null)
      		{
        		ListNode temp = cur.next;
        		cur.next = pre;
        		pre = cur;
        		cur = temp;
      		}
      		return cur;
    	}
      
    }
}
```

1、getMidNode()获取前半链表的最后一个节点，中间节点作为前半链表的最后一个节点

​		偶数链表为中间偏左节点，奇数链表为中间节点

2、对于后半链表的首节点做反转链表操作

3、反转后的链表与前半链表逐一判断

4、将后半链表再次反转，恢复链表原有结构

# 4、合并有序链表

#### [21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

递归实现

```java
class Solution 
{
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) 
    {
      if(l1 == null)
        return l2;
      if(l2 == null)
        return l1;
      if(l1.val < l2.val)
      {
        l1.next = mergeTwoLists(l1.next,l2);
        return l1;
      }
      else
      {
        l2.next = mergeTwoLists(l2.next,l1);
        return l2;
      }
    }
}
```



# 5、链表排序

#### [148. 排序链表](https://leetcode-cn.com/problems/sort-list/)

给你链表的头结点 `head` ，请将其按 **升序** 排列并返回 **排序后的链表** 。



递归实现，时间复杂度为O(nlogn)

```jav
class Solution 
{
    public ListNode sortList(ListNode head) 
    {
    	//递归结束条件
    	if(head == null || head.next == null)
    		return head;
    	ListNode mid = getMidNode(head);
    	//切断为head,temp两个链表
    	ListNode temp = mid.next;
    	mid.next = null;
    	//左右链表分别排序
    	ListNode left = sortList(head);
    	ListNode right = sortList(temp);
    	//合并有序链表
    	return mergeTwoLists(left,right);
    }
    
    //合并2个有序链表
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) 
    {
      if(l1 == null)
        return l2;
      if(l2 == null)
        return l1;
      if(l1.val < l2.val)
      {
        l1.next = mergeTwoLists(l1.next,l2);
        return l1;
      }
      else
      {
        l2.next = mergeTwoLists(l2.next,l1);
        return l2;
      }
    }
    
    //获取链表的中间(偏左)节点
    public ListNode getMidNode(ListNode head)
      {
        ListNode slow = head;
        ListNode fast = head;
        while(fast.next != null && fast.next.next != null)
        {
          slow = slow.next;
          fast = fast.next.next;
        }
        return slow;
      }
    
}
```

1、通过快慢指针从中点处将链表切断

2、将左右两个子链表分别排序

3、合并两个有序链表

​	递归结束条件：子链表只有1个或0个节点



# 6、链表相加

#### [2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

给你两个 非空 的链表，表示两个非负的整数。它们每位数字都是按照 逆序 的方式存储的，并且每个节点只能存储 一位 数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
      boolean flag = fasle;
      ListNode t1 = l1;
      ListNode t2 = l2;
      ListNode head = new ListNode(0);
      ListNode cur = head;
      int t;
      while(t1 != null || t2 != null)
      {
        //t1 t2可能为null，所以判断后方可赋值
        int i1 = t1 == null ? 0:t1.val;
        int i2 = t2 == null ? 0:t2.val;
        t = i1 + i2;
        if(flag)
          t++;
        flag = false;
        if(t>=10)
        {
          t = t-10;
          flag = true;
        }
        cur.next = new ListNode(t);
        cur = cur.next;
        //t1 t2长度不相等，当短链表遍历完成后应当停止遍历
        if(t1 != null)
          t1 = t1.next;
        if(t2 != null)
          t2 = t2.next;
      }
      //最终输出有可能比最长的链表多一位
      if(flag)
        cur.next = new ListNode(1);
      return head.next;
    }      
}

```

1、链表超过10位时，int Long 等数据类型无法存储，因此不能将链表转化为数字进行加减运算

2、两链表同时遍历，短链表结束后应当停止，否则会报空指针异常

# 7、删除倒数第n个节点

#### [19. 删除链表的倒数第 N 个结点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
      ListNode q = new ListNode(0);
      q.next = head;
      ListNode slow = q;
      ListNode fast = head;
      for(int i=0;i<n;i++)
      {
        fast = fast.next;
      }
      while(fast != null)
      {
        slow = slow.next;
        fast = fast.next;
      }
      slow.next = slow.next.next;
      return q.next;
    }
}
```

1、将slow向前移一位，而不是将fast向后移一位，是因为链表长度=n时，fast超出范围

2、return必须是q.next，而非head，因为可能head节点被删除。