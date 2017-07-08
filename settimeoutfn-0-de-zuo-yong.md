在 zepto 源码中，$.fn 对象 有个 ready 函数，其中有这样一句 setTimeout\(fn,0\);

```
$.fn = {
    ready: function(callback){
      // don't use "interactive" on IE <= 10 (it can fired premature)
      //
      // document.readyState：当document文档正在加载时,返回"loading"。当文档结束渲染但在加载内嵌资源时，返回"interactive"，并引发DOMContentLoaded事件。当文档加载完成时,返回"complete"，并引发load事件。
      // document.documentElement.doScroll：IE有个特有的方法doScroll可以检测DOM是否加载完成。 当页面未加载完成时，该方法会报错，直到doScroll不再报错时，就代表DOM加载完成了
      //
      // 关于 setTimeout(fn ,0) 的作用 可以参考文章：http://www.cnblogs.com/silin6/p/4333999.html
      if (document.readyState === "complete" ||
          (document.readyState !== "loading" && !document.documentElement.doScroll))
        setTimeout(function(){ callback($) }, 0)
      else {
        // 监听移除事件
        var handler = function() {
          document.removeEventListener("DOMContentLoaded", handler, false)
          window.removeEventListener("load", handler, false)
          callback($)
        }
        document.addEventListener("DOMContentLoaded", handler, false)
        window.addEventListener("load", handler, false)
      }
      return this
    },
}
```

时间设为 0 ，就是要立即执行，那为什么还要特意将 fn 套到 setTimeout 里面呢？



一、线程

1、浏览器的内核是多线程的，它们在内核控制下相互配合以保持同步，一个浏览器通常由以下常驻线程组成：GUI 渲染线程，javascript 引擎线程，浏览器事件触发线程，定时触发器线程，异步 http 请求线程。

* **GUI 渲染线程**：负责渲染浏览器界面 HTML 元素,当界面需要重绘\(Repaint\)或由于某种操作引发回流\(reflow\)时,该线程就会执行。在 Javascript 引擎运行脚本期间, GUI 渲染线程都是处于挂起状态的,也就是说被”冻结”。即 GUI 渲染线程与 JS 引擎是互斥的，当JS引擎执行时GUI线程会被挂起，GUI 更新会被保存在一个队列中等到 JS 引擎空闲时立即被执行。

* **javascript 引擎线程**：也可以称为 JS 内核，主要负责处理 Javascript 脚本程序，例如 V8 引擎。Javascript 引擎线程理所当然是负责解析 Javascript 脚本，运行代码。浏览器无论什么时候都只有一个 JS 线程在运行 JS 程序。

* **浏览器事件触发线程**：当一个事件被触发时该线程会把事件添加到待处理队列的队尾，等待 JS 引擎的处理。这些事件可以是当前执行的代码块如定时任务、也可来自浏览器内核的其他线程如鼠标点击、AJAX 异步请求等，但由于JS的单线程关系所有这些事件都得排队等待 JS 引擎处理。

* **定时触发器线程**：浏览器定时计数器并不是由 JavaScript 引擎计数的, 因为 javaScript 引擎是单线程的, 如果处于阻塞线程状态就会影响记计时的准确, 因此通过单独线程来计时并触发定时是更为合理的方案。

* **异步 http 请求线程**：在 XMLHttpRequest 在连接后是通过浏览器新开一个线程请求， 将检测到状态变更时，如果设置有回调函数，异步线程就产生状态变更事件放到 JavaScript 引擎的处理队列中等待处理。

举个例子，看看这些线程如何配合工作的：

例子1:异步请求是由线程 JavaScript 执行线程、HTTP 请求线程 和 事件触发线程 共同完成的。JavaScript 执行线程 执行异步请求代码，这时浏览器会开一条新的 HTTP 请求线程 来执行请求，JavaScript 执行线程则继续执行 执行队列 中剩下的其他任务。然后在未来的某一时刻 事件触发线程 监视到之前的发起的 HTTP 请求已完成，它就会把完成事件的回调代码插入到 JavaScript 执行队列尾部 等待 JavaScript 执行线程空闲时来处理。

