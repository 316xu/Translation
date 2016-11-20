# 【译】SOLID Part 4：依赖倒置原则

[原文地址](https://code.tutsplus.com/tutorials/solid-part-4-the-dependency-inversion-principle--net-36872)

作者：Patkos Csaba

> 这篇博客是 [SOLID 原则](http://blog.csdn.net/u013538630/article/details/53143277) 的一部分
>
> [<< SOLID: Part 3 - 里氏代换原则 & 接口隔离原则](http://blog.csdn.net/u013538630/article/details/53151962)

[单一职责（SRP）](http://blog.csdn.net/u013538630/article/details/53118717)，[开闭原则](http://blog.csdn.net/u013538630/article/details/53151962)，里氏代换原则，接口隔离原则以及依赖倒转原则。在编程的过程中应当牢记这五种敏捷原则。

​	谈论 SOLID 原则中那个更重要是不公平的。但是可能没有哪个能比依赖倒置原则（下面简称 DIP）对你的代码造成更多的影响。如果你发现其它的原则难以运用，那就从依赖倒置原则开始，

## 定义

> A. 高层模块不应该依赖低层模块。两者都应该依赖于抽象
>
> B. 抽象不应该依赖于细节。

​	[Robert C. Martin](http://www.8thlight.com/our-team/robert-martin) 在他的书 [Agile Software Development, Principles, Patterns, and Practices](http://www.amazon.com/Software-Development-Principles-Patterns-Practices/dp/0135974445/ref=sr_1_1?s=books&ie=UTF8&qid=1378755964&sr=1-1&keywords=robert+c+martin) 中定义了这个原则。它是 SOLID 原则的最后一个。



## 现实世界中的 DIP

​	在开始写代码之前，我想告诉你一个故事。我们在 Syneto 不是一直都很关心我们的代码。几年前我们知道的不多并且觉得自己已经尽力了，我们的一些工程也做的不怎么样。我们来来回回的失败和尝试。

​	Uncle Bob（Robert C. Martin）的 SOLID 原则和整洁架构改变了我们的工作方式，将我们的代码从一种难以名状的状态拯救了回来。我将会通过讨论伴以例证的方式证明 DIP 能够带给你的进步。

​	大多数网页工程包含三个主要的技术：HTML，PHP 和 SQL。具体是哪个版本和哪种数据库和我们讨论的问题没有关系。我们关注的是网页上显示的内容来自数据库，两者通过 PHP 交互。

​	???

​	我见过很多工程在 HTML 的 PHP 标签里写 SQL 语句，或者是用 PHP 输出页面，然后在页面里读取 `$_GET` 和 `$_POST`。但是为什么这种写法是错的？

![](https://cdn.tutsplus.com/net/uploads/2014/02/html-php-sql-cross-dependencies.png)

​	上图形象的展示了刚刚讨论的情况。箭头代表显示的依赖，可以这么说，各个组件都依赖了其它所有组件。如果我们改变了数据库里的一张表，我们可能连 HTML 文件都要改。或者改变了 HTML 的一个 field，结果是要给数据库中的一个列改名字。再看第二张图表，我们需要改 PHP 文件如果 HTML 变了，在一些糟糕的情况下，当我们将 HTML 的内容都放在 PHP 文件里时，我们肯定要通过修改 PHP 文件来改变网页内容。毫无疑问的是，依赖存在于类和模块之间。

![](https://cdn.tutsplus.com/net/uploads/2014/02/html-php-sql-stored-procedures.png)

​	在上面这张图中，数据库查询返回了由数据库数据生成的 PHP 代码。这段代码执行了其他的数据库查询，也返回了一段 PHP 代码，这个循环不断继续知道最后获得了一些数据，可能是 UI。

​	我知道这种方式对于大部分人来说听起来有点离谱，但是在你将来的职业生涯中一定会遇到用这种方式构建的工程。大部分现有的工程，不管它用的是哪种语言，都是一些不关心或者不知道怎么做更好的程序员按照旧的原则来做的。如果你读完了这些教程，你将会站在一个更高的层次。你做好了准备面对你的工作，写出更好的代码。

​	另一种选择时重复前辈犯的错误，一直就这么下去。在 Syneto 工作的时候，当我们的一个工程由于它老旧的交叉依赖架构到了一个没法控制的地步时，我们不得不永远的抛弃它。那时我们决定永远都不要在走回这条路。从那时开始，我们努力实现遵循了 SOLID 原则，特别是实现了依赖倒置的整洁架构。

![HighLevelDesign](https://cdn.tutsplus.com/net/uploads/2014/02/HighLevelDesign.png)

这个架构让人感到惊叹的是它依赖的指向：

* 用户接口（大多数情况下是 MVC 框架）或者其它的传递机制，这些都依赖于业务逻辑。业务逻辑是很抽象的一个东西。用户接口很具体。UI 只是工程的一个细节，并且它很易变。没有什么应该依赖 UI 或者依赖你的 MVC 框架。
* 另一个有趣的发现是你的持久化数据和数据库依赖于业务逻辑。你的业务逻辑是不应该知道数据库的。这允许你按照你的意愿修改持久层。如果明天你想将 MySql 换成 PostgreSQL 或者纯文本，那你就能够这样的。当然了，你得实现一个特定的持久层，但是你业务逻辑一行代码都不用改。这里有一个关于持久数据的更详细的教程 [Evolving Toward a Persistence Layer](http://code.tutsplus.com/tutorials/evolving-toward-a-persistence-layer--net-27138)
* 最后，在业务逻辑的右边，我们有所有关于创建业务逻辑的类。这些是在程序入口创建的类和类的工厂。很多人认为这些应该属于业务逻辑，但是当它们创建了业务对象的时候，它们唯一的职责就是做这个。它们的就是帮助我们创建别的类。它们创建的业务对象和工厂本身是相互独立的。我们也可以使用别的模式，比如简单工厂等。这都不重要，只要业务对象生成后能工作就好了。