---
title: [vue]webpack3 最佳实践篇
---
vue-render:  
https://www.cnblogs.com/iiiiiher/articles/9465311.html

es6 模块的导入导出  
https://www.cnblogs.com/iiiiiher/p/9060206.html

复习
--

```
webpack作用:
1.处理依赖
2.转换语法

```

```
// 注意： webpack, 默认只能打包处理 JS 类型的文件，无法处理 其它的非 JS 类型的文件；
// 如果要处理 非JS类型的文件，我们需要手动安装一些 合适 第三方 loader 加载器；
// 1. 如果想要打包处理 css 文件，需要安装 cnpm i style-loader css-loader -D
// 2. 打开 webpack.config.js 这个配置文件，在 里面，新增一个 配置节点，叫做 module, 它是一个对象；在 这个 module 对象身上，有个 rules 属性，这个 rules 属性是个 数组；这个数组中，存放了，所有第三方文件的 匹配和 处理规则；

```

```
// 注意： webpack 处理第三方文件类型的过程：
// 1. 发现这个 要处理的文件不是JS文件，然后就去 配置文件中，查找有没有对应的第三方 loader 规则
// 2. 如果能找到对应的规则， 就会调用 对应的 loader 处理 这种文件类型；
// 3. 在调用loader 的时候，是从后往前调用的；
// 4. 当最后的一个 loader 调用完毕，会把 处理的结果，直接交给 webpack 进行 打包合并，最终输出到  bundle.js 中去

```

```
// 回顾 包的查找规则：
// 1. 找 项目根目录中有没有 node_modules 的文件夹
// 2. 在 node_modules 中 根据包名，找对应的 vue 文件夹
// 3. 在 vue 文件夹中，找 一个叫做 package.json 的包配置文件
// 4. 在 package.json 文件中，查找 一个 main 属性【main属性指定了这个包在被加载时候，的入口文件】

// 注意： 在 webpack 中， 使用 import Vue from 'vue' 导入的 Vue 构造函数，功能不完整，只提供了 runtime-only 的方式，并没有提供 像网页中那样的使用方式；

```

```
// 复习 在普通网页中如何使用vue：
// 1. 使用 script 标签 ，引入 vue 的包
// 2. 在 index 页面中，创建 一个 id 为 app div 容器
// 3. 通过 new Vue 得到一个 vm 的实例


```

```
// 总结梳理： webpack 中如何使用 vue :
// 1. 安装vue的包：  cnpm i vue -S
// 2. 由于 在 webpack 中，推荐使用 .vue 这个组件模板文件定义组件，所以，需要安装 能解析这种文件的 loader    cnpm i vue-loader vue-template-complier -D
// 3. 在 main.js 中，导入 vue 模块  import Vue from 'vue'
// 4. 定义一个 .vue 结尾的组件，其中，组件有三部分组成： template script style
// 5. 使用 import login from './login.vue' 导入这个组件
// 6. 创建 vm 的实例 var vm = new Vue({ el: '#app', render: c => c(login) })
// 7. 在页面中创建一个 id 为 app 的 div 元素，作为我们 vm 实例要控制的区域；

```

```
// 注意： export default 向外暴露的成员，可以使用任意的变量来接收
// 注意： 在一个模块中，export default 只允许向外暴露1次
// 注意： 在一个模块中，可以同时使用 export default 和 export 向外暴露成员

// 注意： 使用 export 向外暴露的成员，只能使用 { } 的形式来接收，这种形式，叫做 【按需导出】
// 注意： export 可以向外暴露多个成员， 同时，如果某些成员，我们在 import 的时候，不需要，则可以 不在 {}  中定义
// 注意： 使用 export 导出的成员，必须严格按照 导出时候的名称，来使用  {}  按需接收；
// 注意： 使用 export 导出的成员，如果 就想 换个 名称来接收，可以使用 as 来起别名；

```

