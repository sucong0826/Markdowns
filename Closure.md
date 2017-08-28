# What is Closure ?

### 简介
在编程语言中，闭包（Closure），也是词法闭包或者函数闭包，是用于在把**函数作为第一等级**的语言中实现词法范围的**名称绑定**的技术。操作上，一个闭包是一种**记录**，这个记录存储了**函数与环境**。所提及的**环境**：是一种**映射**，这样的映射关系关联了每一个函数的自由变量（变量在本地被使用，但是定义在一个封闭范围中）和创建闭包时绑定名称的值或者引用。一个闭包，不像一个普通的函数，它（闭包）允许函数去访问那些被捕获的变量（captured variables），通过闭包复制的变量的值或者引用，甚至当在函数之外的范围被调用。

例如，如下的程序代码段定义了一个高阶函数（**一个高阶函数需要一个或者多个函数作为参数，然后返回一个函数作为结果，其他所有的函数就都是一阶函数**），高阶函数为`startAt`，有一个参数是`x`，它内部有一个嵌套函数`incrementBy`，嵌套函数`incrementBy`有对`x`的访问权限，因为`incrementBy`是在`x`的词法作用范围之内，即使`x`并不是`incrementBy`的本地变量。函数`startAt`返回了一个闭包，这个闭包从对`startAt`的调用中得到包括了x值的拷贝，或者x引用的拷贝，而`incrementBy`函数完成了`x`和`y`的加法计算。

```JavaScript
function startAt(x)
  function incrementBy(y)
    return x + y
   return incrementBy
   
variable closure1 = startAt(1)
variable closure2 = startAt(5)
```

注意，作为`startAt`返回的一个函数，变量`closure1`和`closure2`是函数类型。虽然`closure1`和`closure2`参照了同一个函数`incrementBy`，但是所关联的环境是不同的，而且在两个不同的调用中，因此这个闭包将会用不同的值绑定`x`的名字到两个不同的变量上，从而将函数评估为不同的结果。

## 历史和词源
闭包的概念是在1960年为了lambda计算的机械评估而发展出来的，在1970年在PAL语言中首次被完全实现的。

## 匿名函数
闭包的术语通常被误解地来表示**匿名函数**的概念。这很有可能是因为很多程序员同时学习到了这两种概念，或认为以一些小的帮助函数形式出现（helper）的就是匿名闭包。一个匿名函数是一个**没有名字的函数的字面表示形式**，而闭包是一个**函数的实例，一个非本地变量已经被绑定到值，又被绑定到存储空间的值**。

一个匿名函数（函数字面表达，lambda抽象）是一个函数定义，这匿名数没有绑定到一个修饰符上（identifier）。

匿名函数通常：  
1. 作为参数被传递到高阶函数中
2. 用来构建需要返回函数的高阶函数的的返回结果

如果一个函数只使用一次或者几次，一个匿名函数可能比使用一个命名的函数更加轻量，更加方便。匿名函数在一些函数式的编程语言和以函数为第一等级的语言中随处可见。

匿名函数是**嵌套函数的一种形**式，它允许访问那些包含这个匿名函数中的非本地变量。这意味着**匿名函数需要使用闭包来实现**。不像有名字的内嵌函数，如果不在**fixpoint operator**或者将他们绑定到一个名字上的话，是无法实现递归的。

```Python
def f(x):
  def g(y):
    return x + y
  return g
  
def h(x):
  return lambda y : x + y 

>>> a = f(1)
>>> b = f(1)
>>> h(1)(5)
>>> f(1)(5)
```
`a`和`b`都是闭包，或者，具有闭包值的变量，在这两种情况下都是通过从闭包函数返回带有自由变量的嵌套函数的参数x。然而，在第一种情况下，嵌套函数有名字，`g`函数，而第二种情况就是匿名的。闭包不需要被分配到变量上，可以被直接使用，这种用法可被认为是一个**匿名闭包**。

尤其注意，被嵌套函数的定义不是**它们自己的闭包（themselves closures）**，他们有自己还没被绑定的变量。只有使用参数的值评估封闭函数时，嵌套函数的自由变量才会被绑定，从而创建一个闭包，通过封闭函数返回这个闭包。

