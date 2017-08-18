### Java Tricks系列（三）StringBuilder：“不要骗我”

#### 写在开始

好久没有更新Tricks系列的小文章了，最近每天早晨都在看《Java编程思想》，里面有些内容非常适合trick系列的东西，自己做了尝试，稍作调整和整理，拿出来分享一下。今天这篇是关于StringBuilder的，听我慢慢道来。

### StringBuilder的简介

StringBuilder在Java SE5之后引入，在它之前，Java提供了StringBuffer来完成字符串的很多操作，但是因为StringBuffer是线程安全的。既然它是线程安全，想必要有一部分开销来保证线程安全，那么在处理大量字符串操作的时候，它的效率很低一些。Java SE5之后，提供了StringBuilder来完成字符串的操作，虽然它无法保证线程安全，但在处理字符串资源上，它相比于“+”的重载与StringBuffer来说，性能上要优秀很多。

    A modifiable CharSequence sequence of characters for use in creating strings. 
	This class is intended as a direct replacement of StringBuffer for non-concurrent use; 
	unlike StringBuffer this class is not synchronized.
    
    The majority of the modification methods on this class return this so that method calls can be chained together. For example: new StringBuilder("a").append("b").append("c").toString();

-------

### 抛出问题

首先我们先梳理一个Best Practice的部分，在Java中虽然没有显式的C++的操作符重载，但是对于“+”来说，在操作字符串上依然是重载的，也就说，它在执行算数运算时，是加法，但在执行字符串操作时，就被隐式地重载了，变成了字符串拼接操作。 这个很好理解，“+”的重载操作JVM会替我们做一次优化，不过它依然存在效率的问题，我们看如下的代码。
	
	public class StringTester {
		public String concatStrings(String[] fields) {
	        String result = "";
	        for(int i = 0; i < fields.length; i++) {
	            result += fields;
	        }
	        return result;
	    }
	
	    public String appendStrings(String[] fields) {
	        StringBuilder result = new StringBuilder();
	        for(String item : fields) {
	            result.append(item);
	        }
	
	        return result.toString();
	    } 
	}

这段代码中包含了两个方法，第一个方法：`concatStrings(String[] fields)`利用“`+`”进行字符串拼接，第二个方法：`appendStrings(String[] fields)`利用StringBuilder完成字符串操作。两个方法目的相仿，途径不同，看看会有什么样的结果？（想做深层次的优化，还要从JVM原理去看代码，这是很有帮助的。）

我们利用javap逆向得到JVM的字节码内容 `javap -c StringTester`。

如下：

	public java.lang.String concatStrings(java.lang.String[]);

    Code:
       0: ldc           #2                  // String
       2: astore_2
       3: iconst_0
       4: istore_3
       5: iload_3
       6: aload_1
       7: arraylength
       8: if_icmpge     36
      11: new           #3                  // class java/lang/StringBuilder
      14: dup
      15: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
      18: aload_2
      19: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      22: aload_1
      23: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/Object;)Ljava/lang/StringBuilder;
      26: invokevirtual #7                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      29: astore_2
      30: iinc          3, 1
      33: goto          5
      36: aload_2
      37: areturn

我们从code line 8开始看，这里有指令`if_icmpge`,含义为“比较栈顶两个int数值的大小，当结果大于或等于0时执行跳转”，后跟跳转行数 line 36。我们先不用关注到底比较的结果如何，因为concatString()方法中执行比较的只有loop-for中的条件比较，那么暂且我们知道从这个指令之后，开始执行for中的代码指令。“+”是一个重载操作符，但是无论怎样，它都要在JVM中对应一条执行指令，我们可以看到，Code 11执行了一个new指令，备注为class java/lang/StringBuilder，简单立即，这段代码生成了一个StringBuilder的对象，接着指令完成了初始<init>操作，并且真正字符串的拼接操作都是利用StringBuilder的append()方法完成的。

	小结1：“+”的重载操作符是利用StringBuilder的append来完成的。

