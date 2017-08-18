## Java防空指南（NullPointerException），看这一篇就差不多了

### 前言
------
作为一名程序员，尤其是将Java语言作为常用编程语言的人们来说，NullPointerException实在是在普通不过的了。但是即使是普通且常见的问题，没有任何人敢保证自己实现的代码可以万无一失，尤其是在NullPointerException面前。同时，作为NullPointerException(NPE)的设计者来说，这也是一个很头疼的问题。 在我阅读的过程中一些优秀的文章中有意无意的会提及NPE的事情，所以，今天无论如何我要把Java以及Android系统中的防空指南写出来，为他人也为自己梳理一遍，以作备用。 其实我很早就想写这篇文章，但是拖延症晚期的我还是拖了很久，这其中99%的时间都在用来修复bug，这对于程序员来说是伤害蛮大的一件事，修复bug的背后应该是考虑为何出现了这个bug，而不是主要为了完成任务而修复问题。所以，在我反思的过程中，NPE是一个拦路虎，所以我决定不能拖延下去，趁热将这篇写出来，结合准备的资料，一气呵成。 在Google上，StackOverflow上大家可以搜索关于NPE的任何资料，资料非常多，但是也很杂，所以我希望用最清晰且最全面的方式给大家带来我个人的理解，以及优秀文章中的理解。

### 目录
------
1. 什么空指针异常（NPE）？
2. NPE Classics 典型场景
3. Java与Android的防空指南
4. 参考与引用

------

### 一、 什么是空指针异常？（NPE : NullPointerException)
------
空指针异常，英文名称**NullPointerException**。 要想解决NPE，我们就要从根源开始了解NPE到底是个什么东西。在我看来，要想弄懂NPE，至少要从以下三个点切入：

1. 指针		(**Pointer**)
2. 空状态    (**Null**)
3. 异常      (**Exception**)

为什么要先了解指针？正如NPE的名字一样，我们得到这个异常就知道是指针的问题，但是有些同学会问Java中不是没有指针么？这句话是不正确的或者是片面理解的，Java语言借鉴其他语言的优势与特点的同时，也会改进其他的语言的问题和缺点，其中一个就是指针。指针的概念是**C语言**中的核心特色，但是这样的核心特色却也带了很多问题，如指针的管理不当会造成对应的内存错误，间接引用坏指针，误解指针运算等等。所以Java语言的设计者认识到了**指针**带来的好处和问题，因此在Java语言中放弃了由开发人员操作和管理指针而交由系统来进行处理，并不是指针在Java语言中不存在了。

#### 理解指针（What is Pointer ?）
> Quoted from 《深入理解计算机系统》
>**指针(Pointer)**是C语言中的核心特色，指针以一种统一的方式，对不同数据结构中的元素产生引用。指针类型并不是**机器代码**中的一部分；它们是C语言提供的一种抽象，帮助程序员避免寻址错误。

简而言之，一个指针用来对一个元素产生对应的引用。看如下的结构图和示例：
| NAME     | TYPE     | VALUE               |
 -----------|-----------|-------------------|
 int *num   |int        |Address(0x205)

