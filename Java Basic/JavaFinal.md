## Final of Java，看这一篇就差不多了 ##

### 前言 ###

在Java中final是一个关键字，最近在研究和整理项目的代码，发现final出现的频次有些高，而且有些用法也是不知乎所以，所以一直也想整理一篇关于Java中final关键字使用和原理的文章，来梳理一下自己的思路。同时，也想从JVM一层来反过来看看final在Java程序源码中起到了怎样的作用。所以这篇文章的整体还是在于分析与梳理，而且有些点自己也是没有头绪，如果有理解不对或偏离的地方，请大家多指正并提出意见。

### 目录 ###
----------
- 介绍
- 使用
- 解惑
- 总结
- 参考与致谢


### 介绍 ###
----------
**final**是Java中的一个关键字，这个关键字有着很多种不同的用法，而且在不同的环境下，语义也不尽相同。所以，要想理解好**final**， 我们就需要将**final**在Java中的藏身之地一网打尽。  

**final**是一个关键字，在Java中表示为一个修饰符（Modifier），有时候对我自己来说，我也很好奇，这些修饰符是怎么起到作用的，例如我们举一个例子来说，一个被**final**修饰的类是无法有子类的，那么它为什么不能有子类，修饰符只是在Java语言层面上限制了这个关系，那么程序运行的时候系统是怎么知道的呢？后面也会简单的介绍这个内容。  

那么，**final**既然是一个修饰符，在Java中，**final** 能修饰哪些东西呢？基本上可以概括的说，在Java中基本可以修饰面向对象的绝大部分元素。我们可以通过如下的代码段来看**final**修饰的部分。

1.修饰类（Class）  
`public final class SystemUtils {...}`  
`public class OuterClass {  final class InnerClass{...}  }`

2.修饰方法（Method）   
`public final void foo() {...}`

3.修饰域（Field）  
`public final int fee = 25;`  
`private static final float POINT_X = 2.6f;`  
`public void foo() {    
   final int type = 3;  
}`

4.修饰方法参数（Method Argument）  
`public void foo(final int x, final int y) {...}`

由上面的代码片段我们可以看出来**final**关键字基本涵盖了所有的能出现的地方。那么我们就由外向内的来看，一层一层的来分析**final**的作用。  

其实，无论是在官方文档，还是一些blog上，对于**final** 的使用都是有很多独到的见解，并且也只是按照Class/Method/Field来分的，没有Method Argument这一个层次。这个后面我会具体的说明为什么我要单独说明Method Argument。

