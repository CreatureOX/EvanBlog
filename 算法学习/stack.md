# 栈
* 栈（stack）又名堆栈，它是一种运算受限的线性表。其限制是仅允许在表的一端进行插入和删除运算。这一端被称为栈顶，相对地，把另一端称为栈底。*
* 栈就是一个桶，先进后出（FILO/先进后出）*

``` C
#include <stdio.h>
#include <stdlib.h>
#define maxn 11
#define T int

int stack[maxn];
int TOP = -1; // 栈顶指针

void pop(){
    TOP--;
}

void push(T buf){
    stack[++TOP] = buf;
}

int empty(){
    return TOP == -1;
}

void clear(){
    TOP = -1;
}
 
int size(){
    return TOP + 1;
}

T top(){
    return stack[TOP];
}
 
int main(){
    clear();
    T buf;
    buf = 1;
    push(buf);
    printf("stack top element is %d\n",top());
    printf("stack empty is %s\n",empty()==-1?"true":"false");
    printf("stack size is %d\n",size());
    pop();
    printf("stack size is %d\n",size());

    return 0;
}
```
