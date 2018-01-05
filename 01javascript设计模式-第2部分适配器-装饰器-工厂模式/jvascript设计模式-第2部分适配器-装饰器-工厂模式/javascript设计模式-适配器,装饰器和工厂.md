### javascript设计模式-适配器,装饰器和工厂
![Alt text](./giphy-downsized.gif)

###前言
本文为译文,初次翻译,如有误导,请多多包含,阅读英文,可直接戳底下原文链接即可
作者:joseph Zimmerman

    要求
    前提知识
    基础的javascript编程知识
    用户级别
    所有

你已经来到javascript设计模式系列中的第二部分,自从接触第一部分已经有一段时间了,所以,你也可能想要在单例,复合,外观设计模式上提升自己,第三部分讨论了另外3种设计模式:代理,观察者,和命令

这一次,您将了解关于适配器,装饰器和工厂模式
### 适配器
这个适配器模式允许你将接口转换成(或适应)你的需要,这个可以通过创建另一个具有所需接口的对象来实现,并连接到你想要改变的接口对象

适配器图案的结构如下所示
![](http://i.imgur.com/N9rWaBI.png)
### 为什么你需要适配器？
通常情况下,你正在处理一个应用程序时,并且你决定需要更换一个应用程序,例如,用于保存日志或者某种性质的库,当你引进新的库来替换它时,它不可能具有完全相同的界面,从这里你有两个选择
1. 浏览所有的代码,并改变引用旧库里的所有内容
2. 创建一个新的适配器,使新的库可以使用,并与旧的库具有相同的界面

显然,在一些情况下,你的应用将是足够的轻小,或者对旧的库有足够的参考,觉得它更合适了,就可以回到代码中并更改它以匹配新的库,而不是使代码复杂化,抽象,但是,在大多数情况下,只需要创建适配器就可以实现省时

### 适配器的示例

为什么你不采取上述假设的日志方案,并将其用与我们的示例代码中?最初在你的代码中,你可以使用大多数浏览器内置的控制台来制作日志,但是有一些问题,它并不是内置到所有的浏览器(特别是旧版本浏览器)中，你看不到发生的日志,当其他用户使用你的应用程序时,你无法抓取看到自己测试时遇到的任何问题,所以,你决定要一个日志记录器,而不是使用Ajax将这些日志发送到服务器来处理,以下是你的新Ajax logger库的API,可能如下所示

    AjaxLogger.sendLog(arguments);
    AjaxLogger.sendInfo(arguments);
    AjaxLogger.sendDebug(arguments);
    etc...

显然,这个库的作者并没有意识到你会尝试用这个来代替控制台,所以他或者她觉得在每个方法的名字开始添加发送都是有必要的,现在你问,为什么我编辑这个库,改变这个方法的名称?在这里有好几个很好的理由不这样做,如果你需要更新该库,你的库将会被覆盖,所以你需要返回并在此更改,另外,你如果您从库内传送的网络中下载该库,则无法编辑他们,所以让我们来构建一个新的库适应新的库并与控制台具有相同的界面
    

    var AjaxLoggerAdapter = {
    log: function() {
        AjaxLogger.sendLog(arguments);
    },
    info: function() {
        AjaxLogger.sendInfo(arguments);
    },
    debug: function() {
        AjaxLogger.sendDebug(arguments);
    },...
    };



### 你怎么使用它?
我敢打赌,任何使用过控制台的人直接通过它的引用来调用它,所以,你如何获得每个调用console.xxx来引用新的适配器而不是控制台?如果你使用的是抽象(像工厂)检索控制台,那么你可以在该抽象层中进行更改,但是已经说明了,每个人都直接引用到控制台,那么,javascript是一种动态的语言,它允许在运行时更改事物,所以为什么不用新的Ajax日志适配器来覆盖控制台?
    
    window.console = AjaxLoggerAdapter;

那很简单,不是吗？小心,如果你在其它人使用的代码中执行此操作,则控制台不再像用户期望的那样运行,另外,不要因为这个例子的简单而被愚弄,在许多的情况下,你将无法轻松的映射到彼此的方法(比如发送日志记录),你可能不得不实际执行一些你自己的逻辑与将来的用户界面转换与新库保持兼容

### 装饰器
装饰器模式是与许多其他模式非常不同的,装饰器模式解决了在类上添加或更改功能,不为每个功能组合创建子类的问题,例如,你有一个默认功能类的汽车类,汽车有几个可以添加到车上的可选功能(例如,电动锁,电动车窗和空调),如果你想尝试着用子类化处理这个问题,你将共有8个类来覆盖所有的组合

* 汽车
* 电动锁
* 电动车窗
* 汽车空调
* 电动汽车锁和电动汽车窗
* 电动汽车锁和空调
* 电动汽车窗和空调
* 电动汽车窗和电动锁和空调

这个可以快速的通过添加另外一个选项,这又可以增加8个子类,很快就会失控,这个可以使用装饰器模式解决,所以每次添加一个新的选项时,你只创建一个类,而不是将类的数量加倍
### 装饰器的结构
这个装饰器通过使用与基础对象具有相同接口的装饰器对象包装基础对象(car)来工作,装饰器有几个可以处理方法的选项:

1. 它可以完全覆盖它所包含的对象的方法
2. 如果装饰器不影响方法的行为,它可以直接传递到包装对象
3. 这个装饰器可以在将调用传递到包装对象之前或之后添加行为

装饰器并没有限制只包装基础对象,他们也可以包装其他装饰器,因为他们都实现了相同的界面,一般结构上看起来像如下图所示
![](http://i.imgur.com/XXmpwhq.png)
### 一个装饰器的例子
让我们看看上面的汽车插图(illustration),并把它编写成代码,首先构建基础车类

```
var Car = function() {
    console.log('Assemble: build frame, add core parts');
};
 
// The decorators will also need to implement this interface
Car.prototype = {
    start: function() {
        console.log('The engine starts with roar!');
    },
    drive: function() {
        console.log('Away we go!');
    },
    getPrice: function() {
        return 11000.00;
    }
};
```
通过将短语发送到控制台来显示发生的情况,使事情变得更加简单,显然,你可以添加更多的功能,因为汽车是一个非常复杂的东西,但是为来这个演示,我相信不会保持简单中的简单了

现在创建CarDecorator类,这意味着是一个不被实例化的抽象类,但是应该被子类化以创建真正的装饰器

```
var CarDecorator = function(car) {
    this.car = car;
};
 
// CarDecorator implements the same interface as Car
CarDecorator.prototype = {
    start: function() {
        this.car.start();
    },
    drive: function() {
        this.car.drive();
    },
    getPrice: function() {
        return this.car.getPrice();
    }
};
```
这里需要注意几点,首先,`CarDecorator`构造函数需要一辆汽车,或者采用了一个实现与car相同接口的对象,Car包含`CarDecorator`的多个Car和子类,还要注意，所有的方法只是将请求传递给包装对象,这是我们所有装饰器应用于未被更改的方法的默认行为


现在创建所有的装饰器,自动你继承`CarDecorator`，你只需要覆盖更改的功能，否则CarDecorator可以正常工作

```
var PowerLocksDecorator = function(car) {
    // Call Parent Constructor
    CarDecorator.call(this, car);
    console.log('Assemble: add power locks');
};
PowerLocksDecorator.prototype = new CarDecorator();
PowerLocksDecorator.prototype.drive = function() {
    // You can either do this
    this.car.drive();
    // or you can call the parent's drive function:
    // CarDecorator.prototype.drive.call(this);
    console.log('The doors automatically lock');
};
PowerLocksDecorator.prototype.getPrice = function() {
    return this.car.getPrice() + 100;
};
var PowerWindowsDecorator = function(car) {
    CarDecorator.call(this, car);
    console.log('Assemble: add power windows');
};
PowerWindowsDecorator.prototype = new CarDecorator();
PowerWindowsDecorator.prototype.getPrice = function() {
    return this.car.getPrice() + 200;
}; 
var AcDecorator = function(car) {
    CarDecorator.call(this, car);
    console.log('Assemble: add A/C unit');
};
AcDecorator.prototype = new CarDecorator();
AcDecorator.prototype.start = function() {
    this.car.start();
    console.log('The cool air starts blowing.');
};
AcDecorator.prototype.getPrice = function() {
    return this.car.getPrice() + 600;
};
```
在这个例子当中,每当我创建一个向原始文件添加功能的方法时,我只是调用`this.car.x（）`，而不是调用父类的方法,一般来说，调用父母的方法会更加明智,但是因为这样做很简单,所以我认为直接使用汽车的属性会更加简单,如果我需要改变CarDecorator中内置的默认行为来做某些事情，而不仅仅是传递发送请求，那么这可能会伤害到我，但是考虑到这只是一个例子，我没有预见到这一点


所以，现在你拥有所有需要的元素，让它们采取行为

```
var car = new Car(); // log "Assemble: build frame, add core parts"
// give the car some power windows
car = new PowerWindowDecorator(car);// log "Assemble: add power windows"
// now some power locks and A/C
car = new PowerLocksDecorator(car); // log "Assemble: add power locks"
car = new AcDecorator(car); // log "Assemble: add A/C unit"
// let's start this bad boy up and take a drive!
car.start(); // log 'The engine starts with roar!' and 'The cool air starts blowing.'
car.drive(); // log 'Away we go!' and 'The doors automatically lock'
```
使用你想要的功能创建汽车很简单,只需创建您需要的汽车和装饰器,同时将车辆的当前功能添加到装饰器,创建代码相当长，特别是与实例化一个对象（如汽车电动锁和汽车电动窗户,和汽车空调）相比较,不要担心，我会在有关工厂模式的部分中给出更好的方法来创建这些装饰器对象.在此期间，如果您想阅读更多的装饰图案，您可以在我的个人博客上阅读“JavaScript Design Patterns：Decorator”

### 工厂
工厂从其预期目的获得了名称,简化了对象的创建,简单的工厂抽象出新关键字的所有这些用法,以便如果该类名更改或替换为不同的名称,则只需要在一个位置更改它。此外，它还建立了一个一站式服务,用于创建许多不同类型的对象或具有不同选项的单一对象类型,标准规范有点难以用这么少的话来解释,所以我稍后再说一遍
### 一个简单的JavaScript工厂
这个例子是一个简单的工厂,使我们遇到的难题与上面的装饰器示例,创建具有功能的汽车的实际代码太长了,使用工厂，您将这一切都剪切到一个单一的函数调用,从包含单个函数的对象文字开始,在JavaScript中,一个对象文字/单例是一个简单的工厂的构建方式。在传统的面向对象编程语言中，这将是一个静态类

```
var CarFactory = {
    makeCar: function(features) {
        var car = new Car();
        // If they specified some features then add them
        if (features && features.length) {
            var i = 0,
                l = features.length;
            // iterate over all the features and add them
            for (; i < l; i++) {
                var feature = features[i];
                switch(feature) {
                    case 'powerwindows':
                        car = new PowerWindowsDecorator(car);
                        break;
                    case 'powerlocks':
                        car = new PowerLocksDecorator(car);
                        break;
                    case 'ac':
                        car = new ACDecorator(car);
                        break;
                }
            }
        }
        return car;
    }
}
```
这个工厂的一个功能是makeCar,这很重要,首先,它接受的一个参数是一个字符串数组,它映射到不同的装饰器类, makeCar创建一个简单的Car对象,然后遍历功能并对其进行装饰,现在，而不是至少有四行代码来创建具有所有功能的汽车，我们只需要一个:

```
Var myCar = CarFactory.makeCar(['powerwindows', 'powerlocks', 'ac']);
```
您甚至可以调整makeCar功能,以确保只使用任何类型的装饰器中的一种(如果它们实际上是在工厂制作的),如果您想看到一个例子，还有其他一些很好的使用工厂模式的方法，您可以在我的个人博客中查看JavaScript Design Patterns：Factory
### 标准工厂
标准的工厂模式与简单的工厂完全不同,但当然还有创造对象的作用。而不是使用单例,但是，只需在类上使用抽象方法即可,例如，假装有几个不同的汽车制造商在那里，他们都有自己的商店。在大多数情况下，所有的商店都使用相同的销售方法，除了制造不同的汽车。所以他们都从相同的原型继承他们的方法，但是他们实现自己的制造过程

让我们把这个例子变成代码来帮助看看我在说什么。首先创建一个仅仅是子类的汽车店，并且有一个方法 - 制造商 - 这是一个存根。它会抛出一个错误，除非它被子类覆盖

```
/* Abstract CarShop "class" */
var CarShop = function(){};
CarShop.prototype = {
    sellCar: function (type, features) {
        var car = this.manufactureCar(type, features);

        getMoney(); // make-believe function
        return car;
    },
    decorateCar: function (car, features) {
        /*
        Decorate the car with features using the same
        technique laid out in the simple factory
        */
    },
    manufactureCar: function (type, features) {
        throw new Error("manufactureCar must be implemented by a subclass");
    }
};
```
请注意，卖汽车的制造商。这意味着制造商需要由一个子类来实现，以便您销售一辆汽车。所以创建一对的汽车店，看看他们如何实现它

```
/* Subclass CarShop and create factory method */
var JoeCarShop = function() {};
JoeCarShop.prototype = new CarShop();
JoeCarShop.prototype.manufactureCar = function (type, features) {
    var car;
    // Create a different car depending on what type the user specified
    switch(type) {
        case 'sedan':
            car = new JoeSedanCar();
            break;
        case 'hatchback':
            car = new JoeHatchbackCar();
            break;
        case 'coupe':
        default:
            car = new JoeCoupeCar();
    }

    // Decorate the car with the specified features
    return this.decorateCar(car, features);
};
/* Another CarShop and with factory method */
var ZimCarShop = function() {};
ZimCarShop.prototype = new CarShop();
ZimCarShop.prototype.manufactureCar = function (type, features) {
    var car;
    // Create a different car depending on what type the user specified
    // These are all Zim brand
    switch(type) {
        case 'sedan':
            car = new ZimSedanCar();
            break;
        case 'hatchback':
            car = new ZimHatchbackCar();
            break;
        case 'coupe':
        default:
            car = new ZimCoupeCar();
    }
    // Decorate the car with the specified features
    return this.decorateCar(car, features);
};
```
这些方法在它们的工作方式基本上是相同的，除了它们各自创建不同的汽车（为了简洁，我省略了汽车课程的实现，因为它们不是必需的）。关键是制造方法是制造方法。工厂方法在父类上是抽象的，由子类实现，其作业是创建对象（每个工厂方法也使用相同的接口创建对象）
### 完成接触
这涵盖了工厂模式背后的基础。如果您想了解更多关于工厂模式的信息，请访问我的个人博客上的JavaScript设计模式：工厂（简单工厂）和JavaScript设计模式：工厂第2部分（标准工厂）
### 从哪里去
它包含了JavaScript 设计模式系列的第2部分,希望它能让您了解如何在JavaScript中实现适配器,装饰器和工厂模式,在本系列的第1部分中,将介绍另外3种设计模式:单例，复合和外观第3部分讨论了3种设计模式:代理,观察者和命令

第三个也是最后一个JavaScript设计模式的帖子不应该太远，但是如果你有耐心，我的个人博客已经完成了JavaScript设计模式的系列，并且包含了一些我不会在这个系列中展示的模式：Joe Zim的JavaScript博客上的JavaScript设计模式

`总结`

本文是自己初次翻译,忠于原文,如有误导,请多多包含,阅读原文可点击下方阅读原文,主要讲述来javascript设计模式-适配器,装饰器和工厂,主要是想看看老外的写作代码思维,关于设计模式,概念性的东西比较多,也比较抽象,什么单例,适配器,工厂,观察者,命令行模式都是为了提供清晰的结构,移除不必要的复杂性,为提高性能而生的,并将大型代码库中的不同部分的连接进行解耦,最终目的是为了使代码更易于理解和管理,维护,各种设计模式有着不同的应用场景,好比编程工具箱中的各种工具,当然,我了解的也只是一点皮毛,更多的内容值得自己学习和探索的

![Alt text](./code.jpg)
更多内容尽在微信`itclan`公众号






