---
layout: post
title: 'NSSet\NSArray\NSDictionary的区别'
date: 2018-01-24
author: 李大鹏
cover: ''
tags: Objective-C
---

#### 一、NSSet\NSArray\NSDictionary 的区别

- NSSet 和 NSArray 都是对象容器，用于存储对象，属于集合； NSSet ， NSMutableSet 是无序的集合，在内存中存储方式是不连续的，NSArray 是有序的集合，在内存中存储位置是连续的；
- NSSet 和 NSArray 区别是：在搜索一个一个元素时 NSSet 比 NSArray 效率高，主要是它用到了一个算法 hash；开发文档中这样解释：You can use sets as an alternative to arrays when the order of elements isn’t important and performance in testing whether an object is contained in the set is a consideration—while arrays are ordered, testing for membership is slower than with sets.比如你要存储元素 A，一个 hash 算法直接就能直接找到 A 应该存储的位置；同样，当你要访问 A 时，一个 hash 过程就能找到 A 存储的位置。而对于 NSArray，若想知道 A 到底在不在数组中，则需要便利整个数组，显然效率较低了；
- NSSet，NSArray 都是类，只能添加 cocoa 对象，如果需要加入基本数据类型（int，float，BOOL，double 等），需要将数据封装成 NSNumber 类型。
- NSSet 与 NSDictionary 的区别，当有重复的 key 插入到字典 NSDictionary 时，会覆盖旧值，而集合 NSSet 则什么都不做，保证了里面的元素不会重复。

#### 二、NSSet 的实现原理

#### 1. Hash 表

哈希表（Hash table，也叫散列表），是根据键（Key）而直接访问在内存存储位置的数据结构。也就是说，它通过计算一个关于键值的函数，将所需查询的数据映射到表中一个位置来访问记录，这加快了查找速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。

#### 2. 开放定址法

这种方法也称再散列法，基本思想是：当关键字 key 的 hash 地址 p=F(key)出现冲突时，以 p 为基础，产生另一个 hash 地址 p1，如果 p1 仍然冲突，再以 p 为基础，再产生另一个 hash 地址 p2，。。。知道找出一个不冲突的 hash 地址 pi，然后将元素存入其中。

#### 3. 底层实现原理

NSSet 添加 key，key 值会根据特定的 hash 函数算出 hash 值，然后存储数据的时候，会根据 hash 函数算出来的值，找到对应的下标，如果该下标下已有数据，开放定址法后移动插入，如果数组到达阈值，这个时候就会进行扩容，然后重新 hash 插入。查询速度就可以和连续性存储的数据一样接近 O(1)了。
