---
layout: post
title:  "Python中的递归"
date:   2017-05-25 15:34:53 +0800
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
---

## 闲话
从去年年初就想找时间学一下Python这门语言，一直到前不久才付诸实践，通过自己看书以及网上的很多资料，也算是在一步步的进行中，个人博客也是在自己慢慢摸索着直到最近才有空搞出个框架，借助GitHub上的开源模板，也算是有些样子，废话少说，接下来开始记录我在学习Python这门语言上遇到的一些自认为比较重要的知识点
## 正文
我们都知道，函数内部可以调用其自身，那么这就是递归函数，具体概念不赘述，有需要的请移步[廖雪峰的官方网站](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431756044276a15558a759ec43de8e30eb0ed169fb11000)，这里着重以汉诺塔算法记录一下Python的递归函数，加深理解：

有3个柱子A、B、C，n个盘子由大到小堆叠在A柱，现在需要将A柱的盘子借助B柱全部移动到C柱，并且在C柱子由大到小堆叠。

编写函数`hanoi(n, a, b, c)`参数`n`代表A柱的盘子数量，`a` `b` `c`分别代表三个柱子，下面开始实现函数：



```python
def hanoi(n, a, b, c):			#实现函数
    if n == 1:				#当盘子数量n为1时，此时不需要借助B移动
        print(a,'->',c)		
    else:					
        hanoi(n-1, a, c, b)		#将n个盘子借助B移动至C，那首先需要将n-1个盘子借助C移动至B
        hanoi(1, a, b, c)		#将n个盘子除去n-1个移动至B的盘子，此时为n-(n-1)盘子移动至C
        hanoi(n-1, b, a, c)		#最后一步，将在B柱上的n-1个盘子通过A移动至C
hanoi(3 ,'A','B','C')
```

运行代码，期望的输出结果：

```python
A -> C
A -> B
C -> B
A -> C
B -> A
B -> C
A -> C
```

通过递归函数，可以简单的实现汉诺塔算法。
