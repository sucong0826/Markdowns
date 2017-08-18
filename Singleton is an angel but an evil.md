### 写在最前面 ###
越来越感觉编程的世界是真正符合自然的世界，真正遵循自然的规律。在程序的世界中事情几乎是平等的，即使不平等也是有据可循，任何技术问题也可讨论的，无论结果如何，总是0与1的差别。 同样真实世界中的事物放到程序世界中也变得那么客观起来，正如这篇文章的主角**Singleton Design Pattern**一样。

看其他人的代码总是能发现问题，这是review的好处之一，对于自己来说也是一个提升的手段。所以如果想更好的提高技术水平，多去看别人的代码也是非常非常有效果的，其中对于模棱两可的知识点可以学习并且掌握使用，同时对于有问题的或者有自己想法可以写的更好的地方可以给出自己实现，并讨论哪种方式更好。而这篇文章《Singleton is an angel but an evil》也正是我在分析别人的代码时，发现的一个问题，在一个项目里单例的使用频率非常高，我个人认为这并不是一个好现象，或者说用的不是很合理，甚至是滥用。如果希望更加深入地了解单例，希望这篇文章可以对大家有帮助，如果您非常熟悉或深有研究，也希望您不吝赐教。

### 目录 ###
------ 
- 什么是单例模式？
- 争论与诟病。
- 单例模式是魔鬼！
- 单例模式是天使！
- 总结
- 附录


### 什么是单例模式？ ###
------
单例是软件工程设计模式中的一种，英文名是***Singleton Pattern***，中文名为单例模式。从字面来看这个设计模式的浅层次含义那就是单一实例，对于Java语言来说即在系统中一个模板（Class）只存在一个实例。***Singleton Pattern***（以下称为单例模式）的设计概念是从数学中的单例概念得到并演化出来。

> **Quote from Wikipedia**  
> *In mathematics, a singleton, also known as a unit set, is a set with exactly one element. For example, the set {0} is a singleton.*

从上面的内容来看，单例模式的设计意图算是很容易理解的。但是，容易理解并不代表在对于设计理念和使用就合理或者恰到好处。对于一个单元组来说其中只有一个元素，那么这个单元组就是一个单例。在Java中，基本的实现都是通过限制实例化来实现单例模式，下面给出单例的类图。  

