学习源码，除了学习对一些方法的更加聪明的代码实现，同时也要学习源码的设计，把握整体的架构。（推荐对源码有一定熟悉了之后，再看这篇文章）



目录结构：  
第一部分：zepto 设计分析  
第二部分：underscore 设计分析

  
第一部分： zepto 设计分析  
zepto 是一个轻量级的 Javascript 库。相对于 jquery 来说在 size 上更加小，主要是定位于移动设备。它是非常好的学习源码的入门级 javascript 库。这里重点说一下，这个库的设计，而对于详细的源码学习大家可以 star 我的 github 源码学习项目（https://github.com/JiayiLi/source-code-study） 进行关注。

让我们先看看把所有代码删除只剩下这几行的 zepto ：

```
 var Zepto = (function() {
     return $
 })() 
 window.Zepto = Zepto 
 window.$ === undefined && (window.$ = Zepto)
```

一个匿名自执行函数返回 $ 传递给了 Zepto。然后把 Zepto 挂到 window 上，使其成为全局 window 的一个属性，同时，如果 window.$ 符号没有被占用，那么 $ 会被赋值为 Zepto，故可以全局范围内使用 $。

然后咱们再来看看赋值给 zepto 的匿名自执行函数的核心代码具体干了什么：

```
var Zepto = (function() {
    //zepto和$是Zepto中的两个命名空间，用来挂载静态函数
    var $，zepto = {};

    function Z(dom, selector) {
        var i, len = dom ? dom.length: 0
        for (i = 0; i < len; i++) this[i] = dom[i] this.length = len this.selector = selector || ''
    }

    zepto.Z = function(dom, selector) {
        return new Z(dom, selector)
    }
    zepto.init = function(selector, context) {....
        return zepto.Z(dom, selector)
    }

    //$()的返回值能够调用$.fn中的方法
    $ = function(selector, context) {
        return zepto.init(selector, context);
    }

    $.fn = {
        // 里面有若干个工具函数
    };

    zepto.Z.prototype = $.fn;
    $.zepto = zepto

    return $ //返回$,赋值给Zepto
})()

window.Zepto = Zepto
//当$未被占用时就把Zepto赋值给$
window.$ === undefined && (window.$ = Zepto)
```

首先定义了 两个变量 zepto 和 $ ，还有一个 构造函数 Z 。  
对于变量 zepto ，给其定义了两个方法 Z 和 init。init 方法中调用了 zepto 的 Z 方法，而在 zepto 的 Z 方法中 则 实例化了 构造函数 Z。  
对于 变量 $ ，则是个函数，内部调用了 zepto 的 init 方法 ，也就是最后返回了 构造函数 Z 的 新实例 。同时也给 $ 上定义了一个属性 fn，fn 是一个 对象，这个对象里面实现了多个工具函数，比如 concat、slice、each、filter 等。

然后将 刚才定义的 变量 zepto 中的 Z 方法的原型 即构造函数的原型 指向了 $.fn ，这样 调用 $ 函数所返回的 Z 实例 ，就继承了 $.fn 中的方法。

最后通过 $.zepto = zepto 将内部 API 导出。如果有需要可以调用 zepto 上的内部方法。

Z 实例 实际上一个 对象数组，即可以模拟数组操作的对象。

咱们再用个图来梳理一下思路：![](/assets/133import.png)

－－－－－

学习并感谢：

