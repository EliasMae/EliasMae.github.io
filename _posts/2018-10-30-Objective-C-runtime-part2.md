---
layout:     post
title:      "Objective-C 中的 runtime 机制及其应用场景(二)"
subtitle:   "消息转发机制"
date:       2018-10-30 
author:     "Jacob"
tags:
- iOS

---



> *此博文旨在记录自己的学习过程，参考资料：wiki、Apple、《Effective Objective-C 2.0》等。*

#### *消息转发机制*

##### *基本概念*

上一篇提到 OC 是一种动态语言，在运行时才会去查找需要执行的方法，用 OC 的术语就是说在运行时进行消息的传递。所以，如果我们向类发送了其无法理解的消息，在编译期间是不会报出异常的，因为在运行时可以继续向类中添加方法，这也是 OC 动态性的体现，基于此，编译器在编译时还不确定类中是否有某个方法实现，当对象接收到无法解读的消息时，就会启动 **消息转发机制** ，我们可以在这个过程中去告诉对象如何处理未知消息。

##### *我们遇到过的消息转发处理*

在日常的工作开发中，我们免不了会遇到一些常见的崩溃问题，比如下面的这种报错信息：

```
-[__NSDictionary0 lowercaseString]: unrecognized selector sent to instance 0x6000003900e0
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[__NSDictionary0 lowercaseString]: unrecognized selector sent to instance 0x6000003900e0
```

这段报错信息表明，类型为 `__NSDictionary0` 的消息接收者无法解读名为 `lowercaseString` 的方法，其实我们也很清楚，`NSDictionary` 类中原本也没有 `lowercaseString` 方法，这种异常情况就是消息转发过程以失败告终的体现。

**消息转发** 分为两个大阶段。第一阶段叫做 **动态方法解析** ，先征询接收者所属的类，看其是否能动态的添加方法来处理当前无法识别的 **selector**。第二阶段涉及 **完整的消息转发机制** ，如果第一阶段已经执行完毕，那么接收者就没办法再动态添加方法。此时，**运行时系统(runtime)** 会请求接收者用其它手段来处理。首先，要查找有没有其它对象能处理这条消息，如果有，运行时系统就会把消息转发给那个对象，消息转发过程结束。如果没有其它对象可以处理这条消息，则启动 **完整的消息转发机制 **，运行时系统会把与消息有关的东西封装到 `NSInvocation` 对象中，再给接收者 **最后一次** 处理这条消息的机会。

##### *动态方法解析*

在对象接收到无法解读的消息时，首先会调用其所属类的类方法：

`+ (BOOL)resolveInstanceMethod:(SEL)sel`

方法参数 `sel` 就是当前未能识别的 selector，返回值是 BOOL 类型，表示这个类能否新增一个 **实例方法** 来处理这个 selector，如果不是 **实例方法** ，而是 **类方法** 的话，这里需要调用的方法则应该是：`+ (BOOL)resolveClassMethod:(SEL)sel`  。

使用这种方法的前提是相关方法的实现代码已经写好，只等着在运行时，动态插入到类里面即可。在日常的开发工作中，我们会见到 `@dynamic`  关键字，它的语义与 **动态方法解析** 十分吻合，`@dynamic` 关键字会告诉编译器：不要自动创建实现属性所用的实例变量，也不要为其创建存取方法。然后，在编译器编译这些属性代码时，就算发现没有存取方法也不会报错，因为它知道代码在运行时还会有动态方法解析步骤。例：

```objective-c
/**
 * .h 文件
 */
@interface class_name : NSObject
// 定义一个字符串属性
@property (nonatomic, copy) NSString *string;
@end
  
@implementation class_name
  
/**
 * 将 string 声明为 @dynamic
 * 此时，编译器不会自动为其生成 实例变量 & 存取方法
 */
@dynamic string;

// 重点：实现动态方法解析
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    NSString *selectorString = NSStringFromSelector(sel);
    NSLog(@"selectorString is %@", selectorString);
  	...
      
    return YES;
}

```

运行程序，可以看到输出结果：

```c
selectorString is setString:
```

这里简单记录一下处理流程：由于 `@dynamic` 关键字的原因，编译器并没有自动为 `string` 生成存取方法，然后在运行期间，`class_name` 接收到无法处理的 selector，也就是打印结果中的 `setString` ，此时，我们在 `resolveInstanceMethod`  方法里，可以通过 `class_addMethod` 方法为这个类动态添加存取方法。如下：

```objective-c
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    NSString *selectorString = NSStringFromSelector(sel);
    NSLog(@"selectorString is %@", selectorString);
  
  	/**
     * 如果 selector 为 set 方法
     * 利用 class_addMethod 方法为这个类添加 method_name 的方法
     * "v@:@" 表示 void 方法 且有一个参数传入
     */
  	if ([selectorString hasPrefix:@"set"]) {
        class_addMethod(self, sel, (IMP)method_name , "v@:@");
    } else {
        ...
    }
      
    return YES;
}
```

未完见下篇。