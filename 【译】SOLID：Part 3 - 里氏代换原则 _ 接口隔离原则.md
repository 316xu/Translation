# 【译】SOLID：Part 3 - 里氏代换原则 & 接口隔离原则



[原文地址](https://code.tutsplus.com/tutorials/solid-part-3-liskov-substitution-interface-segregation-principles--net-36710)

作者：Patkos Csaba

> 这篇博客是 [SOLID 原则](http://blog.csdn.net/u013538630/article/details/53143277) 的一部分
>
> [<< SOLID: Part 2 - 开闭原则](http://blog.csdn.net/u013538630/article/details/53151962)
>
> \>\> SOLID: Part 4 - 依赖倒转原则

[单一职责（SRP）](http://blog.csdn.net/u013538630/article/details/53118717)，[开闭原则](http://blog.csdn.net/u013538630/article/details/53151962)，里氏代换原则，接口隔离原则以及依赖倒转原则。在编程的过程中应当牢记这五种敏捷原则。

​	因为里氏代换原则（LSP）和接口隔离原则（ISP）都很简单并容易例证，所以在这篇文章里会一起说。



## 里氏代换原则（LSP）

> 子类不能破坏父类的类型定义

​	这个原则是 Barbara Liskov 在 1987 年在一个会议上提出的，并在 1994 年和 Jannette Wing 共同推出书面说明。原始的定式见下：

> Let q(x) be a property provable about objects x of type T. Then q(y) should be provable for objects y of type S where S is a subtype of T.

​	后来随着 [Robert C. Martin](http://www.8thlight.com/our-team/robert-martin) 的书 [Agile Software Development, Principles, Patterns, and Practices](http://www.amazon.com/Software-Development-Principles-Patterns-Practices/dp/0135974445/ref=sr_1_1?s=books&ie=UTF8&qid=1378755964&sr=1-1&keywords=robert+c+martin) 出版，这条定义一里氏代换的名字被大众所知。

​	让我们看一下 Robert C. Martin 给出的定义：

> SubTypes must be substitutable for their base types.

​	尽可能简单的来说，从使用者的角度来说，子类不应该破化父类的功能。这里用一个简单的例子来论证这一观点。

```php
class Vehicle {
 
    function startEngine() {
        // Default engine start functionality
    }
 
    function accelerate() {
        // Default acceleration functionality
    }
}
```

​	提供抽象类 `Vehicle` 和它的两种实现：

```php
class Car extends Vehicle {
 
    function startEngine() {
        $this->engageIgnition();
        parent::startEngine();
    }
 
    private function engageIgnition() {
        // Ignition procedure
    }
 
}
 
class ElectricBus extends Vehicle {
 
    function accelerate() {
        $this->increaseVoltage();
        $this->connectIndividualEngines();
    }
 
    private function increaseVoltage() {
        // Electric logic
    }
 
    private function connectIndividualEngines() {
        // Connection logic
    }
 
}
```

​	如果一个应用类可以使用 `Vehicle` 那么久可以使用它们俩。

```php
class Driver {
    function go(Vehicle $v) {
        $v->startEngine();
        $v->accelerate();
    }
}
```

​	这是一个简单的模板模式的应用，就像我们在 OCP 那里用的那样

![](https://cdn.tutsplus.com/net/uploads/2014/01/template_method1.png)

​	根据我们在 OCP 中的经验，我们可以发现里氏代换和 OCP 有着很强烈的联系。事实上，“违反了 LSP 就是潜在的违反 OCP”（Robert C. Martin），并且模板模式就是一个经典的演示 LSP 的例子，同时也是 OCP 的一种实现方式。



## 一个经典的违反了 LSP 的例子

​	为了彻底的阐明 LSP，我们将会看一个经典的例子，它很有意义而且容易理解。

```php
class Rectangle {
 
    private $topLeft;
    private $width;
    private $height;
 
    public function setHeight($height) {
        $this->height = $height;
    }
 
    public function getHeight() {
        return $this->height;
    }
 
    public function setWidth($width) {
        $this->width = $width;
    }
 
    public function getWidth() {
        return $this->width;
    }
 
}
```

​	让我们从一个基本的几何长方形 `Rectangle` 开始。它就是一个有 `width` 和 `height` 的简单类。假设我们的应用已经在工作了，这个类已经被好几个用户使用了。现在，有新需求加入，用户需要正方形了。

​	在实际的几何学中，正方形是长方形的一种特例。所以我们会尝试通过用继承 `Rectangle` 的方式来实现 `Square`。我们经常说子类就是父类，这种表达第一眼看上去是符合 LSP 的。

​	![](https://cdn.tutsplus.com/net/uploads/2014/01/SquareRect.png)

​	但是在实际编码中 `Spuare` 真的是 `Rectangle` 吗？

```php
class Square extends Rectangle {
 
    public function setHeight($value) {
        $this->width = $value;
        $this->height = $value;
    }
 
    public function setWidth($value) {
        $this->width = $value;
        $this->height = $value;
    }
}
```

​	正方形是长宽相等的长方形，我们也可以像上面那样通过继承实现它，虽然看起来比较奇怪。我们通过覆写 setter 和 getter 让长宽相等。但是这么做会对应用代码产生什么影响呢？

```php
class Client {
 
    function areaVerifier(Rectangle $r) {
        $r->setWidth(5);
        $r->setHeight(4);
 
        if($r->area() != 20) {
            throw new Exception('Bad area!');
        }
 
        return true;
    }
 

```

​	可以假想一个应用类验证长方形的面积，如果不对就抛出异常。

```php
function area() {
    return $this->width * $this->height;
}
```

​	我们把上面的方法加到 `Rectangle` 中以提供面积。

```php
class LspTest extends PHPUnit_Framework_TestCase {
 
    function testRectangleArea() {
        $r = new Rectangle();
        $c = new Client();
        $this->assertTrue($c->areaVerifier($r));
    }
 
}
```

​	上面的例子是能够通过测试的。如果 `Square` 的定义是正确的，那么它也应该能通过这个测试。毕竟从数学上来说正方形总是长方形，但是在我们的程序里呢？

```php
function testSquareArea() {
    $r = new Square();
    $c = new Client();
    $this->assertTrue($c->areaVerifier($r));
}
```

​	测试很简单并且它崩溃了。当我们运行它的时候一个异常被抛出了。

```php
PHPUnit 3.7.28 by Sebastian Bergmann.

Exception : Bad area!
#0 /paht/: /.../.../LspTest.php(18): Client->areaVerifier(Object(Square))
#1 [internal function]: LspTest->testSquareArea()
```

​	所以，我们的类 `Spuare` 实际上并不是 `Rectangle`。它打破了几何学的定律。它违反了里氏代换原则。

​	我特别喜欢这个例子因为它除了违反了 LSP 也说明面向对象编程并不仅仅是将现实生活映射到代码里。每一个对象都必须是一个概念的抽象。如果入门尝试将真实的对象一一对应到程序里去，那么就会总是出错。

## 接口隔离原则

​	单一职责是关于角色和高层架构的。开闭原则负责的是类的设计和功能拓展。里氏代换原则是关于子类型化和继承的。而接口隔离原则（ISP）是关于和应用代码交互的业务逻辑。

​	所有的模块化程序都会提供一些应用代码可以使用的接口。可能是接口类或者是实现了外观模式的对象。具体用的是那种模式并不重要。它们在与应用代码交互的过程中有一个共性。接口可以连接同一个工程的不同模块，或者连接一个第三方的库。这同样也没什么区别。通信是通信，应用是应用，不管是谁在写这个代码。

​	所以我们该怎么样定义这些接口呢。我们可以考虑我们的模块并且列出所有我们想要提供的功能。

![](https://cdn.tutsplus.com/net/uploads/2014/01/hugeInterface.png)



​	这看起来是一个不错的开始，在模块里定义了我们想实现的方法。不过一个这样的开始可能会导致两种结果。

* 出现一个实现了 `Vechile` 中所有方法的的复杂的 `Car` 或者 `Bus` 类。在复制这个文件的时候你就会觉得它太大了。

* 或者很多小一点的类比如 `LightsControl`，`SpeedControl` 或者 `RadioCD`，他们都继承了整个接口但是实际上只提供了一部分功能。

  很明显两种都没法被接受。

  ![](https://cdn.tutsplus.com/net/uploads/2014/01/specializedImplementationInterface.png)

  ​	让我们采取另一种方法。将这个接口分割成更小的部分，将每个接口特殊化。这会让每个类只关心自己的接口。实现了接口的类将会被用作车辆的不同组件，像是上图那样的车。车辆将会使用依赖于接口使用实现的类。所以下图会更有价值。

  ![](https://cdn.tutsplus.com/net/uploads/2014/01/carUsingInterface.png)

  ​	但是这么做从根源上改变了我们对架构的认知。`Car` 变成了用户而不是实现。我们仍然想让我们的用户有办法使用我们的整个模块，也就是一辆车。

  ![](https://cdn.tutsplus.com/net/uploads/2014/01/oneInterfaceManyClients.png)

  ​	假设我们解决了实现的难题并且有了一个稳定的逻辑。最容易做的到就是提供一个包含了所有实现的接口给用户（我们例子中的 `BusStation`, `HighWay`, `Driver` 等等），用户向用什么就用什么。这样基本上是将行为选择的职责甩给了用户。你可以在很多久工程里看到这样的解决方案。

  > The interface-segregation principle (ISP) states that no client should be forced to depend on methods it does not use.
  >
  > 接口给力原则代表着用户不应该被要求依赖于它们不需要的方法。

  ​	然而，这种实现会有它的问题。现在所有的用户都要依赖所有的方法。为什么一个 `BusStation` 要知道公交车灯的状态呢 ，或者它跟司机在听什么广播有关系吗？当然没关系，但它就这么做了。如果我们考虑单一职责，它和现在我们讨论的比较接近。如果 `BusStation` 依赖于很多独立的实现，甚至用不到的一些，那么当其中一个变化的时候它也就要跟着变了。对于编译语言来说这是对的，但是我们仍旧能够看到 `LightControl` 对于 `BusStation` 的影响。

  ​	接口是属于使用它们的用户而不是它们的实现。所以，我们在设计接口的时候要尽量满足用户的需求。我们并不是总能够知道我们的用户是谁。但是我们能够将我们的接口划分为更小的部分，这样它们能够更好的满足用户的精确需求。

  ![](https://cdn.tutsplus.com/net/uploads/2014/01/segregatedInterfaces.png)

  ​	当然，这会某种程度上回导致重复。但是记住，接口只是计划中的方法的名字。它没有实现任何的逻辑。所以这种重复是微小而可控的。

  ​	所以，让用户依赖且仅依赖于他们需要的接口是有很大好处的。用户可能需要好几个接口，这没问题，只要它使用了接口提供的方法。

  ​	另一个诀窍是考虑我们的业务逻辑，一个类在需要的时候可以实现好几个接口。所以我们可以在接口间为相同的方法提供一个单一的实现。这些隔离的接口将会强迫我们从用户的角度来思考我们的代码，这反过来会推动程序变得弱耦合于容易测试。所以，我们不仅让代码对用户变得更友好，也让代码变得更容易理解，测试和执行。

## 最后的思考

​	LSP 教会我们为什么不能将现实一对一的带入到程序对象中和子类该如果遵从父类。我们也把它带入到我们已经知道的别的原则中。

​	ISP 将会我们要更多的考虑使用者。尊重它们的需求将会让代码更好以及更好的作为一个程序员活着。