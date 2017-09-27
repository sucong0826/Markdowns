# Collecting Parameter 收集参数模式 -  难寻的模式与技巧

## 前言

关于**Collecting Parameter**，我最初关注它是在学习JUnit框架内容时，在《JUnit in Action》一书中，它被定义为一种Pattern（模式），可任凭搜索，关于它的资料少之又少，Google，Wiki很难找到一些内容来学习了解它。

我希望写这篇Blog来填充一下它的资料库，既是学习，也是贡献。进入正题！

## 模式与技巧

的确，关于**Collecting Parameter**资料少之又少。暂且这里把它叫做**收集参数**的模式，这是一个直译。目前我只在《Think in Patterns》中找到了它的身影，其次就是一些Blogs中有提到它。所以要当做一种设计模式来阐述它确实有些困难（虽然在《Think in Patterns》它以设计模式的身份出现），其次它又是一个不错的技巧且偏向一种模式来呈现和使用。

当然，无论怎么叫法如何，都当做技术来学习，至于其他的内容都通过实践来验证。这里暂且就叫**Collecting Parameter Pattern - 收集参数模式**。


## 什么是收集参数模式？

### 1. 正确理解Collecting Parameter的意图

如果仅从名字来分析这个模式，肯定会和此模式的原本意图产生歧义。参数，在一门编程语言中是函数组成的一部分。

```Java
public double add(double n1, double n2) {...}
```

```JavaScript
function (x) {
  ...
}
```
无论是`n1/n2`还是`x`，都是函数的参数，的确是这样。那么所谓的收集参数就是收集某一个函数的参数么？答案是：NO！收集参数绝不是收集一个函数的参数。

所以，千万不要误解Collecting Parameter就是收集参数的意思。它真实的意思是：
> 函数的参数是用来收集信息的。

这才是Collecting Parameters想要表达的意思。

### 2. 利用Collecting Parameter模式收集信息

在讲解该模式如何运作之前，先学一个基础内容，信使（Messenger），也就是**Data Object**。

我们都知道，假设在一个三维的坐标系上，任何一个点（Point）都可以被转化成一个向量（Vector）。

POINT(x, y, z) -> VECTOR

创建一个`Space`类，用来处理点到向量的转换。
```Java
public class Space {
    public static void translate(int x, int y, int z, Vector v) {
        // some oeraptions...
    }
}
```
什么是信使（Messenger）呢？
> It simplifies packages information into an object to be passed around, instead of passing all the pieces around separately.

根据上面的内容得知，信使的作用就是**将要传递的一条一条内容放到一个对象里去传递**。这样不仅会提高代码的可读性，而且易于测试与维护。在上面的例子中，`Space`类中的方法`translate`接收了三个参数`x`，`y`，`z`分别表示了三个维度坐标点的整型数值。如果`translate`方法要以转换为目的，那么这段代码的可读性与维护性就很差，而且对于一个点（Point）的其他信息也会一并失去控制，所以信使要发挥作用，也就是一个Data Object。

```Java
public class Point {
    public int x, y, z;
    public Point(int x, int y, int z) {
        this.x = x;
        this.y = y;
        this.z = z;
    }
    
    public Point(Point p) {
        this.x = p.x;
        this.y = p.y;
        this.z = p.z;
    }
    
    public String toString() {
        ...
    }
}
```
这里说一个**小技巧**：

虽然Java的特征之一有要求体现封装性，为了不让外部可以随意改动对象的域，这没问题。但是，在有些特殊情况下，我们可以打破这个规则，为了**效率**。假设以**Point**为例，我们依然建议将`int x,y,z`设置为`private`来遵守原则，但是，当`Space`类需要计算大量的`Point`对象或一个大型的`Point`数组时，为了效率可以将它们设置为`public`而可以被直接访问，这样做的话：

- 如果编译器不支持accessor的内联，那么大量的访问点`Point`通过accessors(getter/setter)会创建许多StackFrame（栈帧），虽然只有短短一行或几行代码，但是也会有资源浪费的情况。

- 如果编译器支持内联，那么可以省去内联优化，加快一些编译的速度。

