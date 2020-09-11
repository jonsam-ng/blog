---
title: 彻底搞懂 JS 闭包各种坑
---
闭包是 js 开发惯用的技巧，什么是闭包？**闭包指的是：能够访问另一个函数作用域的变量的函数**。清晰的讲：闭包就是一个函数，这个函数能够访问其他函数的作用域中的变量。eg:

```
function outer() {
     var  a = '变量1'
     var  inner = function () {
            console.info(a)
     }
    return inner    // inner 就是一个闭包函数，因为他能够访问到outer函数的作用域
}


```

很多人会搞不懂匿名函数与闭包的关系，实际上，闭包是站在作用域的角度上来定义的，因为 inner 访问到 outer 作用域的变量，所以 inner 就是一个闭包函数。虽然定义很简单，但是有很多坑点，比如 this 指向、变量的作用域，稍微不注意可能就造成内存泄露。我们先把问题抛一边，思考一个问题：**为什么闭包函数能够访问其他函数的作用域** ?

从堆栈的角度看待 js 函数  
　　基本变量的值一般都是存在栈内存中，而对象类型的变量的值存储在堆内存中，栈内存存储对应空间地址。基本的数据类型: Number 、Boolean、Undefined、String、Null。

```
var  a = 1   //a是一个基本类型
var  b = {m: 20 }   //b是一个对象


```

对应内存存储：

![](http://upload-images.jianshu.io/upload_images/7155532-841a10bf5a08a51b.png)

当我们执行 b={m:30}时，堆内存就有新的对象 {m：30}，栈内存的 b 指向新的空间地址( 指向{m：30} )，而堆内存中原来的{m：20} 就会被程序引擎垃圾回收掉，节约内存空间。我们知道 js 函数也是对象，它也是在堆与栈内存中存储的，我们来看一下转化：

```
var a = 1;
function fn(){
    var b = 2;
    function fn1(){
        console.log(b);
    }
    fn1();
}
fn();


```

![](http://upload-images.jianshu.io/upload_images/7155532-ee4142a5b829d016.png)

**  
栈是一种先进后出的数据结构：  
1 在执行 fn 前，此时我们在全局执行环境 (浏览器就是 window 作用域)，全局作用域里有个变量 a；  
2 进入 fn，此时栈内存就会 push 一个 fn 的执行环境，这个环境里有变量 b 和函数对象 fn1，这里可以访问自身执行环境和全局执行环境所定义的变量  
3 进入 fn1，此时栈内存就会 push 一个 fn1 的执行环境，这里面没有定义其他变量，但是我们可以访问到 fn 和全局执行环境里面的变量，因为程序在访问变量时，是向底层栈一个个找，如果找到全局执行环境里都没有对应变量，则程序抛出 underfined 的错误。

4 随着 fn1() 执行完毕，fn1 的执行环境被杯销毁，接着执行完 fn()，fn 的执行环境也会被销毁，只剩全局的执行环境下，现在没有 b 变量，和 fn1 函数对象了，只有 a 和 fn(函数声明作用域是 window 下)  
**

在函数内访问某个变量是根据函数作用域链来判断变量是否存在的，而函数作用域链是程序根据函数所在的执行环境栈来初始化的，所以上面的例子，我们在 fn1 里面打印变量 b，根据 fn1 的作用域链的找到对应 fn 执行环境下的变量 b。所以**当程序在调用某个函数时，做了一下的工作：准备执行环境，初始函数作用域链和 arguments 参数对象**

我们现在看回最初的例子 outer 与 inner

```
function outer() {
     var  a = '变量1'
     var  inner = function () {
            console.info(a)
     }
    return inner    // inner 就是一个闭包函数，因为他能够访问到outer函数的作用域
}
var  inner = outer()   // 获得inner闭包函数
inner()   //"变量1"


```

当程序执行完 var inner = outer()，其实 outer 的执行环境并没有被销毁，因为他里面的变量 a 仍然被被 inner 的函数作用域链所引用，当程序执行完 inner(), 这时候，inner 和 outer 的执行环境才会被销毁调；《JavaScript 高级编程》书中建议：由于闭包会携带包含它的函数的作用域，因为会比其他函数占用更多内容，过度使用闭包，会导致内存占用过多。

现在我们明白了闭包，已经对应的作用域与作用域链，回归主题：  
**坑点 1： 引用的变量可能发生变化**

```
function outer() {
      var result = [];
      for （var i = 0； i<10; i++）{
        result.[i] = function () {
            console.info(i)
        }
     }
     return result
}


```

看样子 result 每个闭包函数对打印对应数字，1,2,3,4,...,10, 实际不是，因为每个闭包函数访问变量 i 是 outer 执行环境下的变量 i，随着循环的结束，i 已经变成 10 了，所以执行每个闭包函数，结果打印 10， 10， ..., 10  
怎么解决这个问题呢？

```
function outer() {
      var result = [];
      for （var i = 0； i<10; i++）{
        result.[i] = function (num) {
             return function() {
                   console.info(num);    // 此时访问的num，是上层函数执行环境的num，数组有10个函数对象，每个对象的执行环境下的number都不一样
             }
        }(i)
     }
     return result
}


```

**坑点 2: this 指向问题**

```
var object = {
     name: ''object"，
     getName： function() {
        return function() {
             console.info(this.name)
        }
    }
}
object.getName()()    // underfined
// 因为里面的闭包函数是在window作用域下执行的，也就是说，this指向windows


```

**坑点 3：内存泄露问题**

```
function  showId() {
    var el = document.getElementById("app")
    el.onclick = function(){
      aler(el.id)   // 这样会导致闭包引用外层的el，当执行完showId后，el无法释放
    }
}

// 改成下面
function  showId() {
    var el = document.getElementById("app")
    var id  = el.id
    el.onclick = function(){
      aler(id)   // 这样会导致闭包引用外层的el，当执行完showId后，el无法释放
    }
    el = null    // 主动释放el
}


```

**技巧 1： 用闭包解决递归调用问题**

```
function  factorial(num) {
   if(num<= 1) {
       return 1;
   } else {
      return num * factorial(num-1)
   }
}
var anotherFactorial = factorial
factorial = null
anotherFactorial(4)   // 报错 ，因为最好是return num* arguments.callee（num-1），arguments.callee指向当前执行函数，但是在严格模式下不能使用该属性也会报错，所以借助闭包来实现


// 使用闭包实现递归
function newFactorial = （function f(num){
    if(num<1) {return 1}
    else {
       return num* f(num-1)
    }
}） //这样就没有问题了，实际上起作用的是闭包函数f，而不是外面的函数newFactorial


```

** 技巧 2：用闭包模仿块级作用域 **  
es6 没出来之前，用 var 定义变量存在变量提升问题，eg:

```
for(var i=0; i<10; i++){
    console.info(i)
}
alert(i)  // 变量提升，弹出10

//为了避免i的提升可以这样做
(function () {
    for(var i=0; i<10; i++){
         console.info(i)
    }
})()
alert(i)   // underfined   因为i随着闭包函数的退出，执行环境销毁，变量回收


```

当然现在大多用 es6 的 let 和 const 定义。