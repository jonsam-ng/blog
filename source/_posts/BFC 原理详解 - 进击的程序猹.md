---
title: BFC 原理详解
---
一. BFC 是什么
----------

在解释 BFC 是什么之前，需要先介绍 Box、Formatting Context 的概念。

### 1.BOX：CSS 布局的基本单位

`Box`是`CSS`布局的对象和基本单位， 直观点来说，就是一个页面是由很多个 `Box` 组成的。元素的类型和 `display` 属性，决定了这个 `Box` 的类型。 不同类型的 `Box`， 会参与不同的 **Formatting Context**（一个决定如何渲染文档的容器），因此 Box 内的元素会以不同的方式渲染。让我们看看有哪些盒子：

*   block-level box:display 属性为 block, list-item, table 的元素，会生成 block-level box。并且参与 block fomatting context；
  
*   inline-level box:display 属性为 inline, inline-block, inline-table 的元素，会生成 inline-level box。并且参与 inline formatting context；
  
*   run-in box: css3 中才有， 这儿先不讲了。
  

**注：**这里补充一些 block-level box 一些知识

#### 1.1 block-level box

*   w3.org 中对块级元素的定义
  

> Block-level elements are those elements of the source document that are formatted visually as blocks (e.g., paragraphs). The following values of the ‘display’ property make an element block-level: ‘block’, ‘list-item’, and ‘table’.

大意：块级元素是那种源文档被格式化为可视块了的元素，然后使这个元素变成块级元素的 display 属性取值如下： ‘block’, ‘list-item’, 和 ‘table’。

> Block-level boxes are boxes that participate in a block formatting context. Each block-level element generates a principal block-level box that contains descendant boxes and generated content and is also the box involved in any positioning scheme

大意：块级盒 block-level box 是这种参与了块级排版上下文的一种盒子，每个块级元素都生成了一个包含后代盒子和生成的内容的主要块级盒，并且这个盒子参与了任何定位的计算

*   block-level box 盒模型
  

