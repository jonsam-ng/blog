---
title: float 详解
---
使得元素脱离文档流有两种方式。第一种是通过定位，第二种方法就是使用浮动。  
浮动 (float)，是一个我们即爱又恨的属性。爱，因为通过浮动，我们能很方便地布局；恨，因为浮动之后遗留下来太多的问题需要解决，特别是 IE6-7（以下无特殊说明均指 windows 平台的 IE 浏览器）。如果不详细了解浮动，可能会觉得它很神秘也很复杂，很多网站的效果好看，但是自己就是做不出来。  
很多初学者都会有这样的疑问：浮动从何而来？为什么要设置浮动？我们为何要清除浮动？清除浮动的原理是什么？  
现在我们就深入剖析 float 的奥秘，解开这些问题的答案。

##### 一、什么是浮动？浮动最常见的问题是什么？

最简单的解释，浮动可以使得多个块级元素可以放置在同一行上。例如：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>z-index</title>
</head>
<body>
  <div class="main left">
    .main:我是主要内容
  </div>
  <div class="side left">
    .side:我是侧边栏
  </div>
</body>
</html>

.main {
    height: 60px;
    width: 50%;
    background: #FFE3D7;
}
.left {
    float: left;
}
div {
    padding: 15px 20px;
    font-size: 14px;
    color: #333;
}
.side {
    width: 20%;
    background: lightblue;
}


```

![](http://upload-images.jianshu.io/upload_images/7367863-ccee03e8610541e3.png)

float1.png

这种浮动有什么用？其实很多网站的两栏或者三栏结构都是利用浮动来实现的。例如：

![](http://upload-images.jianshu.io/upload_images/7367863-8b73c30c2f3acb42.png)

float2.png

然而浮动初学者经常出现的问题是，浮动元素脱离的文档流，因此，它无法使得其他元素和其相对位置关系保持正常。也无法使得自己的父容器正确计算自己的大小。对于上面的例子，如果我们使得 main 元素和 side 元素有一个父容器 wrap，并且父容器还有一个兄弟节点 footer：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>z-index</title>
</head>
<body>
    <div class="warp">
        <div class="main left">
            .main:我是主要内容
        </div>
        <div class="side left">
            .side:我是侧边栏
        </div>
    </div>
    <div class="footer">
        我是一个footer
    </div>
</body>
</html>

.main {
    height: 60px;
    width: 50%;
    background: #FFE3D7;
}
.left {
    float: left;
}
div {
    padding: 15px 20px;
    font-size: 14px;
    color: #333;
}
.side {
    width: 20%;
    background: lightblue;
}
.warp {
    border: 1px solid blue;
    width: 300px;
    margin: 30px auto 5px;
    background: #F5F5F5;
}
.footer {
    border: 1px solid #ccc;
    background: #EEE;
    width: 300px;
    margin: 5px auto 30px;
}


```

