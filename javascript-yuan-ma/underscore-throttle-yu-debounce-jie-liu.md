在看 underscore.js 源码的时候，接触到了这样两个方法，很有意思：

我先把实现的代码撂在下面，看不懂的可以先跳过，但是跳过可不是永远跳过哦～

一个是 throttle：

```
_.throttle = function(func, wait, options) {
    var context, args, result;

    // setTimeout 的 handler
    var timeout = null;

    // 标记时间戳
    // 上一次执行回调的时间戳
    var previous = 0;

    // 如果没有传入 options 参数
    // 则将 options 参数置为空对象
    if (!options)
      options = {};

    var later = function() {
      // 如果 options.leading === false
      // 则每次触发回调后将 previous 置为 0
      // 否则置为当前时间戳
      previous = options.leading === false ? 0 : _.now();
      timeout = null;
      result = func.apply(context, args);
      // 这里的 timeout 变量一定是 null 了吧
      // 是否没有必要进行判断？
      if (!timeout)
        context = args = null;
    };

    // 以滚轮事件为例（scroll）
    // 每次触发滚轮事件即执行这个返回的方法
    // _.throttle 方法返回的函数
    return function() {
      // 记录当前时间戳
      var now = _.now();

      // 第一次执行回调（此时 previous 为 0，之后 previous 值为上一次时间戳）
      // 并且如果程序设定第一个回调不是立即执行的（options.leading === false）
      // 则将 previous 值（表示上次执行的时间戳）设为 now 的时间戳（第一次触发时）
      // 表示刚执行过，这次就不用执行了
      if (!previous && options.leading === false)
        previous = now;

      // 距离下次触发 func 还需要等待的时间
      var remaining = wait - (now - previous);
      context = this;
      args = arguments;

      // 要么是到了间隔时间了，随即触发方法（remaining <= 0）
      // 要么是没有传入 {leading: false}，且第一次触发回调，即立即触发
      // 此时 previous 为 0，wait - (now - previous) 也满足 <= 0
      // 之后便会把 previous 值迅速置为 now
      // ========= //
      // remaining > wait，表示客户端系统时间被调整过
      // 则马上执行 func 函数
      // @see https://blog.coding.net/blog/the-difference-between-throttle-and-debounce-in-underscorejs
      if (remaining <= 0 || remaining > wait) {
        if (timeout) {
          clearTimeout(timeout);
          // 解除引用，防止内存泄露
          timeout = null;
        }

        // 重置前一次触发的时间戳
        previous = now;

        // 触发方法
        // result 为该方法返回值
        result = func.apply(context, args);
        // 引用置为空，防止内存泄露
        // 感觉这里的 timeout 肯定是 null 啊？这个 if 判断没必要吧？
        if (!timeout)
          context = args = null;
      } else if (!timeout && options.trailing !== false) { // 最后一次需要触发的情况
        // 如果已经存在一个定时器，则不会进入该 if 分支
        // 如果 {trailing: false}，即最后一次不需要触发了，也不会进入这个分支
        // 间隔 remaining milliseconds 后触发 later 方法
        timeout = setTimeout(later, remaining);
      }

      // 回调返回值
      return result;
    };
  };
```

一个是debounce：

```
_.debounce = function(func, wait, immediate) {
    var timeout, args, context, timestamp, result;

    var later = function() {
      // 定时器设置的回调 later 方法的触发时间，和连续事件触发的最后一次时间戳的间隔
      // 如果间隔为 wait（或者刚好大于 wait），则触发事件
      var last = _.now() - timestamp;

      // 时间间隔 last 在 [0, wait) 中
      // 还没到触发的点，则继续设置定时器
      // last 值应该不会小于 0 吧？
      if (last < wait && last >= 0) {
        timeout = setTimeout(later, wait - last);
      } else {
        // 到了可以触发的时间点
        timeout = null;
        // 可以触发了
        // 并且不是设置为立即触发的
        // 因为如果是立即触发（callNow），也会进入这个回调中
        // 主要是为了将 timeout 值置为空，使之不影响下次连续事件的触发
        // 如果不是立即执行，随即执行 func 方法
        if (!immediate) {
          // 执行 func 函数
          result = func.apply(context, args);
          // 这里的 timeout 一定是 null 了吧
          // 感觉这个判断多余了
          if (!timeout)
            context = args = null;
        }
      }
    };

    // 嗯，闭包返回的函数，是可以传入参数的
    // 也是 DOM 事件所触发的回调函数
    return function() {
      // 可以指定 this 指向
      context = this;
      args = arguments;

      // 每次触发函数，更新时间戳
      // later 方法中取 last 值时用到该变量
      // 判断距离上次触发事件是否已经过了 wait seconds 了
      // 即我们需要距离最后一次事件触发 wait seconds 后触发这个回调方法
      timestamp = _.now();

      // 立即触发需要满足两个条件
      // immediate 参数为 true，并且 timeout 还没设置
      // immediate 参数为 true 是显而易见的
      // 如果去掉 !timeout 的条件，就会一直触发，而不是触发一次
      // 因为第一次触发后已经设置了 timeout，所以根据 timeout 是否为空可以判断是否是首次触发
      var callNow = immediate && !timeout;

      // 设置 wait seconds 后触发 later 方法
      // 无论是否 callNow（如果是 callNow，也进入 later 方法，去 later 方法中判断是否执行相应回调函数）
      // 在某一段的连续触发中，只会在第一次触发时进入这个 if 分支中
      if (!timeout)
        // 设置了 timeout，所以以后不会进入这个 if 分支了
        timeout = setTimeout(later, wait);

      // 如果是立即触发
      if (callNow) {
        // func 可能是有返回值的
        result = func.apply(context, args);
        // 解除引用
        context = args = null;
      }

      return result;
    };
  };
```



