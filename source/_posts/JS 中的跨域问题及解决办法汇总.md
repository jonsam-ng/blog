---
title: JS 中的跨域问题及解决办法汇总
---
### **一、什么是跨域？**

在了解跨域之前，首先要知道什么是同源策略（same-origin policy）。简单来讲同源策略就是浏览器为了保证用户信息的安全，防止恶意的网站窃取数据，禁止不同域之间的 JS 进行交互。对于浏览器而言只要域名、协议、端口其中一个不同就会引发同源策略，从而限制他们之间如下的交互行为：

1.Cookie、LocalStorage 和 IndexDB 无法读取；

2.DOM 无法获得；

3.AJAX 请求不能发送。

跨域的严格一点的定义是：只要协议，域名，端口有任何一个的不同，就被当作是跨域。

如下表所示：

| URL | 说明 | 是否允许通信 |
| --- | --- | --- |
| http://www.a.com/a.js  
http://www.a.com/b.js | 同一域名下 | 允许 |
| http://www.a.com/lab/a.js  
http://www.a.com/script/b.js | 同一域名下不同文件夹 | 允许 |
| http://www.a.com:8000/a.js  
http://www.a.com/b.js | 同一域名，不同端口 | 不允许 |
| http://www.a.com/a.js  
https://www.a.com/b.js | 同一域名，不同协议 | 不允许 |
| http://www.a.com/a.js  
http://70.32.92.74/b.js | 域名和域名对应 ip | 不允许 |
| http://www.a.com/a.js  
http://script.a.com/b.js | 主域相同，子域不同 | 不允许 |
| http://www.a.com/a.js  
http://a.com/b.js | 同一域名，不同二级域名（同上） | 不允许（cookie 这种情况下也不允许访问） |
| http://www.cnblogs.com/a.js  
http://www.a.com/b.js | 不同域名 | 不允许 |

特别注意两点：

第一，如果是协议和端口造成的跨域问题 “前台” 是无能为力的，

第二：在跨域问题上，域仅仅是通过 “URL 的首部” 来识别而不会去尝试判断相同的 ip 地址对应着两个域或两个域是否在同一个 ip 上。

“URL 的首部” 指 window.location.protocol +window.location.host，也可以理解为 “Domains, protocols and ports must match”。

### **二、为什么浏览器要限制跨域访问呢？**

原因就是安全问题：如果一个网页可以随意地访问另外一个网站的资源，那么就有可能在客户完全不知情的情况下出现安全问题。比如下面的操作就有安全问题：

1. 用户访问 www.mybank.com，登陆并进行网银操作，这时 cookie 啥的都生成并存放在浏览器；

2. 用户突然想起件事，并迷迷糊糊的访问了一个邪恶的网站 www.xiee.com；

3. 这时该网站就可以在它的页面中，拿到银行的 cookie，比如用户名，登陆 token 等，然后发起对 www.mybank.com 的操作；

4. 如果这时浏览器不予限制，并且银行也没有做响应的安全处理的话，那么用户的信息有可能就这么泄露了。

### **三、为什么要跨域？**

既然有安全问题，那为什么又要跨域呢？ 有时公司内部有多个不同的子域，比如一个是 location.company.com , 而应用是放在 app.company.com , 这时想从 app.company.com 去访问 location.company.com 的资源就属于跨域。

### **四、解决跨域问题的方法：**

**1. 跨域资源共享（CORS）**

`CORS（Cross-Origin Resource Sharing`）跨域资源共享，定义了必须在访问跨域资源时，浏览器与服务器应该如何沟通。`CORS`背后的基本思想就是使用自定义的 HTTP 头部让浏览器与服务器进行沟通，从而决定请求或响应是应该成功还是失败。

服务器端对于`CORS`的支持，主要就是通过设置`Access-Control-Allow-Origin`来进行的。如果浏览器检测到相应的设置，就可以允许 Ajax 进行跨域的访问。

只需要在后台中加上响应头来允许域请求！在被请求的 Response header 中加入以下设置，就可以实现跨域访问了！

如下所示：

```
//指定允许其他域名访问
'Access-Control-Allow-Origin:*'//或指定域
//响应类型
'Access-Control-Allow-Methods:GET,POST'
//响应头设置
'Access-Control-Allow-Headers:x-requested-with,content-type'
//指定允许其他域名访问
'Access-Control-Allow-Origin:*'//或指定域
//响应类型
'Access-Control-Allow-Methods:GET,POST'
//响应头设置
'Access-Control-Allow-Headers:x-requested-with,content-type'

```

