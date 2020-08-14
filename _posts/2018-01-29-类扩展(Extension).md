---
layout: post
title: '类扩展(Extension)'
date: 2018-01-29
author: 李大鹏
cover: ''
tags: Objective-C
---

#### 1. 一般用扩展做什么？

- 声明私有属性
- 声明私有方法
- 声明私有成员变量

#### 2. 类扩展的使用

```
// 类扩展写在.m中
@interface ViewController ()
@property (nonatomic, strong) NSString *worker;
- (void)printSth; // 扩展方法
@end
```

#### 3. 扩展的特点

- 编译时决议
- 只以声明的形式存在，没有具体类的实现，多数情况下寄生于宿主类的.m 中，可以理解为内部的私有声明
- 不能为系统类添加扩展

#### 4. 分类和扩展的区别

- 分类是运行时决议，扩展是编译时决议
- 分类可以有声明和实现，扩展只有声明，实现写在宿主类中
- 可以为系统类添加分类，不能为其添加扩展