最后，一个闭包与一个函数的区别仅在于那些非本地变量超出了作用范围之外的自由变量（free variables），否则定义环境和执行环境就会一致了，而且没有办法去区分这些。例如，在如下的程序中，函数有一个自由变量`x`，在全局范围中没有绑定到任何非本地变量，这些函数在定义了`x`的相同的环境中执行，所以无论是否是闭包，它都是无形的。

```
x = 1
l = [1, 2, 3]

def f(x):
  return x + y
 
map(f, 1)
map(lambda y: x + y, 1)
```

```
x = 0·
def f(x):
  return x + y

def g(z):
  x = 1
  return f(z)
  
g(1)
```

## Applications
闭包的使用是**和语言关联**的，这些语言是把函数作为第一等级对象的，同时这些高阶函数会返回函数作为返回结果，或者作为参数被传递到其他的函数调用中。如果有自由变量的函数是第一等级的话，会返回一个创建的闭包。闭包例如在JavaScript中通常被用来处理回调（callbacks），尤其是事件处理（event handlers），闭包被用在动态网页的交互部分。传统的命令语言，如Algol， C，Pascal，是不支持嵌套函数的，或者不支持当外部封闭函数退出时调用内部嵌套函数的。因此避免在这些函数中使用闭包。

闭包通常用来实现**连续传递风格**，利用这种方式**隐藏状态**。如构建这样的对象或控制结构是可以使用闭包来实现的。在一些语言中，一个闭包可能会在一个函数定义在另一个函数中出现，而且内部函数引用了外部函数的本地变量。在运行时，当外部函数执行时，会形成一个闭包，是由外部函数的变量通过闭包组成了内部函数的代码和引用。

#### First-class functions
闭包在那些以函数为第一等级的值的方式出现在一些语言中，换句话说，一些语言允许函数以参数的形式传递，然后从函数调用返回，绑定变量的名字等。看如下的例子：
```Scheme
; Return a list of all books with at least THRESHOLD copies sold.
(define (best-selling-books threshold)
  (filter
    (lambda (book)
      (>= (book-sales book) threshold))
      book-list))
```
在这个例子中，lambda表达式出现在函数`best-selling-books threshold`之内，当lambda表达式被评估时，Scheme创建了一个闭包，这个闭包由lambda表达式和一个指向`threshold`变量的引用组成，这个变量在lambda表达式中是一个自由变量。

接下来，这个闭包被传递到一个`filter`函数中，这个函数将会重复地调用来决定哪些书籍应该被添加到结果列表，哪些应该被放弃掉。因为闭包本身有一个引用指向`threshold`，每次`filter`函数调用这个闭包的时候，就会闭包就可以使用这个变量。而这个`filter`函数本身可能被定义在一个完全独立的文件中。

这有一个用JavaScript写的相同的例子。
```JavaScript
// Return a function that approximates the derivative of f
// using an interval of dx, which should be appropriately small.
function derivative(f, dx) {
  return function (x) {
    return (f(x + dx) - f(x)) / dx;
  };
}
```
在没有闭包的语言中，一个自动的本地变量的生命周期与声明这个变量的栈桢的执行周期保持一致。在有闭包的语言中，只要引用指向这些变量，那么它们的存在时间就和闭包一样长。这和某些实现了垃圾回收的语言是大致相同的。

#### State Representation 状态表述
一个闭包可以关联一组函数的私有变量，这些变量会坚持到几组调用之后。这些变量的作用域仅仅包括了封闭函数的，所以它不能被其他程序代码访问。在一些状态型的语言中，闭包可以被用来实现状态表述和信息隐藏的范式，因为闭包的封闭变量是不定性拓展的，所以在一次调用中构建的值在下一次调用中是可用的。闭包以这样的方式使用就不在有参照透明度，而且不在是纯函数了，尽管如此，他们通常被使用在非纯函数语言中，例如Scheme。