**2. 通过 jsonp 跨域**

JSONP 是 JSON with Padding（填充式 json）的简写，是应用 JSON 的一种新方法，只不过是被包含在函数调用中的 JSON，例如：

```
callback({"name","trigkit4"});

```

JSONP 由两部分组成：回调函数和数据。回调函数是当响应到来时应该在页面中调用的函数，而数据就是传入回调函数中的 JSON 数据。

**JSONP 的原理：**通过 script 标签引入一个 js 文件，这个 js 文件载入成功后会执行我们在 url 参数中指定的函数，并且会把我们需要的 json 数据作为参数传入。所以 jsonp 是需要服务器端的页面进行相应的配合的。（即用 JavaScript 动态加载一个 script 文件，同时定义一个 callback 函数给 script 执行而已。）

在 js 中，我们直接用`XMLHttpRequest`请求不同域上的数据时，是不可以的。但是，在页面上引入不同域上的 js 脚本文件却是可以的，jsonp 正是利用这个特性来实现的。 例如：有个 a.html 页面，它里面的代码需要利用 ajax 获取一个不同域上的 json 数据，假设这个 json 数据地址是 http://example.com/data.php，那么 a.html 中的代码就可以这样：

```
<script type="text/javascript">
    function dosomething(jsondata){
        //处理获得的json数据
    }
</script>
<script src="http://example.com/data.php?callback=dosomething"></script>
​<script type="text/javascript">
    function dosomething(jsondata){
        //处理获得的json数据
    }
</script>
<script src="http://example.com/data.php?callback=dosomething"></script>

```

js 文件载入成功后会**执行**我们在 url 参数中**指定的函数**，并且会把我们需要的 json 数据作为参数传入。所以 jsonp 是需要服务器端的页面进行相应的配合的。

```
<?php
$callback = $_GET['callback'];//得到回调函数名
$data = array('a','b','c');//要返回的数据
echo $callback.'('.json_encode($data).')';//输出
?>
​<?php
$callback = $_GET['callback'];//得到回调函数名
$data = array('a','b','c');//要返回的数据
echo $callback.'('.json_encode($data).')';//输出
?>

```

最终，输出结果为：`dosomething(['a','b','c']);`

