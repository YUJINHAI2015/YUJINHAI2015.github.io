---
title:  运行时系统结构（第三篇）
date: 2020-05-16 21:52:34
tags:
categories: runtime
---
## 运行时系统结构

[TOC]

> 有编译器和运行时系统库组成

1、编译器：输入源码文件，生成使用运行时系统库的代码，输出可执行的二进制文件。

2、运行时系统库：是一个标准函数库，翻译Objective-C代码。



### 1、对象消息--》转换成运行时系统库代码

> 会调用运行时系统库中的函数`objc_msgSend()`

原来：`[接收器 消息];`

转换后：`objc_msgSend(接收器，选择器，参数);`

注意：每条消息都是动态转换的，也就是接收器和对应的方法都是在运行的时候决定的。



### 2、类和对象--》转换成运行时系统代码

> 会生成运行时数据结构

objc中的类  ---》objc_class

objc中的对象 ---》 objc_object

```objc
// 类：
typedef struct objc_class *Class;

// 对象：
struct objc_object 
{
  Class isa;
};

// id: 是一个结构指针
typedef struct objc_object 
{
  Class isa;
} *id;
```



> 代码查看

{% asset_img image-20200517005116741.png output %}

可以看到这段代码：

test1: isa指针（a0cffb0a 01000000）和val变量（07b20100 00000000）

test2: isa指针（a0cffb0a 01000000）和val变量（0e640300 00000000）

Calculator类：0x10afbcfa0 是低字节排序

因为test1和test2都是同一个类的实例，所以他们的isa指针都是指向该类的内存地址。



### 3、运行系统库API

- 数据类型：
  - 类定义数据结构（类、方法、实例变量、分类、IMP、SEL）
  - 实例数据类型（id、objc_object和objc_super)
  - 值（BOOL)
- 函数：
  - 对象消息
  - 类函数
  - 实例函数
  - 协议函数
  - 方法函数
  - 属性函数
  - 选择器函数

### 4、使用运行时系统创建类

```objc
@implementation ViewController

NSString* hello(id self, SEL _cmd) {
    NSLog(@"helloworld");
    return @"";
}
- (void)viewDidLoad {
    [super viewDidLoad];
        // 创建一个类
    Class OtherClass = objc_allocateClassPair([NSObject class], "MyClass", 0);
    // 函数将以动态的方式添加到类中的方法实现中
    Method description = class_getInstanceMethod([NSObject class], @selector(description));
    const char *types = method_getTypeEncoding(description);
    class_addMethod(OtherClass, @selector(hello), (IMP)hello, types);
      // 注册该类
    objc_registerClassPair(OtherClass);

    // 使用
    id other = [[OtherClass alloc] init];
    
//    NSLog(@"%@", objc_msgSend(other, NSSelectorFromString(@"hello")));

}
@end

```



### 5、使用运行时系统传递对象消息

步骤：当对象发送消息时，运行时系统会查找类方法缓存和虚函数表。

- 运行时系统库中的方法数据类型

  ```objc
  struct objc_method
  {
      SEL method_name; // 方法的名称
      char *method_types; // 方法参数的数据类型
      IMP method_imp; // IMP变量
  };
  typedef objc_method Method;
  ```

- 虚函数表：存储IMP类型数据的数组

- 消息派发：objc_msgSend() 函数会寻找（与消息指定的接收器和选择器对应的）IMP指针。

对象--> 类对象--> 元类

1、当向对象发送消息时：运行时系统会通过相应的类实例方法虚函数表

2、当向类发送消息时：运行时系统会通过该类的元类类方法虚函数表




