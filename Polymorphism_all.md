# 多态性，看这一篇就可以了
## 前言
多态性（Polymorphism）是一个很大的话题，在很多的编程语言中都有它的身影。在我学习Java并利用Java工作的几年期间，我也经常会反思一个问题：“**到底什么是多态？**”。随着我经验的增加，抱着弄清什么是多态的目的，写了这篇文章。

写这篇文章的目的：**了解多态性的一切。**

## 目录
要想彻底弄懂多态性，分为以下几个方向：
- 到底什么是多态性？
- 为什么要有多态性？
- 多态性的原理（工作机制）
- 多态性的应用
- 总结

# 一、多态性的原理
先从原理出发，看看多态是如何在程序中表现的。至于什么是多态性？为什么要有多态性？我们放在后边，这些总结的话可以不急着说。

这里，我们以Java作为阐述帮助的编程语言。

在Java中，多态性从我们编写代码的一刻开始就存在了，这是第一个阶段，**程序员的意识阶段**。这会引出一个话题，“什么是多态性？”，这里不急着解释这个问题，在意识阶段，我们只要记住一点就可以了，那就是**多态性的出现使得我们编写的程序会通用，也就是所谓的可复用性更高，会降低成本**。虽然这话在我看来是屁话，但对于程序员来说，这是一个指导方针，要时刻牢记心中的话。以至于我们如何使用多态性完成这个指导方针，我会在《多态性的应用》一节说明。

OK，进入正题。多态性的原理分为几个阶段。从源代码被编译器（compiler）编译期间，多态性就正式开始了它的showtime。

第一个阶段：编译期间的多态性，又叫做“**Static Polymorphism 静态多态性**”。静态多态性是相对的，当然，这是相对于动态多态性（Dynamic Polymorphism）而言的，后者会在稍后篇幅讲解。编译器的主要工作就是要将源码（如Test.java）编译成字节码文件（Test.class）。在这个过程中，静态多态性开始工作。

### Static Polymorphism 静态多态性
静态多态性，由于是在编译时期完成的，所以又叫编译期多态性（compile-time polymorphism）。在编程语言中，可以利用静态分发（static dispatch）来完成。

静态多态性是多态性的一种，它在编译时期完全被解析。那么解析什么呢？**解析函数**。在很多编程语言中，函数（function）是已经成为一等公民（first-class citizen），在编译期间，有一部分函数需要被解析出来。

先来说什么是“解析”。
>Examples are templates in C++, and generic programming in other languages, in conjunction with function overloading (including operator overloading). Code is said to be monomorphised, with specific data types deduced and traced through the call graph, in order to instantiate specific versions of generic functions, and select specific function calls based on the supplied definitions.

简单的说，解析是**通过调用图推导和跟踪特定的数据类型，以实例化通用函数的特定版本，并根据提供的定义选择特定的函数调用**。看了这么一句话，估计会有点懵。在Java语言中，一个静态多态性的特征是：**Function Overloading - 函数重载**或者称作“**Method Overloading - 方法重载**”。
#### 1.Function Overloading or Method Overloading
所谓方法或者函数的重载，**即可以创建多个相同名字的函数却有不同实现的能力**。调用一个重载的函数是与函数所依赖的环境相关，也是根据合适的上下文来选择某一个重载的实现。方法或函数重载很好识别：
```Java
public class Task {
    public void doTask() {
        // do something about initializing
    }
    
    public void doTask(Work work) {
       // do work
    }
}
```
在`Task`中的成员方法`doTask()`与`doTask(Work work)`就是重载函数。这里大家可能会产生疑问，函数或者方法的重载与静态多态性有什么关系？要想弄清这个问题，必须弄清楚以下几个问题：
1. 为什么重载要发生在编译期间？
2. 编译器是如何处理重载的？
3. 重载如何体现多态性的？