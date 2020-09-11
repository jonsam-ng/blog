---
title: jQuery 四种事件绑定方式. bind(),.live(),.delegate(),on() 的区别
---
[.bind()](http://api.jquery.com/bind/), [.live()](http://api.jquery.com/live/), 和 [.delegate()](http://api.jquery.com/delegate/) 之间的区别并不明显。但是理解它们的不同之处有助于写出更简洁的代码，并防止我们的交互程序中出现没有预料到的 bug。

### 基础

#### DOM 树

首先，图形化的 HTML 文档能帮助我们更好的理解。一个简单的 HTML 页面看起来应该像这样

![](http://chart.apis.google.com/chart?cht=gv&chl=graph%7Bwindow--document--h1;document--p--span;p--a;document--h2;document--form--input;form--submit;%7D&chs=550x300)

#### 事件冒泡（也称作事件传递）（Event bubbling aka event propagation）

点击一个链接，触发绑定在链接元素上的 click 事件，进而触发绑定到这个元素的 click 事件的函数。

`$(``'a'``).bind(``'click'``,``function``() { alert(``"That tickles!"``) });`

所以一次点击会触发一个 alert。  
![](http://chart.apis.google.com/chart?cht=gv&chl=digraph%7Ba%5Bcolor=red%5D;%22That%20tickles%21%22%5Bshape=rectangle%5D;a-%3E%22That%20tickles%21%22%5Bcolor=red%5D%5Blabel=click%5D%5Bfontcolor=red%5D%7D)  
然后，这个 click 事件会从 DOM 树向上传递，传播到父元素，然后传递给每一个祖先元素。  
![](http://chart.apis.google.com/chart?cht=gv&chl=digraph%7Ba%5Bcolor=red%5D;window-%3Edocument%5Blabel=%22document%20%3E%20p%20%3E%20a.click%22%5D%5Bdir=back%5D%5Bcolor=red%5D%5Bfontcolor=red%5D;document-%3Eh1%5Bdir=none%5D;document-%3Ep%5Blabel=%22p%20%3E%20a.click%22%5D%5Bdir=back%5D%5Bcolor=red%5D%5Bfontcolor=red%5D;p-%3Espan%5Bdir=none%5D;document-%3Eh2%5Bdir=none%5D;document-%3Eform%5Bdir=none%5D;form-%3Einput%5Bdir=none%5D;form-%3Esubmit%5Bdir=none%5D;p-%3Ea%5Blabel=%22a.click%22%5D%5Bdir=back%5D%5Bcolor=red%5D%5Bfontcolor=red%5D;%7D&chs=550x350)

在 DOM 树中， document 是根节点。  
现在我们能容易的解释[.bind()](http://api.jquery.com/bind/), [.live()](http://api.jquery.com/live/), 和 [.delegate()](http://api.jquery.com/delegate/) 之间的差别了

#### .bind()

`$(``'a'``).bind(``'click'``,``function``() { alert(``"That tickles!"``) });`

这是最直接的绑定方法。jQuery 扫描文档找到所有 $(‘a’) 元素，然后给每一个找到的元素的 click 事件绑定处理函数。

```
$( "#members li a" ).bind( "click", function( e ) {} ); 
$( "#members li a" ).click( function( e ) {} );

```

  

上面的两行代码所完成的任务都是一致的，就是把 event handler 加到全部的匹配的 <a> 元素上。这里存在着一些效率方面的问题，一方面，我们隐式地把 click handler 加到所有的 a 标签上，这个过程是昂贵的; 另一方面在执行的时候也是一种浪费，因为它们都是做了同一件事却被执行了一次又一次（比如我们可以把它 hook 到它们的父元素上，通过冒泡可以对它们中的每一个进行区分，然后再执行这个 event handler）。

优点：

*   这个方法提供了一种在各种浏览器之间对事件处理的兼容性解决方案
*   非常方便简单的绑定事件到元素上
*   .click(), .hover()... 这些非常方便的事件绑定，都是 bind 的一种简化处理方式
*   对于利用 ID 选出来的元素是非常好的，不仅仅是很快的可以 hook 上去 (因为一个页面只有一个 id), 而且当事件发生时，handler 可以立即被执行(相对于后面的 live, delegate) 实现方式

缺点：

*   它会绑定事件到所有的选出来的元素上
*   它不会绑定到在它执行完后动态添加的那些元素上
*   当元素很多时，会出现效率问题
*   当页面加载完的时候，你才可以进行 bind()，所以可能产生效率问题

  

#### .live()

`$(``'a'``).live(``'click'``,``function``() { alert(``"That tickles!"``) });`　　

　　jQuery 绑定处理函数到 $(document) 元素，并把 ‘click’ 和 ‘a’ 作为函数的参数。有事件冒泡到 document 节点的时候，检查这个事件是不是 click 事件，target element 能不能匹配 ‘a’ css 选择器，如果两个条件都是 true，处理函数执行。

live 方法也可以绑定到指定的元素（或者说 “上下文 (context)”）而不用绑定到 document，比如：

$('a',`$(``'#container'``)`[0]).live(...);　

优点：

*   这里仅有一次的事件绑定，绑定到 document 上而不像. bind() 那样给所有的元素挨个绑定
*   那些动态添加的 elemtns 依然可以触发那些早先绑定的事件，因为事件真正的绑定是在 document 上
*   你可以在 document ready 之前就可以绑定那些需要的事件

缺点：

*   从 1.7 开始已经不被推荐了，所以你也要开始逐步淘汰它了。
*   Chaining 没有被正确的支持
*   当使用 event.stopPropagation() 是没用的，因为都要到达 document
*   因为所有的 selector/event 都被绑定到 document, 所以当我们使用 matchSelector 方法来选出那个事件被调用时，会非常慢
*   当发生事件的元素在你的 DOM 树中很深的时候，会有 performance 问题

#### .delegate()

`$(``'#container'``).delegate(``'a'``,``'click'``,``function``() { alert(``"That tickles!"``) });`　　

jQuery 扫描文档找到 $(‘#container’)，绑定处理函数到他的 click 事件，’a’ css 选择器作为函数的参数。当有事件冒泡到 $(‘#container’)，检查事件是不是 click，并检查 target element 是不是匹配 css 选择器，如果两者都符合，执行函数。

注意这次和 .live() 方法很相似，除了把事件绑定到特定元素与跟元素的区别。精明的 JS’er 或许会总结成 $(‘a’).live() == $(document).delegate(‘a’)，真的是这样吗？ 不，不全是。

优点：

*   你可以选择你把这个事件放到那个元素上了
*   chaining 被正确的支持了
*   jQuery 仍然需要迭代查找所有的 selector/event data 来决定那个子元素来匹配，但是因为你可以决定放在那个根元素上，所以可以有效的减小你所要查找的元素。
*   可以用在动态添加的元素上

缺点：

*   需要查找那个那个元素上发生了那个事件了，尽管比 document 少很多了，不过，你还是得浪费时间来查找。

#### 为什么 .delegate() 比 .live() 好

jQuery 的 delegate 方法比 live 方法更应该成为首选有一个原因。考虑以下的场景：

`$(``'a'``).live(``'click'``,``function``() { blah() });` `// or` `$(document).delegate(``'a'``,``'click'``,``function``() { blah() });` 　　

#### 速度

上面第二个执行比第一个快，因为第一个会遍历整个文档查找 $(‘a’) 元素，并保存为 jQuery 对象，但是 live 方法只需要传一个字符串参数’a'而已，$() 方法并不知道我们会用链式表达式在后面用上. live()。

delegate 方法就只需要找到并存贮 $(document) 元素就够了。

**        有一种 hack 是在 $(document).ready() 之外调用 live 方法，这样就会立即执行。这时候 DOM 还没有填充，也就不会查找元素或创建 jQuery 对象。**  

#### 灵活性和链式语法

这方面 live 方法依然令人费解。想一下，它链在 $(‘a’) 对象，但实际上是在 $(document) 对象起作用。因为这个原因，在链式表达式中使用 live 让人很不安，我觉得 live 方法变成一个全局的 jQuery 方法 $.live(‘a’,…) 会更有意义。

#### 只支持 css 选择器

最后，live 方法有一个最大的缺点，只能用 css 选择器，用起来很不方便。

#### 为什么使用 .live() 或 .delegate() 而不用 .bind()

最后，bind 方法看起来更清晰，更直接，是吗？但是这里有两个原因我们推荐 delegate 或 live:

*   绑定事件处理函数到还不存在 DOM 中的元素。 bind 方法直接绑定函数到每个单独的元素，不能绑定到还没有添加到页面里的元素，如果你写了 $(‘a’).bind(…)，然后用 ajax 给页面增加了新的链接，新添加的链接不会绑定事件。live 或 delegate 或者其它绑定到祖先元素的事件，让现在有的元素，或者以后增的元素都可以使用。
*   绑定处理函数到一个元素或者少数几个元素，监听后代元素，而不是绑定 100 个相同的处理函数到单独的元素。这样更有性能优势。

  

.On()

 其实. bind(), .live(), .delegate() 都是通过. on() 来实现的，.unbind(), .die(), .undelegate(), 也是一样的都是通过. off() 来实现的，

看一下，我们用如何用. on() 来改写前面通过 .bind(), .live(), .delegate() 所注册的事件：  


```
$( "#members li a" ).on( "click", function( e ) {} ); 
$( "#members li a" ).bind( "click", function( e ) {} ); 

// Live
$( document ).on( "click", "#members li a", function( e ) {} ); 
$( "#members li a" ).live( "click", function( e ) {} );

// Delegate
$( "#members" ).on( "click", "li a", function( e ) {} ); 
$( "#members" ).delegate( "li a", "click", function( e ) {} );

```

  

优点：

*   提供了一种统一绑定事件的方法
*   仍然提供了. delegate() 的优点，当然如果需要你也可以直接用. bind()

缺点：

*   也许会对你产生一些困扰，因为它隐藏了一前面我们所介绍的三种方法的细节。

结论：

*   用. bind() 的代价是非常大的，它会把相同的一个事件处理程序 hook 到所有匹配的 DOM 元素上
*   不要再用. live() 了，它已经不再被推荐了，而且还有许多问题
*   .delegate() 会提供很好的方法来提高效率，同时我们可以添加一事件处理方法到动态添加的元素上。
*   我们可以用. on() 来代替上述的 3 种方法

  
  
  

#### 阻止冒泡

最后注意一下事件冒泡。通常我们能用这样的方法阻止其他处理函数：

`$(``'a'``).bind(``'click'``,``function``(){  ` `e.preventDefault();   ` `//or   ` `e.stopPropagation();` `})` 　　

但是在这里，用 live 或 delegate 方法绑定的事件会一直传递到事件真正绑定的地方才会执行。这时其他的函数已经执行过了。

**阻止事件冒泡：**

if (window.event) {  
  e.cancelBubble=true;// ie6、7、8 下阻止冒泡  
 } else {  
  //e.preventDefault();  
  e.stopPropagation();// 其它标准浏览器下阻止冒泡  
 }