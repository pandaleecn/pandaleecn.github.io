---
layout: post
title: 'Method-Swizzling、方法的动态添加和解析'
date: 2018-02-07
author: 李大鹏
cover: ''
tags: iOS Runtime
---

#### 1. Method-Swizzling 替换方法

![](http://files.pandaleo.cn/a9f86d402d7fb07f303f40795465c81e.png)

- 现有类中有两个方法，selector1 对应 IMP1，selector2 对应 IMP2。
- 经过 Method-Swizzling 后，selector1 改为对应 IMP2，selector2 改为对应 IMP1。
- 当给对象发送消息 1 时，实际执行 IMP2，发送消息 2 时，实际执行 IMP1。

```
#import <objc/runtime.h>
// 导入runtime 头文件

+ (void)load {
    Method test = class_getInstanceMethod(self, @selector(test));
    Method otherTest = class_getInstanceMethod(self, @selector(otherTest));
    method_exchangeImplementations(test, otherTest);
}

- (void)test {
    NSLog(@"test");
}

- (void)otherTest {
  // 系统层面转化为 objc_msg_send，通过选择器因子查找 IMP 具体实现
  // 实际调用 test 的实现，不会新产生死循环
    [self otherTest];
    NSLog(@"otherTest");
}
// 先打印test，然后打印otherTest，两个方法替换了。

// 实际开发中可以替换view生命周期方法，添加日志能操作
// 替换类的viewDidLoad/ViewWillAppear
```

#### 2. 动态添加方法

- performSelector:，关于 runtime 动态添加方法的特性

```
void testImp (void) {
    NSLog(@"test invoke");
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    // 运行时动态添加方法的执行体
    // para1，目标类
    // para2，方法选择器名称，para2-4，method_t 的成员变量
    // para3，方法具体实现
    // para4, 方法对应返回值类型和个数
    if (self  == @selector(test)) {
      class_addMethod(self, @selector(test), testImp, “v@:”);
      return YES;
    } else {
      ……
    }
}
```

#### 3. 动态方法解析

- @dynamic，编译时语言和动态运行时语言的区别。

  - 动态运行时语言将函数决议推迟到运行时。当属性标记为 dynamic 时，不需要编译器在编译时生成属性的 getter 和 setter，在运行调用时再添加具体实现。
  - 编译时语言在编译器进行函数决议。在运行时无法修改。