### 一、使用（类和方法） ###
这里允许我先介入一段引用，一个国外哥们写的，我在阅读他的文章时觉得很受益，所以贴出来，看一下他怎么评价**final**的使用！
> ### Is that your final answer? ###
> Most Java texts properly describe the usage and consequence of using the `final` keyword, but offer little in the way of the guidance as to when, and how often, to use `final`. In my experience, `final` is vastly overused for classes and methods(***generally because developers mistakenly believe it will enhance performance***), and underused where it will do the most good -- in declaring class instance variables. 
>  
> [http://www.ibm.com/developerworks/java/library/j-jtp1029/index.html](http://www.ibm.com/developerworks/java/library/j-jtp1029/index.html "Is that your final answer?")   
> 这是我认为说**final**最贴心的一篇文章了。

为什么我要这里贴出这么一段话，我希望我的这篇分享不会让大家对**final**有误解。没错，**final** 确实可以在某些情况使用的如文中提到的一样，但是这不代表它可以被随意使用或者滥用，这里我们只需要知道或者掌握**final**的使用方法，而如何在场景中合理使用是需要我们参考优秀的代码或者丰富的经验来完成**final**的使用。

好，有了上面的预先说明，我们开始说明**final**的第一层级，对于类的修饰。   
**final** 是一个修饰符（Modifier）可以来修饰一个类。 《Thinking in Java》一书中提到，我们利用**final**加以修饰元素，无外乎两个原因，一个是设计（Design）原因，一个是效率（Efficiency）原因，这是由于在不同的环境下**final** 有着不同的语义，所以可能会带来它的一些误解与误用。  

- 设计原因（Design）  
从设计的角度来考虑为**final**类，此时**final** 的语义表明为：**这个类不想在关系结构上做出任何的改变，也不希望有任何人可以继承自这个类，除此之外，就没有更多的限制了。** 以上是我们从类的设计角度来考虑类被**final** 修饰的情况。此时，我们还需要注意一点，一个类被**final**之后，它就禁止了继承关系，那么这个类中的所有方法都是**final** 修饰的，因为他们不会再被重写了，但是类中的域不会因此也被修饰为**final**的，大家需要注意这一点，不要误解。

- 效率原因（Efficiency）  
在我们说明**final**如何在效率上起到作用的时候，我们首先需要掌握一个知识点，即方法的**内联（inline）** 。我们在掌握了这个知识点之后，可能对于对于**final** 修饰方法也就一并掌握了。我们要说明final为一个类（Class）带来效率上的好处，还真的得研究到蛮深入的地步，这个深入的地步可以到JVM对于方法的调用处理，也可以深入到寄存器如何存储指令，在这里我们就一切从简的说，只要能把意思说通就可以了，想深入的研究的同学可以把JVM的知识学起来，同时Wikipedia上也提供了好多参考。  

言归正传，一个类被**final**修饰后，它的方法默认被修饰为**final** ，这时方法的内联起到作用了。对于Java语言的编译器来说，不同于C/C++，我们无需刻意地利用内联做什么，例如Java Hotspot Compiler这样的编译器，会自动地进行函数内联优化。那我们说了一堆，*What is the fucking inline?*（这是我看了几遍之后的心情！）现在我可以清淡如水的说一句： **内联（inline）在Java中就是编译器为程序做的一种优化操作。** 这句话算我看了10几遍之后的到一个最简单的总结了，那么想要理解**内联（inline）** 带来的优化，我们还要在补充一些知识。（理解内联真的需要很多知识点的串联），就是JVM的方法调用部分的知识。我们一定要明白一点，方法的调用和方法执行绝对不是一回事，这点在我去理解AspectJ的`call`和`execution`的时候，体会尤为深刻，现在看了JVM之后，有了进一步的了解。“方法调用阶段唯一的任务就是确定被调用方法的版本，不会涉及方法内部的具体运行过程。”，我们都非常熟悉一个`.java`文件在被编译器编译后得到了一个对应的`.class`文件，这个文件是一个二进制流文件，我们在类中定义的方法此时都被翻译成了字节码信息保存了起来，而方法的信息对应常量池入口地址都存在在一个`method_info`表中，同时这里也包含了access_flags，也就是编译器如何对**final**修饰的内容进行检查。
![ClassFile(part)](http://i.imgur.com/HR4yLnQ.png)

在`cp_info constant pool`和`method_info`包含了一个类中方法所有的信息，而在常量池中的方法信息便组成了一个**符号引用**，也是就说一个符号引用只是描述了一个方法的信息，而不是一个方法在实际运行时内存中的实际的入口地址（**直接引用**），那么也就说有一些方法需要在运行时才能知道目标方法的直接引用。

那么问题就来了，并不是所有的方法都是这样的装载-解析-执行过程。在一个class的解析阶段，JVM会完成一个任务，将那些不需要在执行阶段才知道直接引用的方法（如多态性）解析出来，即将常量池中的符号引用转换为直接引用。换句话说，我们要调用的目标方法在编译器编译的时期就要确定好是谁，而不是动态的等待哪个方法。那这跟内联有什么关系呢？我们接着向下分析。

在JVM中，方法被调用的instruction（指令）一共有5条：  
> 1. invokestatic	  ：调用静态方法  
> 2. invokespecial    ：调用私有方法、构造器方法、父类方法  
> 3. invokevirtual    ：调用虚方法  
> 4. invokeinterface  ：调用接口方法  
> 5. invokedynamic	  ：先在运行时动态解析出调用点限定符所引用的方法，然后再执行该方法  

在上面的给出的5条指令中，JVM利用`invokevirtual`指令调用虚方法，但是这里面有一个特殊的就是**final**修饰的方法，虽然调用**final**修饰的方法也是利用`invokevirtual`调用的，但是由于**final**修饰的方法无法被覆盖，所以也就无须对方法接受进行多态选择。那么final方法就和`invokestatic`调用的静态方法、`invokespecial`调用的私有方法等一并在解析阶段将符号引用解析为直接引用，所生成的直接引用将包含一个指向实际操作码的指针。

那绕了一大圈，我们终于回到了**内联（inline）**的分析上（这里写的有点啰嗦了），首先我们明确一点，内联是发生在编译期的一个优化操作，所做的优化操作的意义在于两个字，**“替换”** ，这种策略就是非常常见的一种利用**空间置换时间**的一个策略。我通过一个代码段来说明内联在做什么。  

优化前的原始代码：
   
    public static class Car {
    	double price;
    	final double getPrice() {
    		return price;
    	}
    }
    
    public void countDiscount() {
    	y = car.getPrice();
    	// ...
    	z = car.getPrice();
    	discount = y - z;
    }

内联后的代码：  

    public void countDiscount() {
    	y = car.price;
    	// ...
    	z = car.price;
    	discount = y - z;
    }

从以上的代码中，我们就可以很容就看出来内联的含义了，从代码的变化结果来看编译器所做的优化操作，依然都在`public void countDiscount()`方法中，`car`的调用不再是`getPrice()`方法，而是`price`这个返回结果，无论是`y`或者`z`，调用都完成了简化，这种简化是体现在方法的简化，这样做的好处显而易见，**消除了更多方法的调用**，较少方法的调用就意味在虚拟机栈中消耗的资源更少（如不会创建新的StackFrame栈帧），所有的调用都是由内联方法自己发起，自己完成弹栈压栈操作、恢复寄存器（恢复执行上下文）等。其实，朝着简单的方向理解，内联就是在执行***copy+replace***的动作，为什么这么说？因为我们在黏贴别人的或者重复的代码时候也在执行者一种类似于内联的操作，只不过，我们没有做到优化。被执行内联的方法会创建出很多个副本，比如我们的`countDiscount`方法中就有两处，如果还有别的方法有调用那么依然会被副本替换掉。这样减少了额外的调用，减少了系统开销，时间上会看似或确定有一定的缩短。

之所以我也没有用确定的语气说内联一定会带来优化效果，这是因为策略的问题，利用空间换时间，本身就需要一个平衡点（break-even），如果一个方法过于大，copy的副本数量过于多，那么这样的平衡就会被打破，优化的目的反而失去了意义。对于这种break-even我还是推荐两篇阅读材料。  

----------
1.Wikipedia上的关于内联的介绍  
[https://en.wikipedia.org/wiki/Inline_expansion#Language_support](https://en.wikipedia.org/wiki/Inline_expansion#Language_support "Inline expansion")  

2.IBM developerWorks上的一个blog，写的很贴心。  
[http://www.ibm.com/developerworks/java/library/j-jtp1029/index.html](http://www.ibm.com/developerworks/java/library/j-jtp1029/index.html "Is that your final answer?")  
 
----------

当然，针对于Java平台来说，不同的编译器对于内联的处理不尽相同，甚至不同语言内联也不尽相同，而且内联优化的意义不止在于对资源上，而是为了`Further Optimizations`，更深入的优化，所以，以上只是优化中的一点点收益的地方，大家不要以偏概全，如果想知道内联更多的细节，请自行阅读相关书籍吧。

#### 小结 ####
一切从简，这里**final**修饰了类和方法，基本上叙述了一遍，分析了一部分原理。如有疑问，欢迎大家吐槽并指正，thanks here! 
 
小结如下:  
1. final修饰类，更多从设计（Design）的角度去考虑吧，一个被**final**修饰的类无法子类化，即不能被继承。  
2. final修饰类，类中的方法默认都是**final** 修饰的。  
3. final修饰方法，如果从设计（Design）的角度去考虑，如果类之间体现了继承关系，那么**final** 修饰的方法则不能被子类重写或覆盖。如果没有体现继承关系，就从效率的角度考虑吧，但是**请切记：对于Java虚拟机来说编译器在编译期间会自动进行内联优化，这是由编译器决定的，对于我们开发人员来说，我们一定要设计好break-even的平衡，不要滥用final。**

### 三、使用（域） ###
----------
哈哈，相比于类和方法，**final**修饰域（Field）来说就简单多了，这个简单不是原理简单，是没有那么复杂的情况。  
**final**能修饰的域在我总结来看，2种类型。  

1. 基本数据类型   
2. 引用类型

首先是基本数据类型，这个是我们在写代码的时候，最常见的使用方法了。用法很简单：  

    public static final int ORDER = 1;  
    public final double fee = 25.62;
其实我一直认为**final**修饰基本数据类型的时候是最能体现**final**语义的一个用法。 处于一下两点考虑的时候，我们就要用**final**来修饰一个基本数据类型，注意：是基本数据类型。  

1. 程序编译期间的常量，它永远不会变。  
2. 在运行期间为一个**final**修饰的域初始化一个值，不希望它会发生变化。

那么这两点，就对应这我们代码中的第一条和第二条。我们先来一个图示。
![From class file to JVM](http://i.imgur.com/0sVmXt8.png)

在编译后得到的`.class`文件中，有这么一块内容，叫常量池。我们先不说这里面包含了其他什么东西，光从名字上来看，常量池一定要包含常量！没错，常量池中的确包含了常量，当然还有其他的内容，我们也不需要关心，那么一个类中被**final**修饰的域在这个时候就会被放入这个大池子中。至于为什么这么做？原因很简单，为了**效率**。 其实将一个基本数据类型修饰为**final**的目的最单纯最美好，就是希望它不要变。这样系统有就可以做一些优化操作，将这些常量值装在需要计算的过程中，让它们充当类似于宏的身份，换句话说，编译器可以在编译期间提前完成一些计算工作，省去了在运行时对于变量的相对复杂的操作。那么到这里就完成了么？其实不是的，这里要补充的一点就是一个编译期间的类文件中，常量池中的基本数据类型的常量是不知道具体的值是什么，换句话说，在文件编译过后，虽然知道一个域是常量，但是至于这个常量的具体内容是什么，此时是无从知晓的。具体原因在《Thinking in Java》中有这样的回答：

> *This difference shows up only when the values are initialized at run time, since the compile-time value are treated the same by the compiler.(And presumably optimized out of existence.)*

只有当运行时，常量才会真正的被赋值，对于`static`和没有`static`修饰的基本数据类型来说，是有差异的，差异就在于`static`修饰的域是在类载入的时候进行初始化的，所有实例共享同一个常量，同时Java虚拟机没有把它当作类变量，在使用它的任何类的常量池或者字节码流中直接存放的是它表示的常量值。这也就是图示中从执行引擎执行字节码开始之后，对应的常量被赋值，存放于内存中的方法区内。对了，一般情况下`final`和`static`修饰的常量要**大写**。 

基本数据类型的常量初始化的几个方式（如有遗漏请补充）：
    
    public class ConstTest {
    	// 直接初始化
    	private final int fee = 1;
		private static final ORDER = 1;
    
    	{
    		// 利用初始化块初始化
    		fee = 2;
    	}

		static {
			ORDER = 2;
		}
    
    	// 在构造方法中初始化
    	public ConstTest() {
    		fee = 3;
    	}
    }

对于引用类型来说，如果有**final**修饰一个引用类型变量，不是说明这个引用类型指向的实际地址的对象不可变，而是说这个引用不能再指向其他地址的对象，而对象本身是可以改变的。如书上说差不多，这确实有点迷惑。不过问题也不大，就说明一个变量的引用不能变而已，被固定了。对于这点，我就不做过多的解释了，因为它确实没什么典型的例子我能想到的，如果您有好的典型，也请在这里批注补充。

### 四、使用（内部类）###
------
在文章开始分类的时候，我们特意把这个**final**修饰方法的参数单独拿出来单独作为一个小结，之所以这么做，是因为这里面还有一些内容值得我们学习的。先上一段代码：

    public class Parcel {
    	public Destination destination(final String dest) {
    		return new Destination() {
    			public String readLabel() {
    				return dest;
    			}
    		};
    	}
    
    	public static void main(String[] args) {
    		Parcel p = new Parcel();
    		Destination where = p.destination("China");
			System.out.println(where.readLabel());
    	}
    }
看到了，**final**又出现了，这次**final**出现的场景在一个方法的列表中，对于出现方法列表中的final来说一共有两个含义，这里我们先结合方法的内部类来说明。首先如果使用编译器编写这段代码的时候，如果我们不对`dest`增加`final`参数，那么一个IDE是会报错。为什么会报错，我们先从方法执行入手。

首先，我们定义了一个`destination`方法，返回一个`Destination`类型的对象，同时传入一个`String`类型的`dest`名字的参数，但是，这个参数要求必须为`final`，接下来在方法的内部完成了一个创建和返回的动作，创建了`Destination`对象，并返回，但是我们需要给出Destination的具体实现，里面需要实现一个方法就是`readLabel`，它很简单，返回`label`就结束了。那么方法到了return返回了结果意味着结束。但是，问题来了，`readLabel`方法并没有执行啊！ 但是方法结束后该方法的栈帧已经被虚拟机栈弹出了，如果按照我们的想象，`label`还没用就没有了，这就不好了！
显然JVM不会这么做，在临释放`dest`之前，就将这个dest变量做了一次备份操作。当我们在创建Destination的对象的时候，dest就会被存入Destination实例中一个名字为`dest`的变量中，编译器必须检测对局部变量的访问，为每一个变量建立对应的数据域，并将局部变量拷贝到构造器中，以便**将这些数据域初始化为局部变量的副本**。

那么我们将方法参数列表中的变量修饰为final的，防止了这个变量在方法中被修改，因此就做到了局部变量与在内部类建立的拷贝副本保持了一致。

### 解惑 ###
------
在这个小结呢，我们来潜潜地分析一下编译器是如何感知**final**的存在，同时使用完成语义上的限制的（利用Class来举例子）。
在Java语言中，**final** 是一个修饰符，是一个**Modifier**，其实编译器根本不知道它的名字是什么，即使知道它叫final也没什么用，因为它不感兴趣，它只对数字感兴趣。尽管它不感兴趣，但是Java LanguageTools的源码中还是定义了一个枚举类Modifier.java



    // See JLS sections 8.1.1, 8.3.1, 8.4.3, 8.8.3, and 9.1.1.
    // java.lang.reflect.Modifier includes INTERFACE, but that's a VMism.

    /** The modifier {@code public} */          PUBLIC,
    /** The modifier {@code protected} */       PROTECTED,
    /** The modifier {@code private} */         PRIVATE,
    /** The modifier {@code abstract} */        ABSTRACT,
    /**
     * The modifier {@code default}
     * @since 1.8
     */
     DEFAULT,
    /** The modifier {@code static} */          STATIC,
    /** The modifier {@code final} */           FINAL,
    /** The modifier {@code transient} */       TRANSIENT,
    /** The modifier {@code volatile} */        VOLATILE,
    /** The modifier {@code synchronized} */    SYNCHRONIZED,
    /** The modifier {@code native} */          NATIVE,
    /** The modifier {@code strictfp} */        STRICTFP;

    /**
     * Returns this modifier's name in lowercase.
     */
    public String toString() {
        return name().toLowerCase(java.util.Locale.US);
    }

然而弄了枚举，编译器还是不知道什么是什么，为此，tools又定义了Flags类，它就有用多了。`final`修饰符在这里定义如下：  

    public static final int FINAL= 1<<4;

对于限制什么的，Java最擅长的就是利用mask掩码进行位运算，好像开关一样来控制需要的情况，对于访问控制标志位（Access_Flags）来说也一样的做法，看看源码：  
![Flags.java](http://i.imgur.com/acE9kL4.png)  

Java的沙箱为了保证装载的类文件的安全性，会在验证阶段对字节码流做多次的验证，那么其中就包括对各个类之间的二进制兼容的检查，其中就包括，  

- 检查**final**的类不能拥有子类  
- 检查**final**的方法不能被覆盖  

至于检查的方法就是根据Access_Flags做位校验了。如果有不满足的这里就不深入说了，内容比较庞杂，有兴趣的同学可以参考《深入Java虚拟机》一书。

### 总结 ###
------
1. final可以修饰类，方法，域。
2. final的使用很简单，合理且精确的使用final需要些经验和原理支撑。
3. final要从设计角度和效率角度综合考虑，对于方法和类来说，切勿滥用final。
4. final修饰域这是很有效的做法，可以适当减轻系统计算的负担。
5. final还有一部分内容涉及到并发，可能是我没有涉及到的。
6. final和private可以使用，编译器不会报错，但是没有什么意义。
7. final不能和abstract一起使用，因为语义是冲突的，很好理解。

以上就是我个人对**final**的一个理解和总结，资历尚浅，内容可能还不够有有深度，自己也认为有很多地方还可以在完善的，但是也希望别人给出意见，综合提升，所以欢迎大家吐槽、指责、交流。

### 参考和致谢 ###
------
BOOKS:  

- 《Thinking in Java》  
- 《Java 核心技术卷 I》  
- 《深入Java虚拟机》

BLOGS:  

- Inline Expansion  
[https://en.wikipedia.org/wiki/Inline_expansion#Language_support](https://en.wikipedia.org/wiki/Inline_expansion#Language_support)  

- Why is this class final?  
[http://www.ibm.com/developerworks/java/library/j-jtp1029/index.html](http://www.ibm.com/developerworks/java/library/j-jtp1029/index.html)

感谢顾（老大），在我给他讲解**final**使用问题的时候，让我认识到自己说不清楚，有想搞明白的冲动。  
感谢丹总，中午在商场一路请教他关于**final**的知识，给我讲了一路。