## Other uses 其他使用
- 因为闭包是延迟评估的，即直到它们被调用之前，闭包是不会做任何事情的，闭包可以用来定义**控制结构（control structure）**。例如，所有的Smalltalk的标准控制框架，包括分支（if/then/else）和循环（while／for），都是使用那些接受闭包的对象来被定义的。用户可以轻松地定义他们的控制结构。
- 在那些实现分发的语言中，封闭了相同环境的多元函数可以被生成，使得它们可以通过选择环境来进行私有地沟通。在Scheme中：
```Scheme
(define foo #f)
(define bar #f)

(let ((secret-message "none"))
  (set! foo (lambda (msg) (set! secret-message msg)))
  (set! bar (lambda () secret-message)))
  
(display (bar)) : prints "none"
(newline)
(foo "meet me by the docks at midnight")
(display (bsr))
```
- 闭包可以用来实现面向对象系统。

## Implementation and theory 实现与理论
闭包的典型实现是使用一个包含了一个指向函数代码的指针（**pointer to the function code : 函数指针指向内存中可执行的代码，而不是引用数据值**）的数据结构（**data structure**）来实现的，加上一个当闭包创建时的函数字面环境的表述（即一组可用的自由变量）。这个环境将非本地名称绑定到闭包被创建的字面环境中的变量上，至少将这些自由变量的生命周期拓展到和闭包本身的周期一样长。当稍后进入一个闭包时，可能会有一个不同的语义环境，该函数的执行是用被闭包捕捉的非局部变量，而不是当前环境的捕获的非局部变量。

如果一门语言是运行时的内存模型在一个线性栈上分配自动变量（automatic variable），那么想要轻松地实现完整的闭包并不是一件容易的事情。在这样的语言中，当一个函数返回的时候这个函数的自动变量将自动释放。然而，一个闭包需要它引用的自由变量存活着，直到闭包内函数执行完毕。因此，这些自由变量一定要被分配内存空间，持续到它们不再被需要，典型地就是通过堆内存分配，而不是栈分配，因为这些自由变量持续存活，它们的生命周期一定要被管理，直到所有引用它们的闭包不再使用为止。

这也解释了为什么本来支持闭包的语言也使用了垃圾回收机制。可选择的方案是对于非局部变量的手动管理（显式在对堆上分配内存空间然后用完释放），或者，使用栈分配，对于这样的语言来接受某些特定用例将会导致未定义的行为，这归咎于**悬挂指针（dangling pointers）** 释放自动变量的原因，如C++的lambda表达式或C语言中的嵌套函数。*funarg*问题（函数式参数问题）描述了以栈为基础实现函数的编程语言中以函数为第一等级对象的困难。

在拥有不可变对象（如：Erlang）的严格的函数式编程语言中，实现自动内存管理是很容易的，因为变量引用中没有可能的循环。例如，在Erlang中，所有的参数和变量都分配在堆上，但引用他们要额外存储在栈上一份，当函数返回的时候，这些引用仍然有效，堆内存的清理是依靠递增的垃圾回收器完成的。

闭包与并发计算的Actor模型的Actors角色关系密切，其中，函数的词汇环境中的值称为`acquaintances`。在并发的编程语言中，对于闭包来说一个很重要的问题就是在闭包中的变量是否可以被更新，而且，如果可以被更新，这些变量该如何被同步。Actors提供了一种方案。

闭包与函数对象（function objects）关系密切；这种从前到后的转换被认为是去函数化（Defunctionalization）或者lambda lifting；

## Differences in semantics 不同的语义
### 词汇环境
作为不同的语言不总是有一个共同的词汇环境的定义，但是关于闭包的定义可能相同。常用的词汇环境的极简定义定义了一组在作用域所有变量的绑定。这也正是任何一门语言的闭包需要去捕捉的。然而，变量绑定的含义扔有所不同。在命令式语言中，变量绑定到可以存储值的相对内存空间上。尽管绑定的相对内存空间不会在运行时发生变化，但是绑定空间的值可以变化。在这样的语言中，因为闭包捕捉了这样的绑定，在任何一个变量上的操作，无论是否从闭包上完成，都会同一个相对内存空间上执行动作。这通常叫做通过引用捕捉变量。这有一个例子解释了这个概念，ECMAScript中，有如下代码：
```ECMAScript
var f, g;
function foo() {
  var x;
  f = function() { return x++; };
  g = function() { return --x; };
  x = 1;
  alert('inside foo, call to f(): ' + f());
}

foo();
alert('call to g(): ' + g());
alert('call to f(): ' + f());
```
注意变量`f`和`g`所引用的函数`foo()`和闭包都使用由局部变量`x`表示的相同的相对存储空间。

