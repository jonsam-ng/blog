---
title: 如何使用 Flexbox 和 CSS Grid，实现高效布局
---

CSS 浮动属性一直是网站上排列元素的主要方法之一，但是当实现复杂布局时，这种方法不总是那么理想。幸运的是，在现代网页设计时代，使用 Flexbox 和 CSS Grid 来对齐元素，变得相对容易起来。

使用 Flexbox 可以使元素对齐变得容易，因此 Flexbox 已经被广泛使用了。

同时，CSS Grid 布局也为网页设计行业带来了很大的便利。虽然 CSS Grid 布局未被广泛采用，但是浏览器逐渐开始增加对 CSS Grid 布局的支持。

虽然 Flexbox 和 CSS Grid 可以完成类似的布局，但是本次，我们学习的是如何组合使用这两个工具，而不是只选择其中的一个。在不久的将来，当 CSS Grid 布局获得完整的浏览器支持时，设计人员就能够利用每个 CSS 组合的优势，来创建最有效和最有趣的布局设计。

测试 Flexbox 和 CSS Grid 的基本布局
===========================

我们从一个很简单且熟悉的布局类型开始，包括标题，侧边栏，主要内容和页脚等部分。通过这样一个简单的布局，来帮助我们快速找到各种元素的布局方法。

下面是需要创建的内容： 

