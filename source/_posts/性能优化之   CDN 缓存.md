---
title: 性能优化之   CDN 缓存
---
2018.08.07 19:44:38 字数 1442 阅读 458

> 网站性能优化是一个大活儿，按工种划分的话，分前端、后端和 db 等，作为一名前端工程师，这系列文章只聊前端工程师应该知道的关于网站性能优化的那些事儿。

> cdn——维基百科给出的解释是：内容分发网络（Content delivery network 或 Content distribution network，缩写：CDN）。简单来说它主要的工作是把我们需要被分发的内容分发到世界各地的各个节点上，让世界各地的人都可以在距离最近的网络节点拿到想要拿到的内容，减少网络传输距离从而达到加速的目的（需要提过资源绝对地址告诉 cdn 厂商，让厂商去智能拉取）。

![](http://upload-images.jianshu.io/upload_images/11015875-6fd7a262fbd61594.png)

image.png

> 如图 1 所示：当用户发起内容请求时，通过 cdn 厂商的智能 DNS 域名解析拿到 cdn 厂商边缘节点服务器的 ip（cdn 厂商会在运营商注册），然后向边缘节点服务器发起请求，请求内容数据 (这件事情由浏览器完成)，边缘节点会检测当前节点是否有数据，如果没有就去 front（父级节点，父级可能还会有父级节点，不同的网络环境策略会略有不同）节点要，如果还找不到就去源站拿，并依次序返回。如果某个边缘节点可以找到，会先校验内容有效期，当确定有效期之后返回给用户。注：“有效期校验有多种方式和 http 协议相关，内容比较多，我们留到下一期（前端 - 网站性能优化——缓存）再聊。”

> 知道了 cdn 是怎么一回事儿之后，咱们再聊聊前端如何利用 cdn 来优化网站性能。前端需要被加速的文件大致包括 js、css、图片、视频、和页面等文件, 页面文件比较特殊 (有动态和静态之分) 我们稍后再聊，先聊聊 js、css、图片和视频文件。这些文件和页面（html\jsp\aspx 等）最大的区别是：这些文件都是静态的，改动较小，了解上面 cdn 工作原理之后我们就可以发现这类静态文件最适合做 cdn 加速。我们把这些静态文件通过 cdn 分发到全国乃至世界的各个节点，用户就可以在距离最近的边缘节点拿到所需要的内容，从而提升内容下载速度加快网页打开速度达到性能优化的目的。接下来我们聊聊页面，页面分动态页面 (如：jsp 等) 和静态页面（html）。

```
>    动态页面：当收到用户请求时服务器会在服务端对页面进行一次后台渲染把数据渲染到页面之后再返回给用户（当然，服务端也可以做缓存）。

    静态页面：收到用户请求时，服务端不做渲染工作直接返回给用户。



```

> 动态页面是不适合做 cdn 加速的。原因：参照上面讲的 cdn 工作原理，由于页面是动态的，内容的有效期就比较活跃。假如我们对动态页面做了 cdn 加速，那么场景应该是这样的：用户——> 边缘节点（验证有效期发现失效）——> 源站。经过这个过程才能拿到页面，这样并没有起到加速的作用反而更慢了，那我们还不如直接去源站拿 (当然我们可以要求 cdn 厂商做定制化开发)。

> 静态页面（html）也是比较适合做 cdn 加速的。但是静态页面也分纯静态页面和非纯静态页面。

```
    纯静态页面：只指直接通过浏览器输入地址就可访问的，后台服务器没有做鉴权登录等认证。
    非纯静态页面：需要通过鉴权登录等认证才能访问的html页面。



```

> 如果非纯静态页面做 cdn 加速，参照上述的 cdn 工作原理，会出现用户没有通过任何服务器鉴权认证也可以正常在 cdn 边缘节点拿到想要访问页面（要求 cdn 厂商做定制化开发也可以避免这种情况）。

> 不过我们可以采用前后端彻底分离的方式（js 发 ajax 请求的方式验证用户是否可以通过鉴权）来解决动态页面和非纯静态页面不适合做 cdn 加速的问题。

```
    注：前后端分离实践我会在另外一个系列文章中分享



```

> 当你的网站使用上 cdn 加速，我相信你的页面加载速度会有一个非常可观提升。

> 另外有一个我们不得不关注的问题我在想这篇文章中说一下，那就是：“浏览器对同一 ip 进行请求的最大并发连接数的问题”。不同浏览器的并发数量不一样：IE11 、IE10 、chrome、Firefox 的并发连接数是 6 个，IE9 是 10 个（如何查看浏览器并发连接数请自行 google）。

> 如果页面静态资源（图片等）过多（大于 6 个）会存在资源请求等待的情况。目前现实状况是大多用户带宽越来越大，但是咱们的静态资源并非那么大，很多文件都是几 k 或者几十 k，6 个文件加起来都小于带宽。这样就导致了资源的浪费。解决方案是：用多个不同 IP 的服务器来存储这些文件，并在页面中通过绝对路径的方式引用（要求同一 IP 的文件不超过 6 个）。这样就可以尽可能的减少资源请求等待的情况。

> 总结：不同 ip 服务器存储静态文件再结合上 cdn 加速。页面加载速度就又会上升一个档次。

[![](https://upload.jianshu.io/users/upload_avatars/11015875/c8e678e3-b936-4af9-98e1-e63d211a7fb2?imageMogr2/auto-orient/strip|imageView2/1/w/120/h/120/format/webp)](https://www.jianshu.com/u/c817dc83befd)

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

*   前端开发者丨 http 请求 https:www.rokub.com 前言见解有限， 如有描述不当之处， 请帮忙指出，...
  
*   pdf 下载地址：Java 面试宝典 第一章内容介绍 20 第二章 JavaSE 基础 21 一、Java 面向对象 21 ...
  
*   Android 自定义 View 的各种姿势 1 Activity 的显示之 ViewRootImpl 详解 Activity...
  
*   北京时间 6 月 22 日 20:00，世界杯小组赛第二轮继续进行，五届世界杯得主巴西队将迎战哥斯达黎加队。首场比赛，巴西没...