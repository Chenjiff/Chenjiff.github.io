---
layout: post  
title: "从Netty中学习面向接口编程"  
subtitle: "java"  
author: "ChenJ"  
header-img: "img/post-bg-imgs/3-sight.png"  
header-mask: 0.4  
catalog: true  

tags: 后路
---

* TOC
{:toc}
---

>  本文通过 Netty 中的 EventLoop 和 EventLoopGroup 来理解类和接口设计的原则和用意。

### Question

先介绍一下这两个接口吧，从使用角度上看，EventLoop表示一个事件循环器，Netty中唯一的实现类是SingleThreadEventLoop（除了EmbeddedEventLoop，它用于单元测试，实际不会使用到），这个类中自带一个线程，可以通过这个线程来执行提交给它的任务，实际就是Executor的扩展；而EventLoopGroup则是前者的集合，可以理解为一个执行池。

实际使用的简明流程为：向Bootstrap传入一个EventLoopGroup，之后Bootstrap会调用这个EventLoopGroup来进行某些操作，而EventLoopGroup的底层便是通过调用自己的next()来获得一个EventLoop并使用它来进行真正的工作。

下面是常用到的Nio操作相关类的类图：

* NioEventLoop： 

 ![](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/back/NioEventLoop.png)  

* NioEventLoopGroup：  

  ![](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/back/NioEventLoopGroup.png)  

1. 可以看到NioEventLoop中EventLoop接口还继承了EventLoopGroup接口，这也是我一开始困惑的地方，明明后者是前者的组合，按理说EventLoop中有一个parent()方法来获得所属的EventLoopGroup就应该足够了，为什么还要实现EventLoopGroup接口呢？

2. 另外，例如NioEventLoop类图中的SingleThreadEventLoop、SingleThreadEventExecutor和AbstractEventExecutor，类在继承时有时需要额外实现一些接口，而这些接口之间又是存在继承的，那么这和类的继承之间的关系是什么呢？什么时候需要额外实现接口呢？

### Exploration

*注意：下文中的面向对象指使用 Java 中的 class 来设计类，而面向接口指使用 interface*

#### 理解Java中的extends

继承是初学时最被强调的东西，动不动有个共同特征就可以就可以来个继承关系，但是现在需要明白：继承要慎用。

**extends学术上严格定义是is-a关系。**务必要遵循这条定义，下文会说。

* extends的目的

  继承是为了什么呢，结合我的理解和网上的资料，我认为继承是为了代码重用，可以体现在两个方面：

1. 子类可以直接使用父类的方法，这个就不用说了。
2. 可以通过多态使得父类直接使用子类的方法。

* extends可能的缺点

1. 破坏了类的封装性
2. 子类使用到了父类的方法，产生了耦合，容易导致软件复杂度失控
3. 无法通过继承达到多个类代码的重用（属于java语言限制）
4. 父类的方法子类无条件继承，很容易造成方法污染
5. 从父类中继承的方法，是一种静态的复用。不能在运行时发生改变，不够灵活
6. 重写时容易出错，例如重名等

