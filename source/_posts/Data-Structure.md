---
title: '[Data-Structure]'
date: 2018-11-16 21:45:57
tags:
- Data Structure
- 数据结构
toc: ture
---



## 总览

* 



<!--more-->



## 链表

**链式存储中两个元素在内存中互不相邻**

**每一个元素都有一个指针域，一般存储到下一个元素的指针**

**优点：插入和删除时间复杂度为O(1)，不浪费内存，添加元素申请内存，删除释放内存**

**缺点：访问的时间复杂度最坏为O(n)**

**分类：单向链表，双向链表，循环链表**

### 反转链表

```java
public ListNode reverse(ListNode head) {
    ListNode prev = null;			
    while (head != null) {
        ListNode next = head.next;	//新建next指向head下一结点
        head.next = prev;			//第一次：第一个结点指向null，第二次：第二个结点指向第一个结点
        prev = head;				//prev指向排完的最后的结点
        head = next;				//head指向下一个结点
    }
    return prev;
}

public class ListNode {
	public int data;
	public ListNode next;
}
public ListNode reverseList(ListNode pHead){
	ListNode pReversedHead = null; //反转过后的单链表存储头结点
	ListNode pNode = pHead; //定义pNode指向pHead;
	ListNode pPrev = null; //定义存储前一个结点
	while(pNode != null){
		ListNode pNext = pNode.next; //定义pNext指向pNode的下一个结点
		if(pNext==null){ //如果pNode的下一个结点为空，则pNode即为结果
			pReversedHead = pNode;
		}
		pNode.next = pPrev; //修改pNode的指针域指向pPrev
		pPrev = pNode; //将pNode结点复制给pPrev
		pNode = pNext; //将pNode的下一个结点复制给pNode
	}
	return pReversedHead;
}

//递归算法
public ListNode reverseList3(ListNode pHead){
	if(pHead==null || pHead.next == null){ //如果没有结点或者只有一个结点直接返回pHead
		return pHead;
	}
	ListNode pNext = pHead.next; //保存当前结点的下一结点
	pHead.next = null; //打断当前结点的指针域
	ListNode reverseHead = reverseList3(pNext); //递归结束时reverseHead一定是新链表的头结点
	pNext.next = pHead; //修改指针域
	return reverseHead;
}


```