![](https://segmentfault.com/img/bVCruR)  
![](https://segmentfault.com/img/bVCruS)

*   block-level box 特性
  

> 块状元素排斥其他元素与其位于同一行，可以设定元素的宽（width）和高（height），块级元素一般是其他元素的容器，可容纳块级元素和行内元素。
> 
> 块状元素具有流体特性，即：在默认情况下（非浮动、绝对定位等），水平方向会自动填满外部的容器

### 2.Formatting context

Formatting context 是 W3C CSS2.1 规范中的一个概念。它是页面中的一块渲染区域，并且有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素的关系和相互作用。最常见的 Formatting context 有 Block fomatting context (简称 BFC) 和 Inline formatting context (简称 IFC)。  
CSS2.1 中只有 BFC 和 IFC, CSS3 中还增加了 GFC 和 FFC。

### 3.BFC 定义

BFC(Block formatting context) 直译为 "块级格式化上下文"。它是一个独立的渲染区域，只有 Block-level box 参与， 它规定了内部的 Block-level Box 如何布局，并且与这个区域外部毫不相干。

> 解释：
> 
> > BFC 是 W3C CSS 2.1 规范中的一个概念，它决定了元素如何对其内容进行定位，以及与其他元素的关系和相互作用。当涉及到可视化布局的时候，Block Formatting Context 提供了一个环境，HTML 元素在这个环境中按照一定规则进行布局。一个环境中的元素不会影响到其它环境中的布局。比如浮动元素会形成 BFC，浮动元素内部子元素的主要受该浮动元素影响，两个浮动元素之间是互不影响的。这里有点类似一个 BFC 就是一个独立的行政单位的意思。也可以说 BFC 就是一个作用范围。可以把它理解成是一个独立的容器，并且这个容器的里 box 的布局，与这个容器外的毫不相干。
> > 
> > 另一个通俗点的解释是：在普通流中的 Box(框) 属于一种 formatting context(格式化上下文) ，类型可以是 block ，或者是 inline ，但不能同时属于这两者。并且， Block boxes(块框) 在 block formatting context(块格式化上下文) 里格式化， Inline boxes(块内框) 则在 inline formatting context(行内格式化上下文) 里格式化。任何被渲染的元素都属于一个 box ，并且不是 block ，就是 inline 。即使是未被任何元素包裹的文本，根据不同的情况，也会属于匿名的 block boxes 或者 inline boxes。所以上面的描述，即是把所有的元素划分到对应的 formatting context 里。

canvas 会设立一个 BFC，这也是最外层的 formatting context 了，问题的复杂性在于有些块级盒内部也可以产生 BFC（至少它必须也能包含块级盒），于是说 BFC 是可以嵌套。不是所有块级盒内部都可以产生 BFC，比如说要是这盒里面连块级盒都没有，都是行内盒那就产生 IFC。不过，只要它的子节点里面有一个块级盒，它就产生 BFC，那些行内元素，会自动套一个匿名的块级行盒。

### 4.BFC 的布局规则

*   内部的 Box 会在垂直方向，一个接一个地放置
  
*   Box 垂直方向的距离由 margin 决定。属于同一个 BFC 的两个相邻 Box 的 margin 会发生重叠
  
*   每个元素的 margin-box 的左边， 与包含块 border-box 的左边相接触 (对于从左往右的格式化，否则相反)。即使存在浮动也是如此。
  
*   BFC 的区域不会与 float box 重叠。
  

> 关于这条规则的几点说明：
> 
> > 当容器有足够的剩余空间容纳 BFC 的宽度时，所有浏览器都会将 BFC 放置在浮动元素所在行的剩余空间内。  
> > 当 BFC 的宽度大于容器剩余宽度时，最新版本的浏览中只有 firefox 会在同一行显示，其它浏览器均换行。

*   BFC 就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。
  
*   计算 BFC 的高度时，浮动元素也参与计算
  

二. 哪些元素会生成 BFC
--------------

*   根元素
  
*   float 属性不为 none
  
*   position 为 absolute 或 fixed
  
*   display 为 inline-block, table-cell, table-caption, flex, inline-flex
  
*   overflow 不为 visible
  

> 关于 overflow:visible：
> 
> > overflow:visible 的块盒就不产生 BFC，不但不产生 BFC，啥 FC 都不产生，它的子元素直接搞进自己外层的 BFC 鸟：：  
> > overflow:visible 这个限制只对所谓的块盒（既包含块级盒、自己又是块级盒）存在，有些盒内部也能包含块级元素，但是它本身又不是块级元素（比如 display 为 table-cell、inline-block、或者盒本身是 flex item 等），因为外面不是 BFC，所以它们不论如何一定会给包含的块级盒创建一个新的 BFC 出来。
> 
> 关于浮动：
> 
> > 浮动是个行级的行为，当遇到浮动元素的时候，会首先 "假装" 它是个行内元素进行排版，排好后就往浮动的方向挤到挤不过去为止（遇到边界或者其它浮动元素）。  
> > 某一方向有 clear 的时候，浮动元素总是挤到边界，在垂直方向上的行为类似 "换行"。  
> > 排好一个浮动元素之后，这一行就要重排一次。所以说浮动元素会造成行级的 reflow。重排的时候，行盒会躲开浮动元素。之后的块级盒（不论是行盒还是其它盒）也都会躲开浮动元素排布。

三. BFC 的作用及原理
-------------

### 1. 自适应的两栏布局

代码：

```
<style>
    body {
        width: 300px;
        position: relative;
    }
    .aside {
        width: 100px;
        height: 150px;
        float: left;
        background: #f66;
    }
    .main {
        height: 200px;
        background: #fcc;
    }
</style>
<body>
    <div class="aside"></div>
    <div class="main"></div>
</body>

```

页面：  
![](https://segmentfault.com/img/bVCroJ)

> 根据 BFC 布局规则第 3 条：
> 
> > 每个元素的 _margin-box_ 的左边， 与包含块 _border-box_ 的左边相接触 (对于从左往右的格式化，否则相反)。即使存在浮动也是如此。

因此，虽然存在浮动的元素 aslide，但 main 的左边依然会与包含块的左边相接触。

> 根据 BFC 布局规则第四条：
> 
> > BFC 的区域不会与 float box 重叠。

我们可以通过通过触发 main 生成 BFC， 来实现自适应两栏布局。

```
.main {
    overflow: hidden;
}

```

当触发 main 生成 BFC 后，这个新的 BFC 不会与浮动的 aside 重叠。因此会根据包含块的宽度，和 aside 的宽度，自动变窄。效果如下：

页面：  
![](https://segmentfault.com/img/bVCrpp)

**对比**： 实现布局的另一种方式利用块状元素流体特性实现的自适应布局

> 利用块状元素流体特性实现的自适应布局
> 
> > 常用方法：浮动或者定位 + margin 撑开
> > 
> > 不足之处：我们需要知道浮动或绝对定位内容的尺寸。然后，流体内容才能有对应的 margin 或 padding 或 border 值进行位置修正。

### 2. 清除内部浮动

代码：

```
<style>
    .par {
        border: 5px solid #fcc;
        width: 300px;
    }
 
    .child {
        border: 5px solid #f66;
        width:100px;
        height: 100px;
        float: left;
    }
</style>
<body>
    <div class="par">
        <div class="child"></div>
        <div class="child"></div>
    </div>
</body>

```

页面：  
![](https://segmentfault.com/img/bVCrpI)

> 根据 BFC 布局规则第六条：
> 
> > 计算 BFC 的高度时，浮动元素也参与计算

为达到清除内部浮动，我们可以触发 par 生成 BFC，那么 par 在计算高度时，par 内部的浮动元素 child 也会参与计算。

```
.par {
    overflow: hidden;
}

```

效果如下：  
![](https://segmentfault.com/img/bVCrti)

### 3. 防止 margin 重叠

代码：

```
<style>
    p {
        color: #f55;
        background: #fcc;
        width: 200px;
        line-height: 100px;
        text-align:center;
        margin: 100px;
    }
</style>
<body>
    <p>Haha</p>
    <p>Hehe</p>
</body>

```

页面：  
![](https://segmentfault.com/img/bVCrtr)  
两个 p 之间的距离为 100px，发送了 margin 重叠。

> 根据 BFC 布局规则第二条：
> 
> > Box 垂直方向的距离由 margin 决定。属于同一个 BFC 的两个相邻 Box 的 margin 会发生重叠

我们可以在 p 外面包裹一层容器，并触发该容器生成一个 BFC。那么两个 P 便不属于同一个 BFC，就不会发生 margin 重叠了。

代码：

```
<style>
    .wrap {
        overflow: hidden;
    }
    p {
        color: #f55;
        background: #fcc;
        width: 200px;
        line-height: 100px;
        text-align:center;
        margin: 100px;
    }
</style>
<body>
    <p>Haha</p>
    <div class="wrap">
        <p>Hehe</p>
    </div>
</body>

```

效果如下:  
![](https://segmentfault.com/img/bVCrtJ)

### 4. 总结

> 以上的几个例子都体现了 BFC 布局规则第五条：
> 
> > BFC 就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。

因为 BFC 内部的元素和外部的元素绝对不会互相影响，因此， 当 BFC 外部存在浮动时，它不应该影响 BFC 内部 Box 的布局，BFC 会通过变窄，而不与浮动有重叠。同样的，当 BFC 内部有浮动时，为了不影响外部元素的布局，BFC 计算高度时会包括浮动的高度。避免 margin 重叠也是这样的一个道理。

参考
--

1. [前端精选文摘：BFC 神奇背后的原理](http://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html)  
2.[CSS 之 BFC 详解](http://sentsin.com/web/529.html)  
3.[BFC 、IFC](http://blog.sina.com.cn/s/blog_877284510101jo5d.html)  
4.[CSS 深入理解流体特性和 BFC 特性下多栏自适应布局](http://www.zhangxinxu.com/wordpress/2015/02/css-deep-understand-flow-bfc-column-two-auto-layout/)  
5.[CSS 布局](http://www.cnblogs.com/winter-cn/archive/2013/05/11/3072929.html)