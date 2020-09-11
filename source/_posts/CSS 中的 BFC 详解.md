---
title: CSS 中的 BFC 详解
---
**引言：**

这篇文章是我对 BFC 的理解及总结，带你揭开 BFC 的面纱。你将会知道 BFC 是什么，形成 BFC 的条件，BFC 的相关特性，以及他的实际应用。

**一、何为 BFC**

       BFC（Block Formatting Context）格式化上下文，是 Web 页面中盒模型布局的 CSS 渲染模式，指一个独立的渲染区域或者说是一个隔离的独立容器。

**二、形成 BFC 的条件**

      1、浮动元素，float 除 none 以外的值；   
      2、定位元素，position（absolute，fixed）；   
      3、display 为以下其中之一的值 inline-block，table-cell，table-caption；   
      4、overflow 除了 visible 以外的值（hidden，auto，scroll）；

**三、BFC 的特性**

      1. 内部的 Box 会在垂直方向上一个接一个的放置。  
      2. 垂直方向上的距离由 margin 决定  
      3.bfc 的区域不会与 float 的元素区域重叠。  
      4. 计算 bfc 的高度时，浮动元素也参与计算  
      5.bfc 就是页面上的一个独立容器，容器里面的子元素不会影响外面元素。

看到这里是不是有丈二和尚摸不着头脑的感觉，下面我就用案例来帮助理解认识：

**四、实践是检验真理的唯一标准**

_（1）BFC 中的盒子对齐_
---------------

特性的第一条是：内部的 Box 会在垂直方向上一个接一个的放置。

![](https://images2017.cnblogs.com/blog/948888/201711/948888-20171119211903906-1604046404.png)

浮动的元素也是这样，box3 浮动，他依然接着上一个盒子垂直排列。并且所有的盒子都左对齐。

html：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
    <div>
        <div></div>
        <div></div>
        <div></div>
        <div></div>
    </div>

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

css:

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
        div {
            height: 20px;
        }
        
        .container {
            position: absolute;  /* 创建一个BFC环境*/
            height: auto;
            background-color: #eee;
        }
        
        .box1 {
            width: 400px;
            background-color: red;
        }
        
        .box2 {
            width: 300px;
            background-color: green;
        }
        
        .box3 {
            width: 100px;
            background-color: yellow;
            float: left;
        }
        
        .box4 {
            width: 200px;
            height: 30px;
            background-color: purple;
        }

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

_（2）外边距折叠_
----------

特性的第二条：垂直方向上的距离由 margin 决定

在常规文档流中，两个兄弟盒子之间的垂直距离是由他们的外边距所决定的，但不是他们的两个外边距之和，而是以较大的为准。  
![](https://images2017.cnblogs.com/blog/948888/201711/948888-20171119213530124-1270666391.png)

html：

```
    <div>
        <div></div>
        <div></div>
    </div>

```

css：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
        .container {
            overflow: hidden;
            width: 100px;
            height: 100px;
            background-color: red;
        }
        
        .box1 {
            height: 20px;
            margin: 10px 0;
            background-color: green;
        }
        
        .box2 {
            height: 20px;
            margin: 20px 0;
            background-color: green;
        }

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

 这里我门可以看到，第一个子盒子有上边距（不会发生 margin 穿透的问题）；两个子盒子的垂直距离为 20px 而不是 30px，因为垂直外边距会折叠，间距以较大的为准。

 那么有没有方法让垂直外边距不折叠呢？答案是：有。特性的第 5 条就说了：bfc 就是页面上的一个独立容器，容器里面的子元素不会影响外面元素，同样外面的元素不会影响到 BFC 内的元素。所以就让 box1 或 box2 再处于另一个 BFC 中就行了。

![](https://images2017.cnblogs.com/blog/948888/201711/948888-20171119214603984-522419798.png)

html：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
    <div>
        <div>
            <div></div>
        </div>
        <div></div>
    </div>

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

css：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
    .container {
        overflow: hidden;
        width: 100px;
        height: 100px;
        background-color: red;
    }
    
    .wrapper {
        overflow: hidden;
    }
    
    .box1 {
        height: 20px;
        margin: 10px 0;
        background-color: green;
    }
    
    .box2 {
        height: 20px;
        margin: 20px 0;
        background-color: green;
    }

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

_**（3）不被浮动元素覆盖** _

以常见的两栏布局为例。

左边固定宽度，右边不设宽，因此右边的宽度自适应，随浏览器窗口大小的变化而变化。

![](https://images2017.cnblogs.com/blog/948888/201711/948888-20171119221124124-157922272.png)

html：

```
    <div></div>
    <div></div>

```

css：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
        .column:nth-of-type(1) {
            float: left;
            width: 200px;
            height: 300px;
            margin-right: 10px;
            background-color: red;
        }
        
        .column:nth-of-type(2) {
            overflow: hidden;/*创建bfc */
            height: 300px;
            background-color: purple;
        }

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

还有三栏布局。

左右两边固定宽度，中间不设宽，因此中间的宽度自适应，随浏览器的大小变化而变化。

![](https://images2017.cnblogs.com/blog/948888/201711/948888-20171119224204765-1347734437.png)

html：

```
    <div>
        <div></div>
        <div></div>
        <div></div>
    </div>

```

css：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
        .column:nth-of-type(1),
        .column:nth-of-type(2) {
            float: left;
            width: 100px;
            height: 300px;
            background-color: green;
        }
        
        .column:nth-of-type(2) {
            float: right;
        }
        
        .column:nth-of-type(3) {
            overflow: hidden;  /*创建bfc*/
            height: 300px;
            background-color: red;
        }

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

也可以用来防止字体环绕：

众所周知，浮动的盒子会遮盖下面的盒子，但是下面盒子里的文字是不会被遮盖的，文字反而还会环绕浮动的盒子。这也是一个比较有趣的特性。

![](https://images2017.cnblogs.com/blog/948888/201711/948888-20171119222632796-1452266331.png)             ![](https://images2017.cnblogs.com/blog/948888/201711/948888-20171119222701452-741699368.png)

html：

```
    <div></div>
    <p>你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好
       你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好
       你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好你好
    </p>

```

css：

（1）环绕

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
        .left {
            float: left;
            width: 100px;
            height: 100px;
            background-color: yellow;
        }
        
        p {
            background-color: green;
            /* overflow: hidden; */
        }

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

（2）利用 bfc 防止环绕

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
        .left {
            float: left;
            width: 100px;
            height: 100px;
            background-color: yellow;
        }
        
        p {
            background-color: green;
            overflow: hidden;
        }

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

_**（4）BFC 包含浮动的块**_

这个是大家再熟悉不过的了，利用 overflow:hidden 清除浮动嘛，因为浮动的盒子无法撑出处于标准文档流的父盒子的 height。这个就不过多解释了，相信大家都早已理解。

**总结**

我希望这篇文章已经向你展示了 BFC 的相关特性，以及他们如何影响元素在页面上的视觉定位。所有的例子都展示了它们在实际案例中的应用，这应该会使得他们更加清晰。

如果你有任何东西想要补充，请在评论中留言，欢迎共同讨论。