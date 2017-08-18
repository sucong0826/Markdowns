# 让注解更加有用（译文）

设计注解要保守且小心一些

在这个讨论中，作为先导功能的是一个充分得到普遍好评的库，名字是Project Lombok.这个库可以使你避免使用注解来写示例。例如，为一个类添加`@Data`注解，那么Lombok将会生成`getters`和`setters`方法，加上那些额外的需要在JavaBean中体现的方法，如`toString()`方法。Lombok方式很吸引我，对于很多的开发者来说它也是实用，清晰且有影响的。除此之外，这个工程包含了一个工具，叫做"DELOMBOK"，这个工具可以移除注解然后向类中插入示例模板，用这种方法你可以简单地移除对Lombok的依赖。在注解的表达和角色中，它们是保守的，而且这个工程有一个清楚的，可逆的实现。更多的注解应该遵循这样的方式。

注解，当它们出现在Java中，就遵循了一个相仿的简洁的模型。这些注解首先出现在Java5中，它们优雅且简洁，而且企图不想做很多事情：`@Override`和`@Deprecated`告诉了我们一些关于代码的信息，正如`@SuppressWanings`所表达为某些工具正在做的事情一样。意图在于那些工具，尤其是IDE，会使用这些标志来发出警告和体型。没有任何一个注解会改变程序的行为。Java语言小组持续了这种保守的行为并且在Java7中增加了`@SafeVarargs`，在Java8中增加了面向函数式变成的函数式接口`@FunctionalInterface`。

除了我提到的这些性质，这些**注解同样语义清晰。这是一个关键点，而且经常在注解设计中被忽略了**。为了简洁，注解的作者通常将对注解的理解性强行推给开发者。看起来没有深层次的语义但实际缺乏基础语法的协作能力。一个主要的违反者就是`not-null`限定符家族。`@NotNull`就是这样的一个注解，`@NonNull`同样如此。在FindBugs下面，这个注解有不同的语义，正如`@Nullable`在Checker框架下和FindBugs框架下含义不同一样。可这不是着手做事情的方法，**如果你是注解的作者，注解一定要清晰，请一定要避免同义词和同音异义词**。

比较这些言简意赅的注解，Java EE引入了注解的拓展使用。很快，一系列的注解代替了代码，企业级程序的编码性质也由此改变。Java EE已获得了一组注解嵌入的语法，这样的语法跨越了描述和命令。这种风格的跨越式语法的到来再次盘活了Java EE，用更简单的编码方法避开了先前方式的负担。

然而，这个进步鼓舞了一大批框架去使用甚至滥用注解，其中的很多都是不建议的。这些框架引入了复杂性，同时也没有足够好的文档来做为向导提供给开发者阅读和使用。作为注解，成为了在别处定义动作的复杂难懂的标志，你最终会放弃这样的注解而使用原来最正确的编码方式继续完成任务。许多注解带给我的第二个问题就是：**没有足够的文档**。除非注解的含义是绝对的透明。**为注解制定详尽的文档，尤其是框架类型注解，也不要在沟通上做过了头，得不偿失。**

最后，我需要强调让注解对与开发者或者一个开发团队来说是一个好建议的重要性。在这方面，我对IDE开发商创建所有权的注解体系持怀疑态度。所有的IDE都做了这样的拓展，但我会挑选一个我最常用的例子。IntelliJ IDEA使用注解来传达一个最小但很明智的约定实现设计（DbC）风格，强制传入参数而且返回值。我为JetBrains提供了这样便利的方法鼓掌，IDE强制了方法的契约。

IntelliJ这样使用语法，如：  

	@Contract("_, null -> null")

它意味着被标注的方法接受两个参数，而且如果第二个参数空的话返回一个`null`。我很喜欢这个注解，但是我觉得很不爽就是因为它依赖于IDE（即使其它的IDE会跳过这个注解，其它的IDE不识别。现在我在我的代码中留下了一个神奇，但是在某些IDE下没有用，这回浪费开发者的时间去理解这些无用的注解）。除此之外，我们的整个团队并不是使用统一IDE，那么一些代码将不会有这些检查，我对于DbC风格坚持的希望也就变成了折衷的方法或者依赖于IDE了。

注解在Java编程中很重要的一部分，而且角色是可以扩展的。但是一些新的注解应该比以往**更加小心谨慎，命名相对于前任们要细心些，要有良好的说明文档，要避免引入限制性的依赖**。

