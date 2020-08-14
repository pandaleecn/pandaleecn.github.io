---
layout: post
title: 'UITableView相关'
date: 2018-01-18
author: 李大鹏
cover: ''
tags: iOS UI
---

#### 1. 重用机制

- cell = [tableView dequeueReusableCellWithIdentifier:identifier];
  ![2020-08-14-16-10-47](http://files.pandaleo.cn/2020-08-14-16-10-47.png)
- 自定义 UI 控件，字母索引条

#### 2. 数据源同步

在新闻和资讯类 App 中，经常会遇到需要数据源同步的场景。  
![](http://files.pandaleo.cn/f472037a4f9a8b5145549c8464c40b4f.png)

- tableView 在多线程的情况下，对修改和访问数据源同步的问题。
- 常见的解决方案分为以下两种：
  - 并发访问、数据拷贝，存在额外开销
    ![](http://files.pandaleo.cn/98fc1080e4f4bd0095c545318ce31e99.png)
    - 主线程数据拷贝，之后将结果给子线程使用，在子线程中进行网络请求、数据解析、预排版等操作
    - 在子线程进行操作的过程中，主线程删除一行数据 relaod 之后数据会消失，进行其他操作
    - 子线程返回结果，再次 reload 主线程，子线程返回的数据仍然包含主线程的删除内容，出现异常
    - 主线程删除时记录删除操作，在子线程返回数据更新时，同步删除操作，删除子线程拷贝的数据后 reload 主线程 UI
  - 串行访问，存在刷新延迟
    ![](http://files.pandaleo.cn/011a7c171c5ee22884e7fe9410694e1f.png)
    - 使用 GCD 串行队列，子线程网络请求回的数据在串行队列中进行预排版
    - 主线程删除数据时，等待串行队列子线程执行完成后执行删除操作
    - 删除后和子线程同步数据删除
    - 回到主线程 reload 刷新 UI
