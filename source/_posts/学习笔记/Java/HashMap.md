---
title: HashMap详解
date: 2019-10-25
categories:
    - 学习
tags:
    - java
---

### 术语介绍

* 桶： 就是hashmap的table数组
* bin： 就是挂在数组上的链表
* TreeNode： 红黑树
* capacity： table总容量
* MIN_TREEIFY_CAPACITY ：64   转化为红黑树table最小大小
* TREEIFY_THRESHOLD ：8  转化为红黑树的阈值
* loadFactor： 0.75  table扩容因子，当实际length大于等于 capacity*loadFactor时会进行扩容，并且扩容是按照2的整数次幂
* threshold： capacity*loadFactor

### HashMap为什么每次扩容都是以 2的整数次幂进行扩容

首先我们看hashMap的源码可知当新put一个数据时会进行计算位于table数组(也称为桶)中的下标，

``` java
 int index =key.hashCode()&(length-1);
 
 例如 hashCode十进制：201314 二进制：11 0001 0010 0110 0010
 假设初始大小为16则：
 index = 11 0001 0010 0110 0010 & 1111 = 0010
 假设初始大小为11则：
 index = 11 0001 0010 0110 0010 & 1010 = 0010
```

因为是将二进制按位与，(16-1) 是 1111，尾数都为1，这样就能保证计算后的index既可以是奇数也可以是偶数，并且只要传过来的key足够的分散，那么按位与的时候获取的index就会减少重复，这样就减少了hash碰撞以及hashMap的查询效率。所以HashMap每次扩容都是以2的整数次幂进行扩容，确保(length-1)以后尾数都为1。

* **通过与元素的hash值进行与操作，能够快速定位到数组下标 相对于取模运算，直接进行与操作能提高计算效率**。在CPU中，所有的加减乘除都是通过加法实现的，而与操作时CPU直接支持的。

* **扩容时简化计算数组下标的计算量 因为数组每次扩容都是原来的两倍，所以每一个元素在新数组中的位置要么是原来的index，要么index = index + oldCap**

### HashMap中初始化大小为什么是16

从HashMap每次扩容都是2的整数次幂进行扩容中，我们知道要确保HashMap的length要为2的整数次幂，那为什么是16呢？因为如果你分配太大的话会浪费资源，分配的小就会导致HashMap很容易扩容，所以就使用16作为初始大小

### 为什么链表的长度为8是变成红黑树？为什么为6时又变成链表？

我们都知道，链表的时间复杂度是O(n)，红黑树的时间复杂度O(logn)，很显然，红黑树的复杂度是优于链表的，既然这么棒，那为什么
HashMap为什么不直接就用红黑树呢,因为树节点所占空间是普通节点的两倍，所以只有当节点足够多的时候，才会使用树节点。也就是说，
节点少的时候，尽管时间复杂度上，红黑树比链表好一点，但是红黑树所占空间比较大，综合考虑，认为只能在节点太多的时候，红黑树
占空间大这一劣势不太明显的时候，才会舍弃链表，使用红黑树

那为什么选择8才会选择红黑树呢？

源码上说，为了配合使用分布良好的hashCode，树节点很少使用。并且在理想状态下，受随机分布的hashCode影响，链表中的节点遵循
泊松分布，而且根据统计，链表中节点数是8的概率已经接近千分之一，而且此时链表的性能已经很差了。所以在这种比较罕见和极端的
情况下，才会把链表转变为红黑树。因为链表转换为红黑树也是需要消耗性能的，特殊情况特殊处理，为了挽回性能，权衡之下，才使用
红黑树，提高性能。也就是大部分情况下，HashMap还是使用的链表，如果是理想的均匀分布，节点数不到8，HashMap就自动扩容了

>总结：

* 虽然红黑树的时间复杂度O(logn)小于链表的时间复杂度是O(n)，但是红黑树所占的空间节点是普通节点的两倍，综合考虑节点少的情况才会使用链表。
* 如果hashCode理想均匀分布，那么节点数不到8，HashMap就自动扩容了。不均匀那就使用红黑树代替链表
* [参考连接](https://blog.csdn.net/baidu_37147070/article/details/98785367)

#### HashMap扩容时死循环问题

因为HashMap不是线程安全，在1.8之前两个线程并发调用put方法，而其中一个线程刚好需要扩容(头插法)，需要进行链表的重组，从而导致的死循环问题

* [死循环](https://juejin.cn/post/7055084070790758436?utm_source=gold_browser_extension)