***Andrew Binstock, Editor in Cheif  
javamag_us@oracle.com  
@platypusguy***

原文：



> # Keeping Annotations Useful    #
> When designing annotations, be conservative and circumspect.  
> 
> In this issue, the lead feature is about a rightly well-regarded library named Project Lombok. This library enables you to avoid writing boiler-plate code by using annotations. For example, add @Data to a class and Lombok will generate the getters and setters, plus other methods you’re likely to want in a JavaBean-style data class— toString() and so on. Lombok’s approach is attractive to me—and to many developers— in part because it’s useful, compact, and clear.  In addition, the project includes a tool called “delombok,” which can remove the annotations and insert the boilerplate code into your classes. In this way, you can easily remove the dependency on Lombok. The annotations are conservative in their expression and their roles, and the project has a clean, reversible implementation. Many more annotations should follow this approach.
> 
> Annotations as they appear in Java itself follow a similar understated model. Those anno-tations, irst unveiled in Java 5, were elegant and concise and didn’t attempt to do too much: @Override and @Deprecated told you something about the code, while @SuppressWarnings told the tools that you knew what you were doing. The intent was that tools, especially IDEs, would use these markers to issue warnings and reminders. None of the annotations actually changed pro-gram behavior. This conservative approach by the Java language team continued in Java 7, when @SafeVarargs was added, and in Java 8, when @FunctionalInterface was delivered. 
> 
> In addition to the qualities I’ve already men-tioned, these annotations are unambiguous. This is a key and often overlooked aspect of annota-tion design. In the quest for brevity, authors too often push the respon-sibility of intelligibility onto developers. Look no further than the lack of coordination in basic syntax. A prime ofender here is the family of not-null qualiiers. @NotNull is such an annotation, and so is @NonNull. This annota-tion has a diferent meaning under FindBugs. Likewise, @Nullable means diferent things to the Checker Framework and FindBugs. This is not the way to go about things. If you author annotations, be clear and deinitely avoid syn-onyms and homonyms. 
> 
> In comparison to these short, pithy annotations, Java EE intro-duced the extensive use of annota-tions. Soon, a series of annotations replaced code, and the nature of enterprise programming thereby changed. Java EE acquired a sort of embedded syntax that straddled descriptions and commands. The advent of this style reinvigorated Java EE by making it far easier to code and by getting rid of the heaviness of its forebears. 
> 
> However, this advance inspired legions of frameworks to use and overuse annotations, many of which were uninspired formula-tions. They introduced complexity without good enough documenta-tion by which to navigate the code. As the annotations became com-plex markers for actions deined elsewhere, you ended up chasing your tail just to determine what the code you had right before you actu-ally did. This was less than entirely fun, which brings me to the second problem with many annotations: insuicient documentation. Unless the meaning is utterly transparent (and even then, as the preceding examples demonstrated), docu-ment the annotation thoroughly, especially in frameworks. Err on the side of overcommunication. 
> 
> Finally, I need to stress the importance of making annota-tions a sound proposition for the developer and, by extension, the developer’s team. In this regard, I am leery of IDE vendors’ cre-ation of their own proprietary annotation systems. All IDEs do this to some extent, but I’ll pick an example from the one I use most. IntelliJ IDEA uses annota-tions to deliver a minimal but clever implementation of design by contract (DbC)–style enforce-ment of passed parameters and return values. I applaud JetBrains for providing a handy way to have the IDE enforce method contracts (and identify potential coding errors that are inconsistent with the contract requirements). 
> 
> IntelliJ uses syntax like this: @Contract("_, null -> null"). It means that the tagged method accepts two parameters and returns a null if the second param-eter is null. Much as I like this annotation, I feel uncomfortable fully committing to it because it creates a dependence on the IDE. (Even though another IDE will skip the annotation it doesn’t recognize, I’ve now left an unused artifact in my code that might create wasted time for downstream developers trying to understand its function.) In addition, if my whole team is not using the same IDE, then some code won’t have these tests and my hope of consistent DbC enforce-ment is either compromised or IDE-dependent. 
> 
> Annotations are an important part of programming in Java, and their role is likely to expand. But new annotations should be devised with far greater circumspection than in the past, named with care-ful attention to predecessors, and documented well, and they should avoid the introduction of restric-tive dependencies.