例子2:定时触发（setTimeout 和 setInterval）是由浏览器的 定时器线程 执行的定时计数，然后在定时时间结束时把定时处理函数的执行代码插入到 JavaScript 执行队列的尾端（所以用这两个函数的时候，实际的执行时间是大于或等于指定时间的，不保证能准确定时的）。



2、javascript 是单线程的，同一个时间只能做一件事。

这里说一下 js调用栈（call stack），可以从根本上理解单线程的执行过程。  
推荐一个神器网站：[http://latentflip.com/loupe/](http://latentflip.com/loupe/%20)可以用来图形化调用栈的过程，大家可以把例子在网站上运行一下，好用到疯掉。



js 调用栈（call stack）：函数被调用时，就会被加入到调用栈顶部，执行结束之后，就会从调用栈顶部移除该函数，这种数据结构的关键在于后进先出，即 LIFO（last-in，first-out）。



举个例子：来自（[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)） 

![](/assets/8import.png)

```
function f(b) {
    var a = 12;
    return a + b + 35;
}
function g(x) {
    var m = 4;
    return f(m * x);
}
g(21);
```

调用 g 函数 的时候，创建了第一个 堆\( Heap \) 栈\(stack\) 帧 ，包含了 g 的参数和局部变量。当 g 调用 f 的时候，第二个 堆栈帧 就被创建、并置于第一个 堆栈帧 之上，包含了 f 的参数和局部变量。当 f 返回时，最上层的 堆栈帧 就出栈了（剩下 g 函数调用的 堆栈帧 ）。当 g 返回的时候，栈就空了。



再举个例子:

```
function test() {
    setTimeout(function() {
        alert(1)
    },1000);
    alert(2);
}
test();
```

在执行函数 test 的时候，test 先入栈，如果不给 alert\(1\)加 setTimeout，那么 alert\(1\)第 2 个入栈，最后是 alert\(2\)。但现在给 alert\(1\)加上 setTimeout 后，alert\(1\)就被加入到了一个新的堆栈中等待，并1s后执行，因此实际的执行结果就是先 alert\(2\)，再 alert\(1\)。



3、任务队列（消息队列）：

* 函数分为两种：同步和异步。

　　　　同步函数：如果在函数A返回的时候，调用者就能够得到预期结果\(即拿到了预期的返回值或者看到了预期的效果\)，那么这个函数就是同步的。

　　　　例子：

```
console.log('Hi’);   //函数返回时，就看到了预期的效果：在控制台打印了一个字符串
```

                异步函数即如果在函数A返回的时候，调用者还不能够得到预期结果，而是需要在将来通过一定的手段得到，那么这个函数就是异步的。

　　　　例子：

```
setTimeout(fn, 1000);//setTimeout是异步过程的发起函数，fn是回调函数。
```

* 任务也分为两种：同步任务和异步任务。

　　　　同步任务：在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务。  
　　　　异步任务：主线程发起一个异步请求（即执行异步函数），相应的工作线程（浏览器事件触发线程、异步http请求线程等）接收请求并告知主线程已收到\(异步函数返回\)；主线程可以继续执行后面的代码，同时工作线程执行异步任务；工作线程完成工作后，将完成消息放到任务（消息）队列，主线程通过事件循环过程去取任务（消息），然后执行一定的动作\(调用回调函数\)。

　　　　图中主线程即 Stack，任务队列即 Queue。

![](/assets/23import.png)

* 任务队列：任务（消息）队列是一个先进先出的队列，它里面存放着各种任务（消息）。

* 事件循环（event loop）：事件循环是指主线程重复从任务（消息）队列中取任务（消息）、执行的过程。取一个任务（消息）并执行的过程叫做一次循环。

　　　　事件循环中有事件两个字的原因：任务（消息）队列中的每条消息实际上都对应着一个事件——dom事件。  
　　　　例子：

```
 var button = document.getElement('#btn');
 button.addEventListener('click',function(e) {
     console.log();
 })；
```



