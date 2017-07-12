方法一：双循环   （时间复杂度比较高，性能一般。）
A、(1)
```javascript
function unique(arr) {
    var newArr = [];
    var len = arr.length;
    var isRepeat;
  
    for(var i=0; i<len; i++) {  //第一次循环
        isRepeat = false;
        for(var j=i+1; j<len; j++) {  //第二次循环
            if(arr[i] === arr[j]){
                isRepeat = true;
                break;
            }
        }
        if(!isRepeat){
            newArr.push(arr[i]);
        }
    }
    return newArr;
}
```
输出 newArr 结果：![Alt text](/assets/qu1.png)

B、(2)
```javascript
function unique(arr) {
    var newArr = [];
    var len = arr.length;
    for(var i=0; i<len; i++){ // 第一次循环
        for(var j=i+1; j<len; j++){ // 第二次循环
            if(arr[i] === arr[j]){
                j = ++i;
            }
        }
        newArr.push(arr[i]);
    }
    return newArr;
}
```
输出 newArr 结果：![Alt text](/assets/qu2.png)
tip:  j = ++ i; 等价于 j = j+1; i = i+1;  
整体思路就是 如果是重复元素，则跳过重复元素，不对其进行 push 操作。

方法二：Array.prototype.indexOf() 
A、(3)
```javascript
function unique(arr) {
    return arr.filter(function(item, index){ //item 表示数组中的每个元素，index 是每个元素的出现位置。
       return arr.indexOf(item) === index; // indexOf 返回第一个索引值
   });
}
```
输出 新 arr 结果：![Alt text](/assets/qu3.png)
tip：var new_arrary = arr.filter(callback[, thisArg]);  其中 callback 用来测试数组的每个元素的函数。调用时使用参数 element, index, array。
返回true表示保留该元素（通过测试），false则不保留。thisArg可选，执行 callback 时的用于 this 的值。
整体思路就是索引不是第一个索引，说明是重复值。

B、(4)
```javascript
function unique(arr) {
    var newArr = [];
    arr.forEach(function(item){ //一次循环，item 即为数组中的每一项
        if(newArr.indexOf(item) === -1){
            newArr.push(item);
        }
    });
    return newArr;
}
```
输出 newArr 结果：![Alt text](/assets/qu4.png)

方法三：Array.prototype.sort() 
A、(5)
```javascript
function unique(arr) {
    var newArr = [];                  
    arr.sort();
    for(var i = 0; i < arr.length; i++){
        if( arr[i] !== arr[i+1]){
            newArr.push(arr[i]);
        }
    }
    return newArr;
}
```
输出 newArr 结果：![Alt text](/assets/qu5.png)
tip: 整体思路就是 先进行排序，然后比较相邻元素。

B、(6)
```javascript
function unique(arr) {
  var newArr = [];                  
  arr.sort();
  var newArr = [arr[0]];
  for(var i = 1; i < arr.length; i++){
    if(arr[i] !== newArr[newArr.length - 1]){
     newArr.push(arr[i]);
    }
  }    
  return newArr;
}
```
输出 newArr 结果：![Alt text](/assets/qu6.png)
tip：整体思路就是 先进行排序，将原数组中的第一个元素复制给结果数组，然后检查原数组中的第 i 个元素 与 结果数组中的最后一个元素是否相同。

