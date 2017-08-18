## 模板模式，看这一篇就差不多了

### 写在开始

实践出真知，这是个真理。书看的多了，事件好事情，至少对我来说，是这样的。最近在解决一个问题的时候，在《Thinking in Java》的帮助下，快速的解决了一个设计问题。所以，利用周末的时间，整理一下思路，同时，还有我是如何想到解决方式的。思路 + 尝试 = 方案。

今天的主题，是仿照Java GUI库，如Swing，解决问题的一套方案。我将提及方案介绍、思路分析、实战代码，三个部分。请注意：**文章的主要指导来自于《Thinking in Java》（第四版）以及《Head First Design Pattern》（Original Version）MindView《Thinking in Patterns》**

关键词：**模板模式**    **控制框架(Control Framework)**  **实例生成器(Generator)**


## 目录

- 模板模式介绍
- 控制框架介绍
- 实例生成器
- 实战分析

## 一、 模板模式介绍

模板模式，全名：**模板方法模式（Template Method Pattern）**。

让我们先来看一个例子，理解一下什么是模板方法模式（以下例子选取自《Head First Design Patterns》）:

在一家Coffee店，有两种非常畅销的饮品，一种是主打的Coffee，另一种是中式的茶品。我们暂且将这两种饮品命名为：

1. Coffee Beverage
2. Tea Beverage

通过询问得知了两种饮品的大致制作方式如下：  

1. 烧水
2. 在热水中搅拌咖啡/将茶包浸泡
3. 将咖啡/茶导入杯中
4. 根据客人喜好加入其他配料，如糖，牛奶等

在实际的开发中，肯定会遇到这样的流程化或者标准的执行动作的场景。通常，在这种情况下，为了要完成某一个特定业务，需要执行一系列连续的准备任务，或者执行某些标准函数来完成准备工作。正如上面的例子，其实只要是热饮品，制作流程都可以抽象成如上的4个步骤，但是可能针对于某个步骤，不同类型的饮品可能略有差别。

现在，我们先按照如上步骤提供基础的实现：

首先是Coffee的代码实现：

```Java
public class Coffee {

  // preparation works
  void prepareBeverage() {
  boilWater();
  blendCoffeeGrinds();
  pourInCup();
  addMilk();
  }

  // boil water
  private void boilWater() {
// implementation
  }

  // blender coffee grinds
  private void blendCoffeeGrinds() {
// implementation
  }

  // pour in cup
  private void pourInCup() {
// implementation
  }

  // add milk into coffee
  private void addMilk() {
// implementation
  }
}
```

接着是Tea的实现：
```Java
public class Tea {

  // preparation works
  void prepareBeverage() {
      boilWater();
      steepTeaBag();
      pourInCup();
      addLemon();
  }

  // boil water
  private void boilWater() {
    // implementation
  }

  // steep tea bag into boiled water
  private void steepTeaBag() {
    // implementation
  }

  // pour in cup
  private void pourInCup() {
    // implementation
  }

  // add lemon into tea
  private void addLemon() {
    // implementation
  }
}
```

OK，我们现在得到了`Coffee`和`Tea`的基础实现，不管怎样，当有顾客需要这两种饮品，可以分别创建他们的对象，然后调用`prepareBeverage()`方法，就可以完成饮品的制作过程。

但是，这样的程序，拓展性以及维护性都是很差的。这样的说的原因很简单，无论增加任何一种饮品，都要按照新的类型进行编写，虽然它们的流程几乎一致。同时，一旦要求饮品的制作流程要求统一化，麻烦就大了，必定面对着代码和结构的重构，虽然这样是对的，但代价是昂贵的。

所以，为了业务的**可拓展性**和**维护性**，对这块业务进行设计。我们最终的目的就是希望**抽象出一套模板实现逻辑**。

我们简单分析一下`Coffee`和`Tea`的实现，摆在面前的困难有几个：
- Coffee和Tea的制作流程有细微差异：Coffee需要将研磨的咖啡进行搅拌，而Tea是需要将茶包浸泡。在根据顾客的口味上，Coffee一般需要配置牛奶或者糖分，而茶品如红茶一般需要添加些柠檬等。这是第一条差异。

- 基于模板，我们希望Coffee和Tea的制作流程统一管理。但是，Coffee和Tea都是基于自己的流程提供了代码实现，这样做就偏离了目标。所以，第二条就是需要它们的制作流程模板化。

