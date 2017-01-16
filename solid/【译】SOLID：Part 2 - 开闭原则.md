# 【译】SOLID：Part 2 - 开闭原则

[原文地址](https://code.tutsplus.com/tutorials/solid-part-2-the-openclosed-principle--net-36600)

作者：Patkos Csaba

> 这篇博客是 [SOLID 原则](http://blog.csdn.net/u013538630/article/details/53143277) 的一部分
>
> [SOLID: Part 1 - 单一职责原则](http://blog.csdn.net/u013538630/article/details/53118717)
>
> SOLID: Part 2 - 开闭原则



[单一职责（SRP）](http://blog.csdn.net/u013538630/article/details/53118717)，开闭原则，里氏代换原则，接口隔离原则以及依赖倒转原则。在编程的过程中应当牢记这五种敏捷原则。



## 定义

> 软件的实体（类，模块，方法等）应该对于扩展开放，对于修改关闭

​	开闭原则（OCP）是由法国程序员 [Bertrand Mayer](https://en.wikipedia.org/wiki/Bertrand_Meyer) 与1998年发布在他的书 [Object-Oriented Software Construction](https://go.redirectingat.com/?id=1342X589339&site=code.tutsplus.com&xs=1&isjs=1&url=http%3A%2F%2Fwww.amazon.com%2FObject-Oriented-Software-Construction-CD-ROM-Edition%2Fdp%2F0136291554&xguid=bab51d2fa75e643490c471c19a030b6e&xuuid=b717c4be4aaf0f9cdab83a7fe41591d3&xsessid=eea9b261a1b2b3dd0c184d7446459673&xcreo=0&xed=0&sref=https%3A%2F%2Fcode.tutsplus.com%2Ftutorials%2Fsolid-part-2-the-openclosed-principle--net-36600&pref=http%3A%2F%2Fblog.csdn.net%2Fu013538630%2Farticle%2Fdetails%2F53118717&xtz=-480) 中。

​	在21世纪初，这条原则被 [Robert C. Martin](http://www.8thlight.com/our-team/robert-martin) 收录在他的书 [Agile Software Development, Principles, Patterns, and Practices](http://www.amazon.com/Software-Development-Principles-Patterns-Practices/dp/0135974445/ref=sr_1_1?s=books&ie=UTF8&qid=1378755964&sr=1-1&keywords=robert+c+martin) 中，使得这条原则被更多的人知道。

​	我们在这里会简单的讨论以下问题，当我们需要一个新的功能，如何设计我们的模块，类和方法以达到不改变现有代码，仅仅编写新的代码的目的。这听起来可能会有点奇怪，特别是当我们使用的是　Java, C, C++ 或者 C 的时候，因为它们不仅需要代码，还需要二进制文件。当我们添加新需求的时候我们想做到不需要重新部署二进制文件，可执行文件和 DLL 文件。



## SOLID 中的 OCP

​	当我们渐渐深入这个系列的教程，我们可以把新的原则带入到已经讨论过的原则中考虑。我们已经讨论过[单一职责（SRP）](http://blog.csdn.net/u013538630/article/details/53118717)了，在上篇文章里我们提到了一个模块只能有一个改变的原因。让我们将 OCP 和 SRP 放在一起考虑，可以发现它们是互为补充的。严格按照 SRP 写出来的代码会很接近 OCP 或者经过一些细节上的改变就可以实现 OCP。假设我们有一份符 SRP 的代码，现在要引入一个新的功能，这将会让这份代码有了第二个改变的原因，这样 OCP 和 SRP 都被违背了。同样的，一份只有主函数改变才改变并且不随着添加功能改变原有结构的代码，那么它当然是符合 OCP 的，同时也是很接近 SRP 的。

​	当让，这并不意味着符合 SRP 的一定符合 OCP，反之也不成立，但是在大多数情况下，实现了其中一个的再实现另一个都不难。



## 一个明显不符合 OCP 的例子

​	从纯粹的研究角度上来看，开闭原则很简单。一个关系连接两个类，像下面这张图，就违反了 OCP。

![](https://cdn.tutsplus.com/net/uploads/2014/01/violate1.png)

​	`User` 直接使用了 `Logic`。如果我们需要通过继承来实现另一个 `Logic` ，这样我们就有两个 `Logic` 可以用了，这个时候原来的 `Logic` 需要改变。`User` 和 `Logic`直接联系，所以我们没有办法在不改变现有逻辑的情况下提供一个新的 `Logic`。当我们使用的是静态语言时，甚至连 `User` 也需要改变了。如果我们说的是动态语言，所有的东西都要重新编译了，这是我们极力避免的情况。



## 把代码呈上来

​	如果只看上面的代码，可能会得出一个结论，一个类只要引用了另一个类就违反了 OCP，严格来说，这个结论其实是对的。但你很可能不会去遵守 OCP，就是当你觉得与其遵守 OCP 还不如改代码的时候，或者修改架构不如改已有代码方便的时候。

​	假设我们想要写一个类，它能够提供正在下载文件的百分比。我们需要两个核心类，`Progress` 和 `File`，且容我假设我们会像下面这样使用它们。

```php
function testItCanGetTheProgressOfAFileAsAPercent() {
    $file = new File();
    $file->length = 200;
    $file->sent = 100;
 
    $progress = new Progress($file);
 
    $this->assertEquals(50, $progress->getAsPercent());
}
```

​	在这个测试函数中我们是 `Progress` 的使用者，我们想获得文件下载的百分比。`File` 作为 `Progress` 的信息源来使用。一个文件有 length 属性和代表已下载的 sent 属性。我们不关心这些数值是怎么发生改变的，就当在别的地方发生了一些神奇的操作吧。所以在测试函数里直接设置它们的值就好了。

```php
class File {
    public $length;
    public $sent;
}
```

​	`File` 是一个只有两个属性的简单类。 当然在实际情况中会有更多的属性和方法，比如文件名，路径，相对路径，父目录，类型和权限等等。

```php
class Progress {
 
    private $file;
 
    function __construct(File $file) {
        $this->file = $file;
    }
 
    function getAsPercent() {
        return $this->file->sent * 100 / $this->file->length;
    }
 
```

​	`Progress` 通过一个 `File` 对象来构造。在 `Progress` 里有一个有用的方法 `getAsPercent()`，它将 `File` 中的 sent 和 length 转化成百分比返回。

```
Testing started at 5:39 PM ...
PHPUnit 3.7.28 by Sebastian Bergmann.
.
Time: 15 ms, Memory: 2.50Mb
OK (1 test, 1 assertion)
```



​	这些代码看起来没问题，但是它也确实违反了开闭原则。但是为什么？并且怎么解决？



## 改变需求

​	所有软件在发展的过程中都需要新功能。我们希望让这个程序还能够播放音乐而不仅只能下载。`File` 的长度是用字节表示的，但音乐的长度应该用秒来表示。我们希望能够给听众一个播放进度条，但是我们怎么去重用之前的类呢？

​	暂时还没法做到这一点。我们的进度是和 `File` 绑定起来的。它只能识别文件，虽然它也确实能识别音乐文件。为了做到这一点我们必须去改变原有代码，我们要让 `Progress` 能够同时识别 `Music` 和 `File`。如果我们的设计遵守 OCP，我们就不需要改变已有的 `Progress` 和 `File`，只要重用他们就好了。



## 方案一：利用 PHP 的动态特性

​	动态语言能够在运行时判断对象的类别。这让我们可以不指定构造函数参数的参数类型。

```php
class Progress {
 
    private $file;
 
    function __construct($file) {
        $this->file = $file;
    }
 
    function getAsPercent() {
        return $this->file->sent * 100 / $this->file->length;
    }
 
}
```

​	这样我们就可以向 `Progress` 传任何类型了。

```php
class Music {
 
    public $length;
    public $sent;
 
    public $artist;
    public $album;
    public $releaseDate;
 
    function getAlbumCoverFile() {
        return 'Images/Covers/' . $this->artist . '/' . $this->album . '.png';
    }
}
```

​	上面这样一个 `Music` 就可以工作了。让我们像测试 `File` 那样来测试它。

```php
function testItCanGetTheProgressOfAMusicStreamAsAPercent() {
    $music = new Music();
    $music->length = 200;
    $music->sent = 100;
 
    $progress = new Progress($music);
 
    $this->assertEquals(50, $progress->getAsPercent());
}
```

​	基本上来说，所有可测量的内容都可以和 `Progress` 协同工作。所以改一下变量的名字可能更符合现在的情况：

```php
class Progress {
 
    private $measurableContent;
 
    function __construct($measurableContent) {
        $this->measurableContent = $measurableContent;
    }
 
    function getAsPercent() {
        return $this->measurableContent->sent * 100 / $this->measurableContent->length;
    }
 
}
```

​	但是现在还有一个严重的问题。当我们将 `File` 指定为参数类型时，我们明确的知道我们的类能够处理什么，传错了参数就会有错误提示。

```php
Argument 1 passed to Progress::__construct()
must be an instance of File,
instance of Music given.
```

​	但是在没有参数类型的时候，我们的程序就要求传入的对象必须有 `length` 和 `sent` 两个属性。否则程序就会出 Refused bequest 的问题。

> Refused bequest: a class that overrides a method of a base class in such a way that the contract of the base class is not honored by the derived class. ~Source Wikipedia.
>
> 拒收的遗赠：子类只需要父类部分方法和数据

​	在 [Detecting Code Smells](https://tutsplus.com/course/detecting-code-smells/) 更详细的描述了这种烂代码。简单来说，我们不希望访问不遵守规定的的方法或对象。当我们有参数类型时，`File` 就是限制。但现在我们什么限制都没有了，就算是个字符串都能被当作参数穿进去，结果就是发生一些很挫的错误。

```php
function testItFailsWithAParameterThatDoesNotRespectTheImplicitContract() {
    $progress = new Progress('some string');
    $this->assertEquals(50, $progress->getAsPercent());
}
```

​	一个这样的测试将会出现 Refused bequest 的问题

```
Trying to get property of non-object.
```

​	虽然两个例子的结果都是程序崩溃了，但之前的例子输出了详细的错误提示，而现在的例子输出十分晦涩难懂。我们没有办法通过错误提示知道是那个变量（例子中的字符串）出了问题以及是在访问那个属性的时候出了问题。这样就很难调试和解决问题。 程序员必须阅读 `Progress` 并理解它才行。在这个例子里，我们没有通过参数类型来进行约束，而是通过 `Progress` 的行为来约束，这个约束只有在 `Progress` 里才是明确的，在别的地方都没有任何限制了。在这个例子中，约束是由 `getAsPercent()` 对于 `sent` 和 `length` 的访问决定的。但是在实际使用中，这种隐晦的约束可能会很复杂，不是看几秒钟代码就能理解的。

​	这个解决办法是不被推荐的，除非接下来要说的几个都不适用，或者实现它们会对架构造成重大影响。



## 方案二：使用策略模式

​	这里会提供一种常见并且适合用于实现 OCP 的方案，简单而有效

![](https://cdn.tutsplus.com/net/uploads/2014/01/strategy.png)

​	这种策略模式引入了接口的使用。接口是面向对象编程（OOP）中的一种特殊的实体，它在应用类和服务类之间提供了一些约束。两种类都满足约束并且保证有需要的属性和方法。这样就可以有多钟服务类，它们毫不相关，但是都遵守同一套约束以被同一个应用类使用。

```php
interface Measurable {
    function getLength();
    function getSent();
}
```

​	在接口里我们可以只定义行为。这就是为什么不直接使用公有属性而使用 getter 和 setter 的原因。将其它的类弄成这样也不麻烦，IDE 可以为我们做这些工作。

```php
function testItCanGetTheProgressOfAFileAsAPercent() {
    $file = new File();
    $file->setLength(200);
    $file->setSent(100);
 
    $progress = new Progress($file);
 
    $this->assertEquals(50, $progress->getAsPercent());
}
```

​	按照惯例，从测试程序开始。将会使用 setter 去设置值。严格考虑，`Measurable` 接口也需要定义 setter。但是要注意该在里面写什么。这个接口定义的是应用类（比如 `Progress` ）和服务类（比如 `File` ）之间的约束。`Progress` 需要去设置 `File` 的下载量吗？应该是不需要的。所以 setter 不应该被定义在这个接口里。同样的，如果你在这里定义 setter，就要强制所有的服务类实现 setter。对于它们中的一部分可能是必要的，但是别的可能不需要。假设我们现在要让 `Progress` 去显示炉子的温度。类 `OvenTemperature` 在构造函数中初始化温度，或者从另一个类中获取值。为这个类添加 setter 会很奇怪。

```php
class File implements Measurable {
 
    private $length;
    private $sent;
 
    public $filename;
    public $owner;
 
    function setLength($length) {
        $this->length = $length;
    }
 
    function getLength() {
        return $this->length;
    }
 
    function setSent($sent) {
        $this->sent = $sent;
    }
 
    function getSent() {
        return $this->sent;
    }
 
    function getRelativePath() {
        return dirname($this->filename);
    }
 
    function getFullPath() {
        return realpath($this->getRelativePath());
    }
 
}
```

​	稍稍的修改了一下 `File` 类以满足上面的要求。它继承了 `Measure` 接口并有必要的 setter 和 getter。 `Music` 也是类似的。

```php
class Progress {
 
    private $measurableContent;
 
    function __construct(Measurable $measurableContent) {
        $this->measurableContent = $measurableContent;
    }
 
    function getAsPercent() {
        return $this->measurableContent->getSent() * 100 / $this->measurableContent->getLength();
    }
 
}
```

​	`Progress` 也需要一点小小的升级。我们现在可以在构造函数里为它加上参数类型 `Measurable` 了。现在我们终于有了一个明确的约束了，可以保证 `Progress` 访问的方法始终是存在的了，因为他们都继承自 `Measurable`。`File` 和 `Music` 也能保证提供了 `Progress` 需要的所有方法都存在了。

​	想学习更多关于这种策略模式的话，请看课程 [Agile Design Patterns](https://tutsplus.com/course/agile-design-patterns/)

### 关于接口命名的笔记

​	人们习惯于用大写的 I 作为接口名的开头，或者末尾接上 Interface，像是 `IFile` 或者 `FileInterface`。这是一种过时的命名法。我们可以抛弃匈牙利命名法，也不需要为了方便识别而在名字上加上类型，IDE 可以在一瞬间为我们识别所有的东西。这允许我们更多的关注抽象出的到底是什么。

​	接口是提供给应用类使用的。当命名时一定要思考应用类并忘记接口的具体实现。当我们将接口命名为 Measurable 只需要考虑 Progress。如果我需要进度，我需要一个什么东西来提供百分比的值呢？答案很简单，可以度量的东西就可以了，所以就将这个接口命名为 Measurable。

​	另一个这么做的原因是接口的实现可能来自不同的领域。在我们的例子中就包括了文件和音乐。这样我也也可以很好的将 `Progress` 应用到计时器这样的东西上去。



## 方案三：使用模板方法模式

​	模板方法模式很接近与策略模式，但是模板方法模式使用的是抽象类而不是接口。当我们的产品针对特定用户时或者服务类有着类似的行为，建议使用模板方法。

![](https://cdn.tutsplus.com/net/uploads/2014/01/template_method.png)

​	通过课程 [Agile Design Patterns ](https://tutsplus.com/course/agile-design-patterns/) 学习更多相关知识。



## 高层视图

​	那么，开闭原则对于我们的高层视图造成了什么影响？

![](https://cdn.tutsplus.com/net/uploads/2014/01/HighLevelDesign.png)

​	如果上图代表了我们程序目前的架构，添加一个包括五个类（蓝色）的新模块将会对我们的设计产生一些适度的影响（红色）。

![](https://cdn.tutsplus.com/net/uploads/2014/01/HighLevelDesignWithNewClasses.png)

​	在大多数系统中你不能指望一点都不修改现有代码。但是遵循开闭原则会将这种修改限制在最小范围内。

​	和其它原则一样，不要从一开始就想去考虑所有方面。如果你真的这么做了，你的每个类都会有一个接口。这样的设计将会难以理解和维护。通常来说，安全的做法是决定这儿以后会不会有扩展的需求。很多时候很容易就能想到一个以后可能的新功能需要新的服务类来支持，在这种情况下就从一开始就加上接口。如果你决定不了或者不确定，那么就忽略它。让别的程序员或者你自己在需要的时候再添加接口。



## 最后的思考

​	当需要扩展的时候，立刻就添加接口支持会让修改变得更少更简单。记住，如果某处代码需要修改一次，那么它很可能需要修改第二次。当这种事真的发生的时候，OCP 会节省你大量时间和精力。



​	感谢您的阅读。



