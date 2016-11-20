# 【译】SOLID: Part 1 - 单一职责

[原文地址](https://code.tutsplus.com/tutorials/solid-part-1-the-single-responsibility-principle--net-36074)

作者：Patkos Csaba

>这篇文章是[SOLID 原则](https://code.tutsplus.com/series/the-solid-principles--cms-634)的第一部分
>
>[SOLID: Part 2 - 开闭原则](https://code.tutsplus.com/tutorials/solid-part-2-the-openclosed-principle--net-36600)

单一职责（SRP），开闭原则，里氏代换原则，接口隔离原则以及依赖倒转原则。在编程的过程中应当牢记这五种敏捷原则。

## 定义

>_A Class should have only one reason to change_
>
>_一个类只能有一个改变的原因_

​	出自 [Robert C. Martin](http://www.8thlight.com/our-team/robert-martin) 的著作 [Agile Software Development, Principles, Patterns, and Practices](http://www.amazon.com/Software-Development-Principles-Patterns-Practices/dp/0135974445/ref=sr_1_1?s=books&ie=UTF8&qid=1378755964&sr=1-1&keywords=robert+c+martin) 并在之后发行了 C# 版 [Agile Principles, Patterns, and Practices in C#](https://go.redirectingat.com/?id=1342X589339&site=code.tutsplus.com&xs=1&isjs=1&url=http%3A%2F%2Fwww.amazon.com%2FAgile-Principles-Patterns-Practices-C%2Fdp%2F0131857258&xguid=328b28ed6b61347c5ca1579a4878ebe5&xuuid=9c71985238b7789e6ef3524bb77aad52&xsessid=a758b668582eab6368cf044ae9b92ca3&xcreo=0&xed=0&sref=https%3A%2F%2Fcode.tutsplus.com%2Ftutorials%2Fsolid-part-1-the-single-responsibility-principle--net-36074&xtz=-480)，它是五种 SOLID 敏捷开发原则之一。它的意思是一个类应该只有一个发生变化的原因，意思很简单，但实现起来却比较棘手。

​	但是为什么它会如此重要？

​	在静态编译语言中，有各种各样的原因会导致非期望的重新部署。如果一个类会因为两种原因改变，那么就可能会有两个不同的团队因为不同的原因去修改同一份代码。他们都会按自己的需求修改代码，在这种情况下对于一个编译语言（比如 C++，C# 或 Java），可能会导致模块不适用与其他团队或者这个应用的其他部分的问题。

## 使用者

​	学会如何决定一个类的职责是什么不是一件容易的事。举个例子，开发者为了找出改变的原因回去分析用户行为。使用了我们提供的模块应用或系统的用户可能是一类想改变它的人。下面类出了一些模块和可能的使用者。

* **持久化数据模块** - 用户包括 DBA 和软件架构师。
* **通知模块** - 用户包括办事员，会计和业务人员
* **工资系统的统计模块** - 用户可能包括律师，管理人员和会计
* **图书馆管理系统的图书检索模块** - 用户可能包括图书管理员和顾客

## 任务和角色

​	将所有的实习和角色联系起来可能会很困难。在一家小公司里一个人可能需要承担不同的任务，然而在大公司里可能几个人承担一个任务。所以更合理的办法是考虑任务。但是任务这个东西比较难定义。什么是任务？我们怎么把它找出来？一种简单的办法是想一下负责这些任务的角色并将我们的用户和它联系起来。

​	所以如果我们的使用者有理由改变它，那么角色应该能决定用户对象。这对于将实际的人和概念联系起来有很大的帮助。

> _So a responsibility is a family of functions that serves one particular actor. (Robert C. Martin)_
>
> _一个职责是服务于同一个角色的方法的集合（Robert C. Martin)_

## 改变的根源

​	从这个原因上说，角色变成了方法集合发生改变的根源。由于角色需求的改变，方法集合也随之改变。

> _An actor for a responsibility is the single source of change for that responsibility. (Robert C. Martin)_
>
> _一个职责对应的角色是这个职责发生改变的唯一原因。（Robert C. Martin)_



## 典型例子

### 可以“打印”自己的对象

​	假设我们有封装了书的内容和函数的类`Book`

``` java
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function printCurrentPage() {
        echo "current page content";
    }
}
```

​	这看起来是一个合理的类。它能够提供自己的标题，作者，并且可以翻页，也可以打印当前页面的内容。但是有一个小小的问题，让我们考虑一下操作`Book`的角色，他们会是谁？很容易就能想到两个：图书管理员（比如图书馆员）和数据显示机器（比如我们想把书的内容传到屏幕上）。这是两个完全不一样的角色。

​	混合的业务逻辑是不好的，因为它违反了 SRP。看一下下面的代码

```java
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
}
 
interface Printer {
 
    function printPage($page);
}
 
class PlainTextPrinter implements Printer {
 
    function printPage($page) {
        echo $page;
    }
 
}
 
class HtmlPrinter implements Printer {
 
    function printPage($page) {
        echo '<div style="single-page">' . $page . '</div>';
    }
 
}
```

​	即使这个非常基础的例子也展示了业务逻辑的分离与单一职责，对于设计的灵活性有了很大的好处。

### 可以“保存”自己的对象

一个和上面类似的例子，一个对象保存和恢复自己

```java
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
    function save() {
        $filename = '/documents/'. $this->getTitle(). ' - ' . $this->getAuthor();
        file_put_contents($filename, serialize($this));
    }
 
}
```

我们可以再次定义各种角色比如图书管理系统和持久数据管理。无论什么时候我们想改变持久数据， 我们需要改变这个类。当我们想从一页翻到另一页时，也需要改变它。这里有几种变化。

```java
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
}
 
class SimpleFilePersistence {
 
    function save(Book $book) {
        $filename = '/documents/' . $book->getTitle() . ' - ' . $book->getAuthor();
        file_put_contents($filename, serialize($book));
    }
 
} 
```

新建一个类来保存数据相关的操作，这使得职责分开了，我们也能够更灵活的扩展它而不需要改变原有的`Book`类

### 高层视图

​	在我之前的文章中我频繁的提到和展示下图所示的高层架构

​	![high level architectural](https://cdn.tutsplus.com/net/uploads/2013/12/HighLevelDesign.png)

​	如果我们分析这张图，可以看到单一职责原则的重要性。应用的入口和对象的创建在右边，一个角色对应一个职责。持久化数据被放在下面。一个分离的模块对应一个分离的职责。最后，在左边的是界面和分发机制。剩下要做的就是添加业务逻辑了。

## 程序设计注意事项

​	当准备写一个程序时我们需要考虑很多因素。举个例子，涉及到多种需求的类将会导致一系列的变化。这些变化会是单一职责的一个来源。很可能发生的一种情况是影响一组方法的一组需求有可能会改变。

​	为了迎合用户，我们会想满足尽可能多的需求，从这点上来说，软件的核心价值是易于改变，其次才是功能。然而，为了更好的实现第二点，第一点是必须实现的。为了很好的实现第二点，我们必须要有一个易于改变，易于继承，能容纳新方法的并实现了 SRP 的设计。

​	让我们一步步证明上面的论点	

1. 易于改变的程序才能功能丰富
2. 第二点（功能丰富）代表用户的需求
3. 用户的需求代表了角色的需求
4. 角色的需求代表了这些角色对于改变的需求
5. 改变的需求决定了我们的职责

所以当我们设计软件时我们应该：

1. 找出并定义角色
2. 确定角色的职责
3. 组织方法和类让它们都只有一个确定的职责

## 一个不是很明显的例子

```java
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
    function getLocation() {
        // returns the position in the library
        // ie. shelf number & room number
    }
 
}
```



​	现在它看起来相当合理了。没有方法和持久数据交互。有`turnPage()`方法和别的方法提供了关于这本书的不同信息。然而，可能会有一个问题。通过分析程序。方法`getLocation()`可能存在问题。

​	`Book`的所有方法都和业务逻辑相关。所以我们要从业务角度来看问题。所有如果这个程序是用来帮助图书管理员用来找书然后提供给借书者的话，SPR 原则可能被违反了。

​	用户会对`getTitl`，`getAuthor()`，`getLocation()`中的一个或多个感兴趣。用户可能也有权限找一本书阅读它的前几页去决定要不要这本书。所以阅读者可能会对除了`getLocation()`外的所有方法产生兴趣。因为阅读者是不会对这本书放在图书馆的哪个位置感兴趣的。所以我们确实违反了SPR原则。

```java
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
}
 
class BookLocator {
 
    function locate(Book $book) {
        // returns the position in the library
        // ie. shelf number & room number
        $libraryMap->findBookBy($book->getTitle(), $book->getAuthor());
    }
 
}
```

​	引入一个`BookLocator`，这样图书管理员就可能使用它来找书了。而阅读者就只使用`Book`就好了。当然，这里有很多方式去实现`BookLocator`。它可以通过作者和标题或者一个`Book`对象来获取需要的信息。它总是依赖于业务。重要的是当图书馆改变的时候，`Book`对象不需要改变。同样的，如果我们决定提供一个简介给读者，这不会影响到图书管理员或者找书的过程。

​	然而，如果我们的业务目标是取代图书管理员并且创建自助机器，那么我们需要考虑我们第一个例子中的SPR。读者同时也是管理员，他们需要找到书并从自助系统中借阅。这只是一种可能性。重要的是你一定要经常考虑你的业务。

## 最后的思考

​	在编码的过程中一定要频繁的考虑单一职责原则。它对于模块的设计会有很大的影响，对于降低模块间的依赖和实习低耦合很大的帮助。但是凡事总有两面，我们从程序一开始就想用SPR来设计，希望尽可能的想到更多的角色，但从设计的角度来看从一开始就尝试考虑所有的情况是有风险的。过度的考虑 SPR 可能导致的是不成熟的优化而不是一个更好的设计，它会使你的程序过于分散而导致一个单一职责的模块难以理解。

​	所以，无论何时你当你发现一个类或模块因为各种原因需要改变时，不要犹豫不决，通过必要的步骤去实现SRP，然而要注意不成熟优化。