![Singleton Class Diagram](http://upload-images.jianshu.io/upload_images/3902272-85fccac5d02e09f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

这是基本的单例模式实现的类图，从图中可以看到，`singleton`是`Singleton`类中的一个私有的静态的对象，那么它就是这个唯一的实例。对于`Singleton`类来说，将构造器的访问权限限制为`private`，这样可以有效的控制外部通过`Singleton`类通过构造器进行实例化，同时提供一个`public static`的`getInstance()`方法来返回这个唯一的实例，这样最基本也是最实用的单例类就完成了，附上代码。 

	public final class Singleton {
	
	    private static Singleton singleton;
	
	    private Singleton() {}
	
	    public static Singleton getInstance() {
	      if (singleton == null) {
	          singleton = new Singleton();
	      }
	
	      return singleton;
	    }
	}

单例模式的实现在我看来并不是很晦涩难懂的，也并不是这篇文章想要表达的重点，对于单例模式的其他实现方式我会在末尾部分给出。

单例模式的设计意图就是**为了保证一个类有且仅有一个实例，并且为它的一些客户端类（Client Class）提供一个全局的访问点**。（虽然也可以通过其他的方式无法保证只有一个实例）**延迟初始化或加载**也是单例模式的一个主要用法，关于单例模式的用法我们会在后续的章节中给出更具体的解释。其实，对于关于Singleton的争论的问题相信大家都各执己见，这篇文章也是希望从好与坏的两个方面来说明为什么单例模式是争论不休的，客观给出自己的看法。


### 争论与诟病 
------
Design Pattern，是软件工程中的设计模式。为什么我说程序世界是客观且自然的，因为有这样反对派别的存在，Anti-Pattern，反设计模式。 Anti-Pattern这个术语是真实存在的，在1995年的时候，Andrew Koenig看到了Design Patterns这本书后得到了启发，三年之后，AntiPatterns这本书就诞生了，anti-pattern这个术语就开始流行了起来。其中，在anti-pattern中关于Singleton的讨论是最多的。Anti-Pattern的出现想表达的就是利用Design Pattern的思路去解决问题有可能是对于问题来说是一种很糟糕的方案。

Singleton，单例模式就是其中被诟病的一个设计模式。经过了这么多年的讨论与沉淀，目前，Singleton这种设计模式可以在特定的环境下承担特定的职责，这是单一职责的创建型设计模式，它被诟病的主要原因无非就在下面几点：  

- Misuse 误用
- Abuse	 滥用

既然存在这误用与滥用，那么到底什么是误用和滥用？ 上图来说明吧！
![Misuse Case 1](http://upload-images.jianshu.io/upload_images/3902272-98fbce20549cb8d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Misuse Case 2](http://upload-images.jianshu.io/upload_images/3902272-581d9af05c25a7bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

既然在分析系统中存在的问题，那么大家对于系统中存在多少个单例的实现肯定心中有数，我也可以基本保证大部分开发者的单例模式的实现基本都是维持在这个阶段，一个理解了单例模式基本语义的阶段。这是我随便在现存系统中找到的单例的实现，这也就是我想表达的误用，这种误用其实很好理解，就是误用在单例模式的语义阶段。这种阶段的带来的困惑就是看上去只要在系统中这个类希望它只有一个实例，就要设计这个类为单例模式，同时这样的实现看上去和执行时都不会出现什么问题，所以，单例模式的实现就变得随处可见了，简单地说，对单例模式浅显的认识导致了它的误用，那误用之后又没有得到应有的反馈，滥用的情况也就随之就出现了，对于这些类为什么说它们是误用，就请看下面的段落吧。

#### 补充
其实在我读了很多的blog和文章之后，我决定将误用和滥用分开来说明，这两个原因造成的后果就是Singleton作为evil的原因，相比与这些明显的问题，对于这个设计模式本身的实现来说就变得小巫见大巫了。

### 单例模式是魔鬼！
------
对于这个标题我来说几句，说单例模式是魔鬼，这个词也是让我思忖了很久的，但最终还是选择了这个词与来表达它的问题以及这些问题带来的影响。 问题主要会影响：  

1. 测试（Test）
2. 耦合（Coupling）
3. 无边界（Boundaryless）

#### 测试与耦合 （Testing & Coupling） ####

通常在以下条件下，自动化的单元测试是最高效的：  

- *Coupling between classes is only as strong as it needs to be.*
- *It is simple to use mock implementations of collaborating classes in place of production implementations.*

以上这两条该怎么理解呢，如果大家对单元测试有所了解，那么这两条就很好理解了。类与类之间的耦合度越低，一个单独的类就越容易进行测试。当类与类之间都是高度的耦合在一起的时候，单元测试就变得愈发的困难，而且得到的bug也越难划分和分离。

在OOP的编程思想中，类是最基本的单元，每一个类也要保证一些基本的原则，其中，单一职责的原则就是一个类需要保证的，那么单元测试的任务就是来测试这些类是否如他们自己声明的一样，是可以独立于系统其他的部分而完成自己的职责。在单元测试中，让单元测试更加有效，测试执行更加迅速的一个通用的手段就是mock对象。我们在下面的例子中来演示它：

    public class MyTestCase extends TestCase {
    	...
    	public void TestBThrowsException() {
    		MockB b = new MockB();
    		b.throwExceptionFromMethodC(NoSuchElementException.class);
    		
    		// Pass in the mock version.
			A a = new A(b);
    		try {
    			a.doSomethingThatCallsMethodC();
    		} catch(NoSuchElementException ex) {
    			// check exception params
    		}
    	}		
    }

首先，这是单元测试的case，为了测试当类B抛出一个异常的时候，类A是如何响应的。
`MockB b = new MockB();`就是我们上面提到的mock对象，`MockB`模仿了B类，然后对象`b`直接调用了一个`throwExceptionFromMethodC(NoSuchElementException.class);`，这个方法会抛出一个`NoSuchElementException`。自动化的单元测试中，有这样的一些tips：

> From 《**Use your singletons wisely**》  
> *It is much simpler to simulate behavior than it is to recreate that behavior. The simulation requires less specialized knowledge of target class than recreating the scenario would.*

含义就是测试中模拟一个行为比重建一个行为更加简单，因为模拟行为不需要对测试目标掌握更多的内容，但是重建一个行为就不同了。在上面提供的示例中，MockB的对象b直接调用`throwExceptionFromMethodC(NoSuchElementException.class)`来抛出异常，这就是一次重建行为，所以它需要对目标类掌握更多的信息，如它需要知道要抛出的是一个`NoSuchElementException`异常同时是仅在调用`throwExceptionFromMethodC`方法的时候抛出的，这就是单例模式给单元测试带来的影响。第二段代码，`A`在创建实例的时候需要传入一个MockB的对象`b`，在try-catch中，`A`的对象`a`调用了一个`doSomethingThatCallsMethodC()`方法，这个方法不传入任何参数，也是就是说它不需要对B的信息有所了解，就能通过调用这个`doSomethingThatCallsMethodC()`方法来测试调用`C()`方法时是否有异常抛出，同时它也只捕捉`NoSuchElementException`异常，这就是一次模拟的行为。从代码中可以看出，对于单元测试来说，模拟行为确实比重构行为更加简单，对于测试场景需要的信息也更少，但是得到的测试结果却更准确，更客观。

上面讲了一堆，那么单例模式怎么影响了单元测试呢？这就是上面提到的第二点，耦合（Coupling）。刚刚说明了单元测试需要的是低耦合的mock方式，这样的单元测试是最有效的，单例模式也就违反了这些原则。通常得到单例类的唯一实例的方式都是基本相同的，如`Xxx().getInstance().xxx();`这样嵌入到某一个客户端类的某个方法中去，如提到的MethodC，`C()`，假设要测试`C()`方法是否抛出异常，也只能进行场景重构了，单例类已经和客户端类紧紧地耦合在一起，这样就无法更好单独地测试客户端类，mock的方式也就变得有心无力了。

#### 无边界 （Boundaryless） ####
> From **Wikipedia 'Singleton Pattern'**  
> *Singleton introduces unnecessary restrictions in situations where a sole instance of a class is not actually required, and introduces global state into an application.*

Singleton将全局状态引入了一个系统中，这样系统就可以在任何地方对这个全局状态进行处理或者访问。这样，有可能在原本看起来相对独立清晰的模块或者组件之间通过singleton架起了很多的桥梁，也就是说singleton模糊了边界，而且让类与类之间轻易地建立起了合作的关系。在Anti-Pattern中有这样一句话：

> From **'AntiPatterns'**   
> *I know where you live.*

这句话很形象，“我知道你在哪！”，因为Singleton提供的API去得到它的唯一实例实在是很方便，这也是为什么很多朋友希望把类在满足语义的条件下轻易地创建了一个单例类。可能那种方式既便捷又很酷，造成了单例类的误用与滥用。那么单例模式带来的无边界会引起怎样的问题？

1. 耦合度增加（为单元测试带来困难）
2. 违反里氏替换原则（类的设计不正确，灵活性、可复用性差）

我们直接看代码：    
	
	public class Deployment {
    	...
    
    	public void deploy(File targetFile) {
    		Deployer.getInstance().deploy(this, targetFile);
    	}
    
    	...
    }

现在我们从错误的角度来考虑，为什么认为这种实现是合理的？首先，我们可以轻易并且自信地认为系统只需要一个`Deployer`就足矣，毕竟完成的文件部署工作有全局状态的特性，所以我们需要把`Deployer`做成一个Singleton，况且单例模式也很好管理，不会创建出很多的实例，这没有任何问题，同时单例模式的实现提供的`getInstance()`又是那么的简洁与方便，所以，我们暂且认定它为单例模式的类好了。

现在我们稍微做出一些调整，单例模式就变得不再那么实用而且会发现我们想错了。以上的实现中，`Deployment`（部署类）`与Deployer`（部署者类）是紧紧耦合在一起的，无论是当前的`deploy()`方法，或者甚至直接在其他的`xx()`方法中直接调用，都是与客户端类紧紧耦合的。 Deployer在业务领域范围内的概念很大，现在我们希望将`Deployer`的业务做一次拆分，那么单例模式带来的结果可能就是灾难性的了。可能在一个模块，一个组件，甚至一个系统中，Deployer可能使用的地方高达几百处或者更多，那么假设我们只是简单地调整了`deploy(this, targetFile)`方法增加一个参数，兄弟，没啥好说的，周末加班吧，可能你想砸键盘或者砸电脑的想法都会有的。

那么，我们看看这样的实现会不会好一些：  
	
	public class Deployment {
		priavte Deployer deployer;
		
		public Deployement(Deployer aDeployer) {
			this.deployer = aDeployer;
		}
	
		public void deploy(File targetFile) {
			deployer.deploy(this, targetFile);
		}
	}

在OOP中（如Java），推荐使用组合或聚合的方式来降低类与类之间的耦合，那么以上给出的实现中就是利用组合的方式来降低类之间的耦合，相比于直接在方法中调用`getInstance()`方法，association（组合）就是解开耦合的有效手段。同时，组合的方式也体现了Java的多态性。我们为系统或者客户端留下了更多的可能性，而不是`getInstance()`方法这样，强加于客户端中的实现。那么现在，假设将`Deployer`的业务进行拆分，得到了`SystemFileDeployer`,`ExternalFileDeployer`等，`Deployment`类也无需做出任何变化，只要系统或者客户端根据实际业务要求传递不同`Deployer`就可以了。 这样的实现同样不会违反里氏替换原则（Liskov Substitution Principle），我们先来了解什么是里氏替换原则。

里氏替换原则：**子类必须能够替换他们的基类型。**

- 如果每一个类型为T1的对象o1，都有类型为T2的对象o2，使得以T1定义的所有程序P在所有的对象o1都替换为o2时，程序P的行为没有发生变化，那么类型T2是类型T1的子类型。  

- 一个软件实体如果使用一个基类的话，那么一定适用其子类，而且它根本不能察觉出对象和子类对象的区别。只有衍生类替换基类的同时，软件实体的功能没有发生变化，基类才能真正被复用。  

- 里氏替换原则是继承复用的准则。  

- 应当尽量从抽象类继承，而不从具体类继承。
  
- 一个继承是否符合里氏替换原则，可以判断该继承是否合理。  

由于第一种`Deployer`是单例模式的原因，根本无法对Deployer进行替换，如果想完成替换工作，工作必须是手动的去查找出所有依赖于`Deployer`的客户端类，然后变更代码。即使变更了代码也依然违反了里氏替换原则，因为依赖于`Deployer`的客户端类的行为发生了变化。


### 单例模式是天使！
------
是不是看完了上面的介绍，可能会让大家再次面对单例模式的时候变得畏首畏尾？ 没错，我们确实应该这样，的确应该在选择Singleton的时候三思。

合理的辨析一个设计是否应该为单例模式前，先问问自己几个问题，也是检验标准：
> 
> Quote from 《 Use your singletons wisely 》
>   
> - Will every application use this class exactly the same way? 						(keyword: **exactly**)  
> - Will every application ever need only one instance of this class? 					(keyword: **ever** & **one**)  
> - Should the clients of this class be unaware of the application they are part of?	

以上3条就是检验一个类是否应该被设计为单例模式的判断准则，
  
- 每一个应用（组件/模块）是否以完全一致的方式来使用这个类？
- 每一个应用（组件/模块）是否真的只需要这个类的一个实例呢？
- 对于这个类的客户端类来说，对他们自己是应用中的一部分这件事是否应该保持毫无察觉的状态呢？

如果我们对于以上这3条均给出了“是的”的答案，那么这个类就是可以被设计为单例模式了。如果错误地使用了单例模式，那带来的问题确实很多，所以对于使用单例模式我们还是小心点好。

同时这里我们也给出3条，在你有代码实现之后的反思，也算是反思是否正确的使用了单例模式的准则：

- Where am I going to get an instance of this class?
- Does this object belong to the application or component that I am writing?
- Can I write this class so that customization can be pushed back to its clients?

那么，有哪些经典的场景需要使用单例模式呢？ 我们就以Android系统为例来说明系统中有哪些单例模式。

1. ImageLoader (Universal-ImageLoader) 图片加载库中使用了单例模式
2. SLF4J中的StaticMDCBinder 一个日志系统中是使用了单例模式
3. Logging Service 一个比较经典的单例模式

对于“单例模式是天使”这个章节的例子和最后关于如何说明单例模式的优点，在几番辗转下，我最终还是放弃了，关于为什么我放弃给出各个单例的优点，我会在下面给出说明。 单例模式的优点大部分认知还是停留在对于大型的重量级的组件，如DB操作相关的类，或者一些相对较重的Service，去使用单例模式。但是，使用单例模式的同时又要特别注意时机，在合适时机进行创建，也是单例模式结合延时加载策略的目的。至于其他的优点我的确不想照搬下来，因为我没有实践过，并且也实在找不到实践的价值。

### 总结
------

关于如何用好单例模式，个人觉得我不是很有发言权。 因此在Github上，google上，Stackoverflow上到处搜罗关于Singleton的Classic场景，可是非常遗憾，关于Singleton的经典案例真的是少之又少，可能在Anti-Pattern出现之后，大家对于Singleton的讨论也越来越多，同时项目中血与泪的经验也给开发人员敲响了警钟，就是Singleton真的是一个合格的设计模式么？

我非常好奇的打开了Github然后去看了JakeWharton的几乎所有Android的项目，然后搜索关键字**Singleton**，结果真的让我大吃一惊，无论是关于ORM类的项目，还是DiskLruCache，还是ButterKnife这样的项目，等等，根本都看不到Singleton的踪迹。 同时看了一些书籍上介绍，Android中的LayoutInflater是单例模式，我原本也想顺着书上的思路去写以下这块知识，但是看了Google的给出的API之后，即便是LayoutInflater的实现类PhoneLayoutInflater，也是暴露了public的构造器，在我看来LayoutInflater的实现从严格意义上来说也不是很符合单例模式了。（关于LayoutInflater是否为单例模式，希望大家指正）

因此，对于单例模式的使用给我带来了很深的思考和影响。 在上面一节“单例模式是天使”中，我决定放弃将书上或者我意识到的浅显的单例模式的零星优点搬到这里来，以免让大家误解原来单例模式还是有好处的。 不可否认，单例模式的确有优点，也有部分经典的实现，但是对于经典的实现也存在诟病，所以我们应该避免对于单例模式的误用和滥用。

最后，给出逐条的总结：

1. 这篇文章的风向：如果为非经典场景的前提下，不赞成使用单例模式，就是99%的情况下不要使用单例模式。
2. 单例模式带来的问题：  
	1） 为自动化单元测试带来巨大困难   
	2） 系统/模块/组件之间耦合度增加，违背基本原则（如里氏替换原则）
3. 单例模式的经典实现给出不是很多，即使为经典实现，也存在诟病。
4. 如果真的想使用单例模式，请按照3条写前判断准则和3条写后判断依据来验证自己的实现。
5. 最后一条是我看到的一句话，很有道理：
   
> *The code will always tell you what to do. Just listen.*
> **代码会告诉你应该怎么做，你听就是了**。

关于Singleton，单例模式，就介绍到这里，文章中还有很多需要完善和改进的地方，我也会随着我经验的增加和学习的深入来弥补它的不足，也希望大家不吝赐教，批评指正，共同进步。

### 附录
------

1.**Use your singletons wisely.** 我的文章主要思想和内容由这篇文章指导  
[https://www.ibm.com/developerworks/library/co-single/](https://www.ibm.com/developerworks/library/co-single/ "User your singletons wisely.")

2.**Why singletons are controversial?** Google Code中的一篇文章，内容很有帮助  
[https://code.google.com/archive/p/google-singleton-detector/wikis/WhySingletonsAreControversial.wiki](https://code.google.com/archive/p/google-singleton-detector/wikis/WhySingletonsAreControversial.wiki)

3.**Singleton Pattern** Wikipedia上讲Singleton的的基本内容，也包括了Anti-Pattern的相关介绍  
[https://www.wikiwand.com/en/Singleton_pattern](https://www.wikiwand.com/en/Singleton_pattern "Singleton Pattern")

4.**Simply Singleton** JavaWorld上的文章，关于Singleton的几种实现方式和分析都在上面  
[http://www.javaworld.com/article/2073352/core-java/simply-singleton.html](www.javaworld/article/2073352/core-java/simply-singleton.html "Simply Singleton")

5.**Liskov Subsititution Principle** 里氏替换原则的wiki  
[https://www.wikiwand.com/en/Liskov_substitution_principle](https://www.wikiwand.com/en/Liskov_substitution_principle "Liskov Subsititution")