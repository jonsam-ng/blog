---
title: js 中的预加载与懒加载（延迟加载）
---
  js 中加载分两种：预加载与延迟加载

**  一、  预加载，增强用户的体验，但会加载服务器的负担。一般会使用多种 [CSS](http://www.2cto.com/kf/qianduan/css/)(background)、JS(Image)、HTML(<img />) 。**

        1、什么是预加载

             提前加载图片，当用户需要查看时可直接从本地缓存中渲染

        2、实现预加载的方法

              a、单纯的 css 实现

                   可通过 CSS 的 background 属性将图片预加载到屏幕外的背景上。只要这些图片的路径保持不变，当它们在 Web 页面的其他地方被调用时，浏览器就会在渲染过程中使用预加载（缓存）的图片。简单、高效，不需要任何 JavaScript。

```
#preload-01 { background: url(http://domain.tld/image-01.png) no-repeat -9999px -9999px; } 

```

              b、单纯的 js 预加载图片

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
<div>  
    <script type="text/javascript">  
        <!--//--><![CDATA[//><!--  
            var images = new Array()  
            function preload() {  
                for (i = 0; i < preload.arguments.length; i++) {  
                    images[i] = new Image()  
                    images[i].src = preload.arguments[i]  
                }  
            }  
            preload(  
                "http://domain.tld/gallery/image-001.jpg",  
                "http://domain.tld/gallery/image-002.jpg",  
                "http://domain.tld/gallery/image-003.jpg"  
            )  
        //--><!]]>  
    </script>  
</div>

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

           c、使用 ajax 实现预加载

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
window.onload = function() {  
    setTimeout(function() {  
        // XHR to request a JS and a CSS  
        var xhr = new XMLHttpRequest();  
        xhr.open('GET', 'http://domain.tld/preload.js');  
        xhr.send('');  
        xhr = new XMLHttpRequest();  
        xhr.open('GET', 'http://domain.tld/preload.css');  
        xhr.send('');  
        // preload image  
        new Image().src = "http://domain.tld/preload.png";  
    }, 1000);  
};

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

**参考网址：http://web.jobbole.com/86785/**

　**　二、延迟加载（懒加载）**

          1、什么是懒加载

               懒加载又称延迟加载。

               当访问一个页面时，先把 img 元素或者其他元素的背景图片替换成一张大小 1*1px 图片的路径（只需要请求一次的占位图），只有当图片出现在浏览器的可视区域内时，才设置图片真正的路径，让图片显示出来，这就是图片的懒加载。

          2、懒加载的实现原理

                页面中 img 元素，如果没有 src 属性，浏览器就不会发出请求去下载图片，只有通过 js 设置图片路径，浏览器才会发送请求；

                懒加载的原理是先在页面中把所有的图片统一使用一张占位图进行占位，把真正的路径存在元素的‘data-url’属性中，要使用的时候，在设置。

         3、懒加载的实现步骤

              1)首先，不要将图片地址放到 src 属性中，而是放到其它属性 (data-original) 中。  
              2) 页面加载完成后，根据 scrollTop 判断图片是否在用户的视野内，如果在，则将 data-original 属性中的值取出存放到 src 属性中。  
              3) 在滚动事件中重复判断图片是否进入视野，如果进入，则将 data-original 属性中的值取出存放到 src 属性中。

         4、懒加载的优点

              页面加载速度快、可以减轻服务器的压力、节约了流量, 用户体验好

        **   三、懒加载与预加载的对比**

               1、概念

                   懒加载也叫延迟加载：js 图片延迟加载，延迟加载图片或者符合某些条件是才加载某些图片；

                   预加载：提前加载图片，当用户需要查看时可直接从本地缓存中渲染。（base64 小图片可以通过 css 保存）

                2、区别

                  两种技术的本质：两者的行为相反，一个是提前加载，一个是迟缓甚至不加载。懒加载会对前端有一定的缓解压力作用，预加载则会增加前端的压力。

                3、懒加载的意义及实现方式：

                     懒加载的主要目的是优化前端性能，减少请求数或延迟请求数。

                    方法：

                       a、纯粹的延迟加载，使用 setTimeOut 或者 setInterval 进行加载延迟；

                       b、条件加载，符合某些条件，或者触发了某些事件才开始异步下载；

                       c、可视区加载，即仅加载可以看到的区域，监控滚动条实现。

                4、预加载的意义及实现方式

                     预加载是牺牲前端性能，换取用户体验，使用户的操作得到最快的反映。

                    方法：

                   比如：用 CSS 和 JavaScript 实现预加载；仅使用 JavaScript 实现预加载；使用 Ajax 实现预加载。

                  常用的是 new Image(); 设置其 src 来实现预载，再使用 onload() 方法回调预加载完成事件。只要浏览器吧图片下载到本地，src 就会使用缓存，这是最基本的预加载方法。当 image 下载完图片后，会得到宽和高，因此可以在预加载钱得到图片的大小 (方法是用记时器轮循宽高变化)。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
function loadImage(url,callback) {
    var img = new Image();
    
    img.src = url;
 
    if(img.complete) {  // 如果图片已经存在于浏览器缓存，直接调用回调函数
        
        callback.call(img);
        return; // 直接返回，不用再处理onload事件
    }
 
    img.onload = function(){
        img.onload = null;
        callback.call(img);
    }
}

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

                **     参考网址：https://blog.csdn.net/YiDaShi33/article/details/54316126**

** 四、vue 中的懒加载**

 vue 中的延迟加载是通过 webpack 代码拆分组件实现的。

  在 vue 中，有 3 块不同的延迟加载和代码拆分，使用动态导入：

            a、组件，又称为异步组件

            b、路由器

            c、vuex 模块

     1、 vue 组件的延迟加载

            通过将`import`函数包装到箭头函数中，Vue 将仅在请求时执行它，并在该时刻加载模块。

```
Vue.component('AsyncCmp', () => import('./AsyncCmp'))

```

    2、vue 路由器中的延迟加载

          vue 路由器内置支持延迟加载。假设我们想在 / login 路由中延迟加载一个 Lgin 组件

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
// Instead of: import Login from './login'
const Login = () => import('./login')

new VueRouter({
  routes: [
    { path: '/login', component: Login }
  ]
})

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

   3、延迟加载 vuex 模块

         Vuex 有一种`registerModule`方法可以让我们动态创建 Vuex 模块。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
const store = new Vuex.Store()

...

// Assume there is a "login" module we wanna load
import('./store/login').then(loginModule => {
  store.registerModule('login', loginModule)
})

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

 参考网址：https://alexjoverm.github.io/2017/07/16/Lazy-load-in-Vue-using-Webpack-s-code-splitting/