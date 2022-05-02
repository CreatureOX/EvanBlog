# 链表
是一种线性表，但不按线性顺序存储数据而是在每个节点存储后继指针指向下一个节点。无需预先知道数据大小，充分利用计算机内存空间，实现灵活的内存动态管理。无法随机读取元素，而且空间开销比较大
* 插入的时间复杂度 = O(1)
* 查找的时间复杂度 = O(n)

## 单向链表
单向链表的节点包括两个域：一个信息域和一个指针区域。每个指针指向链表中的下一个节点，而最后一个节点指向一个空值。单向链表仅可单向遍历

``` C
typedef struct SingleNode{
    ElemType data;
    struct SingleNode *next;
}SingleNode;

typedef struct SingleNode *SingleLinkedList;
```

## 双向链表
双向链表的节点包括两个域：一个信息域和两个指针区域。前驱指针指向链表的上一个节点，后继指针指向链表中的下一个节点，而链表中的最初一个节点和最后一个节点指向一个空值或空列表。单向链表可双向向遍历

``` C
typedef struct DoubleNode{
    ElemType data;
    struct DoubleNode *prev;
    struct DoubleNode *next;
}DoubleNode;

typedef struct DoubleNode *DoubleLinkedList;
```

## 循环链表
循环链表的节点首尾相连，单向链表和双向链表都可实现循环链表。指向整个列表的指针可被称为访问指针

``` C
// when create and no element
CycleNode *node = (CycleNode*)malloc(sizeof(CycleNode));
node->next = node;
```

## 链表基本操作
``` C
#include<stdio.h>
#include<stdlib.h>

#define ElemType int
#define bool int
#define true 1
#define false 0

SingleNode* create(ElemType val){
    SingleNode *node = (SingleNode*)malloc(sizeof(SingleNode));
    if(node == null){
        return NULL;
    }else{
        node->val = val;
        node->next = NULL;
        return node;
    }
}

SingleNode* append(SingleNode *head, ElemType val){
    SingleNode *node = create(val);
    if(head == NULL){
        return node;
    }
    SingleNode *p = head;
    while(p->next != NULL){
        p = p->next;
    }
    p->next = node;
    return head;
}

SingleNode* insert(SingleNode *head, int idx, ElemType val){
    SingleNode *p = head;
    SingleNode *node = create(val);
    int cur = 1;
    while(cur < idx && p->next != NULL){
        cur++;
        p = p->next;
    }
    node->next = p->next;
    p->next = node;
    return head;
}

SingleNode* get(SingleNode *head, int i){
    int cur = 1;
    SingleNode *p = head;
    while(p && cur < i){
        p = p->next;
        cur++;
    }
    if(!p||cur>i){
        return NULL;
    }
    return p;
}

bool delete(SingleNode *head, int idx){
    int cur = 1;
    SingleNode *p = head;
    while(cur < idx && p->next != NULL){
        cur++;
        p = p->next;
    }
    if(p->next == NULL || cur > idx){
        return false;
    }
    q = p->next;
    p->next = q->next;
    free(q);
    return true;
}

int len(SingleNode *head){
    int len = 0;
    SingleNode *p = head;
    while(p){
        len++;
        p = p->next;
    }
    return len;
}

void traversal(SingleNode *head){
    SingleNode *p = head;
    while(p){
        printf("%d ", p->data);
        p = p->next;
    }
}
```