方法四：使用对象key来去重 (7)
```javascript
function unique(arr) {
    var newArr = [];
    var len = arr.length;
    var tmp = {};
    for(var i=0; i<len; i++){
        if(!tmp[arr[i]]){
            tmp[arr[i]] = 1;
            newArr.push(arr[i]);
        }
    }
    return newArr;
}
```
输出 newArr 结果：![Alt text](/assets/qu7.png)
tip：整体思路就是 利用了对象（tmp）的 key 不可以重复的特性来进行去重。 但是会出现如下问题需要注意：
1. 无法区分隐式类型转换成字符串后一样的值，比如 1 和 '1' 。
2. 无法处理复杂数据类型，比如对象（因为对象作为key会变成[object Object]）。
3. 特殊数据，比如 '__proto__' 会挂掉，因为 tmp 对象的 __proto__ 属性无法被重写。

 
针对以上问题，解决办法有：
解决问题一、三：可以为对象的 key 增加一个类型，或者将类型放到对象的value中来解决：(8)
```javascript
function unique(arr) {
    var newArr = [];
    var len = arr.length;
    var tmp = {};
    var tmpKey;
    for(var i=0; i<len; i++){
        tmpKey = typeof arr[i] + arr[i];

        console.log(tmpKey); 

        if(!tmp[tmpKey]){
            tmp[tmpKey] = 1;
            newArr.push(arr[i]);
        }
    }
    return newArr;
}
```
 输出 newArr 结果：![Alt text](/assets/qu8.png)
 tip： 代码中第 9 行 console 出来的东西 如图：
![Alt text](/assets/qu9.png)


解决问题二：可以将对象序列化之后作为key来使用。这里为简单起见，使用JSON.stringify()进行序列化。(9)
```javascript
function unique(arr) {
    var newArr = [];
    var len = arr.length;
    var tmp = {};
    var tmpKey;
    for(var i=0; i<len; i++){
        tmpKey = typeof arr[i] + JSON.stringify(arr[i]);
      
        console.log(tmpKey)
        
        if(!tmp[tmpKey]){
            tmp[tmpKey] = 1;
            newArr.push(arr[i]);
        }
    }
    return newArr;
}
```
 输出 newArr 结果：![Alt text](/assets/qu10.png)
 tip： 代码中第 9 行 console 出来的东西 如图：
![Alt text](/assets/qu11.png)
看起来和上面一种方法 console 的没有区别，但是将 测试用例 换成一个 对象  var arr  = [{xiaoming:23,xiaoqing:45},{xiaoming:24,xiaoqing:45}]; 时，可以看到 console 结果如下：![Alt text](/assets/qu12.png)

</br>
以上都是一些比较普遍的解决办法，下面补充一下 es6 中的方法以及对于 null,NaN,undefined,{} 等类型的去重。
一、es6:
方法一： Map (10)
```javascript
function unique(arr) {
    var newArr = [];
    var len = arr.length;
    var tmp = new Map();
    for(var i=0; i<len; i++){
        if(!tmp.get(arr[i])){
            tmp.set(arr[i], 1);
            newArr.push(arr[i]);
        }
    }
    return newArr;
}
```
输出 newArr 结果：![Alt text](/assets/qu13.png)
tip: Map 是一种新的数据类型，类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。可以把它想象成key类型没有限制的对象。它的存取使用单独的get()、set()接口。 

方法二：Set (11)
```javascript
function unique(arr){
    var set = new Set(arr);
    return Array.from(set);
}
```
输出 newArr 结果：![Alt text](/assets/qu14.png)
tip：Set 是一种新的数据结构。它类似于数组，但是成员的值都是唯一的，没有重复的值。Array.from() 方法从类似数组或可迭代对象创建一个新的数组实例。

想要具体了解 Set 和 Map ，查看 http://wiki.jikexueyuan.com/project/es6/set-map.html

方法三：Array.prototype.includes() (12)
```javascript
function unique(arr) {
    var newArr = [];
    arr.forEach(function(item){
        if(!newArr.includes(item)){
            newArr.push(item);
        }
    });
    return newArr;
}
```
输出 newArr 结果：![Alt text](/assets/qu15.png)

二、NaN 等类型数据的去重：
使用测试用例，对上面所有算法进行验证 (以 黄色数字 为测试顺序)
```javascript
var arr = [1,1,'1','1',0,0,'0','0',undefined,undefined,null,null,NaN,NaN,{},{},[],[],/a/,/a/]
 
console.log(unique(arr));
```
得到结果如下：
![Alt text](/assets/qu16.png)
![Alt text](/assets/qu17.png)
![Alt text](/assets/qu18.png)
![Alt text](/assets/qu19.png)

tips：
![Alt text](/assets/qu20.png)