![](http://upload-images.jianshu.io/upload_images/7367863-26b529ac85bb80e4.png)

float3.png

可以看到两个最大的问题：

1.  wrap 元素的高度无法正确覆盖住 main 和 side 元素。
2.  由于 wrap 元素高度计算的问题，footer 元素本身显示的位置非常奇怪。通常解决浮动元素带来的问题有两种主要方式：清除浮动和闭合浮动。那么：

##### 二、该 清除浮动 还是 闭合浮动？

很多人都已经习惯称之为清除浮动，以前我也一直这么叫着，但是确切地来说是不准确的。我们应该用严谨的态度来对待代码，也能更好地帮助我们理解上面的问题。  
1）清除浮动：清除对应的单词是 clear，对应 CSS 中的属性是 clear：left | right | both | none；  
2）闭合浮动：更确切的含义是使浮动元素闭合，从而减少浮动带来的影响。  
两者的区别：

###### 清除浮动：

```
<div class="warp">
        <div class="main left">
            .main:很抱歉，现代浏览器中我没能把warp撑高（float:left）
        </div>
        <div class="side left">
            .side:我也浮动了（float:left）
        </div>
    </div>
    <div class="footer clear">
        .footer:我通过设置clear:both <strong>清除浮动</strong>，虽然位置正确了，但是wrap的高度没变，还是没达到想要的效果
    </div>

.clear {
    clear: both;
}


```

![](http://upload-images.jianshu.io/upload_images/7367863-62671506d579c93d.png)

float4.png

###### 闭合浮动：

```
<div class="warp clearfix">
        <div class="main left">
            .main:warp自己闭合浮动了，所以footer不用再清除浮动了（float:left）
        </div>
        <div class="side left">
            .side:我也浮动了（float:left）
        </div>
    </div>
    <div class="footer clear">
        .footer:warp通过 .clearfix 已经<strong>闭合浮动</strong>了
    </div>

.clearfix:after {
    clear: both;
    content: ".";
    display: block;
    height: 0;
    visibility: hidden;
}
.clearfix {
    *zoom: 1;
}


```

![](http://upload-images.jianshu.io/upload_images/7367863-5916d5ec1e48674d.png)

float5.png

通过以上实例发现，其实我们想要达到的效果更确切地说是闭合浮动，而不是单纯的清除浮动，在 footer 上设置 clear：both 清除浮动并不能解决 wrap 高度塌陷的问题。  
结论：用闭合浮动比清除浮动更加严谨，所以后文中统一称之为：闭合浮动。

##### 三、为何要闭合浮动？

要解答这个问题，我们得先说说 CSS 中的定位机制：普通流，浮动，绝对定位（其中 "position:fixed" 是 "position:absolute" 的一个子类）。  
1）普通流：很多人或者文章称之为文档流或者普通文档流，其实标准里根本就没有这个词。如果把文档流直译为英文就是 document flow，但标准里只有另一个词，叫做普通流（normal flow)，或者称之为常规流。但似乎大家更习惯文档流的称呼，因为很多中文翻译的书就是这么来的。比如《CSS Mastery》，英文原书中至始至终都只有普通流 normal flow（普通流）这一词，从来没出现过 document flow（文档流）  
2）浮动：浮动的框可以左右移动，直至它的外边缘遇到包含框或者另一个浮动框的边缘。浮动框不属于文档中的普通流，当一个元素浮动之后，不会影响到块级框的布局而只会影响内联框（通常是文本）的排列，文档中的普通流就会表现得和浮动框不存在一样，当浮动框高度超出包含框的时候，也就会出现包含框不会自动伸高来闭合浮动元素（“高度塌陷” 现象）。顾名思义，就是漂浮于普通流之上，像浮云一样，但是只能左右浮动。  
正是因为浮动的这种特性，导致本属于普通流中的元素浮动之后，包含框内部由于不存在其他普通流元素了，也就表现出高度为 0（高度塌陷）。在实际布局中，往往这并不是我们所希望的，所以需要闭合浮动元素，使其包含框表现出正常的高度。

##### 四、该如何正确闭合浮动呢？

先看一下闭合浮动的各种方法：

###### 1）添加额外标签

一种方法，通过在浮动元素末尾添加一个空的标签例如 <div class=”clear”></div>，其他标签 br 等亦可。例如：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>z-index</title>
</head>
<body>
    <div class="warp">
        <div class="main left">
            .main:我是主要内容
        </div>
        <div class="side left">
            .side:我是侧边栏
        </div>
        <div class="clear"></div>
    </div>
    <div class="footer">
        我是一个footer
    </div>
</body>
</html>


```

![](http://upload-images.jianshu.io/upload_images/7367863-6de9421df038dde3.png)

float6.png

优点：通俗易懂，容易掌握  
缺点：可以想象通过此方法，会添加多少无意义的空标签，有违结构与表现的分离的思想，在后期维护中将是噩梦，这是坚决不能忍受的，所以我们不推荐使用该方法。

###### 2）使用 br 标签和其自身的 html 属性

这个方法有些小众，br 有 clear=“all | left | right | none” 属性

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>z-index</title>
</head>
<body>
    <div class="warp">
        <div class="main left">
            .main:我是主要内容
        </div>
        <div class="side left">
            .side:我是侧边栏
        </div>
        <br clear="all">
    </div>
    <div class="footer">
        我是一个footer
    </div>
</body>
</html>


```

优点：比空标签方式语义稍强，代码量较少  
缺点：同样有违结构与表现的分离，不推荐使用

###### 3) 父元素设置 overflow：hidden

通过设置父元素 overflow 值设置为 hidden；在 IE6 中还需要触发 hasLayout，例如 zoom：1；

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>z-index</title>
</head>
<body>
    <div class="warp close">
        <div class="main left">
            .main:我是主要内容
        </div>
        <div class="side left">
            .side:我是侧边栏
        </div>
    </div>
    <div class="footer">
        我是一个footer
    </div>
</body>
</html>

.close {
  overflow: hidden;
  *zoom: 1;
}


```

优点：不存在结构和语义化问题，代码量极少  
缺点：内容增多时候容易造成不会自动换行导致内容被隐藏掉，无法显示需要溢出的元素；所以也不推荐使用。

###### 4) 父元素设置 overflow：auto 属性

同样 IE6 需要触发 hasLayout，演示和 3 差不多  
优点：不存在结构和语义化问题，代码量极少  
缺点：多个嵌套后，Firefox 某些情况会造成内容全选；IE 中 mouseover 造成宽度改变时会出现最外层模块有滚动条等，Firefox 早期版本会无故产生 focus 等。

###### 5) 父元素也设置浮动

优点：不存在结构和语义化问题，代码量极少  
缺点：使得与父元素相邻的元素的布局会受到影响，不可能一直浮动到 body，不推荐使用。

###### 6) 父元素设置 display:table

```
.close {
  display: table;
}


```

优点：结构语义化完全正确，代码量极少。  
缺点：盒模型属性已经改变，由此造成的一系列问题，得不偿失，不推荐使用。

###### 7) 使用: after 伪元素

需要注意的是: after 是伪元素（Pseudo-Element），不是伪类（某些 CSS 手册里面称之为 “伪对象”），很多闭合浮动大全之类的文章都称之为伪类，不过要严谨一点，这是一种态度。  
由于 IE6-7 不支持: after，使用 zoom:1 触发 hasLayout。  
优点：结构和语义化完全正确, 代码量中等  
缺点：复用方式不当会造成代码量增加

###### 小结：

通过对比，我们不难发现，其实以上列举的方法，无非有两类：  
其一，通过在浮动元素的末尾添加一个空元素，设置 clear：both 属性，after 伪元素其实也是通过 content 在元素的后面生成了内容为一个点的块级元素；  
其二，通过设置父元素 overflow 或者 display：table 属性来闭合浮动，我们来探讨一下这里面的原理。

##### 五、那么闭合浮动的原理究竟是什么？

在 CSS2.1 里面有一个很重要的概念，但是国内的技术博客介绍到的比较少，那就是 Block formatting contexts（块级格式化上下文），以下简称 BFC。  
CSS3 里面对这个规范做了改动，称之为：flow root，并且对触发条件进行了进一步说明。

###### 那么如何触发 BFC 呢？（满足一下任一条件即可）

*   float 除了 none 以外的值
*   overflow 除了 visible 以外的值（hidden，auto，scroll ）
*   display (table-cell，table-caption，inline-block)
*   position（absolute，fixed）
*   fieldset 元素  
    需要注意的是，display:table 本身并不会创建 BFC，但是它会产生匿名框 (anonymous boxes)，而匿名框中的 display:table-cell 可以创建新的 BFC，换句话说，触发块级格式化上下文的是匿名框，而不是 display:table。所以通过 display:table 和 display:table-cell 创建的 BFC 效果是不一样的。  
    fieldset 元素在 [www.w3.org](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.w3.org) 里目前没有任何有关这个触发行为的信息，直到 HTML5 标准里才出现。有些浏览器 bugs（Webkit，Mozilla）提到过这个触发行为，但是没有任何官方声明。实际上，即使 fieldset 在大多数的浏览器上都能创建新的块级格式化上下文，开发者也不应该把这当做是理所当然的。CSS 2.1 没有定义哪种属性适用于表单控件，也没有定义如何使用 CSS 来给它们添加样式。用户代理可能会给这些属性应用 CSS 属性，建议开发者们把这种支持当做实验性质的，更高版本的 CSS 可能会进一步规范这个。

##### BFC 的特性：

###### 1) 块级格式化上下文会阻止外边距叠加

当两个相邻的块框在同一个块级格式化上下文中时，它们之间垂直方向的外边距会发生叠加。换句话说，如果这两个相邻的块框不属于同一个块级格式化上下文，那么它们的外边距就不会叠加。

###### 2) 块级格式化上下文不会重叠浮动元素

根据规定，一个块级格式化上下文的边框不能和它里面的元素的外边距重叠。这就意味着浏览器将会给块级格式化上下文创建隐式的外边距来阻止它和浮动元素的外边距叠加。由于这个原因，当给一个挨着浮动的块级格式化上下文添加负的外边距时将会不起作用。

###### 3) 块级格式化上下文通常可以包含浮动

通俗地来说：创建了 BFC 的元素就是一个独立的盒子，里面的子元素不会在布局上影响外面的元素，反之亦然，同时 BFC 任然属于文档中的普通流。至此，你或许明白了为什么 overflow:hidden 或者 auto 可以闭合浮动了，这是因为父元素创建了新的 BFC。

##### 六、低版本的 IE 没有 BFC 怎么办？

IE6-7 的显示引擎使用的是一个称为布局（layout）的内部概念，由于这个显示引擎自身存在很多的缺陷，直接导致了 IE6-7 的很多显示 bug。当我们说一个元素 “得到 layout”，或者说一个元素“拥有 layout” 的时候，我们的意思是指它的微软专有属性 hasLayout 被设为了 true。  
IE6-7 使用布局的概念来控制元素的尺寸和定位，那些拥有布局（have layout）的元素负责本身及其子元素的尺寸设置和定位。如果一个元素的 hasLayout 为 false，那么它的尺寸和位置由最近拥有布局的祖先元素控制。  
触发 hasLayout 的条件：

*   position: absolute
*   float: left|right
*   display: inline-block
*   width: 除 “auto” 外的任意值
*   height: 除 “auto” 外的任意值（例如很多人闭合浮动会用到  
    height:1%）
*   zoom: 除 “normal” 外的任意值
*   writing-mode: tb-rl  
    在 IE7 中，overflow 也变成了一个 layout 触发器：
*   overflow: hidden|scroll|auto（这个属性在 IE 之前版本中没有触发 layout 的功能。）
*   overflow-x|-y: hidden|scroll|auto（CSS3 盒模型中的属性，尚未得到浏览器的广泛支持。他们在之前 IE 版本中同样没有触发 layout 的功能）  
    hasLayout 更详细的解释请参见 old9 翻译的大名鼎鼎的《On having layout》一文（[http://www.satzansatz.de/cssd/onhavinglayout.htm](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.satzansatz.de%2Fcssd%2Fonhavinglayout.htm)）。  
    综上所述，若同时考虑 IE 何其他浏览器，更加完善的闭合浮动的方式：

1.  在支持 BFC 的浏览器（IE8+，firefox，chrome，safari）通过创建新的 BFC 闭合浮动；
2.  在不支持 BFC 的浏览器（IE6-7），通过触发 hasLayout 闭合浮动。