![Adressing](http://i.imgur.com/yW5VBic.png)

如图所示，每一个Pointer指针都有一个TYPE（类型）和VALUE（值），TYPE（类型）说明了一个指针都对应一个类型，这个类型表明该**指针指向的是哪一类对象**，如`*num` 都指向int类型的对象。同时每一个指针都有一个值，即VALUE（值），是**指定类型对象的地址**， 如0x205，一个内存地址（address）。从引用的内容中可以得知，指针Pointer并不是机器代码的一部分，而是避免寻址错误，那么什么是寻址呢？这是计算机系统的一个知识，这里简单的介绍一些：所谓**寻址**简单地说就是**寻找指定的内存单元的地址**，在大部分的机器上，多个字节的对象在一块内存空间中被存储为连续的字节序列，一个对象的地址就是所使用内存空间的最小的地址数。举个例子，上面的 `int *num` 是一个int类型的指针，那么VALUE就是这个变量所占用的内存空间中最小的内存地址，假设int是一个32位的，`32bits = 4byte * 8bits` ，所以共计4个字节，连续的内存空间的地址为`0x205, 0x206, 0x207, 0x208`。 那么寻址就是找到这个类似于`0x205` 这个内存地址。

> Quote from Wikipedia
> 
> ***Unlike C, C++, or Pascal, there is no explicit representation of pointers in Java. Instead, more complex data structures like objects and arrays are implemented using references. The language does not provide any explicit pointer manipulation operators. It is still possible for code to attempt to dereference a null reference (null pointer), however, which results in a run-time exception being thrown. The space occupied by unreferenced memory objects is recovered automatically by garbage collection at run-time***

与C/C++/Pascal不一样的是，在Java语言中没有**显式**的指针，这和开始提到的内容是一致的。的确指针的操作和管理实在是对开发者考验很大，所以这种情况在Java语言中得到了简化，使用了**references**，即**引用**。在Java语言的世界中，它是OOP的典型代表，一切事物都可以被视为对象，而这些对象的操纵标识实际上是对象的一个**引用**。Java的设计者有意的将**指针（Pointer）**与**引用（Reference）**这个两个概念区分开，但是这同样是一个有争论的话题，所以，无论是**指针（Pointer）**还是**引用（Reference）**，在我看来，理解原理是很重要的，我想既然NPE的设计者选择了**NullPointerException**作为名字，是合理的，因为Pointer更加贴近基于底层系统的实现吧。

OK，有了以上的关于**指针Pointer** 的简单内容作为铺垫，我们开始向下进发。


#### 理解空状态（What is Null ?）
这里我要说明一下，为什么我要说这是“空状态”。在我理解了上面的内容之后，我更加倾向于“空的状态”，原本我只片面的认为是空的值，即null values。在“理解指针”一节中我们得到一个指针的基本结构包括了一个TYPE和一个VALUE，VALUE这个内容更能说明我们的主题，就是NPE的话题，不过，在绝大部分的系统或者语言中，空状态就是空值，二者可以理解为一个含义，都是说明了指针中VALUE的部分没有值，即NULL(0)，表示该指针没有指向任何地方。只不过在一些对象关系映射型的数据库中（ORM Database）会有区别，所以这段话想表明的就是我理解为空状态，即Pointer为空状态时VALUE是没有值的，但是除了特殊的场合之外，空状态就是Null Pointer的意思。

那么，有了上面的铺垫，我们就很好理解Null Pointer是中的**Null**是什么意思了！即`NULL(0)`，指针没有指向任何地方，这个**“地方”**就是**内存空间**。再结合第一小节中的概念，**NullPointer**的概念就可以大致得出了：指针没有指向一个连续字节序列的最小内存地址，即指针的值为`NULL(0)`。又 连续字节序列可能表示为一个对象，所以也可以理解为指针没有指向一个有效的对象的内存地址。这里允许我引用来自Wikipedia的一段话：
> Quote from Wikipedia
> 
> ***In computing, a null pointer has a value reversed for indicating that the pointer does not refer to a valid object. Programs routinely use null pointers to represent conditions such as the end of a list of unknown length or the failure to perform some actions.***

也就是说当我们使用一个**invalid**的对象的时候，就意味着我们在操作一个带有空值的指针，就会出现一些执行上的问题。那么，写到这里，我个人认为对于空状态（空值）的描述就够了，实际上针对于NullPointer的问题，大家可以各执己见，毕竟这个问题实在是经典且头疼。总之，NPE的核心就是这个部分，为了不让大家对于我的描述而变得迷惑，点到为止就可以了，即**指针没有指向一个对象的内存地址，而它的值是NULL(0)**。

#### 理解异常（What is Null Pointer Exception?）
这个小节一共有三个点，这里我们已经讲过了两个了，那么剩下最后一个就是我们最头疼也是很经典的NPE了。说到NPE，就不得不带入到某一个具体的语言环境中，例如Java语言。这个小节就要来说说Java语言中的NPE。

之前我一直认为，Exception是只存在于Java语言中，但是这样的想法是错误的。C/C++中同样存在异常机制，只不过C++的异常机制是在后面的版本中添加的，C的异常处理没有Java那样丰富和灵活，可读性那么高，所以异常体系虽然不是Java独有的，但Java的异常体系是我个人认为十分优秀的。然而，Java的异常体系十分庞大，在这么一篇文章中，我不会对异常体系做介绍，只希望针对NPE做一些介绍，仅此而已，如果想学习异常机制的话，我个人推荐《Java编程思想》的第12章去深入的学习。

了解NPE，先从官方提供的资料开始入手。看看NPE的层级分类：

![NPE Classification](http://i.imgur.com/Wneus4I.png)

NPE的父类就是**RuntimeException**，那么就说明NPE是一个运行时异常，它会被JVM自动抛出来，所以就没有必要在异常说明中声明它，即不需要：
```
// not suggest
public void update() throws NullPointerException {
	// ...
}

// not suggest
try {
	update();
} catch (NullPointerException npe) {
	// ...
}
```
编译器不会强制要求我们必须对NPE进行检查和处理，所以NPE这样的运行时异常又称为**“免检查异常(Unchecked Exception)”**。虽然我们可以执意的对NPE进行检查，但是我个人也不建议这么做，首先我们希望**RuntimeException**可以尽早的被发现并且处理，所以一旦我们对NPE的问题进行了`try...catch...`这样的包裹，那么实际的问题很有可能就被异常处理器处理掉了，如果存在十分严重的设计或者数据的问题，后果就有些严重了。同时，增加了NPE的检查会让程序的代码可读性降低，这是因为一旦流行了NPE的检查，我们在编写程序的时候就会小心翼翼，畏首畏尾，害怕NPE的问题出现，就会做各种检查来防止NPE问题的出现，虽然这是意识上的好事情，但是代码的可读性下降同样会引起更严重的问题，业务相关的代码就更加凌乱了，同时异常机制会增加额外的系统开销。所以，针对于NPE的情况，我个人建议：从异常的角度来说，**尽早抛出，尽早处理，尽量不拦截**。

看下面这个图，官方给出的一些关于NPE的参考和建议：

![Oracle Suggestions](http://i.imgur.com/5v9k2yr.png)

简单地说就是：什么时候会抛NPE异常呢？  

1. **Calling the instance method of a null object.**    调用了一个空对象（null object）的实例方法时。
2. **Accessing or modifying the field of a null object.**  访问或者修改了一个空对象（null object）的域时。
3. **Taking the length of null as if it were an array.** 当数组时一个空对象的时候，取它的长度时。
4. **Accessing or modifying the slots of null as if it were an array.** 当对数组中的某些null的元素进行访问或者修改的时候。
5. **Throwing null as if it were a Throwable value.** 假如null当作Throwable的值时将会抛出异常。

在我们看到了以上5条可能会抛出NPE的场景后，我更加希望大家对这种**RuntimeException**保持一个清醒的认识，正如Exception的设计者在设计这类**RuntimeException**时，**RuntimeException**所代表的就是**程序编写上的错误**，相比于那些编译器强制要求开发人员必须提供异常处理器（Exception Handler）的**受检查异常（CheckedException）**来说：

 - RuntimeException是无法预知的程序错误。
 - 有了一处RuntimeException可能出现的地方，就会在其他相似的地方出现这个问题，作为开发人员要对相似的处理环节进行检查。

实际上无论在多大的项目中，真正遇到NPE的问题也就是以上列举出现的5条之内的某一些，如果全都命中，那真的就需要好好学习和反思一下了。在下一个小节，我会从实际的项目中结合我出现的项目来说明这些场景，不过，正如我前面提到的，NPE是一个头疼的问题，因为我们需要小心谨慎些，如果一旦处理不慎的话，就十分有可能导致我们的应用程序崩溃，所以，最后的章节还会补上防空指南的内容。

#### 小结 1
------
本小节将NPE（NullPointerException）拆分成了3个部分进行了讲解，Null + Pointer + Exception，从3个角度对NPE进行了分析，所以要理解好NPE，**Pointer**是一个关键。

### 二、NPE Classics
------
如果想说NPE的案例，那我相信大家绝对可以滔滔不绝地说上一整天。没错，NPE可以无处不再又让我们防不胜防。防不胜防？这句话还是有偏见的，其实NPE并没有那么可怕，只要处理得当掌握适当的方式就可以杜绝NPE的骚扰。

在列举NPE之前，我个人推荐的一项任务，就是不断的Review自己的代码或者别人的代码。这绝对是一个很好的上升方法，也是一个绝佳的提升自己代码质量的机会。这是我之前缺少的经验和实际行动。同时，Review的过程中要想，要思考，为什么要这么写代码？会不会出现问题？所以这就是写这篇文章的原因，因为在自己Review自己的代码，也实在是能看出来问题。所以我就先拿自己开刀说明问题。

#### Scene 1： Calling the instance method of a null object.
调用了一个空对象的方法。先解释一下，什么是空对象。理解空对象，就是我们前面提到的Null Pointer。我们的指针或者引用没有携带一个有效的VALUE值，即没有指向一个内存地址。在Java语言中最常见的就是声明一个对象而没有分配给它内存空间。

```
private TextView mTextView = null; // 显示初始化或者默认都可以
```
那么`mTextView`就是一个空对象，即没有初始化的过程，没有在内存中分配一块内存空间给它，指针或者引用就会指向`NULL(0)`。

请看下面这段代码：
![Scene 1](http://i.imgur.com/ktAonZk.png)

知道这段代码出现了NPE的问题，我还在运行的火车上回家，怎么也想不到这段代码会出问题，这个方法很简单，就是利用给定的正则表达式去过滤和校验一个用户的头像URL是否合法，如果合法，就返回一个true，否则返回false。在设计和写这段代码的时候，我还真的考虑到过NPE的问题，不过系统提供的API封装让我过于放心放弃了对NPE的处理。为什么这么说，一般来说API的提供方和API的调用方会有不默契的情况出现，即**提供方希望参数的传入是合法的，也是按照自己的期望传递的**，而**调用方有希望API的设计者的设计足够强大，可以让自己安心地传递而放弃大部分检查**，我敢保证，这在绝大部分的开发者中间是存在这样的问题的。不过这是问题之一，我在设计这个API的实现时，确实考虑到需要增加入参的校验，但是我还是注掉了，原因就在于，我过于相信了系统提供的API（不是说系统的API设计不对，大家不要误解，是我自己的问题所导致），问题就在这行代码：

`Matcher matcher = urlPattern.matcher(avatarUrl);`

这确实是我缺乏经验，看图说话：  
![Why has an issue?](http://i.imgur.com/1Q06Pus.png)

`input.length()`，从`matcher(avatarUrl)`方法一路跟踪到这里，看到了一个`reset(CharSequence input)`方法中有`input.length()` 调用，我瞬间就明白了问题出现的原因，NPE的出现说明了input是一个空对象。而我曾单纯的认为`matcher`方法会对入参进行校验，也没有点击进去看看实现到底是怎样的。这样的话，如果我们传入一个null的字符串类型的对象，NPE就会成功的出现了，然后Crash掉我们的应用程序。

这就是我觉得NPE头疼的原因，因为它会隐藏地很隐蔽，有时为了让代码简洁提高可读性而有意地放弃一些检查，这样的做法是得不偿失的，就像这个问题一样，我们无法快速地得知`matcher`方法会在很远的地方出现NPE。以上，这个例子足够表明***Calling the instance method of a null object***的场景了。

------

#### Scene 2：Accessing or modifying the field of a null object.
访问或修改一个空对象的域。这也是Classic场景，出现NPE的机会非常高。在我们实际的开发场景中也很常见，造成这个问题的原因很简单，和上面的道理是一样的，只不过第一条是访问一个空对象的方法，而这个场景是访问或修改一个空对象的域。看如下的代码：

![Accessing or modifying the field of a null object.](http://i.imgur.com/5ScHkau.png)

首先，我说明一下不建议这样利用public直接修饰域（除非特定场景需要），这样破坏了封装型。在这样一个实体类中，有三个域，如果一切顺利，那我们就可以顺利的将可能从服务器端来的数据映射到这个实体类中，稍作处理后呈现在UI上。但是假设某一次数据请求出现了问题，导致了部分数据没有返回，如`UserBean`，此时`user`对象就会为一个空对象，如果此时我们有这样的设置：  

`mNickNameTv.setText(user.nickName)`或者`mRemarkTv.setText(user.remark)`

NPE就出现了。因为`user`是一个空的对象，那么直接操作`user`对象的域是不存在的，因为`user`这个引用没有指向任何一块内存空间。这种问题的病因就在于**调用方的注意力不足，同时实体给的信息不足导致的**。层次过多的组合让我们在检验的过程中产生“走神”的现象，绝大部分的检查会放在`FriHomePageBean`这样的地方，一部分检查会放在`UserBean`上，但是如果`UserBean`中也组合了其他的实体，那么越向下的部分就越容易产生NPE，所以小心谨慎是一方面，我们还是有方法避免的。

------

#### Scene 3：Array
数组是Java语言中比较有特色的内容之一，至少我是这么觉得的。
特色就在于数组本身就是一个引用类型的对象，而数组所能提供的容器能力，也能让其装入基本数据类型的数据和引用类型的数据。这是很容易混淆的。关于数组出现的NPE，也是十分多的，要想理解**数组**出现的NPE，我们可以简单的打入它的内部看看情况再说！

*Taking the length of null as if it were an array.* 取一个空数组的长度时会出现NPE。大家要注意，“空数组的长度”中的“空数组”的意思是数组对象本身为空，即它是一个空引用或是一个空指针，数组本身没有被分配任何的内存空间，有些类似于Scene 2。所有的数组都有一个固定的成员域，名字为**length**，我们可以通过**length**得到一个数组中有多少个元素，但是我们无法对**length**做任何的修改，只能访问。

```
public class PlayerTest {
	private Player[] groupOne;
	private Player[] groupTwo;
	
	public PlayerTest(int teamNumber, Player[] theSecondGroup) {
		if (teamNumber < 0) { 
			throw new IllegalArgumentException();
		}
		
		groupOne = new Player[teamNumber];
		groupTwo = theSecondGroup;
	}
	
	/**
	 * 通过该方法交换两组中的成员。
	 * 条件为当第二组的人员数量大于第一组的时候。
	 */
	public void changeGroup() {
		if (groupTwo.length > groupOne.length) {
			// change the two groups
			...
		}
	}
	
	/**
	 * 打印所有玩家的名字。
	 */
	public static void printPlayerName(Player[] group) {
		if (group == null) {
			return;
		}
		
		for (Player player : group) {
			System.out.println(player.getName());
		}
	}
}
```
这段代码看上去没有什么问题，可是在一些场景下它就会出现NPE的情况。首先我们分析了任何一个数组都有一个成员域名字叫做**length**，就是`groupTwo.length`所访问的域。在`PlayerTest`的构造器中，如果传入的参数合法，那么我们会顺利创建出`groupOne`数组这个对象，至少我们可以保证`groupOne`是分配了内存空间的，但是`groupTwo`就不一样了，通过构造器传递的情况就会出现传入一个null对象进来的场景，那么`groupTwo`在访问自己的**length**的时候就会出现如同Scene 2一样的问题。所以在考虑使用或者已经使用数组的时候，要多多考虑，使用**length**时，是否已经保证了在任何地方对于`groupTwo`使用没有让其为null的操作，如果有，那么NPE的问题就会找上门来。

此外，数组的NPE的另一个场景就是类似于Scene 1的场景。同样还是上面的代码，我们继续看：
如果我们使用`PlayerTest`中提供的公共静态方法`printPlayerName`来打印所有的玩家名字，就需要传入一个数组对象，假设我们传入的是`groupOne`，依然还是会出现NPE的问题。这是因为数组中装入的元素有些可能是空对象的缘故。

所以针对于数组来说，需要使用它要在保证数组对象本身不为空对象的时候额外的考虑它装入的元素也不能为空对象（某些特定场合除外），需要认真地找出使用这些数组的地点并给出尽量周全的处理。

补充，针对于第五条 *Throwing null as if it were a Throwable value.*，这里我暂时不做分析，因为我会在分析异常机制的时候详细说明这个问题，有点复杂，这里就先埋个伏笔，不过不会影响大家阅读。

#### 小结 2
------ 
针对于NPE的几个场景我分别给出实际的代码和实现，相信大家会遇到比我这个更加复杂更加有说服力的场景，也希望大家可以提出意见，毕竟NPE算是异常中比较典型的，如果有更好的场景请添加，Thanks here.

### 三、Java与Android的防空指南
所谓防空，只要两点要素就可以预防的不错：

 - 手段和方法
 - 意识和经验

第一条凭借外部学习，第二条凭借经验和细心。缺一不可。这里我**预先说明**，关于防空的外部学习，也请允许我照搬一些轮子，前辈总结下来的，照搬并非不可，这是知识的传递，而且，我不想加入个人的理解在里面，既然知识能传递就说明能经得起考究，反而加入自己的理解或许让原本的信息流失，这样就不好了。所以照搬就照搬，只要学会了并且解决问题了，这就是好的。

我的分享参照了来自于这样一篇blog，原文地址附上：
[Java Tips and Best Practices to avoid NPE in Java Applications](http://javarevisited.blogspot.sg/2013/05/ava-tips-and-best-practices-to-avoid-nullpointerexception-program-application.html)

1) Call equals() and equalsIngoreCase() method on known String literal rather unknown object.
什么意思呢？看代码就明白了！

```
Object unknownObj = null;

// wrong way - may cause NullPointerException
if (unknownObj.equals("unknownObj")) {
}

// right way - avoid NPE even if unknownObj is null
if ("unknownObj".equals(unknownObj) {
}
```
评价：这是一个最简单的Java Tip和Best Practice来避免NPE，但是结果却有巨大的提高，因为`equals()`方法是作为Object中的方法，所有子类都会继承这个方法且使用频率很高。这是Scene 1的问题。

------

2) Prefer `valueOf()` over `toString()` where both return same result
> Since calling `toString()` on null object throws NPE, if we can get same value by calling `valueOf()` then prefer that, as passing `null` to `valueOf()` returns `null`, specially in case of wrapper classes like `Integer`, `Float`, `Double`, `BigDecimal`.

```
BigDecimal price = getPrice();
System.out.println(price.toString());			// may throws NPE
System.out.println(String.valueOf(price));		// avoid NPE
```
评价：`toString()`方法比`equals()`方法更加常见，尤其在调试的时候十分好用，不过即便好用的方法也是利用对象去调用方法，所以也是会出现NPE的情景。相比于`toString()`方法，`String.valueOf()`则是String类提供的静态方法，避开对象的直接调用，而变成了参数传递，传入`null`返回`null`，不会出现NPE的情况。 所以，在你不确定一个对象是否为空的时候，请follow这条BP。

------
3) Using null safe methods and libraries
> There are lot of open source library out there, which does the heavy lifting of checking null for you. One of the most common is `StringUtils()` from Apache commons. You can use these method without worrying about NPE.

![Apache StringUtils](http://i.imgur.com/uXCe2Pe.png)

评价：不得不同意，优秀的方法和库都是尽全力保证NPE的问题不会出现，`StringUtils`就是其中之一，而且官方也说明了，**Operations on String that are null safe.**，所以关于字符串的操作推荐使用它，同时，可以看源码来搞清楚到底是怎么做到的***null safe***。

------
4) Avoid returning null from method, instead return empty collection or empty array.
>By returning empty collection or empty array you make sure that basic calls like size(), length() doesn't fail with NPE. Collections class provides convenient empty List, Set and Map as Collections.EMPTY_LIST, Collections.EMPTY_SET, and Collections.EMPTY_MAP which can be used accordingly.

```
public List getOrders(Member member) {
	if (member == null) {
		return Collections.EMPTY_LIST;
	}
	// ...
}
```
评价：这是一条真的好用的Best Practice，尤其在Android的同学开发时，这条BP可以预防很多的NPE，常见的AdapterView类型的组件如ListView，GridView，或者是RecyclerView都需要提供一组数据作为展示用的，但这样的数据集合往往从Server端取得，如果没有，我们就可以传递一个`Collections.EMPTY_LIST`，这样就不会出现关于数据集合的NPE相关的问题了。

------

5) Use of annotation @NotNull and @Nullable
> While writing method you can define contracts about nullability, by declaring whether a method is null safe or not, by using annotations like @NotNull and @Nullable. Modern days compiler, IDE or tool can read this annotation and assist you to put a missing null check, or may inform you about an unnecessary null check, which is cluttering your code. IntelliJ IDE and findbugs already supports such annotation. These annotations are also part of JSR 305, but even in the absence of any tool or IDE support, this  annotation itself work as documentation. By looking @NotNull and @Nullable, programmer can himself decide whether to check for null or not. By the way ,this is relatively new best practice for Java programmers and it will take some time to get adopted.

![With @NonNull Annotation](http://i.imgur.com/0mZSQNB.png)

评价：正如文中提到的一样，这类注解的出现确实帮助我们的代码提升了不少质量，我也确实是它们的粉丝，可以放心且大胆的使用它们。如果以100%来评价，我支持它99%，剩下的1%是需要警示作用的。为什么这么说！因为它是注解，是Annotation。如果大家对注解了解，注解无非那几套打法，源码级别、编译时、运行时处理注解各有特色。不过@NonNull这类的注解都是CLASS级别的，编译时，所以就存在开发人员无视或者忽略它们的情况。即便我对参数加以`@NonNull`， `@NotNull`之类的注解，可是我们依然可以传入null逃避注解的警告。上面贴上的代码也指出了这个问题，即便我使用了`@NonNull`为参数增加了限制，但是API的调用者依然可以无视它，传入任何参数，所以，我还是增加了额外的checking。在Android开发中，Google的`android.support.annotation`下增加了这些注解，同时，作为AndroidStudio的东家JetBrains也为IDE增加了自己的注解库，来提供这些限制。

![Android Studio Infer Nullity](http://i.imgur.com/9xRM0Sv.png)

同时，例如AndroidStudio和Intellij都在Analyze中提供了**Infer Nullity**，这是IDE的一些高级特性，在分析代码的时候十分有用，IDE可以帮助我们推断出哪些对象，哪些位置可能存在NPE的风险，以便我们做出调整。

6) Avoid unnecessary autoboxing and unboxing in your code
autoboxing and unboxing是Java语言中的自动装箱的和拆箱技术。举个简单的例子，如果我们像把1...10这些数字装入装入一个集合中，如List，那么我们在装入这些数字的时候，1会被装箱为一个Integer的对象后放入List中去，这就是**装箱（autoboxing）**，同时如果我们在取出一个数字来使用的时候，它又会被拆箱，成为一个基本数据类型的int类型来使用，这就是**拆箱（unboxing）**。

那么在装箱与拆箱的过程中，是基本数据类型到引用类型的相互转换。看看下面的代码段就知道问题了。

![Autoboxing & Unboxing](http://i.imgur.com/WfyvU0k.png)

从图中我们可以得知，`getNumber()`返回的是一个`Integer`类型的对象，而`set()`方法中调用`getNumber()`之后赋值给一个`int`类型的`result`就是一个拆箱过程，但是这会产生一个NPE的，毕竟`getNumber()`返回的是一个**null**。

如果希望了解更多关于拆箱和装箱的技术细节，请看这篇文章。
[Autoboxing and Unboxing in Java](http://javarevisited.blogspot.com/2012/07/auto-boxing-and-unboxing-in-java-be.html)

7) Follow Contract and define reasonable default value
> One of the best way to avoid NullPointerException in Java is as simple as defining contracts and following them. Most of the NullPointerException occurs because Object is created with incomplete information or all required dependency is not provided. If you don't allow to create incomplete object and gracefully deny any such request you can prevent lots of NullPointerException down the road. Similarly if  Object is allowed to be created, than you should work with reasonable default value. for example an Employee object can not be created without id and name, but can have an optional phone number. Now if Employee doesn't have phone number than instead of returning null, return default value e.g. zero, but that choice has to be carefully taken sometime checking for null is easy rather than calling an invalid number. One same note, by defining what can be null and what can not be null, caller can make an informed decision. Choice of failing fast or accepting null is also an important design decision you need to take and adhere consistently.

嫌它太长，看代码吧！
```
class Address {
	
	// default instance
	public static final Address EMPTY_ADDRESS = new Address("", "", "", 0);

	public Address(String line, String city, String country, int zipCode) {
		// ...
	}
}
```
评价：提供这样一个成员域EMPTY_ADDRESS，也是一个非常不错的Best Practice，尤其在Java 8之后提供了Optional（后面会提到），这样的做法就更加灵活了。当我们会预知到某些地方不能使用null的时候，它就更加有用，可以很好的预防NPE的出现。无论如何，在考虑null的情况时优先使用提供的默认实例绝对会防止NPE的出现的。

------

8) Using **Optional** after Java 8

> What is Optional? As the name suggests, the Optional is a wrapper class which makes a field optional which means it may or may not have values. By doing that, It improves the readability because the fact which was earlier hidden in the code is now obvious to the client.

关于Optional，我这里就不多介绍了，与其我蹩脚的翻译，不如原汁原味的文章来的过瘾，这篇文章很清晰的阐述了如何利用Optional预防NPE，很值得学习，附上地址。

[Optional in Java 8](http://javarevisited.blogspot.com/2017/04/10-examples-of-optional-in-java-8.html)

评价：由于Java 8之后支持了lambda表达式，同时函数式编程思想的引入，预防NPE的问题有了长足的进步，不过兼容性是一个问题，同时在Android中Java 8的特性与Android的结合并不是很好。**Optional**被放到了java.util.*包下，但是在Android中需要开启Jack enable的同时也会损失很多IDE的特性，这样下来有点得不偿失，所以普及的也不是很好。不过Optional是预防NPE的主流趋势，因为它真的灵活，在Java 8之前，Google在Guava中就已经给了Optional的实现，只不过在Java 8中Oracle和Open JDK正式的将其纳入JDK中，提供给开发者使用，所以预防NPE，**Optional**非学不可。

------

9) 麻烦却有效的双重入参校验

无论怎么样，一些有经验的开发者更倾向于入参的校验，没错，我也很喜欢这样的方式，总感觉很放心。其实，入参校验这个话题放到这里有点委屈了，入参校验是必须的，无论什么方法，只要有参数传递，我们就要增加入参校验，也不论是引用类型还是基本数据类型。API的设计者和调用者都有各自的职责去保证对方法的正确使用，那就文档的约束和入参的校验，缺一不可。因此，增加了`null != obj`这样的校验在我看来是非常好的做法，无论设计者还是调用者，都可以在合适的时机和场合下增加这个判断。

------

10) 必杀技：细心

在好的技术，在高明的方案也不如再细心一点，再多检查一遍来的好。其实，正如我开头总结的一样，预防好NPE的要素就是两条，其中一条就是作为一个开发者要对代码足够细心，足够敏感。我觉得这才是真的好用的Best Practice吧！

### 参考与引用
------

1.[Pointer (computer programming)](https://www.wikiwand.com/en/Pointer_%28computer_programming%29)  
这是Wiki上的一个普及贴，关于C和Java的指针的简单介绍都在这上面

2.[NullPointerException Official Doc](https://docs.oracle.com/javase/7/docs/api/java/lang/NullPointerException.html)  
 Oracle提供的官方NullPointerException的文档

3.关于第三节防空指南中内容是这篇blog中提及的。我做了一个**搬运工**的工作。
http://javarevisited.blogspot.jp/2013/05/ava-tips-and-best-practices-to-avoid-nullpointerexception-program-application.html

4.这是JavaWorld上的一篇文章，NPE的问题讲的也很好，不过我不是很支持利用try-catch的方式去处理NPE，所以没有纳入其中，但是在这里会放出链接。
http://www.javaworld.com/article/2072719/effective-java-nullpointerexception-handling.html

5.这里有一些关于NPE的有趣的讨论，有兴趣的可以来看看！
http://wiki.c2.com/?NullPointerException

6.StackOverflow上关于NPE的处理，讨论的很自由也很有趣。
http://stackoverflow.com/questions/218384/what-is-a-nullpointerexception-and-how-do-i-fix-it

7.参考了《深入理解计算机系统》一书。

  第二章 “寻址和字节顺序”
  第三章 “理解指针”
  第九章 “C中常见的与内存相关的错误”

8.参考了《Java编程思想》一书。

第二章 “一切都是对象”
第十二章 “Java标准异常”

9.感谢顾岩，我的老大，给我传授了很多关于NPE的经验和新潮的东西，以及在AndroidStudio使用工具去推断，十分感谢。