详细解释见： [(1)](https://www.cnblogs.com/xz816111/p/9080173.html  ) [(2)](https://lovoedu.gitee.io/javablog/2017/06/01/20170601/  )

*  面向对象

面向对象有着不少缺点，它的唯一优点：是代码复用最简洁明了的实现。因此我们需要使用它时要满足以下前提（但是还是要“享受”它的一些缺点）：

1. 父类中所有的属性和方法，在子类中都适用。
2. 子类不需要再去重用别的类中的代码。

其实满足了is-a关系也就满足了这两个前提了，所以实现继承时一定要死抠定义。代替继承的是使用聚合（复合），也就是常说的**复合优先于继承**。复合相比继承可以减少一些问题例如：不会破坏类的封装性、可以重用多个类的代码等。但设计难度大，耦合度高且不够优雅。

#### 理解面向接口

* 理解interface

interface是一种**声明**或**协议**，它用来表明一个类实现了一些功能，让那些需要使用到这个类的地方准确知道它可以调用这个类的某个方法。

接口继承接口是单纯的协议的扩展，表示拥有更复杂的功能。

* 为什么要面向接口

引用一句我觉得非常贴切的话：

**“面向对象编程语言的问题在于，它总是附带着所有它需要的隐含环境。你想要一个香蕉，但得到的却是一个大猩猩拿着香蕉，而其还有整个丛林。”** — Joe Armstrong(Erlang语言发明人)

为什么面向对象不适合做功能扩展的设计：使用面向对象时，我们在设计具体类时总会有一些不够核心的辅助方法，这就会显得很冗余。使用面向接口时，我们只需要看一个类是否实现了相应接口便可以调用相应核心方法，而不用考虑它是如何实现的，也不用考虑它还有什么其他的功能。

我们把目光拉到所有面向对象的设计上，面向对象在设计时需要对方法进行抽象来定义职能等，而interface就是带着这样的目的被设计出来的，可以说先有面向对象设计再有interface，面向接口实际是广义的面向对象设计里的一种设计模式，即使如c++没有对interface的直接支持也是可以用面向接口来编程的，java中的interface正是在对这种模式的实践考量之后才加入的，所以用它进行面向接口编程符合interface的设计的原意。直接使用抽象类固然也强行可以达到一些效果（毕竟interface可以看成一种特殊的抽象类），但是抽象类的设计目的是代码重用，表明一个is-a关系，而相比下interface是like-a，既然java已经将这种关系提取出来了，我们使用interface 肯定更清晰易用。所以，java设计一个类只能继承一个类，而实现多个接口，只要你使用了正确的设计模式，就能感受到java对这种模式的合理支持。

总结：java设计者处心积虑创造了interface，也造就了它的清晰和易用。

所以我们不难发现，合理的设计思路应该是：**先在接口层面上去设计，好的接口设计可以在大的、模块的层次上大幅简化项目，而继承则可在小的、细碎的层次上简化实现。**

落实到Netty等优秀框架上的设计就是：

**好的接口设计 -> 抽象类实现一些接口，编写一些可重用代码 -> 具体类继承抽象类的代码进行泛化，并可选择实现额外接口满足额外功能。**

到这里就可以看清楚了，接口是核心骨架协议（我个人觉得这样好理解）；接口继承是协议的扩展，可以方便接口方法的重复使用并处理协议的包含关系；类是具体实现；而类继承是代码复用的工具。

### Solution

1. 为什么NioEventLoop中EventLoop接口还继承了EventLoopGroup接口？

   为了行为的统一，所有继承了EventLoop的类同时也继承了EventLoopGroup，因而也可以调用相应方法，只不过在前者中的方法调用的线程就是它本身，next()方法也是返回它自己；而后者会保持着一个线程池，会真正调用不同的线程去执行submit给它的相应task。

   看一下其中register方法的实现就知道了：

   ```java
   public abstract class MultithreadEventLoopGroup extends MultithreadEventExecutorGroup implements EventLoopGroup {
   	//...
       @Override
       public ChannelFuture register(ChannelPromise promise) {
           return next().register(promise);
       }
   }
   ```

   ```java
   public abstract class SingleThreadEventLoop extends SingleThreadEventExecutor implements EventLoop {
       //...
       @Override
       public ChannelFuture register(final ChannelPromise promise) {
           ObjectUtil.checkNotNull(promise, "promise");
           promise.channel().unsafe().register(this, promise);
           return promise;
       }
   }
   ```

   

   这其实是Netty设计精妙之处，使得**用户可以将一个EventLoop视为一个只包含一个线程的EventLoopGroup来使用，在需要传入EventLoopGroup的地方可以直接传入一个EventLoop**。

   面向接口设计时可以借鉴一下这一点，在有聚合（一个包含多个）关系的地方可以参照Netty的这种处理方式。

2. 类在继承时有时需要额外实现一些接口，而这些接口之间又是存在继承的，那么这和类的继承之间的关系是什么呢？什么时候需要额外实现接口呢？

   通过上文的介绍，设计时应是从接口入手，设计好接口之后，可以利用抽象类来实现某些接口的可重用方法（一般这些接口方法位于比较上层的位置），然后在实现某些接口的具体类时就可以继承这些抽象类重用其代码。所以我的答案：

   * 类继承关系一般和接口继承关系是有相似的顺序关系的，这是因为上层接口的方法一定是被更多的类所实现的，所以泛化程度较低，所对应类的层级也往往对应的比较高。
   * 实现最底层的接口是我们的最终目的，前面实现非最底层接口是为了维护代码的可重用性，所以应该看：当前实现到了什么程度了。

*写在末尾：这篇文章参考了许多资料，但是总体仍是按着我个人的思路和想法来叙述的，所以不保证完全正确，只代表着我现在的理解和角度，今后若有新的体会再来修改。*

*END*