另一方面，许多函数式语言，如ML，直接将变量绑定到值上。在这种情况下，一旦绑定，就没有方法去改变变量的值，也没有必要在闭包之间分享它们的状态。

一些语言可以去让你去选择是去捕捉变量的值还是内存空间。例如，在C++ 11中，被捕捉的变量的引用用[&]表示，被捕捉的值用[=]表示。

另一种子集，懒函数式语言（lazy），如Haskell，绑定变量到将要计算的结果上，而不是值。
```Haskell
foo::Fractional a => a -> a -> (a -> a)
foo x y = (\z -> z + r)
    where r = x / y
    
f :: Fractional a => a -> a
f = foo 1 0

main = print (f 123)
```
变量r的绑定是用来计算（x/y）的值，它被定义在`foo`函数中的闭包所捕获，真实的case结果用0做了除法。然而，因为这是一个被捕捉的计算，而不是值，闭包被调用时这个错误只表明了是闭包的错误，事实上，试图使用的是被捕捉的那个绑定。

### Closure Leaving 
然而，更多的差异表现在其他词汇范围结构的行为上，例如`return` `break`和`continue`等声明语句。通常，这样的结构可以通过由封闭的控制语句建立的转义延续来考虑（在break和continue的情况下，这种解释需要根据递归函数调用来考虑循环结构）。在一些语言中，如ECMAScript，`return`是指由关于语句的词汇最内层建立的延续。因此，一个闭包内的`return`语句将控制权转移到调用它的代码部分。然而，在SmallTalk中，表面上相似的运算符`^`调用为方法调用简历的转义继承，忽略任何中间嵌套闭包的转义延续。一个特定闭包的转义延续只能再SmallTalk中被隐式地调用通过到达闭包底部的代码。

```SmallTalk
foo
  | xs |
  xs := #(1 2 3 4).
  xs do : [:x | ^x].
  ^0

bar 
  Transcript show: (self foo printString) "prints 1"
```
```ECMAScrpit
function foo() {
  var xs = [1,2,3,4];
  xs.forEach(function (x) {return x;});
  return 0;
}

alert(foo()); // prints 0
```
以上的代码，表现的结果不同。因为SmallTalk的`^`操作符和JavaScript的`return`操作符不是类似的。在ECMAScript中的例子中，`return x`将会离开内部的闭包去开始`forEach`的循环迭代，而再SamllTalk的例子中，`^x`将会放弃循环而从方法`foo`中返回。

## Closure-like Constructs (Java)
在Java中，类允许被定义在方法内部。他们被称作为局部类。当这样的类没有被命名，它们被称作匿名类或者匿名内部类。一个局部类（或命名，或匿名）可以引用词汇层面上封闭类的名字，或者引用词汇层面上封闭方法的只读变量（被标记为`final`）。
```Java
class CalculationWindow extends JFrame {
    private volatile int result;
    ...
    public void calculateInSeparateThread(final URI uri) {
        new Thread() {
            new Runnable(
                void run() {
                  // it can read final local variables
                  calculate(uri);
                
                  // it can access private fields of the enclosing class
                  result = result + 10;
              }
            ).start();
        }
    }
}
```

 `final`变量的捕捉可以使你去通过值来捕捉变量，即使你想捕捉的变量是非`final`的，仅在类之前，你可以总是复制这个变量到一个临时的`final`变量中。

通过一个引用来捕捉变量是可以通过使用一个`final`引用到一个可变容器来模拟的，例如，一个单一元素的数组。局部类没有能力去改变容器引用本身的值，但是它会改变容器的内容。

随着Java8的到来，闭包将导致如下的代码被执行：
```Java
class CalculationWindow extends JFrame {
    private volatile itn result;
    ...
    public void calculateInSeparateThread(final URI uri) {
        new Thread(() -> 
            calculate(uri);
            result = result + 10;
        ).start();
    }
}
```