```
- vue-router使用
// 1. 导入 vue-router 包
// 2. 手动安装 VueRouter 
// 3. 创建路由对象
4. 将路由对象挂载到 vm 上

login
    account
    goodlist
// 注意： App 这个组件，是通过 VM 实例的 render 函数，渲染出来的， render 函数如果要渲染 组件， 渲染出来的组件，只能放到 el: '#app' 所指定的 元素中；
// Account 和 GoodsList 组件， 是通过 路由匹配监听到的，所以， 这两个组件，只能展示到 属于 路由的 <router-view></router-view> 中去；

```

```
npm i webpack@3 -g //全局安装webpack

```

webpack 手动使用
------------

先创建个目录, npm 初始化下

```
npm init -y

```

目录如下  
![](https://images2018.cnblogs.com/blog/1312420/201808/1312420-20180819001528680-79776339.png)

然后 webpack 手动编译

```
webpack src/main.js dist/bundle.js

```

会自动生成 dist 目录和 dist/bundle.js  
![](https://images2018.cnblogs.com/blog/1312420/201808/1312420-20180819001603110-1913155692.png)

可见 webpack 命令核心使命是编译.

配置 webpack 的配置文件, 使得执行 webpack 更加便利
-----------------------------------

webpack.conf.js

```
let path = require('path');

module.exports = {
  entry: path.resolve('./src/main.js'), //输入
  output: {
    path: path.resolve('./dist'), //产出
    filename: 'bundle.js'
  }
};


```

自此执行 webpack 即可编译了

```
webpack 

```

webpack-dev-server 实现自动编译
-------------------------

每次修改代码, 无需在手动执行 webpack 了

```
npm i webpack@3 webpack-dev-server@2 --save-dev

```

```
let path = require('path');

module.exports = {
  entry: path.resolve('./src/main.js'), //输入
};


```

pakage.json

```
"dev": "webpack-dev-server --open --hot --port 3000"

```

1. 自动打开浏览器  
2. 代码更新后热刷新

自动插入 bundle.js
--------------

```
npm i html-webpack-plugin --save-dev

```

webpack.confg.js 配置

```
let path = require('path');

//1.导入
let HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: path.resolve('./src/main.js'),
  //2.配置个模板
  plugins: [
    new HtmlWebpackPlugin({
      template: 'src/index.html'
    }),
  ]
};


```

![](https://images2018.cnblogs.com/blog/1312420/201808/1312420-20180819004143146-9840274.png)

注意: 本质上预览的 index.html 和 bundle.js 路径在项目根目录下, 而非 src 下.

![](https://img2018.cnblogs.com/blog/1312420/201811/1312420-20181112170505695-1360199555.png)

解析 css
------

```
npm i style-loader css-loader --save-dev
npm i less less-loader --save-dev


```

解析图片和字体文件
---------

```
npm i url-loader file-loader --save-dev

```

```
// 在js中引入图片需要import,或者写一个线上路径
import page from './01.jpg';
console.log(page); // page就是打包后图片的路径
let img = new Image();
img.src = page; // 写了一个字符串 webpack不会进行查找
document.body.appendChild(img);

```

```
- webpack.config.js
// 转化base64只在8192字节一下转化。其他情况下输出图片
{test:/\.(jpg|png|gif)$/,use:'url-loader?limit=8192'},
{test:/\.(eot|svg|woff|woff2|wtf)$/,use:'url-loader'},

```

![](https://img2018.cnblogs.com/blog/1312420/201811/1312420-20181112174156877-479513783.png)

解析 js 高级语法
----------

```
npm i babel-loader babel-core babel-plugin-transform-runtime --save-dev
npm i babel-preset-env babel-preset-stage-0 --save-dev

- webpack3对接babel加一下版本号:

npm i babel-loader@7.1.5 babel-core@6.26.3 babel-plugin-transform-runtime@6.23.0 --save-dev
npm i babel-preset-env@1.7.0 babel-preset-stage-0@6.24.1 --save-dev

```

.babelrc

```
{
  "plugins": [
    "transform-runtime"
  ],
  "presets": [
    "env",
    "stage-0"
  ]
}


```

webpack.config.js

```
let path = require('path');
let HtmlWebpackPlugin = require('html-webpack-plugin');


module.exports = {
  entry: './src/main.js',
  // output: {
  //   path: path.resolve('./dist'),
  //   filename: 'bundle.js'
  // }
  plugins: [
    new HtmlWebpackPlugin({
      template: 'src/index.html'
    })
  ],
  module: {
    rules: [
      {test: /\.css$/, use: ['style-loader', 'css-loader']},
      {test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader']},
      {test: /\.scss$/, use: ['style-loader', 'css-loader', 'sass-loader']},
      {test: /\.(jpg|png|gif|bmp|jpeg)$/, use: 'url-loader?limit=7631&name=[hash:8]-[name].[ext]'},
      {test: /\.(ttf|eot|svg|woff|woff2)$/, use: 'url-loader'},
      {test: /\.js$/, use: 'babel-loader', exclude: /node_modules/},
      {test: /\.vue$/, use: 'vue-loader'}
    ]
  }
};


```

main.js

```
console.log('ok1111');


import './index.css';

class People {
  name = 'maotai'
}

console.log(People.name);


```

index.css

```
body {
    background-color: palegoldenrod;
    background-image: url('./01.jpg');
}


```

babel 补课  
.babelrc

```
{
  "plugins": [
    "transform-runtime"
  ],
  "presets": [
    "env",
    "stage-0"
  ]
}


```

```
// 注意： 如果要通过路径的形式
import './css/index.scss'

// 不写 node_modules 这一层目录 ，默认 就会去 node_modules 中查找
import 'bootstrap/dist/css/bootstrap.css'

// 在 webpack 中，默认只能处理 一部分 ES6 的新语法，一些更高级的ES6语法或者 ES7 语法，webpack 是处理不了的；这时候，就需要 借助于第三方的 loader，来帮助webpack 处理这些高级的语法，当第三方loader 把 高级语法转为 低级的语法之后，会把结果交给 webpack 去打包到 bundle.js 中
// 通过 Babel ，可以帮我们将 高级的语法转换为 低级的语法
// 1. 在 webpack 中，可以运行如下两套 命令，安装两套包，去安装 Babel 相关的loader功能：
// 1.1 第一套包： cnpm i babel-core babel-loader babel-plugin-transform-runtime -D
// 1.2 第二套包： cnpm i babel-preset-env babel-preset-stage-0 -D

// 2. 打开 webpack 的配置文件，在 module 节点下的 rules 数组中，添加一个 新的 匹配规则：
// 2.1 { test:/\.js$/, use: 'babel-loader', exclude:/node_modules/ }
// 2.2 注意： 在配置 babel 的 loader规则的时候，必须 把 node_modules 目录，通过 exclude 选项排除掉：原因有俩：
// 2.2.1 如果 不排除 node_modules， 则Babel 会把 node_modules 中所有的 第三方 JS 文件，都打包编译，这样，会非常消耗CPU，同时，打包速度非常慢；
// 2.2.2 哪怕，最终，Babel 把 所有 node_modules 中的JS转换完毕了，但是，项目也无法正常运行！
// 3. 在项目的 根目录中，新建一个 叫做 .babelrc  的Babel 配置文件，这个配置文件，属于JSON格式，所以，在写 .babelrc 配置的时候，必须符合JSON语法规范： 不能写注释，字符串必须用双引号
// 3.1 在 .babelrc 写如下的配置：  大家可以把 preset 翻译成 【语法】 的意思
        // {
        //   "presets": ["env", "stage-0"],
        //   "plugins": ["transform-runtime"]
        // }
// 4. 了解： 目前，我们安装的 babel-preset-env, 是比较新的ES语法， 之前， 我们安装的是 babel-preset-es2015, 现在，出了一个更新的 语法插件，叫做 babel-preset-env ，它包含了 所有的 和 es***相关的语法

```

支持 vue-render 渲染一个组件
--------------------

```
npm i --save-dev vue-loader@13 vue-template-compiler@2

```

webpack.conf.js

```
let path = require('path');
let HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: path.resolve('./src/main.js'),
    plugins: [
        new HtmlWebpackPlugin({
            template: 'src/index.html'
        })
    ],
    module: {
        rules: [
            {test: /\.js/, use: "babel-loader", exclude: /node_modules/},
            {test: /\.css/, use: ['style-loader', 'css-loader']},
            {test: /\.(jpg|png|gif|jpeg)$/, use: 'url-loader?limit=8192'},
            {test: /\.vue$/, use: 'vue-loader'}
        ]
    }
};


```

main.js

```
import Vue from 'vue';
import App from "./App.vue";

let vm = new Vue({
  el: "#app",
  render: c => c(App),
});


```

App.vue

```
<template>
    <div>app</div>
</template>

<script>

</script>

<style>

</style>


```

支持 vue-render 渲染多个组件
--------------------

```
App
    login
    register


```

![](https://images2018.cnblogs.com/blog/1312420/201808/1312420-20180819165304183-1086022441.png)

sumcom/login.vue

```
<template>
    <div>login</div>
</template>

<script>

</script>

<style>

</style>


```

sumcom/register.vue

```
<template>
    <div>register</div>
</template>

<script>

</script>

<style>

</style>


```

App.vue

```
<template>
    <div>
        <h1>account</h1>
        <router-link to="/login">login</router-link>
        <router-link to="/register">register</router-link>
        <router-view></router-view>
    </div>
</template>

<script>

</script>

<style>

</style>


```

main.js

```
import Vue from 'vue';
import VueRouter from 'vue-router';

Vue.use(VueRouter);
import App from "./App.vue";
import login from "./subcom/login.vue";
import register from "./subcom/register.vue";


let routes = [
  {path: '/login', component: login},
  {path: '/register', component: register},
];
let router = new VueRouter({
  routes
});
let vm = new Vue({
  el: "#app",
  render: c => c(App),
  router
});


```

vue-router 子组件
--------------

![](https://images2018.cnblogs.com/blog/1312420/201808/1312420-20180819170359316-2010013579.png)

subcom/login.vue

subcom/register.vue

main/Account.vue

```
<template>
    <div>
        <h1>account</h1>
        <router-link to="/account/login">login</router-link>
        <router-link to="/account/register">register</router-link>
        <router-view></router-view>
    </div>
</template>

<script>

</script>

<style>

</style>


```

main/GoodList.vue

main.js

```
import Vue from 'vue';
import VueRouter from 'vue-router';

Vue.use(VueRouter);
import App from "./App.vue";


// 注入2个组件
import Account from "./main/Account.vue";
import GoodList from "./main/GoodList.vue";


//注入两个子组件
import login from "./subcom/login.vue";
import register from "./subcom/register.vue";

let routes = [
  {path: '/GoodList', component: GoodList},
  {
    path: '/account',
    component: Account,
    children: [
      {path: 'login', component: login},
      {path: 'register', component: register},
    ],
  },
];
let router = new VueRouter({
  routes
});
let vm = new Vue({
  el: "#app",
  render: c => c(App),
  router
});


```

![](https://images2018.cnblogs.com/blog/1312420/201808/1312420-20180819170523732-2100905319.png)

附: package.json
---------------

```
{
  "name": "01.webpack-study",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server --open --port 3000 --hot"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-plugin-component": "^0.10.1",
    "babel-plugin-transform-runtime": "^6.23.0",
    "babel-preset-env": "^1.6.1",
    "babel-preset-stage-0": "^6.24.1",
    "css-loader": "^0.28.7",
    "file-loader": "^1.1.5",
    "html-webpack-plugin": "^2.30.1",
    "less": "^2.7.3",
    "less-loader": "^4.0.5",
    "node-sass": "^4.5.3",
    "sass-loader": "^6.0.6",
    "style-loader": "^0.19.0",
    "url-loader": "^0.6.2",
    "vue-loader": "^13.3.0",
    "vue-template-compiler": "^2.5.2",
    "webpack": "^3.8.1",
    "webpack-dev-server": "^2.9.3"
  },
  "dependencies": {
    "bootstrap": "^3.3.7",
    "mint-ui": "^2.2.9",
    "vue": "^2.5.2",
    "vue-resource": "^1.3.4",
    "vue-router": "^3.0.1"
  }
}


```

附 webpack.config.js
-------------------

```
let path = require('path');
let HtmlWebpackPlugin = require('html-webpack-plugin');


module.exports = {
  entry: './src/main.js',
  // output: {
  //   path: path.resolve('./dist'),
  //   filename: 'bundle.js'
  // }
  plugins: [
    new HtmlWebpackPlugin({
      template: 'src/index.html'
    })
  ],
  module: {
    rules: [
      {test: /\.css$/, use: ['style-loader', 'css-loader']},
      {test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader']},
      {test: /\.scss$/, use: ['style-loader', 'css-loader', 'sass-loader']},
      {test: /\.(jpg|png|gif|bmp|jpeg)$/, use: 'url-loader?limit=7631&name=[hash:8]-[name].[ext]'},
      {test: /\.(ttf|eot|svg|woff|woff2)$/, use: 'url-loader'},
      {test: /\.js$/, use: 'babel-loader', exclude: /node_modules/},
      {test: /\.vue$/, use: 'vue-loader'}
    ]
  }
};

```

需要理解的点
------

#### 使用 render 渲染, App 是根组件

所以搞清楚子组件的写法.

![](https://img2018.cnblogs.com/blog/1312420/201811/1312420-20181112183353507-865356295.png)

![](https://img2018.cnblogs.com/blog/1312420/201811/1312420-20181112184453975-1000964039.png)

#### App.vue 里的写法

![](https://img2018.cnblogs.com/blog/1312420/201811/1312420-20181112183503131-329601762.png)

#### 子组件进阶

第一步: 用 componet 渲染  
![](https://img2018.cnblogs.com/blog/1312420/201811/1312420-20181112184113279-1232888372.png)

![](https://img2018.cnblogs.com/blog/1312420/201811/1312420-20181112184210542-81899871.png)

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="app">
    <login></login>
</div>

<template id="login">
    <div>login</div>
</template>
<script src="node_modules/vue/dist/vue.js"></script>
<script>
    let login = {
        name: 'login',
        template: "#login"
    };

    let vm = new Vue({

        el: "#app",
        data: {
            msg: "maotai"
        },
        components: {
            login
        }
    })
</script>
</body>
</html>


```

第二阶段: 用 router 渲染

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="app">
    <router-link to="/login">/login</router-link>
    <router-view></router-view>
</div>

<template id="login">
    <div>login</div>
</template>
<script src="node_modules/vue/dist/vue.js"></script>
<script src="node_modules/vue-router/dist/vue-router.js"></script>
<script>
    let login = {
        name: 'login',
        template: "#login"
    };

    let routes = [
        {path: '/login', component: login}
    ];
    let router = new VueRouter({
        routes
    });
    let vm = new Vue({

        el: "#app",
        data: {
            msg: "maotai"
        },
        router,
    })
</script>
</body>
</html>

```

第三阶段: 用 render 渲染. 改变根组件  
https://www.cnblogs.com/iiiiiher/articles/9465311.html

组件的 scope 属性实现原理: 外层 div 上, 加 css 属性选择器
---------------------------------------

![](https://img2018.cnblogs.com/blog/1312420/201811/1312420-20181112225821288-1062289848.png)

vue-cli 使用
----------

![](https://img2018.cnblogs.com/blog/1312420/201811/1312420-20181125103845283-537631177.png)

![](https://img2018.cnblogs.com/blog/1312420/201811/1312420-20181125104146549-1878225799.png)