接着向下说，既然有了信使，就可以修改`Space`部分的代码。
```Java
public class Space {
    public static void translate(Point p, Vector v) {
        // some oeraptions...
    }
}
```

有了新的`translate`方法，逻辑变得更加清晰。其实这部分内容很简单，如果具备Java的OOP的基础，就很好理解。那么，这部分内容和**Collecting Parameter**有什么关系呢？向下看！

> Quote from Think in Patterns
> 
> Messenger’s big brother is the collecting parameter, whose job is to capture information from the method to which it is passed. Generally, this is used when the collecting parameter is passed to multiple methods, so it’s like a bee collecting pollen.

无论是大哥还是小弟，总之，**Collecting Parameter**与**Messenger**的关系很大，密不可分。而**Collecting Parameter**的作用就是**利用一个传递到当前方法的参数去收集信息**。

![bottle](https://i.imgur.com/GqRCG1C.png)

可以这么来比喻，这里有一个瓶子，里面装满了液体。瓶口有些海绵，它专门吸收上层透明的液体。那么**Collecting Parameters**就是这块海绵。它和闭包的意思有些像，都是想从函数中携带些东西出去，但是用途和原理不相同。而它的使用场景是一个Collecting Parameter被传递到多个方法中时，这个模式通常会被使用到。

那么**Collecting Parameter**是如何实现的？
```
// samples from Think in Patterns
public class CollectingParameter extends ArrayList {
   ...
}

public class Filter {
    public void f(CollectingParameter cp) {
        cp.add("accumulating");
    }
    
    public void g(CollectingParameter cp) {
        cp.add("items");
    }
    
    public void h(CollectingParameter cp) {
        cp.add("as we go");
    }
    
    public static void main(String[] args) {
        Filter filter = new Filter();
        CollectingParameter cp = new CollectingParameter();
        filter.f(cp);
        filter.g(cp);
        filter.h(cp);
        String result = "" + cp;
        System.out.println(cp);
    }
}
```
既然是可收集信息的参数，那么再设计这个参数的时候一定要保证一点：**在这个参数中一定有某个方式添加或者插入值**。因此无论是上边提到的`Point`，还是这个例子中的`CollectingParameter`，它们都有添加值，也就是收集信息的能力。在这一点上，更加体现了信使的重要性。

当创建了`Filter`与`CollectingParameter`的实例之后，当在调用`f/g/h`方法时，我们都传递了`CollectingParameter`的实例进去收集各自的信息，收集的内容完全取决于调用的顺序于方法的实现。说到这里，我们不禁会问，这样做到底有什么意义？如果您也能考虑到这一点，那么就证明您没有完全被动地学习与吸收，而是动脑思考它真实地意义所在？这是极其正确的学习方式！

的确，当我第一遍学习与编写的过程中，的确是抱着这个疑问再写，我没有害怕写错，而是写一遍我就思考一遍。通过做对比的方式去研究。

（注：以下内容为我个人理解，毕竟Collecting Parameter的资料很少，能讲到为什么这么做的几乎没有，所以对于个人理解仁者见仁，如果您有更深刻的见解，也请不吝赐教。）

我在思考的过程中，给出了另一种实现，可能更为普遍，为什么不这么写呢？
```Java
public class Filter {
    private CollectingParameter inside;
    public Filter() {
        inside = new CollectingParameter();
    }
    
    public void f() {
        ...
        inside.add("accumulating");
        ...
    }
    
    public void g() {
        ...
        inside.add("items");
        ...
    }
    
    public void h() {
        ...
        inside.add("as we go");
        ...
    }
    
    public CollectingParameter getInsideCP() {
        return this.inside;
    }
}
```
我总是在心里想，为什么不这么写？而要造一个**Collecting Parameter**模式来。在我给出的实现中，结果是一样的，不同的就是信息收集的对象被封闭在了`Filter`之中，而不是作为参数暴露给调用者。

经过我的思考与查证，**Collecting Parameter Pattern**的使用要**根据具体的看场景与需求**。这里先对比着总结性地说：

首先最直观也是最大的变化，`CollectingParameter`从参数的身份变成了一个成员变量。这个身份的转变增加了`Filter`的负担与风险。如果作为收集信息的参数，那么`CollectingParameter`与`Filter`没有直接的关联，换句话说，`Filter`类根本不关心`CollectingParameter`是否存在或它有如何的业务逻辑。而`CollectingParameter`只是在对应的方法中作为参数传递进来拿走需要的信息之后就全身而退了，但既然成了`Filter`的成员，那么`Filter`就要额外的管理`CollectingParameter`，比如需要在自己的构造器内生成实例，还需要提供getter方法来返回它等。或许`Filter`本身的设计指南要求它不要组合一个`CollectingParameter`进来，毕竟二者没有直接的关联，`CollectingParameter`采集到的信息对于`Filter`本身的业务逻辑没有帮助，所以`Filter`**管理了一个它不要的内容，这当然是一个负担，也违反了单一职责的原则，同样会成为测试的阻碍**。

同时风险增加，这个很好理解，`CollectingParameter`的函数会暴露在`Filter`下，虽然`f/g/h`三个函数确实为它提供了需要采集的信息，但对于`Filter`其他的函数来说，同样可见。**如果误用或者恶意破坏，`CollectingParameter`采集到的信息未必是正确有效的，相反，可能是无效的**。

综上，若要使用好Collecting Parameter模式，使用原则就是**一定要针对具体场景与需求做分析，而不要随意使用**。良好的使用会使目标类更加清晰，负担更小，风险更低。但是滥用误用的结果就是类结果混乱，风险与负担更大。

具体如何使用好？ 还是看看人家怎么写的吧！

## 案例场景

### 案例一（正确案例）
------
第一个场景，我们来看看Android的一个案例，如何使用**Collecting Parameter pattern**完成任务的。

在`View`中，有一个方法：
```Java
public void getLocationOnScreen(@Size(2) int[] outLocation) {
    getLocationInWindow(outLocation);
    final AttachInfo info = mAttachInfo;
    if (info != null) {
        outLocation[0] += info.mWindowLeft;
        outLocation[1] += info.mWindowTop;
    }
}
```
相信大家对这个方法不会陌生，它用来计算一个View在屏幕上的位置，那么位置的坐标数据就保存在这个传入的`outLocation`的数组中。计算坐标的任务从头到尾都是利用`CollectingParameter`来完成的，无论是`getLocationOnScreen`还是里面调用的`getLocationInWindow`还是下一级调用的`transformFromViewToWindowSpace`来说，都应用了这种模式。

当然我们可以说，如果在以上这些方法内部构建一个长度为2的数组，作为参数去采集计算结果并作为`getLocationOnScreen`的返回值返回出来，不可以么？如：
```Java
public int[] getLocationOnScreen() {
    int[] outLocation = new int[2];
    getLocationInWindow(outLocation);
    ...
    return outLocation;
}
```
我只能说，不行！想法不错，但是不行。这违反了原本计算方法的设计。在`View`类中提供的API有：
1. `public void getLocationOnScreen(int[] outLocation)`
2. `public void getLocationInWindow(int[] outLocation)`
3. `@hide public void transformFromViewToWindowSpace(int[] inOutLocation)`

同时，方法1和2是同时提供给外部的API，但是1的内部调用了2，而2又调用了3，所以在设计上要将3个方法都返回一个数组这是不可能的，同时`View`类不能管理一个`int[] outLocation`，这样就会出现上面提到的负担与风险，而且关于组件位置的信息是保存在`AttachInfo`中进行管理的。所以，此时`int[] outLocation`就是一个**Collecting Parameter**。

当以设计API的角度来看**Collecting Parameter**模式时，将`int[] outLocation`传递到方法中是必然的。
> Generally, this is used when the collecting parameter is passed to multiple methods, so it’s like a bee collecting pollen.
> 
>通常这个模式在收集信息的参数被传递到多个方法中时被使用，就好像一个蜜蜂收集花粉一样。

### 案例二（正确案例） TestResult in JUnit framework

这部分对于有利用JUnit测试经验的人来说，应该是秒懂的。如果对JUnit没有经验的人，看以下的代码也没问题。

直接先贴一段源码：
![TestResult](https://i.imgur.com/3vExHs8.png)

这是`TestCase`中的一段代码，什么是`TestCase`呢？

`TestCase` 也叫测试用例：任何编写的测试用例都应该扩展了JUnit的TestCase类。它以`testXXX`方法的形式包含一个或者多个测试。一个`TestCase`把具有公共行为的测试归入一组。

那什么是`TestResult`？它表示一个测试结果，测试结果收集了一个`TestCase`的结果。它是收集参数模式(**Collecting Parameter** Pattern)的一个实例。这个测试框架区分了失败（failures）和错误（errors）。一个失败是可以预测的，而且它会被assertions检查。但错误（errors）是不可预期，比如一个数组越界异常。

那`TestResult`是如何完成信息采集的呢？从以上代码可以看到，`public void run(TestResult result)`方法需要一个`TestResult`作为参数，而`result`则调用了自己的方法`run(Test test)`，继续向下看。

![Method_run](https://i.imgur.com/BQQTUQt.png)

![run_protected](https://i.imgur.com/EIR7Ww0.png)
正如我上面介绍的，一个`TestResult`是收集信息的，的确，它会将`TestCase`执行过程中出现的错误和失败都收集起来，通过`addFailure(test, e)`和`addError(test, e)`两个方法就能看到收集的过程。以上的内容和形式都好理解，那么，我们要思考一个问题，JUnit的设计者为什么要这么实现呢？

答案似乎也很明了，以我的观点来看，这是`TestCase`的设计决定的！您可能觉得这是废话，先别急，我们一起来分析一下。

在案例二的第一幅图示中，我们看到一个`public TestResult run()`方法，这个方法并没有要求调用者传递一个`CollectingParameter`进来，也就是`TesetResult`，这是因为对于JUnit暴露给外部的规则并没有要求一定要创建出`TestResult`的实例，所以处于这点考虑，JUnit framework也没有必要在`TestCase`中强制放一个全局的`TestResult`对象。不过无论一个`TestCase`如何执行，得到什么结果，总要通过`TestResult`来表示，这时CollectingParameter模式就发挥用途了。虽然JUnit并不强制要求我们要创建`TestCase`对应的`TestResult`，这是因为系统会我们创建它，为了保持良好的实现，JUnit将`TestResult`与`TestCase`隔离开，目的就是希望`TestResult`只要采集到`TestCase`执行的结果信息就可以了，其它的部分而两者没有关联，也就没有必要通过`TestCase`作为载体来管理`TestResult`，即使是`public TestResult run()`方法中返回的结果也是在方法内部创建了`TestResult`作为返回结果，也不是`TestCase`维护的成员变量。

### 案例三（错误案例）Misleading name of a function

好钢用在刀刃上！如果用错了，可能麻烦会多了。虽然上面我们给出了两个标准使用方法，但是，在自己写代码的时候，想到这里动手写的时候也难免会出问题。什么问题？**容易用错**！

看一段代码：
```Java
public SomeModel GetModel(ViewData viewData) {
	viewdata["someKey"] = "someValue;
	// do some other stuff
 
	return new SomeModel(...); 
}
```
从方法名来看，`GetModel`方法利用`ViewData`的内容来帮助生成一个`SomeModel`。可能我们希望这个方法就是用来生成`SomeModel`，结果它巧合地符合了**Collecting Parameter**模式，这种情况下就会造成歧义。这种歧义体现在方法名与方法实现上的不一致。因此我个人更加推荐Android和JUnit的使用方法，就是**利用返回类型为`void`的函数去实现它**。

> Collecting Parameter Pattern must clear its intent.

## 参考及致谢

1. 《JUnit in Action》 Chapter 2
2. 《Think in Patterns with Java》Messenger and Collecting Parameter Pattern
3. Blog：https://dzone.com/articles/coding-collecting-parameter

## 总结
总结一下，Collecting Parameter模式内容很难找到，这篇算是一个自我总结的过程，这个模式虽然内容很少，但是无论在Android或者JUnit中都有大量的体现，所以从源码出发去体会它还是一个很好的途径。我依然强调要从类设计的角度和实际场景和需求去尝试**Collecting Parameter**模式。至于其他的，等我想到了再补充也不迟。