可这里**一定要注意**：Code 33 执行了goto跳转，这是loop循环的下一次的调用指令，即loop完成了一次，该进行下一次loop操作了。虽然loop在goto的行数是5，但是依然从5行开始向下执行相应的JVM指令，也就是说，11行的new指令依然要被执行很多次，即**只要loop在进行，那么new指令就要被执行，每一次循环都要生成一个StringBuiler的实例，这就是问题所在**。

	小结2：虽然“+”重载操作符执行字符串拼接操作，但JVM对其做了优化，这次优化是利用StringBuilder来完成的。 如果执行一次操作，完全OK，但是循环的拼接操作暴露了其弊端，StringBuilder每次loop会创建实例，这是没有性能上考虑的，不是最好的手段和途径。


那么，StringBuilder来替代“+”之后，会有什么变化呢？来看字节码。

 	public java.lang.String appendStrings(java.lang.String[]);
    Code:
       0: new           #3                  // class java/lang/StringBuilder
       3: dup
       4: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
       7: astore_2
       8: aload_1
       9: astore_3
      10: aload_3
      11: arraylength
      12: istore        4
      14: iconst_0
      15: istore        5
      17: iload         5
      19: iload         4
      21: if_icmpge     43
      24: aload_3
      25: iload         5
      27: aaload
      28: astore        6
      30: aload_2
      31: aload         6
      33: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      36: pop
      37: iinc          5, 1
      40: goto          17
      43: aload_2
      44: invokevirtual #7                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      47: areturn

我们看，从Code line 0开始，就执行了new指令，之后line 17 - 50的loop操作不会有StringBuilder对象再被创建，这就是一个Best Practice。

	小结3：在执行一次字符串拼接操作（"a" + "b"），“+”与StringBuilder性能效率一致，但是，在多次循环拼接中，我们看到了“+”的弊端，所以，我们在要尽量使用StringBuilder去完成字符串操作。

### Trick在哪里？

我要表表述的trick，其实是developer有时候的自以为是，利用了StringBuilder，去欺骗了它，此话怎讲？看代码吧！

	public String appendStrings(String[] fields) {
	        StringBuilder result = new StringBuilder();
	        for(String item : fields) {
	            result.append(item + ":" + "suffix");
	        }
	
	        return result.toString();
	    }

我可以保证，我们在使用StringBuilder的时候，会出现这样的情况，我将上述的代码调整了一下，也是常见的问题之一，即在StringBuilder的append()方法中进行字符串拼接，这对JVM来说，或许是个欺骗性的行为，因为JVM不会对append(item + ":" + "suffix")做优化操作，依然会生成新的StringBuilder对象。 为啥，看看JVM是怎么被骗的吧！

	public java.lang.String appendStrings(java.lang.String[]);
    Code:
       0: new           #3                  // class java/lang/StringBuilder
       3: dup
       4: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
       7: astore_2
       8: aload_1
       9: astore_3
      10: aload_3
      11: arraylength
      12: istore        4
      14: iconst_0
      15: istore        5
      17: iload         5
      19: iload         4
      21: if_icmpge     61
      24: aload_3
      25: iload         5
      27: aaload
      28: astore        6
      30: aload_2
      31: new           #3                  // class java/lang/StringBuilder
      34: dup
      35: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
      38: aload         6
      40: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      43: ldc           #8                  // String :suffix
      45: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      48: invokevirtual #7                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      51: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      54: pop
      55: iinc          5, 1
      58: goto          17
      61: aload_2
      62: invokevirtual #7                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      65: areturn

前面的很相似，我们直接跳过，直接说重点。 Code line 31,惊现new指令，不管怎么样，StringBuilder又要被创建了。 之后完成 item + ":" + suffix操作，这就奇怪了，我们明明使用了append()方法，为啥依然创建了StringBuilder对象呢？

原因很简单，因为他是计算机，它是按章办事，不分等级的。无论何时见到“+”重载操作符，它的操作和动作总是保持一致，那就是创建一个StringBuilder对象给你，既然StringBuilder希望完成优化的任务，那么就不要欺骗它，规规矩矩地做如下的操作：
	
	sb.append(item).append(":").append("suffix");

### 写在最后

写了这么多，无非为了最后最后的一个Best Practice。

**总结：请优先使用StringBuilder（不考虑线程安全的情况下）来完成字符串的操作来代替“+”重载操作符，但是请注意，不要耍花样，每个`append()`方法中不要执行`“+”`的字符串拼接操作，这样无法达到优化目的。**