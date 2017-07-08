是时候写一写 “继承”了，为什么加引号，因为当你阅读完这篇文章，你会知道，说是 继承 其实是不准确的。

一、类  
1、传统的面向类的语言中的类：  
类／继承 描述了一种代码的组织结构形式。举个例子：  
“汽车”可以被看作是“交通工具”的一种特例。  
我们可以定义一个 Vehicle 类和一个 Car 类来对这种关系进行描述。  
Vehicle 的定义可能包含引擎、载人能力等，也就是 所有交通工具，比如飞机、火车和汽车等都有的通用的功能描述。  
在对 Car 类进行定义的时候，重复定义“载人能力”是没有意义的，我们只需要声明 Car 类继承了 Vehicle 的这个基础类就可以了。 Car 实际上是对通用 Vehicle 定义的特殊化。  
Car 现在只是个类，当我们把它更加形象化，比如说是 保时捷、奔驰的时候，就是一个实例化的过程。  
我们也有可能 会在 Car 中定义一个 和 Vehicle 中相同的方法，这是子类对父类针对特定方法的重写，为了可以更加特殊化，更加符合对子类的描述，这个被称作是 多态。

以上就是 类、继承、实例化和多态。

再举个例子：比如房屋的构建  
建筑师设计出来建筑蓝图，然后由建筑工人按照建筑蓝图建造出真正的建筑。建筑就是蓝图的物理实例，本质上是对建筑蓝图的复制。之后建筑工人就可以到下一个地方，把所有的工作重复一遍，再创建一份副本。  
建筑和蓝图之间的关系是间接的。你可以通过蓝图了解建筑的结构，只观察建筑本身是无法获得这些信息的。但是如果你想打开一扇门，那就必须接触真实的建筑才行－－蓝图只能表示门应该在哪，但并不是真正的门。  
一个类就是一张蓝图。为了获得真正可以交互的对象，我们必须按照类来建造（实例化）一个东西，这个东西通常被称为实例。这个对象就是类中描述的所有特性的一份副本。

在传统的面向类的语言中，类的继承，实例化其实就是复制，用一张图：

![](/assets/impor2t.png)

箭头表示复制操作。

子类 Bar 相对于 父类 Foo 来说是一个独立并完全不同的类。子类会包含父类行为的副本，也可以通过在子类中定义于父类中相同的方法名来改写某个继承的行为，这种改写不会影响父类中的方法，这两个方法互不影响。对于 类 Bar 和 实例 b1 、b2 之间也同样是 类通过复制操作被实例化为对象形式。

