# 快速幂取模
## 原理讲解
幂取模即求 **a<sup>b<sup>** % **mod**。
1. 最基础的方法是可以通过循环累乘，最后取模，时间复杂度为O(n)
2. 基础方法在a和b过大时会溢出。存在如下公式：**a<sup>b<sup>** % **mod** = (**a** % **mod**)**<sup>b<sup>** % **mode**。即引理：积的取余等于取余的积的取余。

``` C
#include<stdio.h>

int qmod(int a,int b,int mod){
    int ans=1,base=a%mod;
    while(b){
        if(b&1){
            ans=(ans*base)%mod;
        }
        base=(base*base)%mod;
        b>>=1;
    }
    return ans;
}

int main(){
    printf("%d\n", qmod(2,4,12));
    return 0;
}
```
