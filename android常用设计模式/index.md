# Android常用设计模式


Android中其实到处都是设计模式或者设计模式的联合运用的地方，以下设计模式是Android设计中常见的：

+ Observer模式
+ Abstract Factory模式
+ Adapter模式
+ Template Method模式
+ Composite模式
+ Strategy模式
+ State模式
+ Proxy模式
+ Bridge 模式
+ Iterator模式
+ Mediator模式
+ Facade模式

<!--more-->
Android框架的魅力源自IoC（控制反转）,关于IoC是什么可以参考[【张鸿洋的博客】](http://blog.csdn.net/lmj623565791/article/details/39269193)
Android应用开发使用Java编写，在架构上使用MVC，鼓励组件之间的弱耦合，开发出可重用、可扩展、可维护、灵活性高的代码需要遵循如下准则：

+ "开-闭"原则（OCP）:一个软件实体应该对扩展开发，对修改关闭。在设计一个模块时，应当使这个模块可以在不被修改的前提下被扩展，也就是你可以在不修改源码的情况下，通过简单的继承来改变模块的行为，或者增添一些新的特性。
+ 里氏替换原则（LSP）: 一个软件实体如果使用的是一个基类，那么一定可以使用其子类。而替换子类后，不会对调用的地方带来问题。
+ 依赖倒置原则（DIP）: 要依赖于抽象，但不要依赖于具体。
+ 接口隔离原则（ISP）: 使用多个专门的接口比使用单一的总接口要好。一个类对另外一个类的依赖性应当建立在最小接口上。
+ 合成/聚合复用原则（CARP）: 又称合成复用原则（CRP），就是在一个新的对象里面使用一些已有的对象，使之成为新对象的一部分。新的对象通过向这些对象的委派达到复用已有功能的目的。**简而言之：就是尽量使用CRP，避免使用继承。**
+ 迪米特法则（LoD）: 又称为最少知识原则(LKP），就是说一个对象应该对其他对象尽可能少的了解。**狭义的迪米特法则**指的是如果两个类不必彼此直接通信，那么这两个类就不应该发生直接的相互作用。如果其中一个类需要调用另一个类的方法的话，可以通过第三者转发这个调用。**广义的迪米特法则** 一个模块设计的好与坏的一个重要的标志就是该模块在多大程度上将自己的内部数据与实现有关的细节隐藏起来。信息的隐藏非常重要的原因在于，它可以使各个子系统之间脱耦，从而可以允许它们独立的被开发、优化、使用、阅读及修改。

关于设计模式的的详细介绍可以参考[设计模式读书笔记（开篇）](http://blog.csdn.net/mrljdx/article/details/44854347)这篇文章。

灵活的使用设计模式，可以让你根据需求编写出可重用、可扩展、可维护、灵活性高的代码。
