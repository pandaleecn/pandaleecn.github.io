---
layout: post
title: 'Objective-C 中的对象'
date: 2018-02-04
author: 李大鹏
cover: ''
tags: iOS Runtime
---

本篇文章主要包含：

- 对象在内存中的布局
- 常用 LLDB 指令
- 对象的分类与关系
- 一个实例对象在内存中的数据结构

### 一、对象在内存中的布局

```
@interface NSObject {
  Class isa;
}

// ↓↓↓↓↓↓↓↓↓↓ Clang 编译后如下
struct NSObject_IMPL {
  Class isa;
};

typedef struct objc_class *Class;

NSObject *obj = [[NSObject alloc] init];
```

![2020-09-01-22-05-05](http://files.pandaleo.cn/2020-09-01-22-05-05.png)

```
@interface Student : NSObject {
  @public
  int _no;
  int _age;
}

// ↓↓↓↓↓↓↓↓↓↓ Clang 后
struct Student_IMPL {
  struct NSObject_IML NSObject_IVARS;
  int _no;
  int _age;
}

struct NSObject_IML {
  Class isa;
}

// 使用
Student *stu = [[Student alloc] init];
stu->_no = 4;
stu->_age = 5;

struct Student_IMPL *stu2 = (__bridge struct Student_IMPL *)stu;
NSLog("%d, %d, stu2->_no, stu2->age);
```

![2020-09-01-22-16-25](http://files.pandaleo.cn/2020-09-01-22-16-25.png)

- 一个 Person 对象、一个 Student 对象占用多少内存空间？  
  Person，isa 指针对象 8 字节 + 成员变量（int 4 字节、double 8 字节），当不足 16 的倍数时，倍数向上取整。
- 创建一个实例对象，至少需要 16 个字节。实际使用了 8 个字节，16 个字节是为了内存对齐。
- 创建一个实例对象，实际上分配 16 个字节的倍数，内存对齐。

```
@interface Person ： NSObject
{
  int _age;
}

@interface Student : Person
{
  int _no;
}

//  ↓↓↓↓↓↓↓↓↓↓ Clang 编译
struct NSObject_IMPL {
  Class isa;
}

struct Persion_IMPL {
  struct NSObject_IMPL NSObject_IVARS;
  int _age;
}

struct Student_IMPL {
  struct Person_IMPL Person_IVARS;
  int _no;
}
```

- 实时查看内存数据（也可以使用 LLDB 指令
  ）  
   `Debug -> Debug Workfllow -> View Memory （Shift + Command + M） `
- 查询对象占用的内存
  ```
  #import <objc/runtime.h>
  class_getInstanceSize([NSObject class]);
  ```
- 查询对象实际分配的内存

  ```
  #import <malloc/malloc.h>
  malloc_size((__bridge const void *)obj);
  ```

- 常用 LLDB 指令

  - print、p：打印
  - po：打印对象
  - 读取内存
    - memory read/数量格式字节数 内存地址
    - x/数量格式字节数 内存地址
    - x/3xw 0x10010
  - 格式
    - x 是 16 进制，f 是浮点，d 是 10 进制
  - 字节大小

    - b：byte 1 字节，h：half word 2 字节
    - w：word 4 字节，g：giant word 8 字节

  - 修改内存中的值
    - memory write 内存地址 数值
    - memory write 0x0000010 10

### 二. 对象的分类

#### 1. 实例对象（instance）

```
NSObject *object1 = [[NSObject alloc] init];
NSObject *object2 = [[NSObject alloc] init];
```

- instance 对象就是通过类 alloc 出来的对象，每次调用 alloc 都会产生新的 instance 对象
- object1、object2 是 NSObject 的 instance 对象（实例对象）
- 它们是不同的两个对象，分别占据着两块不同的内存
- instance 对象在内存中存储的信息包括
  - isa 指针
  - 其他成员变量

#### 2. 类对象（class）

```
Class objectClass1 = [object1 class];
Class objectClass2 = [object2 class];
Class objectClass3 = [NSObject class];
Class objectClass4 = object_getClass(object1); // Runtime API
```

- objectClass1 ~ objectClass4 都是 NSObject 的 class 对象（类对象）
- 它们是同一个对象。每个类在内存中有且只有一个 class 对象
- class 对象在内存中存储的信息主要包括
  - isa 指针
  - superclass 指针
  - 类的属性信息（@property）、类的对象方法信息（instance method）
  - 类的协议信息（protocol）、类的成员变量信息（ivar）

#### 3. 元类对象（meta-class）

```
Class objectMetaCClass = object_getClass([[NSObject class]]); // Runtime API
```

- objectMetaClass 是 NSObject 的 meta-class 对象（元类对象）
- 每个类在内存中有且只有一个 meta-class 对象
- meta-class 对象和 class 对象的内存结构是一样的，但是用途不一样，在内存中存储的信息主要包括

  - isa 指针
  - superclass 指针
  - 类的类方法信息（class method）

- `Class objectClass = [[NSObject class] class];` 中的 objectClass 是 class 对象，不是元类对象。
- 查看 Class 是否为 meta-class

```
#import <objc/runtime.h>
BOOL result = clas_isMetaClass([NSObject class]);
```

#### 4. 实例对象、类对象、元类对象之间的关系

- instance 的 isa 指向 class。当调用对象方法时，通过 instance 的 isa 找到 class，最后找到对象方法的实现进行调用；
- class 的 isa 指向 meta-class。当调用类方法时，通过 class 的 isa 找到 meta-class，最后找到类方法的实现进行调用；
- 当 Student 的 instance 对象要调用 Person 的对象方法时，会先通过 isa 找到 Student 的 class，然后通过 superclass 找到 Person 的 class，最后找到对象方法的实现进行调用；
- 当 Student 的 class 要调用 Person 的类方法时，会先通过 isa 找到 Student 的 meta-class，然后通过 superclass 找到 Person 的 meta-class，最后找到类方法的实现进行调用。

#### 5. 对象关系总结

- Root class，根类，没有父类，指向 nil
- 类对象和元类对象都是 objc_class 数据结构，都继承了 NSObject，所以可以通过 isa 指针查找。
- 任何元类指针对象都指向根元类对象，根元类对象的 isa 指针指向自己。
- 如果调用类方法没有对应的实现，但是有同名的实例的方法，是否会发生崩溃？会不会产生实际的调用？回答如下。
- 根元类对象的 superclass 指针指向根类对象。调用类方法不存在的情况下，由于 Root class（meta） 的 isa 指针指向根类对象，会查找实例方法并调用。
- 消息传递过程：调用实例方法时，先通过 isa 找到类对象的方法列表，查找方法实现，没有时顺次查找父类对象方法列表，最后到根类。如果没有找到，会转到消息转发流程。类方法类似，区别在于最后的根元类对象找不到会根据 isa 指针查找根类对象。
  ![2020-09-01-23-14-26](http://files.pandaleo.cn/2020-09-01-23-14-26.png?imageMogr2/thumbnail/!50p)

### 三、一个实例对象在内存中的数据结构

- class、meta-class 对象的本质结构都是 struct objc_class
- 从 64bit 开始，isa 需要进行一次位运算，才能计算出真实地址
  ![2020-09-01-23-36-19](http://files.pandaleo.cn/2020-09-01-23-36-19.png)
  ![2020-09-01-15-08-18](http://files.pandaleo.cn/2020-09-01-15-08-18.png?imageMogr2/thumbnail/!30p)
