---
layout:     post
title:      "Objective-C 中的 runtime 机制及其应用场景(一)"
subtitle:   ""
date:       2018-10-23 
author:     "Mae"
tags:
- iOS
---

# Objective-C 中的 runtime 机制及其应用场景(一)

这篇博文旨在记录自己的学习过程，参考资料：wiki、《Effective Objective-C 2.0》等。

#### *什么是 runtime*

##### *运行时*

runtime 的字面意思就是 运行时，OC 是 C 语言的超集，在 C 语言的基础上为其添加了面向对象特性，而且， OC 与 C++、Java 等面向对象语言不同，它是使用 **消息结构** 的语言，这类语言在运行时所执行的代码是由运行环境决定的，无论函数是否是多态的，总是在 **运行时** 才会去查找所要执行的方法，划重点：**消息结构(messaging structure)**、**运行时 (runtime)**。

##### *消息传递*

消息传递是 OC 中的术语，也是 **runtime** 的核心，如果向某对象传递消息，就会使用动态绑定机制来决定需要调用的方法，当对象接收到消息以后，究竟该调用哪个方法是完全由 **运行时** 决定的，这也使得 OC 成为一门完全的动态语言；例如：

```objective-c
/**
 给名为 obj 的对象发送消息
 obj 为接收者
 messageName 为 selector
 接收参数 paramer
 返回结果 value
 */
id value = [obj messageName:paramer];
```

在底层，所有的方法都是普通的 C 语言函数，编译器在看到此消息后，会将其转换成一条标准的 C 语言函数调用，所调用的函数是消息传递机制中的核心函数 ` objc_msgSend` ，此函数原型为：

```c
void objc_msgSend(id self, SEL cmd, ...)
```

于是，编译器会把上面的例子转换成：

```c
id value = objc_msgSend(object,
                        @selector(messageName:),
                        paramer);
```

`objc_msgSend ` 函数会依据接收者和 selector 的类型来调用适当的方法， 为了完成此操作，该方法会在接收者的类里查找其方法列表，如果找到，就跳转至该方法实现代码，如果找不到，会沿着继承体系继续向上查找，若最终还是找不到，那么就会执行 **消息转发** 操作。

##### *跳转至该方法实现*

下面主要先说一下，**跳转至该方法实现** 这步操作的大致原理，之所以能这样做，是因为，每个 OC 对象的方法在底层都可以视作一个 C 语言函数，其原型与 ` objc_msgSend` 类似，大概 如下：

```c
<return_type> Class_selector(id self, SEL _cmd, ...)
```

每一个 class 里面都有一张 **表格(dispatch table)**[^1]，其中的指针都会指向上面这种函数，而选择器 selector 则是查表所用的 **key **，` objc_msgSend` 等函数正是通过这张表格来寻找对应方法并跳转至其实现的。

[^1]:  *(来自 wiki 的介绍：a **dispatch table** is a table of [pointers](https://en.wikipedia.org/wiki/Pointer_(computer_programming)) to functions or [methods](https://en.wikipedia.org/wiki/Method_(computer_science)).)* 



下一篇会继续记录 runtime 中的消息转发。