![Comparation between Coffee and Tea](http://i.imgur.com/k347mbk.png)

对于以上遇到的问题，我们尝试给出解决方案。观察发现，`Coffee`和`Tea`在制作步骤上稍有差异，但是去抽象这样的制作步骤，可以得到一致的结果。无论是`blendCoffeeGrinds()`还是`steepTeaBag()`，都是制作饮品的关键步骤，所以，我们将其抽象为方法` make()`。

同时，`Coffee`需要增加Milk或者Sugar，而Tea需要增加Lemon等，所以得到抽象方法：`addCondiments()`。

结合第二条，我们将饮品的制作抽象如下：
![Beverage and its subclass class diagram](http://i.imgur.com/61d5NUa.png)

```Java
public abstract class Beverage {

  // make this method final to avoid modifying by subclass.
  final void prepareBeverage() {
    boilWater();
    make();
    pourInCup();
    addCondiments();
  }

  abstract void make();

  abstract void addCondiments()

  void boilWater() {
    // simple implementation
  }

  void pourInCup() {
    // simple implementation
  }
}
```

```Java
public class Coffee extends Beverage {

  // blender coffee grinds
  private void make() {
    // own impl
  }

  // add milk or sugar into coffee
  private void addCondiments() {
    // own impl
  }
}
```

```Java
public class Tea extends Beverage {

  private void make() {
    // own impl
    // steep tea bag
  }

  private void addCondiments() {
    // add lemon into it
  }
}
```

```Java
public class BeverageTest {
  public static void main(String... args) {
    Tea tea = new Tea();
    tea.prepareBeverage();

    Coffee coffee = new Coffee();
    coffee.prepareBeverage();
  }
}
```

OK，以上就是模板方法的一个简单实现。

> Quoted from《Head First Design Patterns》
>
> **The Template Method Pattern** defines the skeleton of an algorithm in method, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm's structrue.

**模板方法定义了算法的步骤，同时允许子类来提供其中一步或者多步的具体实现。**

那么，什么是模板呢？**所谓模板就是一个定义标准的方法**。这个方法可以定义一个算法，而子类可以选择给这个算法提供自己的实现。

说到这里，我就将模板方法的设计模式介绍完毕。它在系统中有什么用？请接着向下看。

## 二、控制框架介绍

什么是控制框架？控制框架，Control Framework。是**一类特殊的应用程序框架， 它用来解决响应事件的需求。主要用来响应时间的系统称作*事件驱动系统*。**

记得大学学习Java的时候，老师要求的软件设计作业，需要利用AWT或者Swing做GUI部分，那时是第一次接触Swing库。不过只是记得利用各种Event做各种回调处理，并没有深入学习。

Swing库事件响应的核心，就是Java的**内部类**与**观察者模式**。无论是一般内部类还是匿名内部类，都可以优雅地解决事件驱动。

```Java
public class MyPanel extends JFrame {
  MyPanel() {
    JPanel panel = new JPanel();
    JButton submitBtn = new JButton("Submit");
    JButton cancelBtn = new JButton("Cancel");
    panel.add(submitBtn);
    panel.add(cancelBtn);

    submitBtn.addActionListener(new ActionListener() {
        @Override
        public void actionPerformed(ActionEvent event) {
            System.out.println("You've pressed Submit button!");
        }
    });
  }
}
```

Swing提供了很多Event事件，它们都用`ActionEvent`来描述。任何需要对组件的事件进行监听的类，都需要实现`ActionListener`这个接口，此接口用来接受Event事件。

`JComponent.java`的部分源码：

```Java
public abstract class JComponent extends Container implements Serializable, TransferHandler.HasGetTransferHandler {
  final class ActionStandin implements Action {
    private final ActionListener actionListener;
    private final String command;
    ...

    ActionStandin(ActionListener actionListener, String command) {
      this.actionListener = actionListener;
      if(actionListener instanceof Action) {
          this.action = (Action) actionListener;
      } else {
        this.action = null;
      }
      this.command = command;
    }

    public void actionPerformed(ActionEvent ae) {
        if (actionListener != null) {
            actionListener.actionPerformed(ae);
        }
    }
  }
  ...
}
```

类似`JButton`这样的Swing组件，都有一个全局变量的`ActionListener`，从`JComponent`的代码可以看出，组件的动作用一个`ActionStandin`来表示，传入的`ActionListener`被转型为`Action`。当有事件发生的时候，会执行`actionListener.actionPerformed(ae)`，这样，传入的内部类实例的`actionPerformed`方法被回调，就可以执行一些UI交互的动作了。

##### 写着这个章节的意义在于，GUI的事件驱动框架给了我一定启发，在Android开发过程中，一个UI界面可能会有很多组件，在业务关联性很强的情况下，结合之前的**模板方法**，一些组件背后的业务逻辑要完成某一项特定任务，我们就可以结合GUI的事件驱动框架来完成。看的云里雾里？没关系，在最后的实战章节，会看到它们是如何结合使用的。

## 三、实例生成器

实例生成器，Generator，是一个非常适合于放在你自己ToolBox里的工具，至少我是这么做的。直接上代码。**这部分内容来自于《Thinking in Java》第15章节**。

```Java
// Generator.java
public interface Generator<T> {
  T next();
}
```

```Java
// Basic Generator
public class BasicGenerator<T> implements Generator<T> {
  private Class<T> type;

  private BasicGenerator(Class<T> type) {
    this.type = type;
  }

  public T next() {
    try {
      return type.newInstance();
    } catch(Exception ex) {
      throw new RuntimeException(ex);
    }
  }

  public static <T> Generator<T> create(Class<T> type) {
    return new BasicGenerator<T>(type);
  }
}
```

```Java
//Generator Test
public class GeneratorTest {
    public static class Book {
        // dummy class
    }

    public static void main(String[] args) {
        Generator<Book> generator = BasicGenerator.create(GeneratorTest.Book.class);
        Book book = generator.next();
        System.out.println(book.toString());
    }
}
```

![GeneratorTest Result](http://i.imgur.com/Xj4X6Ds.png)


它会在接下来的解决方案中起到很大的作用。

## 四、实战分析

先写一段我的心得：我常常在解决问题的时候去回想我之前到底做过什么，学过什么。记得我在InfoQ看过陆奇给百度员工的建议的一段话，“作为程序员，要写有意义的代码。何为有意义的代码？每一行代码都是值得写的，对于那些已经存在的，有价值的代码可以直接使用，对于自己要写的，要问问清楚，这些是否有价值。” 所以，每次我在尝试解决问题的时候，都要先在脑子过一遍，这些代码是否会带来价值，如果没有，那就算了，别写了。

同时，对于以前自己遇到的一个生涯瓶颈，现在也有了新的认识。就是自我学习与项目如何并存。很多人告诉我，它们是互斥的。其实大部分是这样的，但是，我在每次遇到问题都要去套用，所谓套用，就是将问题去和自己学过的知识进行匹配，哪怕只有5%-10%的匹配度，也要去硬性的套用一次，即使失败，我也深入地了解了所学所用。

在这小段，我要摘取一段《Thinking in Java》的作者Bruce Eckel的一句话，在他讲解泛型的章节开篇，他写到：
> 为什么在学习Java泛型的时候要与C++对应部分进行比较。你可以了解Java泛型的局限是什么，以及为什么会有这些限制。最终的目的是帮助你理解，Java泛型的边界在哪里。根据我的经验，理解了边界所在，你才能成为程序高手，正因为只有知道了某个技术不能做到什么，你才能更好地做到所能做的。

好，开始正题。我要表述的是我实际项目中遇到问题及我如何利用以上内容给出解决方案。（**由于这是项目内容，为了保密，我做了一些调整。**）

![Remote Control](http://i.imgur.com/W90cepn.png)

这是实际项目中的一个Remote Control，即**远程遥控器**。 上面一共有6个组件，每个组件都有各自的功能，又有一些组合键的功能，即组件可以单一完成任务，也可以几个组件协同作用，完成特定任务。

在我以前的知识库，拿到这样的需求，肯定就这样完成了。如下：

```Java  
public class RemoteControlActivity extends Activity implements View.OnClickListener {
	
	@Override
	public void onCreate(@NonNull Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_remote_control);
		
		findViewById(R.id.zoomOut).setOnClickListener(this);
		findViewById(R.id.zoomIn).setOnClickListener(this);
		findViewById(R.id.lookUp).setOnClickListener(this);
		findViewById(R.id.lookDown).setOnClickListener(this);
		findViewById(R.id.turnLeft).setOnClickListener(this);
		findViewById(R.id.turnRight).setOnClickListener(this);
	}

	@Override
	public void onClick(View v) {
		int vId = v.getId();
		switch(vId) {
			case R.id.zoomOut:{
				zoomOut;
				break;
			}
			...
		}
	}
	
	private void zoomOut() {
		// implementation
	}
}
```

其实，这样完全是可以解决问题的。但是，这样的代码没有任何设计，没有考虑任何拓展性、健壮性、可维护性。 任何人都可以写出来，而且**没有任何价值**。所以我就在脑袋里搜索，是否可以好好设计一下，利用自己的所学所见。恰好，**GUI的事件驱动**和**模板方法模式**进入我的考虑范围之内，我就抱着试试的态度进行了尝试。

首先，我为整体业务进行了梳理，经过一番梳理，得到了以下结论。

1. **无论各个组件如何组合，每个组合都可以抽象为一个Event，即事件。**
2. **每一个事件（Event）都是可以执行某一特定任务，关于业务层的逻辑一定要放在动作中去执行。**
3. **事件用来执行特定算法，就犹如第一章节中的模板方法中的`prepareBeverage()方法。`**

遂我的第一个类，也就是**抽象的事件驱动模板**，有了雏形。  
![Event Class Diagram](http://i.imgur.com/Jlc4MWl.png)

```Java
public abstract class Event {
	// You can compose your own algorithm in method action().
	abstract void action();

	private int instruction;

    protected int getInstruction() {
        return this.instruction;
    }

    protected void setInstruction(int instruction) {
        this.instruction = instruction;
    }
}
```
在`action()`方法中，我们可以编写特定逻辑的算法。当`action()`方法被调用的时候，它会执行这些特定算法。

接下来，我们需要为事件们提供控制器，何为控制器？即执行某一个事件/某一组特定事件序列。经过整理，我得到了如下的雏形。
![Controller Class Diagram](http://i.imgur.com/bOspbWl.png)

```Java
public class Controller {
    /**
     * single event for being executed by controller
     */
    private Event singleEvent;

    /**
     * A series events being executed by controller.
     */
    private List<Event> seriesEvent;

    public Controller() {
        seriesEvent = new ArrayList<>();
    }

    public void addSingleEvent(Event singleEvent) {
        this.singleEvent = singleEvent;
    }

    public void addSeriesEvent(Event event) {
        if (event != null)
            seriesEvent.add(event);
    }

    public void runSingle() {
        if (singleEvent != null)
            singleEvent.action();
    }

    public void runSeries() {
        if (seriesEvent != null && !seriesEvent.isEmpty()) {
            for (Event event : seriesEvent)
                event.action();
        }
    }
}
```
现在，在这个基础的`Controller`中有了两个方法`runSingle()`和`runSeries()`，它们分别执行了单一事件和序列事件的任务，即，无论现在有什么样的事件，只要经过这个`Controller`，都可以被执行，所以，basic controller的代码就可以了。

接下来，我们来定义系统的各个事件。我总结的事件如下：  

- ZoomIn 	视角缩小 = ZoomInEvent  
- ZoomOut 	视角放大 = ZoomOutEvent
- TurnLeft	视角左移 = TurnLeftEvent
- TurnRight 视角右移 = TurnRightEvent
- LookUp	视角上移 = LookUpEvent
- LookDown  视角下移 = LookDownEvent
- （其他拓展事件）

我们给出具体事件控制器的代码：

```Java  
public class RemoteController extends Controller {

    public static class EventFactory<T> implements Generator<T> {

        private Class<T> type;

        private EventFactory(Class<T> type) {
            this.type = type;
        }

        public T next() {
            try {
                return type.newInstance();
            } catch (Exception ex) {
                throw new RuntimeException(ex);
            }
        }

        public static <T extends Event> Generator<T> create(Class<T> type) {
            return new EventFactory<T>(type);
        }
    }

    public class ZoomInEvent extends Event {

        public ZoomInEvent() {
            // do nothing
        }

        @Override
        public void action() {
            ctrlCam(getInstruction());
        }
    }

    public class ZoomOutEvent extends Event {

        public ZoomOutEvent() {
            // do nothing
        }

        @Override
        public void action() {
            ctrlCam(getInstruction());
        }
    }

    public class ShiftEvent extends Event {

        public ShiftEvent() {
            // do nothing
        }

        // def direction is 0.
        private int direction = 0;

        public void setDirection(int direction) {
            this.direction = direction;
        }

        @Override
        public void action() {
            shift(direction);
        }
    }


    // real biz logic
    private void ctrlCam(int instruction) {

    }

    // shift logic
    private void shift(int direction) {

    }
}

```

那么，如何利用控制器执行事件：

```Java
public class TemplateTest {

    public static void main(String[] args) {
        RemoteController remoteController = new RemoteController();

        RemoteController.EventFactory<RemoteController.ShiftEvent> factory = new RemoteController.EventFactory<>(RemoteController.ShiftEvent.class);
        RemoteController.ShiftEvent shiftEvent = factory.next();
        remoteController.addSingleEvent(shiftEvent);
        remoteController.runSingle();
    }
}
```

## 五、总结

最后，我对以上的内容做些总结，只有一个主题，为什么要这么做！

首先，我觉得很酷！不管内容是不是最合适的，效率最高的，至少我换了一种以前从未尝试过的方式解决了问题，也是一套我自认为成型的方案。所以，不管怎样，我是成功的。

其次，GUI的事件驱动框架，让执行变得可控，无论是单一事件，还是序列事件，都可以很好的执行，同时`action()`方法中只关注真实的算法和业务逻辑，那些UI相关的就放在对应的UI实现就可以了。

实例生成器，将事件的创建封装了起来，让内部实现细节不可见，只需要提供类的class信息，便可以创建对象。利用泛型结合变种的工厂模式来对事件进行创建，创建更加简单。

### 六、资料参考

1. 《Thinking in Java》 第四版 第十章 第十五章
2. 《Thinking in Patterns》 Template Method Pattern
3. 《Head First Design Patterns》
