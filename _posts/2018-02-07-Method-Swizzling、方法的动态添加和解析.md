---
layout: post
title: 'Method-Swizzling、方法的动态添加和解析'
date: 2018-02-07
author: 李大鹏
cover: ''
tags: iOS Runtime
---
#### 1. Method-Swizzling替换方法
![](http://files.pandaleo.cn/a9f86d402d7fb07f303f40795465c81e.png)
```
+ (void)load {
    Method test = class_getInstanceMethod(self, @selector(test));
    Method otherTest = class_getInstanceMethod(self, @selector(otherTest));
    method_exchangeImplementations(test, otherTest);
}

- (void)test {
    NSLog(@"test");
}

- (void)otherTest {
    [self otherTest];
    NSLog(@"otherTest");
}
// 先打印test，然后打印otherTest，两个方法替换了。
// 实际开发中可以替换view生命周期方法，添加日志能操作
```

#### 2. 动态添加方法
* performSelector:  

```
void testImp (void) { }
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    class_addMethod(self, @selector(test), testImp, “v@:”);
}
```  

#### 3. 动态方法解析
@dynamic
* 动态运行时语言将函数决议推迟到运行时。当属性标记为dynamic时，不需要编译器在编译时生成属性的getter和setter，在运行调用时再添加具体实现。
* 编译时语言在编译器进行函数决议。
