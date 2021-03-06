---
layout: post
title: '代理(Delegate)和通知(Notification)'
date: 2018-01-31
author: 李大鹏
cover: ''
tags: Objective-C
---

### 一、代理(Delegate)

#### 1. 定义

- 准确的说是一种软件设计模式
- iOS 当中以@protocol 形式体现
- 传递方式一对一

#### 2. 工作流程

![](http://files.pandaleo.cn/47d8b2c96e4be4a314084d6a2aff9d1a.png)

- 协议：委托方要求代理方将需要实现的接口全都定义在协议中，可以声明属性、方法
- 协议中声明的方法由代理方负责按照协议实现
- 代理方可能返回一个处理结果，委托方调用代理方遵从的协议方法
- 是否必须实现：require 需要实现，option 可选

#### 3. 一般声明 weak 以规避循环引用

![](http://files.pandaleo.cn/40b9f4117a0df27d7aaed3c936b273db.png)
在使用代理模式开发，建立委托方和代理方的关系时，需要注意规避循环引用。

### 二、通知(Notification)

#### 1. 特点

- 是使用观察者模式来实现的用于跨层传递消息的机制。
- 传递方式为一对多。

#### 2. 与代理的区别

- 代理使用代理模式实现，通知使用观察者模式实现
- 代理的传递方式一对一，通知一对多。
  ![](http://files.pandaleo.cn/640e8564c43f266a23749896d60ff5bd.png)

#### 3. 如何实现通知机制？

![](http://files.pandaleo.cn/d99ab308b5465be15d14753c7b65584d.png)

- 在通知中心（NSNotificationCenter）中维护一张表（Notification_Map）
- key 是 notificatioName，value 是 Observers_List
- 列表中每个成员包含通知接收的观察者和观察者调用的方法
