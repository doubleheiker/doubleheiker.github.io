---
title: Palindrome
date: 2018-10-14 09:38:47
tags: [DataStructure,c,c++]
categories: [DataStructure]
description: <center>单链表判断回文串</center>
copyright: true
photo: "http://r.photo.store.qq.com/psb?/V13BnVsC23MbK8/A2*Xr5NMflftPy7SfuEopnfYR6LzA.Q6K2Av40K4Nk0!/r/dFYBAAAAAAAA"
---
---
> 在学习链表时，遇到的一个有意思的问题，记录下思路和算法。

# 思路

1. 使用快慢两个指针找到链表中点，快指针每次移动两个结点，慢指针每次移动一个结点。
   * 如果结点是奇数，中点位置不需要矫正
   * 如果结点是偶数，使慢指针前进一个结点指向下中位数
2. 在慢指针移动的时候，同时修改其next指针，使链表前半部分反序。
3. 最后比较中点两侧的链表是否相等。

时间复杂度：O(n)
空间复杂度：O(n)

---

# 完整代码

```c++
/***** Definition for linklist *****/
/* typedef struct LNode{
	Elemtype data;
	struct LNode *next;
 * }LNode *linklist
*/
bool isPalindorme(LNode head) {
	if (head == null || head->next == null) return true;

	LNode prev = null;
	LNode slow = head;
	LNode fast = head;

	//实现链表前半部分反序排列
	while (fast != null && fast->next != null) {
		fast = fast->next->next;
		LNode next = slow->next;
		slow->next = prev;
		prev = slow;
		slow = next;
	}

	//根据fast指针判断链表奇偶
	if (fast ！= null) {
		slow = slow->next;
	}

	//比较链表前半段和后半段是否相同
	while (slow != null) {
		if (slow->data != prev->data) {
			return false;
		}
		slow = slow->next;
		prev = prev->next
	}

	return true;
}
```