---
title: "MurmurHash算法"
share: false
categories:
  - 数据结构与算法
tags:
  - 算法
  - 哈希
---

MurmurHash是一种非加密型哈希函数，适用于一般的哈希检索操作。已经被很多开源项目如Redis, Memcached, Cassandra, Lucene等应用。对于规律性较强的key，MurmurHash的随机分布特征表现更良好。

## 特点
1. 速度快，比安全散列算法快几十倍
2. 变化足够激烈，相似的字符串如``` acb ```和```acc```能均匀散落在哈希环上
3. 不保证安全性

## JAVA实现
``` java
import java.nio.charset.Charset;
import com.google.common.hash.HashCode;
import com.google.common.hash.HashFunction;
import com.google.common.hash.Hashing;


public class MurmurHashTest {
    public static void main(String[] args) {
        HashFunction function = Hashing.murmur3_32();
        HashCode hashcode = function.hashString("helloWorld!", Charset.forName("utf-8"));
        System.out.println(hashcode.asInt());
    }
}

```