局部类是内部类的一种类型，它声明在一个方法的函数体内。Java同样支持内部类被声明为一个封闭类的非静态成员。它们通常被称为内部类。它们被定义在封闭类内部的，而且有完整的访问封闭类变量实例的权限。由于绑定到了这些实例变量上，一个内部类可能只会通过一种特殊的语法用一个显式绑定到一个封闭类的实例来完成初始化。
```Java
public class EnclosingClass {
    public class InnerClass {
        public int incrementAndReturnCounter() {
            return count++;
        }
    }
    
    private int counter;
    
    {
        counter = 0;
    }
    
    public int getCounter() {
        return counter;
    }
    
    public static void main(String... args) {
        EnclosingClass eci = new EnclosingClass();
        EnclosingClass.InnerClass innerClassInstance = enclosingClassInstance.new InnerClass();
        
        for(int i = enclosingClassInstance.getCounter(); (i = innerClassInstance.incrementAndReturnCounter()) < 10;) {
            System.out.println(i):
        }
    }  
}
```

Java 8，Java支持了函数作为第一等级的对象。这种形式的Lambda表达式被认为是`Function<T, U>`，其中，`T`是域，`U`是假定类型。这个表达式可以被它的`.apply(T t)`方法调用，但不是用一个标准的方法调用。
```Java
public static void main(String... args) {
    Function<String, Integer> length = s -> s.length();
    System.out.println(length.apply("Hello, world!"));
}
```

## Terms 术语表
- Automatic variable 自动变量：在计算机编程语言中，自动变量是一个本地（局部）变量（local variable），当程序流进入并离开这个局部变量的作用域时，它将自动分配和释放。这个作用域是词意的环境，就是一个自动变量被定义在的函数或者代码块的环境。
  > *An automatic variable is a local variable which is allocated and deallocated automatically when program flow enters and leaves the variable's scope. The scope is lexical context, particularly the function or block in which a variable is defined.*

- Dangling Pointer
悬挂指针：悬挂指针和野指针在编程语言中是指指针没有指向一个合适类型的有效对象上。这些都是内存安全违规的特殊情况。

- Funarg problem(functional arguments problem)
函数式参数问题：在计算机科学中，funarg问题指的是在编程语言中使用基于栈的函数内存分配来实现以函数为第一等级对象的困难。这样的问题只会出现在有嵌套函数的主体（即不是通过参数传递）部分直接引用定义在函数定义的环境中的标识符，而不是在函数调用的环境中。

- function object
函数对象：在计算机程序中，一个函数对象是一个允许被调用的对象，或者好似一个被执调用的普通函数，它们通常有相同的语法（一个函数也可以作为函数的参数）。函数对象也通常叫做函数因子（函子）。

- Defunctionalization 
函数降阶：在编程语言中，函数降阶指的是，一个编译时期的转型。这个转型通过使用一阶应用函数代替且消除了高阶函数。

- Lambda lifting：Lambda提升是重组计算机程序的元过程，使得函数在全局范围内彼此独立地定义。一个独立的“lift”将一个局部函数转换为全局函数。

- Lazy evaluation：
在编程语言中，lazy evaluation 延迟评估，或者叫call-by-need，需要时调用，是一种评估策略，这种策略会延迟评估一个表达式，直到这个值被需要，而且这种策略同样会避免重复的值评估，也就是会分享评估结果。这种分享可以通过一个指数因素减少某些功能的运行时间，而不是其他非严格的评估策略，例如按名称调用。

- Name binding 名称绑定
在编程语言中，名称绑定是一种条目与标识符的关联，条目可能是数据（data）或者代码（code）。绑定到一个对象的标识符可以说成是这个对象的引用。机器语言没有内部构建的标识符，但对于程序员来说，命名的对象绑定作为一种服务和强调，是通过编程语言来实现的，换句话说，绑定与机器指令无关，是以编程语言为基础实现的。绑定与作用域（scoping）链接紧密，也就是说作用域决定了名称可以绑定到哪些内存空间上的哪些对象上，且是一条可执行的路径上。

在上下文环境中，使用一个标识符`id`。当前上下文环境为`id`建立了一个绑定，称作绑定事件，在所有的事件中（如表达式，分配，子程序调用），一个标识符表示了它所绑定的内容；这种情况成为应用事件。