[https://www.kancloud.cn/wangfupeng/zepto-design-srouce/173681](https://www.kancloud.cn/wangfupeng/zepto-design-srouce/173681)

[https://segmentfault.com/a/1190000007515865\#articleHeader4](https://segmentfault.com/a/1190000007515865#articleHeader4)

  
-------------

第二部分 underscore 设计分析  


underscore 是一个Javascript 实用库。是函数式编程的典型代表。它是非常好的学习源码的入门级 javascript 库，尤其是学习函数式编程的好材料。这里重点说一下，这个库的设计，而对于详细的源码学习大家可以 star 我的 github 源码学习项目（https://github.com/JiayiLi/source-code-study） 进行查看。

underscore 的所有代码都包裹在匿名自执行函数中，

```
 (function() {
       ...
 }.call(this))   //  通过传入this（浏览器环境中其实就是window对象）来改变函数的作用域
```

大多数的源码设计都是使用的 匿名自执行函数，这样做的好处：  
1、避免全局污染：库中所定义的变量，方法都封装到了该函数的作用域中。  
2、隐私保护：使用方只能获得 库 想暴露在外面的变量方法，而不能访问 不想暴露的内部变量方法。

再说设计之前，这里还要说一个知识点：  
underscore 采用的是典型的函数式编程风格，这与面向对象的编程风格并不相同。  
函数式编程（fp）风格，设计的函数方法并不会属于任何一个对象，对象只是 这些函数方法的参数。  
而面向对象的编程（oop）风格则是 设计的函数方法都隶属于一个对象。作为对象的一个属性。  
如果你还没有明白，这里看一下调用方式的不同：  
函数式编程风格：

```
 var arr = [1, 2, 3];
 _.map(arr,function(item) {
      return item * 2;
 });
```

arr 这个对象只是 map 方法的一个参数，map 并不属于 arr。



面向对象风格：

```
 var arr = [1, 2, 3];
 arr.map(function(item) {
      return item * 2;
 });
```

map是对象arr的一个方法。

看出区别了吗？



回到匿名自执行函数内部，核心代码如下：

    (function() {
         var root = this;

         // 核心函数
         // `_` 其实是一个构造函数
         var _ = function(obj) {
              // 以下均针对 OOP 形式的调用
              // 如果是非 OOP 形式的调用，不会进入该函数内部
              // 如果 obj 已经是 `_` 函数的实例，则直接返回 obj
              if (obj instanceof _) return obj;

              // 如果不是 `_` 函数的实例
              // 则调用 new 运算符，返回实例化的对象
              if (! (this instanceof _)) return new _(obj);

              // 将 obj 赋值给 this._wrapped 属性
              this._wrapped = obj;
         };

         // 将上面定义的 `_` 局部变量赋值给全局对象中的 `_` 属性
         // 即客户端中 window._ = _
         // 服务端(node)中 exports._ = _
         // 同时在服务端向后兼容老的 require() API
         // 这样暴露给全局后便可以在全局环境中使用 `_` 变量(方法)
         if (typeof exports !== 'undefined') {
              if (typeof module !== 'undefined' && module.exports) {
                   exports = module.exports = _;
              }
              exports._ = _;
         } else {
              root._ = _;
         }

         // .....  定义工具函数  如 _.each, _.map 等


         _.mixin = function(obj) {
              // 遍历 obj 的 key，将方法挂载到 Underscore 上
              // 其实是将方法浅拷贝到 _.prototype 上
              _.each(_.functions(obj),
              function(name) {
                   // 直接把方法挂载到 _[name] 上
                   // 调用类似 _.myFunc([1, 2, 3], ..)
                   var func = _[name] = obj[name];

                   // 浅拷贝
                   // 将 name 方法挂载到 _ 对象的原型链上，使之能 OOP 调用
                   _.prototype[name] = function() {
                        // 第一个参数
                        var args = [this._wrapped];

                        // arguments 为 name 方法需要的其他参数
                        push.apply(args, arguments);
                        // 执行 func 方法
                        // 支持链式操作
                        return result(this, func.apply(_, args));
                   };
              });
         };

         // Add all of the Underscore functions to the wrapper object.
         // 将前面定义的 underscore 方法添加给包装过的对象
         // 即添加到 _.prototype 中
         // 使 underscore 支持面向对象形式的调用
         _.mixin(_);

         // Add all mutator Array functions to the wrapper.
         // 将 Array 原型链上有的方法都添加到 underscore 中
         _.each(['pop', 'push', 'reverse', 'shift', 'sort', 'splice', 'unshift'],
         function(name) {
              var method = ArrayProto[name];
              _.prototype[name] = function() {
                   var obj = this._wrapped;
                   method.apply(obj, arguments);

                   if ((name === 'shift' || name === 'splice') && obj.length === 0) delete obj[0];

                   // 支持链式操作
                   return result(this, obj);
              };
         });

         // Add all accessor Array functions to the wrapper.
         // 添加 concat、join、slice 等数组原生方法给 Underscore
         _.each(['concat', 'join', 'slice'],
         function(name) {
              var method = ArrayProto[name];
              _.prototype[name] = function() {
                   return result(this, method.apply(this._wrapped, arguments));
              };
         });
         // 一个包装过(OOP)并且链式调用的对象
         // 用 value 方法获取结果
         _.prototype.value = function() {
              return this._wrapped;
         };

         _.prototype.valueOf = _.prototype.toJSON = _.prototype.value;

         _.prototype.toString = function() {
              return '' + this._wrapped;
         };

    }.call(this))

将this（浏览器环境中其实就是window对象）传入 匿名自执行 函数，并赋值给 root。  
创建 \_ 函数，将其挂到 root 即全局作用域下，故可以全局范围内使用 \_ 。  
然后在 \_ 上定义了工具函数\(函数即对象，可以在对象上添加方法\)，像 \_.each, \_.map 等。  
这样就可以全局使用函数式编程风格的方式 调用 \_ 上的方法了， \_.each, \_.map等。

但同时 underscore 也做了针对面向对象风格的调用方式的兼容。需要通过\_\(\)来包裹一下对象，调用例子：

```
 var arr = [1, 2, 3];
 _(arr).map(function(item) {
      return item * 2;
 });
```

看一下 \_ 函数内部： 

    var _ = function(obj) {
         // 以下均针对 OOP 形式的调用
         // 如果是非 OOP 形式的调用，不会进入该函数内部
         // 如果 obj 已经是 `_` 函数的实例，则直接返回 obj
         if (obj instanceof _) return obj;

         // 如果不是 `_` 函数的实例
         // 则调用 new 运算符，返回实例化的对象
         if (! (this instanceof _)) return new _(obj);

         // 将 obj 赋值给 this._wrapped 属性
         this._wrapped = obj;
    };

如果你采用的是函数式编程风格 调用的话， 不传 obj ，所以不会进入两个 if 判断而直接执行最后一句 this.\_wrapped = obj;。  
如果你采用的是面向对象的编程风格 调用的话，如果 obj 已经是 \_ 函数的实例，则直接返回 obj，如果不是 \_ 函数的实例， 则调用 new 运算符，返回实例化的对象。

那么 面向对象风格的调用方式，是如何拥有所有定义的方法的呢？  
从 代码中的 \_.mixin 函数开始，就是为了兼容 这一种调用。  
定义了一个 \_.mixin 方法 ，并在之后立即执行了，传入了 \_ 作为参数。

```
_.mixin = function(obj) {
    // 遍历 obj 的 key，将方法挂载到 Underscore 上
    // 其实是将方法浅拷贝到 _.prototype 上
    _.each(_.functions(obj),
    function(name) {
        // 直接把方法挂载到 _[name] 上
        // 调用类似 _.myFunc([1, 2, 3], ..)
        var func = _[name] = obj[name];

        // 浅拷贝
        // 将 name 方法挂载到 _ 对象的原型链上，使之能 OOP 调用
        _.prototype[name] = function() {
            // 第一个参数
            var args = [this._wrapped];

            // arguments 为 name 方法需要的其他参数
            push.apply(args, arguments);
            // 执行 func 方法
            // 支持链式操作
            return result(this, func.apply(_, args));
        };
    });
};

// Add all of the Underscore functions to the wrapper object.
// 将前面定义的 underscore 方法添加给包装过的对象
// 即添加到 _.prototype 中
// 使 underscore 支持面向对象形式的调用
_.mixin(_);

// Add all mutator Array functions to the wrapper.
// 将 Array 原型链上有的方法都添加到 underscore 中
_.each(['pop', 'push', 'reverse', 'shift', 'sort', 'splice', 'unshift'],
function(name) {
    var method = ArrayProto[name];
    _.prototype[name] = function() {
        var obj = this._wrapped;
        method.apply(obj, arguments);

        if ((name === 'shift' || name === 'splice') && obj.length === 0) delete obj[0];

        // 支持链式操作
        return result(this, obj);
    };
});

// Add all accessor Array functions to the wrapper.
// 添加 concat、join、slice 等数组原生方法给 Underscore
_.each(['concat', 'join', 'slice'],
function(name) {
    var method = ArrayProto[name];
    _.prototype[name] = function() {
        return result(this, method.apply(this._wrapped, arguments));
    };
});
// 一个包装过(OOP)并且链式调用的对象
// 用 value 方法获取结果
_.prototype.value = function() {
    return this._wrapped;
};

_.prototype.valueOf = _.prototype.toJSON = _.prototype.value;

_.prototype.toString = function() {
    return '' + this._wrapped;
};
```

\_.mixin 函数中将遍历\_的属性，如果某个属性的类型是function，就把该函数挂载到 \_ 原型链上，这样对于 \_ 函数的实例自然就可以调用 \_ 原型链上的方法。

  
对于函数式编程，其实是另一种编程思想，它相较于大家所熟知的面向对象的编程风格来说 ，应该是各有好处。推荐大家看 underscore源码 和 书 《Javascript函数式编程》 深入了解。





------------

学习并感谢：

http://www.jianshu.com/p/e602ce36b6f7

https://yoyoyohamapi.gitbooks.io/undersercore-analysis/content/base/%E7%BB%93%E6%9E%84.html