在开发过程中，经常会遇到处理频率很高的事件或着连续的事件，比如像在 window 的 resize/scroll 事件调用某个事件 A 等，如果不进行处理，这个时候伴随着resize就会执行无数个 事件A ,若事件 A 是个很复杂的函数，需要较多的运算执行时间，响应速度跟不上触发频率，往往会出现延迟，导致假死或者卡顿感，并且很多情况下并不是需要“如此”夸张的频繁调用事件 A。

下面 gif 录制了一个频繁调用 事件 A 的例子：

!\[gif\]\(http://g.recordit.co/DDSbAVb57z.gif\)

我们可以想办法来解决这样的问题，一种方式就是规定一个时间间隔 wait ，每隔固定 wait 固定执行一次函数 A，也就是我们要说的 throttle 方法，另一种方式则是在连续事件结束之后开始 过一个时间间隔 wait 执行一次函数 A，也就是我们要说的 debounce方法。

针对例子 resize 来解释（ 假设时间间隔5s ）：

　　　　throttle 方法：在 resize 的过程中，每隔5s执行一次函数 A；

　　　　debounce方法：在 resize 结束之后，过了 5s ，执行一次函数 A，如果没有到 5s ，就又开始 resize ，那么就重新计时，并不执行函数 A；



如果我的这个解释没有理解，那么很有一个很多人用的电梯的例子与生活结合在一起会更加形象，在这里也引用一下：

想象每天上班大厦底下的电梯。把电梯完成一次运送，类比为一次函数的执行和响应。假设电梯有两种运行方法 `throttle`和 `debounce` ，超时设定为15秒，不考虑容量限制。

* `throttle`
   方法的电梯。保证如果电梯第一个人进来后，15秒后准时运送一次，不等待。如果没有人，则待机。
* `debounce`
   方法的电梯。如果电梯里有人进来，等待15秒。如果又人进来，15秒等待重新计时，直到15秒超时，开始运送。



下面就逐一说一下两个方法：

1、 throttle 方法：

　　 Underscore.js 中 针对这个方法 传入三个参数：func 即你要在密集事件内调用的函数，也就是例子中的 事件 A，wait 即时间间隔，而第三个参数 options 可以传两种选项，一种是  {leading: false}，表示你想禁用第一次首先执行函数 A，如果不传，就表示你想上来先调用一次 函数 A，另一种是 {trailing: false}，表示想禁用最后一次执行函数 A ；  
![](/assets/111116import.png)

（此图来自 http://benalman.com/projects/jquery-throttle-debounce-plugin/）

2、debounce 方法

　　Underscore.js 中 针对这个方法 传入三个参数：func 即你要在密集事件内调用的函数，也就是例子中的 事件 A，wait 即时间间隔，而第三个参数 immediate 可以传 true 或者 false。传为 true， debounce 会在 wait 时间间隔的开始调用这个函数 （注：并且在 waite 的时间之内，不会再次调用）。在类似不小心点了提交按钮两下而提交了两次的情况下很有用。

![](/assets/1111116import.png)

（此图来自 http://benalman.com/projects/jquery-throttle-debounce-plugin/，图中 at\_begain 即可看为 immediate ）



两者使用方法类似：

```
// WRONG 错误的调用方法
$(window).on('resize', function() {
   _.throttle(doSomething, 300); 
});

// RIGHT  正确的调用方法
$(window).on('resize', _.throttle(doSomething, 300));
```

最后，两者既然有区别，那么也有针对不同场景下更好的选择方法：



throttle：

 　　1、DOM 元素动态定位，window 对象的 resize 和 scroll 事件，比如：用户在你无限滚动的页面上向下拖动，你需要判断现在距离页面底部多少。如果用户快接近底部时，我们应该发送请求来加载更多内容到页面。

　　 2、如果用户在 30s 内 input 输入非常块，但你想固定每间隔 5s 就进行一次某个事情。

debounce：

　　 1、AutoComplete中的Ajax请求使用的keypress

　　 2、在进行input校验的时候，“你的密码太短”等类似的信息。



其实选择哪个方法主要取决于你是否想确保在固定的时间间隔进行回调。



别忘了再回头研究一下最上方的源码。

  
-------------------

参考并感谢：

[https://segmentfault.com/a/1190000004909376  ](https://segmentfault.com/a/1190000004909376)  里面有例子，大家可以进去试试

[http://benalman.com/projects/jquery-throttle-debounce-plugin/](http://benalman.com/projects/jquery-throttle-debounce-plugin/)

[https://blog.coding.net/blog/the-difference-between-throttle-and-debounce-in-underscorejs](https://blog.coding.net/blog/the-difference-between-throttle-and-debounce-in-underscorejs)

[ http://drupalmotion.com/article/debounce-and-throttle-visual-explanation](http://drupalmotion.com/article/debounce-and-throttle-visual-explanation)

