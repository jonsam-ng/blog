---
title: Vue 是如何渲染页面的，渲染过程以及原理代码
---

### 一、前言

1.  Vue.[js](http://lib.csdn.net/base/javascript "JavaScript知识库") 框架是目前比较火的 MVVM 框架之一，简单易上手的学习曲线，友好的官方文档，配套的构建工具，让 Vue.js 在 2016 大放异彩，大有赶超 [React](http://lib.csdn.net/base/react "React知识库") 之势。前不久 Vue.js 2.0 正式版已出，在体积优化（相比 1.0 减少了 50%）、性能提升（相比 1.0 提升 60%）、API 优化等各方面都更上一层楼；
  
2.  本文是系列文章，主要想通过对于 Vue.js 2.0 源码的分析，从代码层面解析 Vue.js 的实现原理，帮助读者能够更深入地理解整个框架的思想。此篇文章主要介绍前端渲染部分；
  
3.  不足之处还请批评指正，欢迎一起交流学习。
  

### 二、Vue 的初始化

我们在使用 Vue.js 的时候，最基本的一个使用，就是在 HTML 引入 Vue.js 的库文件，并写如下一段代码：

```
1.var app = new Vue({
2.  el: '#app',
3.  data: {
4.    message: 'Hello Vue!'
5.  }
6.})

```

new Vue，本质就是生成一个 Vue 的对象，我们来了解一下这个生成 Vue 对象的过程是怎样的：

首先，Vue 的入口是 / src/entries/web-runtime-with-compiler.js，这是由 config.js 配置文件决定的。

![](http://img.blog.csdn.net/20161230165915011)

这个入口文件中 import 了很多文件，其中有一条主要的脉络：

`/src/entries/web-runtime-with-compiler.js`   
引用了`/src/entries/web-runtime.js`   
引用了`/src/core/index.js`   
引用了`/src/core/instance/index.js`

其中`/src/core/instance/index.js`是最核心的初始化代码，其中：

![](http://img.blog.csdn.net/20161230165934996)

红框部分，就是整个 Vue 的类的核心方法。其含义给读者解读一下：

```
1.//初始化的入口，各种初始化工作
2.initMixin(Vue) 
3.//数据绑定的核心方法，包括常用的$watch方法
4.stateMixin(Vue)
5.//事件的核心方法，包括常用的$on，$off，$emit方法
6.eventsMixin(Vue)
7.//生命周期的核心方法
8.lifecycleMixin(Vue)
9.//渲染的核心方法，用来生成render函数以及VNode
10.renderMixin(Vue)

```

其中 new Vue 就是执行下面的这个函数：

![](http://img.blog.csdn.net/20161230165949652)

`_init`方法就是 initMixin 中的`_init`方法。

![](http://img.blog.csdn.net/20161230170002621)

至此，程序沿着这个`_init`方法继续走下去。

### 三、Vue 的渲染逻辑——Render 函数

在定义完成 Vue 对象的初始化工作之后，本文主要是讲渲染部分，那么我们接上面的逻辑，看 Vue.js 是如何渲染页面的。在上图中我们看到有一个`initRender`的方法：

![](http://img.blog.csdn.net/20161230170015591)

在该方法中会执行红框部分的内容：

![](http://img.blog.csdn.net/20161230170030918)

而`$mount`方法就是整个渲染过程的起始点。具体定义是在`/src/entries/web-runtime-with-compiler.js`中，根据代码整理成流程图：

![](http://img.blog.csdn.net/20161230170042715)

由此图可以看到，在渲染过程中，提供了三种渲染模式，自定义 Render 函数、template、el 均可以渲染页面，也就是对应我们使用 Vue 时，三种写法：

1. 自定义 Render 函数

```
1.Vue.component('anchored-heading', {
2.    render: function (createElement) {
3.        return createElement(
4.            'h' + this.level,   // tag name 标签名称
5.            this.$slots.default // 子组件中的阵列
6.        )
7.    },
8.    props: {
9.        level: {
10.            type: Number,
11.            required: true
12.        }
13.    }
14.})

```

2. template 写法

```
1.var vm = new Vue({
2.    data: {
3.        // 以一个空值声明 `msg`
4.        msg: ''
5.    },
6.    template: '<div>{{msg}}</div>'
7.})

```

3. el 写法（这个就是入门时最基本的写法）

```
1.var app = new Vue({
2.    el: '#app',
3.    data: {
4.        message: 'Hello Vue!'
5.    }
6.})

```

这三种渲染模式最终都是要得到 Render 函数。只不过用户自定义的 Render 函数省去了程序分析的过程，等同于处理过的 Render 函数，而普通的 template 或者 el 只是字符串，需要解析成 AST，再将 AST 转化为 Render 函数。

记住一点，无论哪种方法，都要得到 Render 函数。

我们在使用过程中具体要使用哪种调用方式，要根据具体的需求来。

*   如果是比较简单的逻辑，使用 template 和 el 比较好，因为这两种都属于声明式渲染，对用户理解比较容易，但灵活性比较差，因为最终生成的 Render 函数是由程序通过 AST 解析优化得到的；
  
*   而使用自定义 Render 函数相当于人已经将逻辑翻译给程序，能够胜任复杂的逻辑，灵活性高，但对于用户的理解相对差点。
  

### 四、Vue 的渲染逻辑——VNode 对象 & patch 方法

根据上面的结论，我们无论怎么渲染，最终会得到 Render 函数，而 Render 函数的作用是什么呢？我们看到在`/src/core/instance/lifecycle.js`中有这么一段代码：

```
1.vm._watcher = new Watcher(vm, () => {
2.    vm._update(vm._render(), hydrating)
3.}, noop);

```

意思就是，通过 Watcher 的绑定，每当数据发生变化时，执行`_update`的方法，此时会先执行`vm._render()`，在这个`vm._render()`中，我们的 Render 函数会执行，而得到 VNode 对象。

![](http://img.blog.csdn.net/20161230170106183)

VNode 对象是什么？VNode 就是 Vue.js 2.0 中的 Virtual DOM，在 Vue.js 2.0 中，相较 Vue.js 1.0 引入了 Virtual DOM 的概念，这也是 Vue.js 2.0 性能提升的一大关键。Virtual DOM 有多种实现方式，但基本思路都是一样的，分为两步：

1. [JavaScript](http://lib.csdn.net/base/javascript "JavaScript知识库") 模拟 DOM 模型树

在 Vue.js 2.0 中 [javascript](http://lib.csdn.net/base/javascript "JavaScript知识库") 模拟 DOM 模型树就是 VNode，Render 函数执行后都会返回 VNode 对象，为下一步操作做准备。在`/src/core/vdom/vnode.js`中，我们可以看到 VNode 的具体[数据结构](http://lib.csdn.net/base/datastructure "算法与数据结构知识库")：

![](http://img.blog.csdn.net/20161230170119905)

VNode 的数据结构中还有 VNodeData、VNodeDirective、VNodeComponentOptions，这些数据结构都是对 DOM 节点的一些描述，本文不一一介绍。读者可以根据源码来理解这些数据结构。（PS：Vue.js 使用了 flow，标识了参数的静态类型，对理解代码很有帮助 ^_^）

2. DOM 模型树通过 DOM Diff [算法](http://lib.csdn.net/base/datastructure "算法与数据结构知识库")查找差异，将差异转为真正 DOM 节点

我们知道 Render 函数执行生成了 VNode，而 VNode 只是 Virtual DOM，我们还需要通过 DOM Diff 之后，来生成真正的 DOM 节点。在 Vue.js 2.0 中，是通过`/src/core/vdom/patch.js`中的`patch(oldVnode, vnode ,hydrating)`方法来完成的。

该方法有三个参数 oldVnode 表示旧 VNode，vnode 表示新 VNode，hydrating 表示是否直接使用服务端渲染的 DOM 元素，这个本文不作讨论，在服务端渲染篇再详细介绍。

其主要逻辑为当 VNode 为真实元素或旧的 VNode 和新的 VNode 完全相同时，直接调用 createElm 方法生成真实的 DOM 树，当 VNode 新旧存在差异时，则调用 patchVnode 方法，通过比较新旧 VNode 节点，根据不同的状态对 DOM 做合理的添加、删除、修改 DOM（这里的 Diff 算法有兴趣的读者可以自行阅读 patchVnode 方法，鉴于篇幅不再赘述），再调用 createElm 生成真实的 DOM 树。

### 五、Vue 的渲染小结

回过头来看，这里的渲染逻辑并不是特别复杂，核心关键的几步流程还是非常清晰的：

1.  new Vue，执行初始化
2.  挂载`$mount`方法，通过自定义 Render 方法、template、el 等生成 Render 函数
3.  通过 Watcher 监听数据的变化
4.  当数据发生变化时，Render 函数执行生成 VNode 对象
5.  通过 patch 方法，对比新旧 VNode 对象，通过 DOM Diff 算法，添加、修改、删除真正的 DOM 元素

至此，整个 new Vue 的渲染过程完毕。