2、javascript 中的类  
然而 javascript 其实并没有类的概念，但是 我们早已经习惯用类来思考，所以 javascript 也提供了一些近似类的语法，我们用它模拟出了类，然而这种 “类”还是与传统面向类语言中的类有不同。  
在继承和实例化的过程中，javascript的对象机制并不会自动执行复制行为。简单来说，javascript中只有对象，并不存在可以被实例化的“类”。一个对象并不会被复制到其它对象，他们只会被关联起来，也就是复制的是其实是引用。（对于复制引用，具体的可以看[【 js 基础 】 深浅拷贝](http://www.cnblogs.com/lijiayi/p/jsdeeepcopy.html) 一文，第一部分）

二、javascript 原型链、“类”和 继承  
1、 \[\[prototype\]\]：javascript 中的对象都有一个特殊的 \[\[prototype\]\] 内置属性，其实就是对于其他对象的引用。他的作用是什么呢？  
当你试图访问对象的属性的时候，就会触发对象的内置操作 \[\[Get\]\]，\[\[Get\]\] 操作就是从对象中找到你要的属性。然而他是怎么找的呢？  
例子：

```
 var testObject = {
     a:2
 };
 console.log(testObject.a) // 2
```

在上面的代码中，当你 console 的时候，会触发 \[\[Get\]\] 操作，查找 testObject 中的 a 属性。对于默认的 \[\[Get\]\]操作，第一步是检查对象本身是否有这个属性，如果有的话就直接使用它。第二步，如果 a 不在 testObject 中，也就是 无法在对象本身中找到需要的属性，就会继续访问对象的 \[\[prototype\]\] 链。

例子：

```
var anotherObject  = {
    a : 2
};

var testObject  = Object.create(anotherObject);

console.log(testObject.a); // 2
```

Object.create\(\) 方法会创建一个对象并把这个对象的 \[\[prototype\]\] 关联 到指定的对象（anotherObject）。  
例子中，testObject 对象的 \[\[prototype\]\] 关联到了 anotherObject。 testObject 本身并没有 a 属性，然而还是可以 console 出testObject.a 为 2，这是 \[\[Get\]\] 从 testObject 的 \[\[prototype\]\] 链中找到的，即 anotherObject 中的属性a。但是倘若 anotherObject中也没有属性 a，并且 \[\[prototype\]\]不为空，就会继续查找下去。这个过程会持续到找到匹配的属性名，或者查找完整条 \[\[prototype\]\] 链未找到，\[\[Get\]\] 操作的返回值是 undefined 。

那么哪里是原型链的尽头呢？  
所有的普通的 \[\[prototype\]\] 链最终都会指向 内置的 Object.prototype 。

2、“类”：在上一部分的内容中，我们已经说到，javascript 中其实是没有传统意义上的“类”的，但我们一直在试图模仿类，主要是利用了 函数的一种特殊特性：所有的函数默认都会有一个名为 prototype 的公有并且不可枚举的属性，它会指向另一个对象。  
例子：

```
function foo(){
    //…
}

foo.prototype; // {}

var a = new foo();
Object.getPrototypeOf(a) === foo.prototype;//true
```

这个对象是在调用 new foo \(\) 时创建的，最后会被关联到 foo.prototype 上。就像例子中的调用 new foo \(\) 时会创建 a ，然后 将 a 内部的 \[\[prototype\]\] 链接到 foo.prototype 所指向的对象。

在传统的面向类的语言中，类可以被复制多次，每次实例化的过程都是一次复制。但在 javascript 中没有类似的复制机制。你不能创建一个类的多个实例，只能创建多个对象，他们的 \[\[prototype\]\] 关联的是同一个对象。因为在默认情况下，并不会进行复制，所以这些对象之间并不会完全失去联系，他们是互相关联的。

就像上面的例子 new foo \(\) 会生成一个新的对象，称为 a，这个新的对象的内部的 \[\[prototype\]\] 关联的是 foo.prototype 对象。最后我们得到两个对象，他们之间互相关联。我们并没有真正意义上初始化一个类，实际上我们并没有从 “类” 中复制任何行为到一个对象中，只是让两个对象互相关联着。

再强调一下： 在 javascript 中，并不会将一个对象（“类”）复制到另一个对象（“实例”），只是将它们关联起来。看一个图：

![](/assets/impor21t.png)

箭头表示 关联。  
这个图就表达了 \[\[prototype\]\] 机制，即 原型继承。  
但是说是 继承其实是不准确的，因为传统面向类的语言中 继承 意味着复制操作，而 javascript （默认）并不会复制对象属性，而是在两个对象之间创建一个关联，这样一个对象可以 委托 访问另一个对象的属性和函数。委托 可以更加准确的描述 javascript 中对象的关联机制。

3、 （原型）继承  
来看个例子：

```
function foo(name){
    this.name = name;
}

foo.prototype.myName = function(){
    return this.name;
}

function bar(name,label){
    foo.call(this,name);
    this.label = label;
}

// 创建了一个新的 bar.prototype 对象并把它关联到了 foo.prototype。
bar.prototype = Object.create(foo.prototype);

bar.prototype.myLabel = function(){
    return this.label;
}

var a = new Bar(“a”,”obj a”);
console.log(a.name) // “a"
console.log(a.label) // "obj a"
```

声明 function bar\(\){} 时，bar 会有一个默认的 .prototype 关联到默认的对象，但是这个对象不是我们想要的 foo.prototype 。因此 我们通过 Object.create\(\) 创建了一个新的对象并把它关联到我们希望的对象上，即 foo.prototype，直接把原始的关联对象抛弃掉。

如果你说为什么不用下面这种方式关联？

```
bar.prototype = foo.prototype
```

因为 这种方式并不会创建一个关联到 foo.prototype 的新对象，它只是让 bar.prototype 直接引用 foo.prototype 。因此当你执行 bar.prototype.myLabel 的赋值语句时会直接修改 foo.prototype 对象本身。

或者说为什么不用new？

```
bar.prototype = new foo();
```

这样的确会创建一个关联到 foo.prototype 的新对象。 但是它同时 也执行了对 foo 函数的调用，如果 foo 函数中有给this添加属性、修改状态、写日志等，就会影响到 bar\(\) 的 “后代” 。

这里补充两点关于 new ，方便理解：

```
function foo(){
    console.log(“test”);
}

var a = new foo(); // test
```

当你执行 var a = new foo\(\); 也就是使用 new 来调用函数，会执行下面四步操作：  
1、创建一个全新的对象  
2、这个新对象会被执行 \[\[prtotype\]\] 连接  
3、这个新对象会绑定到函数调用的 this 上  
4、如果函数没有返回值，那么 new 表达式中的函数调用会自动返回这个新对象。

另一点，当你执行 var a = new foo\(\); 时 ，console 打出 test。foo 只是个普通的函数，当使用 new 调用时，它就会创造一个新对象并赋值给 a，当然也会调用自身。

综上，要创建一个合适的关联对象，最好的方式就是用 Object.create\(\)，这样做也有缺点：就是创建了新对象，然后把旧对象抛弃掉，不能直接修改默认的已有对象了。  
Object.create\(\) 会创建一个 拥有空 \[\[prototype\]\] 连接的对象。它是 es5 新增的方法，让我们来看看在老的环境中如何实现它：

```
if(!Object.create){
    Object.create = function(o){
        function F(){}
        F.prototype = o;
        return new F();
    }
}
```

我们使用了一个空函数 F，通过改写它的 .prototype 属性使其指向想要关联的对象，然后再使用 new F\(\) 来构造一个新对象来进行关联。

三、类式继承设计模式 和 委托设计模式  
这两种模式都是用来实现继承，本质上也就是 关联。

1、类式继承设计模式：  
这个应该是大家最熟悉的，主要就是运用构造函数和原型链实现继承，也就是所谓的面向对象风格。

```
function Foo(who){
    this.me = who;
}

Foo.prototype.identify = function(){
    return "i am “ + this.me;
}

function Bar(who){
    Foo.call(this,who);
}

Bar.prototype = object.create(Foo.prototype);

Bar.prototype.speak = function(){
    alert(“Hello,”+ this.identify()+”.”);
}

var b1 = new Bar(“b1”);
var b2 = new Bar(“b2”);

b1.speak();
b2.speak();
```

子类 Bar 继承了 父类 Foo，然后生成了 b1 和 b2 两个实例。 b1 继承了 Bar. prototype , Bar.prototype 继承了 Foo.prototype。

2、 委托设计模式  
对象关联风格：

```
Foo = {
    init:function(who){
        this.me = who;  
    },
    identify:function(){
        return “i am” +this.me;
    }
};

Bar = Object.create(Foo);
Bar.speak = function(){
    alert(“hello,” + this.identify())
};

var b1 = Object.create(Bar);
b1.init(“b1”);
var b2 = Object.create(Bar);
b2.init(“b2”);

b1.speak();
b2.speak();
```

这段代码同样 利用 \[\[prototype\]\] 把 b1 委托给 Bar 并把 Bar 委托给 Foo，和上一段代码一摸一样，同样实现了三个对象的关联。

3、以上两种模式都实现了三个对象的关联，那么它们的区别是什么呢？  
首先是思维方式的不同：  
      类式继承设计模式：定义一个通用的父类，可以将其命名为 Task，在 Task 中定义所有任务都有的行为。接着定义子类 A 和 B，他们都继承子 Task，并且会添加一些特殊的行为来处理对应的人物。然后你实例化子类，这些实例拥有 父类 Task 的通用方法，也拥有 子类 A 的特殊行为。

```
 委托设计模式：首先定义一个名为Task 的对象，它包含所有任务都可以使用的行为。接着对于每个任务 A 和 B，都会定义一个对象来存储对应的数据和行为。执行 任务 A 需要两个兄弟对象（Task 和 A）协作完成，只是在需要某些通用行为的时候 可以允许 A 对象委托给 Task。在上面的例子中，也就是 Bar 通过 Object.create\(Foo\); 创建，它的 \[\[prototype\]\] 委托给了 Foo 对象。这就是一种对象关联的风格。委托行为意味着某些对象（Bar）在找不到属性或者方法引用时会把这个请求委托给另一个对象（Foo）。

 委托设计模式不是按照父类到子类的关系垂直组织的，而是通过任意方向的委托关联并排组织的。
```

其次，代码实现明显不同，可以感觉到 委托设计模式，也就是对象关联风格更加简洁，这种设计模式只关注对象之间的关联关系。  
最后，委托设计模式更加贴近 javascript 的 “继承”机制 —— 委托机制。



---

学习并感谢

《你不知道的JavaScript》上卷  （炒鸡推荐大家看）

