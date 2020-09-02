---
layout: post
title: 'Runtime的应用和API'
date: 2018-02-07
author: 李大鹏
cover: ''
tags: iOS Runtime
---

### 一、应用

#### 1. 利用 Runtime 遍历所有的属性或者成员变量

- 设置 UITextField 占位文字的颜色

```
self.textField.placeholder = @"请输入用户名";
[self.textField setValue:[UIColor readColor] forKeyPath:@"_placeholderLable.textColor"];
```

#### 2. 字典转模型

- 利用 Runtime 遍历所有的属性或者成员变量
- 利用 KVC 设值

#### 3.替换方法实现

- `class_replaceMethod`
- `method_exchangeImplementations`

### 二、API

#### 1. 类

- 动态创建一个类（参数：父类，类名，额外的内存空间）  
  `Class objc_allocateClassPair(Class superclass, const char *name, size_t extraBytes)`
- 注册一个类（要在类注册之前添加成员变量）  
  `void objc_registerClassPair(Class cls) `
- 销毁一个类  
  `void objc_disposeClassPair(Class cls)`
- 获取 isa 指向的 Class  
  `Class object_getClass(id obj)`
- 设置 isa 指向的 Class  
  `Class object_setClass(id obj, Class cls)`
- 判断一个 OC 对象是否为 Class  
  `BOOL object_isClass(id obj)`
- 判断一个 Class 是否为元类  
  `BOOL class_isMetaClass(Class cls)`
- 获取父类  
  `Class class_getSuperclass(Class cls)`

#### 2. 成员变量

- 获取一个实例变量信息  
  `Ivar class_getInstanceVariable(Class cls, const char *name)`
- 拷贝实例变量列表（最后需要调用 free 释放）  
  `Ivar *class_copyIvarList(Class cls, unsigned int *outCount)`
- 设置和获取成员变量的值

```
void object_setIvar(id obj, Ivar ivar, id value)
id object_getIvar(id obj, Ivar ivar)
```

- 动态添加成员变量（已经注册的类是不能动态添加成员变量的）  
  `BOOL class_addIvar(Class cls, const char * name, size_t size, uint8_t alignment, const char * types)`
- 获取成员变量的相关信息

```
const char *ivar_getName(Ivar v)
const char *ivar_getTypeEncoding(Ivar v)
```

#### 3. 属性

- 获取一个属性  
  `objc_property_t class_getProperty(Class cls, const char *name)`

- 拷贝属性列表（最后需要调用 free 释放）  
  `objc_property_t *class_copyPropertyList(Class cls, unsigned int *outCount)`

- 动态添加属性  
  `BOOL class_addProperty(Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount)`

- 动态替换属性  
  `void class_replaceProperty(Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount)`

- 获取属性的一些信息

```
const char *property_getName(objc_property_t property)
const char *property_getAttributes(objc_property_t property)
```

#### 4. 方法

- 获得一个实例方法、类方法

```
Method class_getInstanceMethod(Class cls, SEL name)
Method class_getClassMethod(Class cls, SEL name)
```

- 方法实现相关操作

```
IMP class_getMethodImplementation(Class cls, SEL name)
IMP method_setImplementation(Method m, IMP imp)
void method_exchangeImplementations(Method m1, Method m2)
```

- 拷贝方法列表（最后需要调用 free 释放）
  `Method *class_copyMethodList(Class cls, unsigned int *outCount)`
- 动态添加方法
  `BOOL class_addMethod(Class cls, SEL name, IMP imp, const char *types)`
- 动态替换方法
  `IMP class_replaceMethod(Class cls, SEL name, IMP imp, const char *types)`
- 获取方法的相关信息（带有 copy 的需要调用 free 去释放）

```
SEL method_getName(Method m)
IMP method_getImplementation(Method m)
const char *method_getTypeEncoding(Method m)
unsigned int method_getNumberOfArguments(Method m)
char *method_copyReturnType(Method m)
char \*method_copyArgumentType(Method m, unsigned int index)
```

- 选择器相关

```
const char *sel_getName(SEL sel)
SEL sel_registerName(const char *str)
```

- 用 block 作为方法实现

```
IMP imp_implementationWithBlock(id block)
id imp_getBlock(IMP anImp)
BOOL imp_removeBlock(IMP anImp)
```

### 三、思考题

- [obj foo]和 objc_msgSend()函数之间有什么关系？
  [obj foo] 在编译器处理后变成了 objc_msgSend 调用，第一个参数是 obj，第二个参数是 foo，开始 runtime 消息传递过程。
- runtime 如何通过 Selector 找到对应的 IMP 地址的？
  - 首先查找当前实例对应类对象的缓存是否有 Selector 的 IMP 实现，如果命中了，就会返回该实现给调用方；
  - 如果没有命中，就会根据当前类的方法列表，查找方法的实现；
  - 当前类没有命中，根据 superclass 指针，查找父类的方法实现。
- 能否向编译后的类中增加实例变量？
  - 首先区分编译后的还是动态添加的类
  - 编译之前，创建的类已经完成了实例变量的布局，class_ro_t。
  - 编译后的类无法修改，不能增加实例变量。
  - 如果是动态添加类，可以在添加过程中，调用注册类方法前添加可以实现。

```

```
