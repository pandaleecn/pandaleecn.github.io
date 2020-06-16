---
layout: post
title: 'UITableView数据源同步方案'
date: 2018-01-18
author: 李大鹏
cover: ''
tags: iOS UI
---

在新闻和资讯类App中，经常会遇到需要数据源同步的场景。  
![](http://files.pandaleo.cn/f472037a4f9a8b5145549c8464c40b4f.png)
常见的解决方案分为以下两种：
##### 1. 并发访问、数据拷贝，存在额外开销  
![](http://files.pandaleo.cn/98fc1080e4f4bd0095c545318ce31e99.png)
##### 2. 串行访问，存在刷新延迟
![](http://files.pandaleo.cn/011a7c171c5ee22884e7fe9410694e1f.png)
