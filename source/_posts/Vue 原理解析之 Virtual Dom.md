---
title: Vue 原理解析之 Virtual Dom
---

`DOM`是文档对象模型 (`Document Object Model`) 的简写，在浏览器中我们可以通过 js 来操作`DOM`，但是这样的操作性能很差，于是`Virtual Dom`应运而生。我的理解，`Virtual Dom`就是在 js 中模拟`DOM`对象树来优化`DOM`操作的一种技术或思路。

本文将对于 Vue 框架 2.1.8 版本中使用的`Virtual Dom`进行分析。

VNode 对象
--------

一个 VNode 的实例对象包含了以下属性

*   `tag`: 当前节点的标签名
  
*   `data`: 当前节点的数据对象，具体包含哪些字段可以参考 vue 源码`types/vnode.d.ts`中对`VNodeData`的定义  
    ![](https://segmentfault.com/img/bVITKL?w=419&h=458)
    
*   `children`: 数组类型，包含了当前节点的子节点
  
*   `text`: 当前节点的文本，一般文本节点或注释节点会有该属性
  
*   `elm`: 当前虚拟节点对应的真实的 dom 节点
  
*   `ns`: 节点的 namespace
  
*   `context`: 编译作用域
  
*   `functionalContext`: 函数化组件的作用域
  
*   `key`: 节点的 key 属性，用于作为节点的标识，有利于 patch 的优化
  
*   `componentOptions`: 创建组件实例时会用到的选项信息
  
*   `child`: 当前节点对应的组件实例
  
*   `parent`: 组件的占位节点
  
*   `raw`: raw html
  
*   `isStatic`: 静态节点的标识
  
*   `isRootInsert`: 是否作为根节点插入，被`<transition>`包裹的节点，该属性的值为`false`
  
*   `isComment`: 当前节点是否是注释节点
  
*   `isCloned`: 当前节点是否为克隆节点
  
*   `isOnce`: 当前节点是否有`v-once`指令
  

VNode 分类
--------

![](https://segmentfault.com/img/bVITTR?w=495&h=540)

`VNode`可以理解为 vue 框架的虚拟 dom 的基类，通过`new`实例化的`VNode`大致可以分为几类

*   `EmptyVNode`: 没有内容的注释节点
  
*   `TextVNode`: 文本节点
  
*   `ElementVNode`: 普通元素节点
  
*   `ComponentVNode`: 组件节点
  
*   `CloneVNode`: 克隆节点，可以是以上任意类型的节点，唯一的区别在于`isCloned`属性为`true`
  
*   `...`
  

createElement 解析
----------------

```
const SIMPLE_NORMALIZE = 1
const ALWAYS_NORMALIZE = 2

function createElement (context, tag, data, children, normalizationType, alwaysNormalize) {
  // 兼容不传data的情况
  if (Array.isArray(data) || isPrimitive(data)) {
    normalizationType = children
    children = data
    data = undefined
  }
  // 如果alwaysNormalize是true
  // 那么normalizationType应该设置为常量ALWAYS_NORMALIZE的值
  if (alwaysNormalize) normalizationType = ALWAYS_NORMALIZE
  // 调用_createElement创建虚拟节点
  return _createElement(context, tag, data, children, normalizationType)
}

function _createElement (context, tag, data, children, normalizationType) {
  /**
   * 如果存在data.__ob__，说明data是被Observer观察的数据
   * 不能用作虚拟节点的data
   * 需要抛出警告，并返回一个空节点
   * 
   * 被监控的data不能被用作vnode渲染的数据的原因是：
   * data在vnode渲染过程中可能会被改变，这样会触发监控，导致不符合预期的操作
   */
  if (data && data.__ob__) {
    process.env.NODE_ENV !== 'production' && warn(
      `Avoid using observed data object as vnode data: ${JSON.stringify(data)}\n` +
      'Always create fresh vnode data objects in each render!',
      context
    )
    return createEmptyVNode()
  }
  // 当组件的is属性被设置为一个falsy的值
  // Vue将不会知道要把这个组件渲染成什么
  // 所以渲染一个空节点
  if (!tag) {
    return createEmptyVNode()
  }
  // 作用域插槽
  if (Array.isArray(children) &&
      typeof children[0] === 'function') {
    data = data || {}
    data.scopedSlots = { default: children[0] }
    children.length = 0
  }
  // 根据normalizationType的值，选择不同的处理方法
  if (normalizationType === ALWAYS_NORMALIZE) {
    children = normalizeChildren(children)
  } else if (normalizationType === SIMPLE_NORMALIZE) {
    children = simpleNormalizeChildren(children)
  }
  let vnode, ns
  // 如果标签名是字符串类型
  if (typeof tag === 'string') {
    let Ctor
    // 获取标签名的命名空间
    ns = config.getTagNamespace(tag)
    // 判断是否为保留标签
    if (config.isReservedTag(tag)) {
      // 如果是保留标签,就创建一个这样的vnode
      vnode = new VNode(
        config.parsePlatformTagName(tag), data, children,
        undefined, undefined, context
      )
      // 如果不是保留标签，那么我们将尝试从vm的components上查找是否有这个标签的定义
    } else if ((Ctor = resolveAsset(context.$options, 'components', tag))) {
      // 如果找到了这个标签的定义，就以此创建虚拟组件节点
      vnode = createComponent(Ctor, data, context, children, tag)
    } else {
      // 兜底方案，正常创建一个vnode
      vnode = new VNode(
        tag, data, children,
        undefined, undefined, context
      )
    }
    // 当tag不是字符串的时候，我们认为tag是组件的构造类
    // 所以直接创建
  } else {
    vnode = createComponent(tag, data, context, children)
  }
  // 如果有vnode
  if (vnode) {
    // 如果有namespace，就应用下namespace，然后返回vnode
    if (ns) applyNS(vnode, ns)
    return vnode
  // 否则，返回一个空节点
  } else {
    return createEmptyVNode()
  }
}

```

简单的梳理了一个流程图，可以参考下

![](https://segmentfault.com/img/bVIT2U?w=609&h=705)

patch 原理
--------

`patch`函数的定义在`src/core/vdom/patch.js`中，我们先来看下这个函数的逻辑

`patch`函数接收 6 个参数：

*   `oldVnode`: 旧的虚拟节点或旧的真实 dom 节点
  
*   `vnode`: 新的虚拟节点
  
*   `hydrating`: 是否要跟真是 dom 混合
  
*   `removeOnly`: 特殊 flag，用于`<transition-group>`组件
  
*   `parentElm`: 父节点
  
*   `refElm`: 新节点将插入到`refElm`之前
  

`patch`的策略是：

1.  如果`vnode`不存在但是`oldVnode`存在，说明意图是要销毁老节点，那么就调用`invokeDestroyHook(oldVnode)`来进行销毁
  
2.  如果`oldVnode`不存在但是`vnode`存在，说明意图是要创建新节点，那么就调用`createElm`来创建新节点
  
3.  当`vnode`和`oldVnode`都存在时
  
    *   如果`oldVnode`和`vnode`是同一个节点，就调用`patchVnode`来进行`patch`
      
    *   当`vnode`和`oldVnode`不是同一个节点时，如果`oldVnode`是真实 dom 节点或`hydrating`设置为`true`，需要用`hydrate`函数将虚拟 dom 和真是 dom 进行映射，然后将`oldVnode`设置为对应的虚拟 dom，找到`oldVnode.elm`的父节点，根据 vnode 创建一个真实 dom 节点并插入到该父节点中`oldVnode.elm`的位置
      
    
    这里面值得一提的是`patchVnode`函数，因为真正的 patch 算法是由它来实现的（patchVnode 中更新子节点的算法其实是在`updateChildren`函数中实现的，为了便于理解，我统一放到`patchVnode`中来解释）。
    

`patchVnode`算法是：

1.  如果`oldVnode`跟`vnode`完全一致，那么不需要做任何事情
  
2.  如果`oldVnode`跟`vnode`都是静态节点，且具有相同的`key`，当`vnode`是克隆节点或是`v-once`指令控制的节点时，只需要把`oldVnode.elm`和`oldVnode.child`都复制到`vnode`上，也不用再有其他操作
  
3.  否则，如果`vnode`不是文本节点或注释节点
  
    *   如果`oldVnode`和`vnode`都有子节点，且 2 方的子节点不完全一致，就执行更新子节点的操作（这一部分其实是在`updateChildren`函数中实现），算法如下
      
        *   分别获取`oldVnode`和`vnode`的`firstChild`、`lastChild`，赋值给`oldStartVnode`、`oldEndVnode`、`newStartVnode`、`newEndVnode`
          
        *   如果`oldStartVnode`和`newStartVnode`是同一节点，调用`patchVnode`进行`patch`，然后将`oldStartVnode`和`newStartVnode`都设置为下一个子节点，重复上述流程  
            ![](https://segmentfault.com/img/bVIVBX?w=667&h=204)
            
        *   如果`oldEndVnode`和`newEndVnode`是同一节点，调用`patchVnode`进行`patch`，然后将`oldEndVnode`和`newEndVnode`都设置为上一个子节点，重复上述流程  
            ![](https://segmentfault.com/img/bVIVCG?w=676&h=221)
            
        *   如果`oldStartVnode`和`newEndVnode`是同一节点，调用`patchVnode`进行`patch`，如果`removeOnly`是`false`，那么可以把`oldStartVnode.elm`移动到`oldEndVnode.elm`之后，然后把`oldStartVnode`设置为下一个节点，`newEndVnode`设置为上一个节点，重复上述流程  
            ![](https://segmentfault.com/img/bVIVEu?w=826&h=224)
            
        *   如果`newStartVnode`和`oldEndVnode`是同一节点，调用`patchVnode`进行`patch`，如果`removeOnly`是`false`，那么可以把`oldEndVnode.elm`移动到`oldStartVnode.elm`之前，然后把`newStartVnode`设置为下一个节点，`oldEndVnode`设置为上一个节点，重复上述流程  
            ![](https://segmentfault.com/img/bVIVFk?w=864&h=214)
            
        *   如果以上都不匹配，就尝试在`oldChildren`中寻找跟`newStartVnode`具有相同`key`的节点，如果找不到相同`key`的节点，说明`newStartVnode`是一个新节点，就创建一个，然后把`newStartVnode`设置为下一个节点
          
        *   如果上一步找到了跟`newStartVnode`相同`key`的节点，那么通过其他属性的比较来判断这 2 个节点是否是同一个节点，如果是，就调用`patchVnode`进行`patch`，如果`removeOnly`是`false`，就把`newStartVnode.elm`插入到`oldStartVnode.elm`之前，把`newStartVnode`设置为下一个节点，重复上述流程  
            ![](https://segmentfault.com/img/bVIVJb?w=869&h=227)
            
        *   如果在`oldChildren`中没有寻找到`newStartVnode`的同一节点，那就创建一个新节点，把`newStartVnode`设置为下一个节点，重复上述流程
          
        *   如果`oldStartVnode`跟`oldEndVnode`重合了，并且`newStartVnode`跟`newEndVnode`也重合了，这个循环就结束了
        
    *   如果只有`oldVnode`有子节点，那就把这些节点都删除
      
    *   如果只有`vnode`有子节点，那就创建这些子节点
      
    *   如果`oldVnode`和`vnode`都没有子节点，但是`oldVnode`是文本节点或注释节点，就把`vnode.elm`的文本设置为空字符串
    
4.  如果`vnode`是文本节点或注释节点，但是`vnode.text != oldVnode.text`时，只需要更新`vnode.elm`的文本内容就可以
  

生命周期
----

`patch`提供了 5 个生命周期钩子，分别是

*   `create`: 创建 patch 时
  
*   `activate`: 激活组件时
  
*   `update`: 更新节点时
  
*   `remove`: 移除节点时
  
*   `destroy`: 销毁节点时
  

这些钩子是提供给 Vue 内部的`directives`/`ref`/`attrs`/`style`等模块使用的，方便这些模块在 patch 的不同阶段进行相应的操作，这里模块定义在`src/core/vdom/modules`和`src/platforms/web/runtime/modules`2 个目录中

`vnode`也提供了生命周期钩子，分别是

*   `init`: vdom 初始化时
  
*   `create`: vdom 创建时
  
*   `prepatch`: patch 之前
  
*   `insert`: vdom 插入后
  
*   `update`: vdom 更新前
  
*   `postpatch`: patch 之后
  
*   `remove`: vdom 移除时
  
*   `destroy`: vdom 销毁时
  

vue 组件的生命周期底层其实就依赖于 vnode 的生命周期，在`src/core/vdom/create-component.js`中我们可以看到，vue 为自己的组件 vnode 已经写好了默认的`init`/`prepatch`/`insert`/`destroy`，而 vue 组件的`mounted`/`activated`就是在`insert`中触发的，`deactivated`就是在`destroy`中触发的

实践
--

在 Vue 里面，`Vue.prototype.$createElement`对应 vdom 的`createElement`方法，`Vue.prototype.__patch__`对应`patch`方法，我写了个简单的 demo 来验证下功能

<p data-height="265" data-theme-id="0" data-slug-hash="rjZKZz" data-default-tab="html,result" data-user="JoeRay" data-embed-version="2" data-pen-title="Vue Virtual Dom">See the Pen [Vue Virtual Dom](http://codepen.io/JoeRay/pen/rjZKZz/) 点击预览 by zhulei ([@JoeRay](http://codepen.io/JoeRay) 点击预览) on [CodePen](http://codepen.io).</p>  
<script async src="[https://production-assets.cod...](https://production-assets.codepen.io/assets/embed/ei.js%22></script>)