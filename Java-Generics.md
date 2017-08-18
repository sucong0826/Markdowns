# Java泛型，看这一篇差不多了

### 写在开始
---------------
无论怎样，在我看来泛型（Generics）在Java中都有着神秘的面纱。经过了工作中的使用，平日的学习和整理，泛型的某些点或者要素依然叫不准，遂奋笔疾书整理了这一篇，从内到外，由浅入深的透彻地分析、整理一次泛型，目的在于巩固与分享。

为了更好的了解泛型的由来，我找到了JSR-00014这篇归档说明来作为我阐述的依据，同时在类型推断（Type Inference），类型擦除（Type erasure），也要从JVM的方面来体会泛型的处理过程，这会更加深入的帮助我们来理解泛型的原理。

好，进入正文吧！Let's start ^.^


## 目录
----------------
- 什么是泛型
- 泛型的使用
- 泛型的原理
------

### 1. 什么是泛型？              
想彻底弄明白什么是泛型比较难，首先我不敢保证这篇文章会起到多大的作用，不过我会尽我所能把我所搜集到的材料整理清楚，同时根据我的理解写一些主观的观点，所以，我也希望读者可以保留自己的观点，我们一起讨论。

![](/JSR_01.jpg)

我们先看看JSR-00014中，泛型说明文档中大家是如何说明泛型的，JSR并没有开篇急于说明什么是泛型及它的语法，而是先说明了泛型的好处。

> Quote from &lt;Adding Generics to the Java Programming Language: Participant Draft Specification&gt;
>
> *The main benefit of adding genericity to the Java programming language lies in the added expressiveness and safety that stems from **making type parameters explicit** and **making type casts implicit**. This is crucial for using libraries such as collections in a flexible, yet safe way.*

要增加泛型（Genericity）到Java中是为了两个优点：
1. 使类型参数显式（make type parameters explicit）
2. 使类型转型隐式（make type casts implicit）

这两个优点是总结出来的，要想凭借这点信息去理解全部要是不大现实的，所以，请先保留一个疑问。不管怎么样，至少在这样的信息中我们得到了一个共同的特性，无论泛型如何总结，都离不开“类型”，这个“**类型**”就是JSR中提到的**type**。 众所周知，Java是面向对象的语言，当我们利用它来描述一个对象的时候，需要为这个对象提供各种信息（数据），那么信息就要对应一个类型。在Java中我们利用一个整型类型（int）来描述年龄，利用浮点型（float）来描述身高或体重，利用字符串类型（引用类型）来描述名字等。的确，如上面提到的，在Java中，原始数据类型（primitive type）和引用类型（reference type）是为数据提供了确切的类型，指导了系统该在内存中如何分配空间存储它们。
