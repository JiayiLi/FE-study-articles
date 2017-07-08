这是一个非常有意思的问题。

在看源码的过程中，总会遇到这样的写法：

```
var triggerEvents = function(events, args) {
    var ev, i = -1, l = events.length, a1 = args[0], a2 = args[1], a3 = args[2];
    switch (args.length) {
      case 0: while (++i < l) (ev = events[i]).callback.call(ev.ctx); return;
      case 1: while (++i < l) (ev = events[i]).callback.call(ev.ctx, a1); return;
      case 2: while (++i < l) (ev = events[i]).callback.call(ev.ctx, a1, a2); return;
      case 3: while (++i < l) (ev = events[i]).callback.call(ev.ctx, a1, a2, a3); return;
      default: while (++i < l) (ev = events[i]).callback.apply(ev.ctx, args); return;
    }
};
```

（ 代码来自 backbone ）

作者会在参数为3个（包含3）以内时，优先使用 call 方法进行事件的处理。而当参数过多（多余3个）时，才考虑使用 apply 方法。

这个的原因就是 call 比 apply 快。

网上有很多例子全方位的证明了 call 比 apply 快。大家可以看看 [call和apply的性能对比](https://github.com/coderwin/__/issues/6) 。这篇文章中的例子，很全面。或者你也可以自己写几个简单的，测试一下。这里要推荐一个神奇网站  [jsperf](https://jsperf.com/) ，用于测试 js 性能。

几个简单的例子：![](/assets/import.png)![](/assets/import1.png)

为什么call 比apply 快？

这里就要提到他们被调用之后发生了什么。

Function.prototype.apply \(thisArg, argArray\)

1、如果IsCallable（Function）为false，即Function不可以被调用，则抛出一个TypeError异常。  
2、如果argArray为null或未定义，则返回调用function的\[\[Call\]\]内部方法的结果，提供thisArg和一个空数组作为参数。  
3、如果 Type（argArray）不是Object，则抛出TypeError异常。  
4、获取argArray的长度。调用argArray的\[\[Get\]\]内部方法，找到属性length。 赋值给len。  
5、定义 n 为ToUint32（len）。ToUint32（len）方法：将其参数len转换为范围为0到2^32-1的2^32个整数值中的一个。  
6、初始化 argList 为一个空列表。  
7、初始化 index 为 0。  
8、循环迭代取出argArray。重复循环 while（index &lt; n）  
            a、将下标转换成String类型。初始化 indexName 为 ToString\(index\).  
            b、定义 nextArg 为 使用 indexName 作为参数调用argArray的\[\[Get\]\]内部方法的结果。  
            c、将 nextArg 添加到 argList 中，作为最后一个元素。  
            d、设置 index ＝ index＋1  
9、返回调用func的\[\[Call\]\]内部方法的结果，提供thisArg作为该值，argList作为参数列表。

Function.prototype.call \(thisArg \[ , arg1 \[ , arg2, … \] \] \)

1、如果 IsCallable（Function）为false，即Function不可以被调用，则抛出一个TypeError异常。  
2、定义argList 为一个空列表。  
3、如果使用超过一个参数调用此方法，则以从arg1开始的从左到右的顺序将每个参数附加为argList的最后一个元素  
4、返回调用func的\[\[Call\]\]内部方法的结果，提供thisArg作为该值，argList作为参数列表。

我们可以看到，明显apply比call的步骤多很多。  
由于apply中定义的参数格式（数组），使得被调用之后需要做更多的事，需要将给定的参数格式改变（步骤8）。 同时也有一些对参数的检查（步骤2），在call中却是不必要的。  
另外一个很重要的点：在apply中不管有多少个参数，都会执行循环，也就是步骤6-8，在call中也就是对应步骤3 ，是有需要才会被执行。

综上，call 方法比 apply 快的原因是 call 方法的参数格式正是内部方法所需要的格式。



---------

参考并学习：

[http://es5.github.io/\#x15.3.4.3](http://es5.github.io/#x15.3.4.3)

[https://stackoverflow.com/questions/23769556/why-is-call-so-much-faster-than-apply](https://stackoverflow.com/questions/23769556/why-is-call-so-much-faster-than-apply)

