underscore的源码中，有很多地方用到了 Array.prototype.slice\(\) 方法，但是并没有传参，实际上只是为了返回数组的副本，例如 underscore 中 clone 的方法：

      // Create a (shallow-cloned) duplicate of an object.
      // 对象的 `浅复制` 副本
      // 注意点：所有嵌套的对象或者数组都会跟原对象用同一个引用
      // 所以是为浅复制，而不是深度克隆
      _.clone = function(obj) {
        // 容错，如果不是对象或者数组类型，则可以直接返回
        // 因为一些基础类型是直接按值传递的
        if (!_.isObject(obj))
          return obj;

        // 如果是数组，则用 obj.slice() 返回数组副本
        // 如果是对象，则提取所有 obj 的键值对覆盖空对象，返回
        return _.isArray(obj) ? obj.slice() : _.extend({}, obj);
      };

这里就涉及到了一个知识点：深浅拷贝。

所谓深浅拷贝，都是进行复制，那么区别主要在于复制出来的新对象和原来的对象是否会互相影响，改一个，另一个也会变。

浅拷贝栗子：

```
var a = ["a","b","c"];
var a_copy = a;
console.log(a === a_slice);
a_slice[0]="f";
console.log(a_slice);
console.log(a);
```

![](/assets/import2.png)深拷贝栗子：

```
var obj = {name:"xixi",age:20};
var obj_extend = $.extend(true,{}, obj); //extend方法，第一个参数为true，为深拷贝，为false，或者没有为浅拷贝。
console.log(obj === obj_extend);
obj.name = "heihei";
console.log(obj);
console.log(obj_extend);
```

![](/assets/import4.png)有了上面的大概认识，让我们从原理深入了解一下深浅拷贝。

**一、基本类型 和 引用类型**

1、ECMAScript 中的变量类型分为两类：

* 基本类型：undefined,null,布尔值\(Boolean\),字符串\(String\),数值\(Number\)
* 引用类型:
  统称为Object类型，
  细分的话，有：Object类型，Array类型，Date类型，Function类型等。

2、不同类型的存储方式：

* **基本数据类型 **保存在 **栈内存，**形式如下：栈内存中分别存储着变量的标识符以及变量的值。

![](/assets/import5.png)

即

```
var a  = “A”;
```

在栈内存中是这样的

![](/assets/import6.png)

* **引用类型 **保存在 **堆内存 **中**，**栈内存存储的是变量的标识符以及对象在堆内存中的存储地址，当需要访问引用类型（如对象，数组等）的值时，首先从栈中获得该对象的地址指针，然后再从对应的堆内存中取得所需的数据。 

![](/assets/import7.png)即

```
var a = {name:“jack”};
```

在内存中是这样的![](/assets/impor8.png)3、不同类型的复制方式：

**基本类型 **的复制：当你在复制基本类型的时候，相当于把值也一并复制给了新的变量。

栗子 1:

```
var a = 1;
var b = a;
console.log(a === b);
var a = 2;
console.log(a);
console.log(b);
```

![](/assets/i1mport.png)

改变 a 变量的值，并不会影响 b 的值。

内存中是这样的：

```
var a = 1;
```

![](/assets/import12.png)

```
var b = a;
```

![](/assets/impor13t.png)

```
a = 2;
```

![](/assets/import14.png)

**引用类型 **的复制：当你在复制引用类型的时候，实际上只是复制了指向堆内存的地址，即原来的变量与复制的新变量指向了同一个东西。

栗子 2：

```
var a = {name:”jack",age:20};
var b = a;
console.log(a === b);
a.age = 30;
console.log(a);
console.log(b);
```

![](/assets/import15.png)改变 a 变量的值，会影响 b 的值。

内存中是这样的：

```
var a = {name:“jack",age:20};
```

![](/assets/impor17t.png)

```
var b = a;
```

![](/assets/impo1rt.png)

```
a.age = 30;
```

![](/assets/impor12t.png)二、明白了上面之后，所谓

**深浅拷贝**：对于仅仅是复制了引用（地址），换句话说，复制了之后，原来的变量和新的变量指向同一个东西，

彼此之间的操作会互相影响，为 **浅拷贝**。

而如果是在堆中重新分配内存，拥有不同的地址，但是值是一样的，复制后的对象与原来的对象是完全隔离，互不影响，为 **深拷贝**。

**深浅拷贝 **的主要区别就是：复制的是引用\(地址\)还是复制的是实例。

所以上面的栗子2，如何可以变成深拷贝呢？

我们可以想象出让 b 在内存中像下图这样，肯定就是深拷贝了。

![](/assets/impor16t.png)那么代码上如何实现呢？