![](https://images2017.cnblogs.com/blog/1191190/201708/1191190-20170824154926855-473423436.png)

如果你的页面使用 jquery，那么通过它封装的方法就能很方便的来进行 jsonp 操作了。`jquery`会自动生成一个全局函数来替换`callback=?`中的问号，之后获取到数据后又会自动销毁，实际上就是起一个临时代理函数的作用。`$.getJSON`方法会自动判断是否跨域，不跨域的话，就调用普通的`ajax`方法；跨域的话，则会以异步加载 js 文件的形式来调用`jsonp`的回调函数。

```
<script type="text/javascript">
    $.getJSON('http://example.com/data.php?callback=?,function(jsondata)'){
        //处理获得的json数据
    });
</script>
​<script type="text/javascript">
    $.getJSON('http://example.com/data.php?callback=?,function(jsondata)'){
        //处理获得的json数据
    });
</script>

```

**JSONP 的优缺点：**

JSONP 的优点是：它不像`XMLHttpRequest`对象实现的 Ajax 请求那样受到同源策略的限制；它的兼容性更好，在更加古老的浏览器中都可以运行，不需要 XMLHttpRequest 或 ActiveX 的支持；并且在请求完毕后可以通过调用 callback 的方式回传结果。

JSONP 的缺点则是：它只支持 GET 请求而不支持 POST 等其它类型的 HTTP 请求；它只支持跨域 HTTP 请求这种情况，不能解决不同域的两个页面之间如何进行`JavaScript`调用的问题。

**CORS 和 JSONP 对比：**

CORS 与 JSONP 相比，无疑更为先进、方便和可靠。

（1）JSONP 只能实现 GET 请求，而 CORS 支持所有类型的 HTTP 请求；

（2）使用 CORS，开发者可以使用普通的 XMLHttpRequest 发起请求和获得说句，比起 JSONP 有更好的错误处理；

（3）JSONP 主要被老的浏览器支持，它们往往不支持 CORS，而绝大多数现代浏览器都已经支持了 CORS；

**3. 通过修改 document.domain 来跨子域**

上面的 jsonp 是来解决 ajax 跨域请求的，那么如果是需要处理 Cookie 和 iframe 该怎么办呢？这时候就可以通过修改 document.domain 来跨子域。两个网页一级域名相同，只是二级域名不同，浏览器允许通过设置 document.domain 共享 Cookie 或者处理 iframe。比如 A 网页是 http://w1.example.com/a.html，B 网页是 http://w2.example.com/b.html，那么只要设置相同的 document.domain，两个网页就可以共享 Cookie。

```
document.domain = 'example.com';
//现在，A网页通过脚本设置一个 Cookie。
document.cookie = "test1=hello";
//B网页就可以读到这个 Cookie。
var allCookie = document.cookie;
​document.domain = 'example.com';
//现在，A网页通过脚本设置一个 Cookie。
document.cookie = "test1=hello";
//B网页就可以读到这个 Cookie。
var allCookie = document.cookie;

```

注意，这种方法只适用于 Cookie 和 iframe 窗口，LocalStorage 和 IndexDB 无法通过这种方法，规避同源政策，而要使用下文介绍的 PostMessage API。   
另外，服务器也可以在设置 Cookie 的时候，指定 Cookie 的所属域名为一级域名，比如. example.com。

```
Set-Cookie: key=value; domain=.example.com; path=/
//这样的话，二级域名和三级域名不用做任何设置，都可以读取这个Cookie。
Set-Cookie: key=value; domain=.example.com; path=/
//这样的话，二级域名和三级域名不用做任何设置，都可以读取这个Cookie。

```

不同的 iframe 之间（父子或同辈），是能够获取到彼此的 window 对象的，但是你却不能使用获取到的 window 对象的属性和方法 (html5 中的 postMessage 方法是一个例外，还有些浏览器比如 ie6 也可以使用 top、parent 等少数几个属性)，总之，你可以当做是只能获取到一个几乎无用的 window 对象。   
首先说明一下同域之间的 iframe 是可以操作的。比如 http://127.0.0.1/JSONP/a.html 里面嵌入一个 iframe 指向 http://127.0.0.1/myPHP/b.html。那么在 a.html 里面是可以操作 iframe 里面的 DOM 的。

```
<iframe src="http://127.0.0.1/myPHP/b.html" frameborder="1"></iframe>
<body>
<script type="text/javascript">
var iframe = document.querySelector("iframe");
iframe.onload = function(){
    var win = iframe.contentWindow;
    var doc = win.document;
    var ele = doc.querySelector(".text1");
    var text = ele.innerHTML="123456";
}
</script>
<iframe src="http://127.0.0.1/myPHP/b.html" frameborder="1"></iframe>
<body>
<script type="text/javascript">
var iframe = document.querySelector("iframe");
iframe.onload = function(){
    var win = iframe.contentWindow;
    var doc = win.document;
    var ele = doc.querySelector(".text1");
    var text = ele.innerHTML="123456";
}
</script>

```

如果两个网页不同源，就无法拿到对方的 DOM。典型的例子是 iframe 窗口和 window.open 方法打开的窗口，它们与父窗口无法通信。如果两个窗口一级域名相同，只是二级域名不同，那么 document.domain 属性，就可以规避同源政策，拿到 DOM。 

**4. 使用 window.name 来进行跨域**

window 对象有个 name 属性，该属性有个特征：即在一个窗口 (window) 的生命周期内, 窗口载入的所有的页面都是共享一个 window.name 的，每个页面对 window.name 都有读写的权限，window.name 是持久存在一个窗口载入过的所有页面中的，并不会因新页面的载入而进行重置。这个属性的最大特点是，无论是否同源，只要在同一个窗口里，前一个网页设置了这个属性，后一个网页可以读取它。   
比如：有一个页面 a.html, 它里面有这样的代码：

```
window.name = "我是a页面设置的";
setTimeout(function(){
    window.location = "http://127.0.0.1/JSONP/b.html";
},1000)
window.name = "我是a页面设置的";
setTimeout(function(){
    window.location = "http://127.0.0.1/JSONP/b.html";
},1000)

```

b.html 页面的代码：

```
console.log(window.name);

```

a.html 页面载入后 1 秒，跳转到了 b.html 页面，结果 b 页面打印出了：

```
我是a页面设置的

```

可以看到在 b.html 页面上成功获取到了它的上一个页面 a.html 给 window.name 设置的值。如果在之后所有载入的页面都没对 window.name 进行修改的话，那么所有这些页面获取到的 window.name 的值都是 a.html 页面设置的那个值。当然，如果有需要，其中的任何一个页面都可以对 window.name 的值进行修改。注意，window.name 的值只能是字符串的形式，这个字符串的大小最大能允许 2M 左右甚至更大的一个容量，具体取决于不同的浏览器，但一般是够用了。   
利用 window.name 可以对同域或者不同域的之间的 js 进行交互。   
那么在 a.html 页面中，我们怎么把 b.html 页面载入进来呢？显然我们不能直接在 a.html 页面中通过改变 window.location 来载入 b.html 页面，因为我们想要即使 a.html 页面不跳转也能得到 b.html 里的数据。答案就是在 a.html 页面中使用一个隐藏的 iframe 来充当一个中间人角色，由 iframe 去获取 b.html 的数据，然后 a.html 再去得到 iframe 获取到的数据。

**5. 使用 HTML5 的 window.postMessage 方法跨域**

上面两种方法都属于破解，HTML5 为了解决这个问题，引入了一个全新的 API：跨文档通信 API（Cross-document messaging）。   
这个 API 为 window 对象新增了一个 window.postMessage 方法，允许跨窗口通信，不论这两个窗口是否同源。目前 IE8+、FireFox、Chrome、Opera 等浏览器都已经支持 window.postMessage 方法。   
举例来说，父窗口 http://a.com 向子窗口 http://b.com 发消息，调用 postMessage 方法就可以了。   
a 页面：

```
<iframe src="http://127.0.0.1/JSONP/b.html" frameborder="1"></iframe>
document.getElementById('frame1').onload = function(){
    var win = document.getElementById('frame1').contentWindow;
    win.postMessage("我是来自a页面的","http://127.0.0.1/JSONP/b.html")
}
<iframe src="http://127.0.0.1/JSONP/b.html" frameborder="1"></iframe>
document.getElementById('frame1').onload = function(){
    var win = document.getElementById('frame1').contentWindow;
    win.postMessage("我是来自a页面的","http://127.0.0.1/JSONP/b.html")
}

```

b 页面通过监听 message 事件可以接受到来自 a 页面的消息。

```
window.onmessage = function(e){
    e = e || event;
    console.log(e.data);//我是来自a页面的
}
window.onmessage = function(e){
    e = e || event;
    console.log(e.data);//我是来自a页面的
}

```

子窗口向父窗口发送消息的写法类似。

```
window.opener.postMessage('我是来自b页面的', 'http://a.com');
//父窗口和子窗口都可以通过message事件，监听对方的消息。
window.opener.postMessage('我是来自b页面的', 'http://a.com');
//父窗口和子窗口都可以通过message事件，监听对方的消息。

```

通过 window.postMessage，读写其他窗口的 LocalStorage 也成为了可能。   
下面是一个例子，主窗口写入 iframe 子窗口的 localStorage。   
父窗口发送消息代码：

```
var win = document.getElementsByTagName('iframe')[0].contentWindow;
var obj = { name: 'Jack' };
// 存入对象
win.postMessage(JSON.stringify({key: 'storage', method: 'set', data: obj}), 'http://b.com');
// 读取对象
win.postMessage(JSON.stringify({key: 'storage', method: "get"}), "*");
window.onmessage = function(e) {
  if (e.origin != 'http://a.com') return;
  // "Jack"
  console.log(JSON.parse(e.data).name);
};
var win = document.getElementsByTagName('iframe')[0].contentWindow;
var obj = { name: 'Jack' };
// 存入对象
win.postMessage(JSON.stringify({key: 'storage', method: 'set', data: obj}), 'http://b.com');
// 读取对象
win.postMessage(JSON.stringify({key: 'storage', method: "get"}), "*");
window.onmessage = function(e) {
  if (e.origin != 'http://a.com') return;
  // "Jack"
  console.log(JSON.parse(e.data).name);
};

```

子窗口接收消息的代码：

```
window.onmessage = function(e) {
  if (e.origin !== 'http://bbb.com') return;
  var payload = JSON.parse(e.data);
  switch (payload.method) {
    case 'set':
      localStorage.setItem(payload.key, JSON.stringify(payload.data));
      break;
    case 'get':
      var parent = window.parent;
      var data = localStorage.getItem(payload.key);
      parent.postMessage(data, 'http://aaa.com');
      break;
    case 'remove':
      localStorage.removeItem(payload.key);
      break;
  }
};
window.onmessage = function(e) {
  if (e.origin !== 'http://bbb.com') return;
  var payload = JSON.parse(e.data);
  switch (payload.method) {
    case 'set':
      localStorage.setItem(payload.key, JSON.stringify(payload.data));
      break;
    case 'get':
      var parent = window.parent;
      var data = localStorage.getItem(payload.key);
      parent.postMessage(data, 'http://aaa.com');
      break;
    case 'remove':
      localStorage.removeItem(payload.key);
      break;
  }
};

```

**6. 通过 WebSocket 进行跨域**

web sockets 是一种浏览器的 API，它的目标是在一个单独的持久连接上提供全双工、双向通信。(同源策略对 web sockets 不适用)

web sockets 原理：在 js 创建了 web socket 之后，会有一个 HTTP 请求发送到浏览器以发起连接。取得服务器响应后，建立的连接会使用 HTTP 升级从 HTTP 协议交换为 web sockt 协议。

只有在支持 web socket 协议的服务器上才能正常工作。

```
var socket = new WebSockt('ws://www.baidu.com');//http->ws; https->wss
socket.send('hello WebSockt');
socket.onmessage = function(event){
    var data = event.data;
}
var socket = new WebSockt('ws://www.baidu.com');//http->ws; https->wss
socket.send('hello WebSockt');
socket.onmessage = function(event){
    var data = event.data;
}

```

**7. 图像 ping（单向）**

**什么是图像 ping：**  图像 ping 是与服务器进行简单、单向的跨域通信的一种方式，请求的数据是通过查询字符串的形式发送的，而相应可以是任意内容，但通常是像素图或 204 相应（No Content）。 图像 ping 有两个主要缺点：首先就是只能发送 get 请求，其次就是无法访问服务器的响应文本。

**使用方法：**

```
var img = new Image();
img.onload = img.onerror = function(){
alert("done!");
};
img.src = "https://raw.githubusercontent.com/zhangmengxue/Todo-List/master/me.jpg";
document.body.insertBefore(img,document.body.firstChild);
var img = new Image();
img.onload = img.onerror = function(){
alert("done!");
};
img.src = "https://raw.githubusercontent.com/zhangmengxue/Todo-List/master/me.jpg";
document.body.insertBefore(img,document.body.firstChild);

```

然后页面上就可以显示我放在我的 github 上某个地方的照片啦。

与 <img> 类似的可以跨域内嵌资源的还有:

(1)<script src=""></script > 标签嵌入跨域脚本。语法错误信息只能在同源脚本中捕捉到。上面 jsonp 也用到了呢。

(2) <link src=""> 标签嵌入 CSS。由于 CSS 的松散的语法规则，CSS 的跨域需要一个设置正确的 Content-Type 消息头。不同浏览器有不同的限制： IE, Firefox, Chrome, Safari (跳至 CVE-2010-0051) 部分 和 Opera。

(3)<video> 和 <audio > 嵌入多媒体资源。

(4)<object>, <embed> 和 <applet > 的插件。

(5)@font-face 引入的字体。一些浏览器允许跨域字体（ cross-origin fonts），一些需要同源字体（same-origin fonts）。

(6) <frame> 和 <iframe > 载入的任何资源。站点可以使用 X-Frame-Options 消息头来阻止这种形式的跨域交互。

**8. 使用片段识别符来进行跨域**

片段标识符（fragment identifier）指的是，URL 的 #号后面的部分，比如 http://example.com/x.html#flag 的 #flag。如果只是改变片段标识符，页面不会重新刷新。   
父窗口可以把信息，写入子窗口的片段标识符。在父窗口写入：

```
document.getElementById('frame').onload = function(){
    var src = "http://127.0.0.1/JSONP/b.html" + '#' + "data";
    this.src = src;
}
document.getElementById('frame').onload = function(){
    var src = "http://127.0.0.1/JSONP/b.html" + '#' + "data";
    this.src = src;
}

```

子窗口通过监听 hashchange 事件得到通知。

```
window.onload = function(){
    console.log("b.html加载完成")
    window.onhashchange = function(){
        var message = window.location.hash;
        console.log(message)//#data
    };  
}
window.onload = function(){
    console.log("b.html加载完成")
    window.onhashchange = function(){
        var message = window.location.hash;
        console.log(message)//#data
    };  
}

```

同样的，子窗口也可以改变父窗口的片段标识符。

```
parent.location.href= target + "#" + hash;

```

本文参考：[js 前端解决跨域问题的 8 种方案（最新最全）](https://www.jb51.net/article/97611.htm)

                 [JS 中的跨域问题](https://www.cnblogs.com/yongshaoye/p/7423881.html)