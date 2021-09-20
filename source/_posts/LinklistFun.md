---
title: 5 LinkListFun()
date: 2018-10-14 19:10:27
tags: [DataStructure,c++]
categories: [DataStructure]
description: <center>5个常见的单链表操作</center>
copyright: true
photos: "http://r.photo.store.qq.com/psb?/V13BnVsC23MbK8/wUdQiMjzyBZGQvyAXhe.5ybOE3vgF0tUgnmCkZCqTX0!/r/dLYAAAAAAAAA"
---
---
# Reverse LinkList

描述：反转一个单链表

1. 迭代
```c++
/***** Definition for linklist *****/
/* typedef struct LNode{
	Elemtype data;
	struct LNode *next;
 * }LNode *linklist
*/
class Solution {
    public LNode reverseList(linklist head) {
        linklist newhead=null;
        linklist now;
        while(head!=null){
            now=head;         //取头
            head=head.next;   //更新原链头
            now.next=newhead; //插入新链
            newhead=now;      //更新新链头
        }
        return newhead;
    }
}
```

2. 递归
```c++
class Solution {
    public LNode reverseList(linklist head) {
         if(head==null||head.next==null)return head;
         LNode newhead=reverseList(head.next);
         head.next.next=head;
         head.next=null;
         return newhead;
    }
}
```

---

# LinkedList Cycle

描述：判断一个链表是否有环

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
// 时间复杂度O(n) 空间复杂度O(1) 
class Solution { 
public: 
	bool hasCycle(ListNode *head) { 
	// 设计两个指针，一快一慢，快指针与慢指针相遇则有环。
		ListNode *slow = head, *fast = head; 
		while (fast && fast->next) { 
			slow = slow->next; 
			fast = fast->next->next; 
			if (slow == fast) return true; 
		} 
		return false; 
	} 
};
```

---

# Merge Two Sorted Lists

描述：将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

```c++
// 时间复杂度O(min(m,n)) 空间复杂度O(1) 
class Solution { 
public: ListNode *mergeTwoLists(ListNode *l1, ListNode *l2) { 
	if (l1 == nullptr) return l2; 
	if (l2 == nullptr) return l1; 
	ListNode dummy(-1); //头结点
	ListNode *p = &dummy; 
	for (; l1 != nullptr && l2 != nullptr; p = p->next) { 
		if (l1->val > l2->val) { 
			p->next = l2; 
			l2 = l2->next; 
		} 
		else { 
			p->next = l1; 
			l1 = l1->next; 
		} 
	} 
	p->next = l1 != nullptr ? l1 : l2; 
	return dummy.next; 
	} 
};
```

---

# Remove Nth Node From End of List

描述：给定一个链表，删除链表的倒数第 n 个节点，并且返回链表的头结点。

```c++
//设两个指针p,q，让q先走n步，然后p,q一起走，直到q走到尾结点，删除p->next即可
// 时间复杂度O(n) 空间复杂度O(1) 
class Solution { 
public: ListNode *removeNthFromEnd(ListNode *head, int n) { 
	ListNode dummy{-1, head}; //头结点
	ListNode *p = &dummy, *q = &dummy; 
	for (int i = 0; i < n; i++) // q先走n步 
		q = q->next; 
	while(q->next) { // 一起走
		p = p->next; 
		q = q->next; 
	} 
	ListNode *tmp = p->next; 
	p->next = p->next->next; 
	delete tmp; return dummy.next; 
	} 
};

```

---

# MiddleNode of List

描述：给定一个带有头结点head的非空单链表，返回链表的中间结点。如果，有两个中间结点，则返回第二个中间节点。

```c++
//同样使用快慢两个指针，当用慢指针slow遍历列表时，让另一个指针fast的速度是slow的二倍，则当快指针到结尾时，slow指针位于中间。
//初始位置都为head时，当fast指向最终的null时，slow也就达到了要求。
class Solution {
	public ListNode middleNode(ListNode head) {
		ListNode slow = head, fast = head;
		while((fast != null) && (fast.next != null)) {
			slow = slow.next;
			fast = fast.next.next;
		}
		return slow;
	}
}
```