![](http://images2017.cnblogs.com/blog/139239/201709/139239-20170920110009884-901638498.jpg)

要完成这个基本布局， Flexbox 需要完成的主要任务包括以下方面：

*   创建完整宽度的 header 和 footer
*   将侧边栏放置在主内容区域左侧
*   确保侧边栏和主内容区域的大小合适
*   确保导航元素定位准确

基本 HTML 结构
==========

```
<div>
    <header>
        <nav>
          <ul>
            <li></li>
            <li></li>
            <li></li>
          </ul>
        </nav>
        <button></button>
    </header>
    <div>
        <aside>
            <h3></h3>
        </aside>
        <section>
            <h2></h2>
            <p></p>
        </section>
    </div><!-- /wrapper -->
    <footer>
        <h3></h3>
        <p></p>
    </footer>
</div><! -- /container -->

```

使用 Flexbox 创建布局
===============

*   Header 样式
    ---------
    

我们从外到内，逐层开始设计，首先将 display: flex; 添加到 container，这也是所有 Flexbox 布局的第一步。接着，将 flex-direction 设置为 column，确保所有部分彼此相对。

```
.container {
    display: flex;
    flex-direction: column;
}

```

通过 display: flex; 自动创建一个全宽的 header（header 默认情况下是块级元素）。通过这个声明，导航元素的放置会变得很容易。

导航栏的左侧有一个 logo 和两个菜单项，右侧有一个登录按钮。导航位于 header 中，通过 justify-content: space-between; 可以实现导航和按钮之间的自动间隔。

在导航中，使用 align-items: baseline; 能够实现所有导航项目与文本基线的对齐，这样也使得导航栏看起来更加统一。

![](http://images2017.cnblogs.com/blog/139239/201709/139239-20170920110530931-1941712144.jpg)

代码如下:

```
header{
    padding: 15px;
    margin-bottom: 40px;
    display: flex;
    justify-content: space-between;
}

header nav ul {
    display: flex;
    align-items: baseline;
    list-style-type: none;
}

```

*   页面内容样式
    ------
    

接下来，将侧边栏和主内容区域使用一个 wrapper 包含起来。具有 .wrapper 类的 div，也需要设置 display: flex; 但是 flex 方向与上述不同。这是因为侧边栏和主内容区域彼此相邻而不是堆叠。

```
.wrapper {
    display: flex;
    flex-direction: row;
}

```

![](http://images2017.cnblogs.com/blog/139239/201709/139239-20170920110514618-1006690725.jpg)

主内容区域和侧边栏的大小设置非常重要，因为重要的信息都在这里展示。主内容区域应该是侧边栏大小的三倍，使用 Flexbox 很容易实现这点。

```
.main {
    flex: 3;
    margin-right: 60px;
}

.sidebar {
   flex: 1;
}

```

总的来说，Flexbox 在创建这个简单的布局时，十分高效。尤其在控制列表元素样式和设置导航与按钮之间的间距方面，特别有用。

使用 CSS Grid 创建布局
================

为了测试效率，接下来使用 CSS Grid 创建相同的基本布局。 

![](http://images2017.cnblogs.com/blog/139239/201709/139239-20170920110549525-1972002482.jpg)

*   Grid 模板区域
    ---------
    

CSS Grid 的方便之处在于，可以指定模板区域，这也使得定义布局变得非常直观。采取这种方法，网格上的区域可以命名并引用位置项。对于这个基本布局，我们需要命名四个项目：

*   header
*   main content
*   sidebar
*   footer

基本 HTML 结构

```
<div>
    <header>
        <nav>
          <ul>
            <li></li>
            <li></li>
            <li></li>
          </ul>
        </nav>
        <button></button>
    </header>
   
    <aside>
        <h3></h3>
        <ul>
            <li></li>
            <li></li>
   　　　　　 <li></li>
    　　　　　<li></li>
   　　　　　 <li></li>
        </ul>
    </aside>
 
    <section>
        <h2></h2>
        <p></p>
        <p> </p>
    </section>
 
    <footer>
        <h3></h3>
        <p></p>
    </footer>
</div>

```

我们按照顺序在 grid container 中定义这些区域，就像绘制它们一样。

grid-template-areas:

        "header header"

        "sidebar main"

        "footer footer";

当前侧边栏位于左侧，主区域内容位于右侧，如果需要，也可以轻松更改顺序。

有一件事要注意：这些名字需要 “连接” 到样式上。所以需要在 header block 中，添加 grid-area: header;。

```
header{
    grid-area: header;
    padding: 20px 0;
    display: grid;
    grid-template-columns: 1fr 1fr;
}

```

HTML 结构与 Flexbox 示例中的相同，但 CSS 与创建网格布局完全不同。

```
.container{
    max-width: 900px;
    background-color: #fff;
    margin: 0 auto;
    padding: 0 60px;
    display: grid;
    grid-template-columns: 1fr 3fr;
    grid-template-areas:
        "header header"
        "sidebar main"
        "footer footer";
    grid-gap: 50px;
}

```

使用 CSS Grid 布局时，在 container 中设置 display: grid; 非常重要。此处声明 grid-template-columns，是为了确保页面的整体结构。这里 grid-template-column 已将侧边栏和主内容区域大小设置为 1fr 和 3fr。fr 是网格的分数单位。

 ![](http://images2017.cnblogs.com/blog/139239/201709/139239-20170920110812306-464478185.jpg)

接下来，需要调整 header 容器中的 fr 单元。将 grid-template-columns 设置为 1fr 和 1fr。这样 header 中就有两个相同大小的列，放置导航项和按钮会很合适。

```
header{
    grid-area: header;
    display: grid;
    grid-template-columns: 1fr 1fr;
}

```

![](http://images2017.cnblogs.com/blog/139239/201709/139239-20170920110829431-1198146622.jpg)

要放置按钮，我们只需要将 justify-self 设置为 end。

```
header button {
    justify-self: end;
}

```

导航的位置按照以下方式设置：

```
header nav {
    justify-self: start;
}

```

使用 Flexbox 和 CSS Grid 创建布局
==========================

最后，我们通过组合 Flexbox 和 CSS Grid 来创建更复杂的布局。

![](http://images2017.cnblogs.com/blog/139239/201709/139239-20170920110911384-498762685.jpg) 

基本的布局如下图所示：

![](http://images2017.cnblogs.com/blog/139239/201709/139239-20170920110920478-1738248932.jpg)

这种布局需要在行和列两个方向上保持一致，所以使用 CSS Grid 实现整体布局十分有效。

![](http://images2017.cnblogs.com/blog/139239/201709/139239-20170920110940540-1063780669.jpg) 

规划对于布局的实现来说，十分重要。

接下来看看代码如何一步步实现。首先 display: grid; 是基本设置，其次内容块之间的间距，可以通过 grid-column-gap 和 grid-row-gap 实现。

```
.container {
  display: grid;
  grid-template-columns: 0.4fr 0.3fr 0.3fr;
  grid-column-gap: 10px;
  grid-row-gap: 15px;
}

```

*   列和行布局
    -----
    

Header 部分横跨所有的列。

```
.header {
  grid-column-start: 1;
  grid-column-end: 4;
  grid-row-start: 1;
  grid-row-end: 2;
  background-color: #d5c9e2;
}

```

也可以使用简写，起始值和结束值位于同一行上，并用斜杠分隔。就像这样：

```
.header {
  grid-column: 1 / 4;
  grid-row: 1 / 2;
  background-color: #55d4eb;
}

```

完成网格布局的构建之后，微调内容就是下一步。

*   导航
    --
    

Flexbox 非常适合放置 header 元素。基本的 header 布局需要设置 justify-content: space-between。

上面的 CSS Grid 布局示例中，需要在导航栏设置 justify-self：start;，在按钮设置 justify-self: end;，但是如果使用 Flexbox，导航的间距会变得很容易设置。

```
.header {
  grid-column: 1 / 4;
  grid-row: 1 / 2;
  color: #9f9c9c;
  text-transform: uppercase;
  border-bottom: 2px solid #b0e0ea;
  padding: 20px 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

```

*   列内容网格
    -----
    

将所需的元素排列在一个方向上，意味所有元素都处在同一横向维度，通常 Flexbox 是实现这种布局的更好选择。此外，Flexbox 可以动态调整元素。使用 Flexbox，可以将所有元素连成一条直线，这也确保了所有元素都具有相同的高度。

*   带有文本和按钮的行内容
    -----------
    

下图是包含了 “额外” 文本和按钮的三个区域。Flexbox 可以轻松设置三列的宽度。

```
.extra {
  grid-column: 2 / 4;
  grid-row: 4 / 5;
  padding: 1rem;
  display: flex;
  flex-wrap: wrap;
  border: 1px solid #ececec;
  justify-content: space-between;
}

```

设计方法总结
======

以上的布局设计中，使用了 CSS Grid 来进行整体布局（以及设计中的非线性部分）。对于网格内容区域的设计，使用 Flexbox 进行样式的排序和微调会更容易实现。
