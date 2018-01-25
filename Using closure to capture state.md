# 使用闭包（closure）去捕捉状态
**Lambda表达式是无状态的，但你的程序必须要有状态。**

在Java语言中，我们宽泛地使用lambda表达式来完成lambda表达式或者闭包（closure）。但是在一些情况下，理解它们（lambda/closure）的区别是十分重要的。lambda表达式是无状态的，而闭包（closure）是携带状态的。用闭包来代替lambda表达式去管理函数式程序是一种很优雅的方式。

## 无状态的生命

在这一些列的教学中，我们已经和lambda一起工作了很久，而且我们也越来越了解它们。它们（lambda表达式）是微小的匿名函数，这些函数会携带者一些参数，执行一些计算，也可能会返回一个结果。Lambda表达式是无状态的，这会在代码中产生一些有趣的影响。

让我们来看一个使用lambda表达式的简单例子。假设我们希望在一组数字中找到偶数，一种方式就是去创建一个函数式的管道（pipeline） ，我们使用`Stream`的lambda表达式，像如下这样：
```
numbers.stream()
  .filter(e -> e % 2 == 0)
  .map(e -> e * 2)
  .collect(toList());
```

我们传递到`filter`中的lambda表达式是一个`Predicate`的函数式接口，它会携带一个数字进去然后验证这个这数字，如果这个数字是偶数，那么就返回`true`，否则返回`false`。传递到`map`的参数是一个`Function`函数式接口，它会携带任何一个数字然后返回它两倍的值。

这些lambda表达式都依赖于传入的参数和常量值。它们都是独用的（self-contained），这意味着它们不会有任何的外部依赖。因为它们依赖传入的参数，可能是常量，所以lambda表达式是无状态的。它们可爱（cute），安静，就像打盹的小孩子。

> NOTE：
> 末尾这句“它们都是独用的（self-contained），这意味着它们不会有任何的外部依赖。因为它们依赖传入的参数，可能是常量，所以lambda表达式是无状态的。”我来补充一下我理解的意思：lambda表达式就认常量，其他的它并不关心。一旦拿到一个满足条件的常量，它就好像把自己封闭起来，与外部环境隔绝，所以说它是无状态的，因为常量哪有什么状态可言呢！

## 为什么我们需要状态

让我来深入地看一个lambda表达式，这个表达式是我们传入到`map`方法中的。
```
.map((e, factor) -> e * factor)
```
假设我们希望计算给定值`e`的`factor`倍数的结果，这个lambda表达式可以适用。但是很不巧，它实际上不可用。因为`map`方法接受一个函数式接口`Function<T, R>`，`T apply(R)`，我们可以理解为输入一个`R`，输出一个`T`。但lambda表达式的函数签名和`Function`的`apply`函数不一致，正因为多了一个参数`factor`，像`BiFunction<T, U, R>`，那就必须有其他的方法让`factor`进入这个lambda表达式中。

## 词汇范围
 函数期望一些变量在自己的范围内。因为它们是匿名函数，所以lambda需要引用的变量也在范围内。一些变量是作为参数被lambda表达式中的，另一些则直接定义在本地。同时，还有一些变量定义在函数外部，而外部环境被称为语义词汇范围（lexical scope）。
 
 这有一个例子来说明它。
 ```
 public static void print() {
   String location = "World";
   Runnable runnable = new Runnable {
       public void run() {
           System.out.println("Hello" + location);
       }
   }
 }
 ```
 
 在`print`方法中，`location`是一个局部变量。然而，这个`run`方法引用了这个`location`，这个参数既不是本地变量也不是`run`方法的变量。这个紧挨着`"Hello"`的`location`变量绑定到了`print`方法的`location`变量上。
 
 一个词汇范围（lexical scope）就是定义一个函数的范围。相反，它也可能是界定范围的界定范围。
 
 在之前的代码中，`run`方法没有定义`location`或者它没有接受`location`作为参数。`run`方法定义范围是一个`Runnable`的匿名内部类。因为`location`没有定义在`Runnable`的这个实例中，那么编译器会持续的向匿名内部类中进行搜索，也就是这个定义范围中的`print`方法。
 
 如果在定义范围内没有出现`location`，那么编译的查找也会持续的在`print`的定义范围内进行，直到变量找到或者查找失败。
 
 ## 在一个lambda表达式中的词汇范围
 
 现在我们来看看用lambda表达式重写上面的方法将会发生什么：
 ```
 public static void print() {
     String location = "World";
     Runnable r = () -> System.out.println("Hello" + location);
 }
 ```
 
 这段代码更加的简洁了，这要多亏了lambda表达式，但是对于`location`词汇范围和绑定是没有变化的。在lambda表达式中的`location`