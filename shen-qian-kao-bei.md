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

* **基本数据类型 **保存在 **栈内存，**形式如下：栈内存中分别存储着变量的标识符以及变量的值。

![](/assets/import5.png)

即 

```
var a  = “A”;
```

在栈内存中是这样的

![](/assets/import6.png)



* **引用类型 **保存在 **堆内存 **中**，**栈内存存储的是变量的标识符以及对象在堆内存中的存储地址，当需要访问引用类型（如对象，数组等）的值时，首先从栈中获得该对象的地址指针，然后再从对应的堆内存中取得所需的数据。 

![](/assets/import7.png)即

```
var a = {name:“jack”};
```

在内存中是这样的![](/assets/impor8.png)3、不同类型的复制方式：



**基本类型 **

的复制：当你在复制基本类型的时候，相当于把值也一并复制给了新的变量。

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

**引用类型 **的复制：当你在复制引用类型的时候，实际上只是复制了指向堆内存的地址，即原来的变量与复制的新变量指向了同一个东西。