利用 **递归** 来实现深复制，**对属性中所有引用类型的值，遍历到是基本类型的值为止。**

```
function deepClone(source){    
  if(!source && typeof source !== 'object'){      
    throw new Error('error arguments', 'shallowClone');    
  }    
  var targetObj = source.constructor === Array ? [] : {};    
  for(var keys in source){       
    if(source.hasOwnProperty(keys)){          
      if(source[keys] && typeof source[keys] === 'object'){           
        targetObj[keys] = source[keys].constructor === Array ? [] : {}; 
        targetObj[keys] = deepClone(source[keys]);    //递归      
      }else{            
        targetObj[keys] = source[keys];         
      }       
    }    
  }    
  return targetObj; 
}
```

检测一下

```
var a = {name:"jack",age:20};
var b = deepClone(a);
console.log(a === b);
a.age = 30;
console.log(a);
console.log(b);
```

![](/assets/12import.png)

三 、最后让我们来看看 一些 js 中的 复制方法，他们到底是深拷贝还是浅拷贝？

1、Array 的 slice 和 concat 方法

两者都会返回一个新的数组实例。

栗子

```
var a = [1,2,3];
var b = a.slice(); //slice
console.log(b === a);
a[0] = 4;
console.log(a);
console.log(b);
```

![](/assets/1import.png)

```
var a = [1,2,3];
var b = a.concat();  //concat
console.log(b === a);
a[0] = 4;
console.log(a);
console.log(b);
```

![](/assets/2import.png)

看到结果，如果你觉得，这两个方法是深复制，那就恭喜你跳进了坑里 

让咱们再看一个颠覆你观念的栗子：

```
var a = [[1,2,3],4,5];
var b = a.slice();
console.log(a === b);
a[0][0] = 6;
console.log(a);
console.log(b);
```

![](/assets/3import.png)

看见了吗？都变啦！！！！

这就是坑，知道吗？？？？

所以你要记住的是 Array的 slice 和 concat 方法 并不是 **真正的深拷贝**，他们其实是披着羊（**qian**）皮（**kao**）的（**bei**）狼。



2、jQuery中的 extend 复制方法

可以用来扩展对象，这个方法可以传入一个参数:deep\(true or false\)，表示是否执行深复制\(如果是深复制则会执行递归复制\)。

栗子：

深拷贝：

```
var obj = {name:'xixi',age:20,company : { name : '腾讯', address : '深圳'} };
var obj_extend = $.extend(true,{}, obj); //extend方法，第一个参数为true，为深拷贝，为false，或者没有为浅拷贝。
console.log(obj === obj_extend);
obj.company.name = "ali";
obj.name = "hei";
console.log(obj);
console.log(obj_extend);
```

![](/assets/33import.png)

浅拷贝：

```
var obj = {name:"xixi",age:20};
var obj_extend = $.extend(false,{}, obj); //extend方法，第一个参数为true，为深拷贝，为false，或者没有为浅拷贝。
console.log(obj === obj_extend);
obj.name = "heihei";
console.log(obj);
console.log(obj_extend);
```

![](/assets/4import.png)

咦，company 的变化 可以看出 深浅复制来（即箭头所指）， 红色方框圈出的地方，怎么和上面 slice 和 concat 的情况一样？难道也是羊？



**其实总结一下就是：**Array 的 slice 和 concat 方法 和 jQuery 中的 extend 复制方法，他们都会复制第一层的值，对于 **第一层 **的值都是 **深拷贝**，而到 **第二层 **的时候 Array 的 slice 和 concat 方法就是 **复制引用 **，jQuery 中的 extend 复制方法 则 **取决于 你的 第一个参数**， 也就是是否进行递归复制。所谓第一层 就是 key 所对应的 value 值是基本数据类型，也就像上面栗子中的name、age，而对于 value 值是引用类型 则为第二层，也就像上面栗子中的 company。



3、JSON 对象的 parse 和 stringify

JOSN 对象中的 stringify 可以把一个 js 对象序列化为一个 JSON 字符串，parse 可以把 JSON 字符串反序列化为一个 js 对象，这两个方法实现的是深拷贝。

栗子：

```
var obj = {name:'xixi',age:20,company : { name : '腾讯', address : '深圳'} };
var obj_json = JSON.parse(JSON.stringify(obj));
console.log(obj === obj_json);
obj.company.name = "ali";
obj.name = "hei";
console.log(obj);
console.log(obj_json);
```

![](/assets/7import.png)

完全的 深拷贝。

  
----------

 参考并感谢：

http://lijundong.com/deep-clone-vs-shallow-clone/

http://www.jianshu.com/p/d9bef2e83163









