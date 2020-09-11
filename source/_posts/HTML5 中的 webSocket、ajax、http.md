---
title: HTML5 中的 webSocket、ajax、http
---
webSocket 与 ajax、web
====================

*   一、webSocket 与 ajax
    *   1、ajax
    *   2、webSocket
*   二、webSocket API
    *   1、事件
    *   2、方法
    *   3、属性
    *   4、常量
*   三、webSocket 与 HTTP
*   四、webSocket 原理
*   五、webSocket 的作用
    *   1、ajax 轮询：
    *   2、long poll
    *   3、webSocket
*   六、Socket.io

先看一个有道释义：

![](https://ask.qcloudimg.com/http-save/yehe-1729928/4e3rssrb94.png?imageView2/2/w/1620)

其实释义的挺形象的，下面我来一一解释哈：

1、聊天室：webSocket 有名的应用就是聊天室了；

2、服务：webSocket 提供客户端请求的服务器和服务；

3、套接字：源 IP 地址和目的 IP 地址以及源端口号和目的端口号的组合叫套接字，webSocket 就是服务端和客户端的结合；

4、协议：webSocket 是基于 TCP 的一种新的网络协议。

**一、webSocket 与 ajax**
======================

作为一个码了还算久代码的前端，说起 webSocket，脑子里最先闪现的当然就是 ajax ajax ajax......ajax 是啥，ajax 刚出来时，可谓轰动一时，让我们愉快地告别那种提交一个表单必须得填完所有信息，然后再把数据转给服务器验证，结果发现有一个小小的输入框里输错了信息，然后又改掉重新提交走着重复的路的痛苦时代，所以它最大的贡献就是局部刷新。当然，不是说有了 webSocket，它就 out 了，ajax 现在依旧好用。下面稍微比较了下 ajax 和 webSocket：

1、ajax
------

#### （1）浏览器主动发送消息给服务器；

#### （2）非实时数据交互（异步，局部刷新）。

![](https://ask.qcloudimg.com/http-save/yehe-1729928/me757wg861.png?imageView2/2/w/1620)

原生写法：

四部曲：**ajax 对象、建立连接、发送请求、获取相应**。

更通俗的用打电话来比喻，那就是：**电话、拨号、说话、听到对方回应**。[demo](https://jojojojo.duapp.com/websocket/ajax.html)

```
//创建一个ajax对象（想打电话，首先得有电话这个对象）


```

```
var XHR = null;  
if (window.XMLHttpRequest) {  
    // 非IE内核  
    XHR = new XMLHttpRequest();  
} else if (window.ActiveXObject) {  
    // IE内核,早期IE的版本写法不同
    XHR = new ActiveXObject("Microsoft.XMLHTTP");  
} else {  
    XHR = null;  
}  


```

```
if(XHR){  
    //建立连接（拨号）
    XHR.open("GET", "ajaxServer.action"，true);  

    //发送请求（说话）
    XHR.send();  
    
    //获取响应（听到对方回应）
    XHR.onreadystatechange = function () {  
        // readyState值说明  
        // 0,初始化,XHR对象已经创建,还未执行open  
        // 1,载入,已经调用open方法,但是还没发送请求  
        // 2,载入完成,请求已经发送完成  
        // 3,交互,可以接收到部分数据 
        // 4,交互完毕
  
        // status值说明  
        // 200:成功  
        // 404:没有发现文件、查询或URl  
        // 500:服务器产生内部错误  
        if (XHR.readyState == 4 && XHR.status == 200) {  
            // 这里可以对返回的内容做处理  
            // 一般会返回JSON或XML数据格式  
            console.log(XHR.responseText);  
            // 主动释放,JS本身也会回收的  
            XHR = null;  
        }  
    };  
}

```

JQuery 写法（so easy，妈妈再也不用担心我的学习啦）：

```
$.ajax({
    type:"post",
    url:url,
    async:true,
    data:params,
    dataType:"json",
    success:function(res){
        console.log(res);
    },
    error:function(jqXHQ){
        alert("发生错误:"+jqXHQ.status);
    }
});

```

2、webSocket
-----------

#### （1）实现了浏览器与服务器全双工 (full-duplex) 通信——允许服务器主动发送信息给客户端；

#### （2）实时数据交互。

**WebSocket 握手过程**  
WebSocket 协议本质上是一个基于 TCP 的协议，WebSocket

连接与 TCP 连接的建立过程类似[4]。但与传统的基于 TCP 连 接的协议有所不同的是 WebSocket 协议需要从 HTTP 协议“过 渡”而来，而这个 “过度” 过程也被称为 WebSocket 协议的握手 过程。因此想要建立基于 WebSocket 的 Web 应用必须首先了解 WebSocket 协议的握手机制，如图 2 给出了 WebSocket 协议握手 过程的示意图。

首先由浏览器客户端向服务器发起 WebSocket 握手请求报 文，这个报文是基于 HTTP 协议的，它告诉服务器客户端想要升 级当前的 HTTP 协议为 WebSocket 协议。服务器收到客户端的 WebSocket 握手请求报文之后会对报头进行解析，如果服务器 理解客户端握手请求报头并且满足升级为 WebSocket 协议的条 件，便会向客户端发送握手应答报文，这个应答报文同样是基于 HTTP 协议的。客户端收到服务器的应答报文后会对该报文进 行一次验证，验证成功之后便会成功升级为 WebSocket 协议，如 果验证失败客户端将会主动断开连接。建立了 WebSocket 连接 之后，双方便可以进行全双工通信， 

WebSocket 握手过程与 TCP 握手过程类似，但是 WebSocket 协议握手采用了更加简洁的方式。  

相对于传统的 TCP 握手，WebSocket 握手协议有以下明显 的特点: 首先，WebSocket 整个握手过程只需要两次握手，相对 于 TCP 的三次握手，WebSocket 简化并且加入了自己的规则。 其次，WebSocket 握手协议是基于 HTTP 协议的，相对于字节流 的解析，ASCII 序列解析起来更加简便。最后，WebSocket 握手 协议引入了随机序列认证机制，易于实现。

两次握手成功预示着双方接下来将升级当前的 HTTP 协议 为 WebSocket 协议。WebSocket 握手请求报文的报头部分除了 必须包含必要的 HTTP 字段 [5]，还须遵循通用消息格式 [RFC 822]，同时又要包含和 WebSocket 紧密联系的字段 [6]，如 Web- Socket 协议的版本信息、客户端随即生成的 Key、必要的 GET 请 求方法等。 

![](https://ask.qcloudimg.com/http-save/yehe-1729928/xx7u4h11yp.png?imageView2/2/w/1620)

```
// Create WebSocket connection.
var socket = new WebSocket('ws://localhost:8080');    //创建一个webSocket实例

// Connection opened
socket.addEventListener('open', function (event) {   //一旦服务端响应WebSocket连接请求，就会触发open事件
    socket.send('Hello Server!');
});

// Listen for messages
socket.addEventListener('message', function (event) {  //当消息被接受会触发消息事件
    console.log('Message from server', event.data);
});

```

二、webSocket API
===============

既然上面写了一部分代码，那不如把 API 全都贴出来，哈哈哈。

首先，创建一个 webSocket 实例：

```
var socket = new WebSocket('ws://localhost:8080');  

```

然后再看下面的的 API。

1、事件
----

### **（1）open**

一个用于连接打开事件的事件监听器。当`readyState`的值变为 OPEN 的时候会触发该事件。该事件表明这个连接已经准备好接受和发送数据。这个监听器会接受一个名为 "open" 的事件对象。

```
socket.onopen = function(e) {
    console.log("Connection open...");
};

```

或者：

```
socket.addEventListener('open', function (event) {
    console.log("Connection open...");
});

```

### （2）message

一个用于消息事件的事件监听器，这一事件当有消息到达的时候该事件会触发。这个 Listener 会被传入一个名为 "message" 的[` MessageEvent `](https://developer.mozilla.org/en/WebSockets/WebSockets_reference/MessageEvent)对象。

```
socket.onmessage = function(e) {
   console.log("message received", e, e.data);
};

```

### （3）error

当错误发生时用于监听 error 事件的事件监听器。会接受一个名为 “error” 的 event 对象。

```
socket.onerror = function(e) {
   console.log("WebSocket Error: " , e);
};

```

### （4）close

用于监听连接关闭事件监听器。当 WebSocket 对象的 readyState 状态变为 CLOSED 时会触发该事件。这个监听器会接收一个叫 close 的[` CloseEvent`](https://developer.mozilla.org/en/WebSockets/WebSockets_reference/CloseEvent) 对象。

```
socket.onclose = function(e) {
   console.log("Connection closed", e);
};

```

2、方法
----

### （1）send

通过 WebSocket 连接向服务器发送数据。

一旦在服务端和客户端建立了全双工的双向连接，可以使用 send 方法去发送消息，当连接是 open 的时候 send() 方法传送数据，当连接关闭或获取不到的时候回抛出异常。

一个通常的错误是人们喜欢在连接 open 之前发送消息。如下所示：

```
// 这将不会工作
var socket= new WebSocket("ws://localhost:8080")
socket.send("Initial data");

```

应该等待 open 事件触发后再发送消息，正确的姿势如下：

```
var socket= new WebSocket("ws://localhost:8080")
   socket.onopen = function(e) {
   socket.send("Initial data");
}

```

### （2）close

关闭 WebSocket 连接或停止正在进行的连接请求。如果连接的状态已经是`closed`，这个方法不会有任何效果。

使用 close 方法来关闭连接，如果连接已经关闭，这方法将什么也不做。调用 close 方法后，将不能再发送数据。close 方法可以传入两个可选的参数，code（numerical）和 reason（string）, 以告诉服务端为什么终止连接。

```
socket.close(1000, "Closing normally");
//1000是状态码，代表正常结束。

```

3、属性
----

| 
属性名

 | 

类型

 | 

描述

 |
| --- | --- | --- |
| 

binaryType

 | 

DOMString

 | 

一个字符串表示被传输二进制的内容的类型。取值应当是 "blob" 或者 "arraybuffer"。 "blob" 表示使用 DOMBlob 对象，而 "arraybuffer" 表示使用 ArrayBuffer 对象。

 |
| 

bufferedAmount

 | 

unsigned long

 | 

调用 send() 方法将多字节数据加入到队列中等待传输，但是还未发出。该值会在所有队列数据被发送后重置为 0。而当连接关闭时不会设为 0。如果持续调用 send()，这个值会持续增长。只读。

 |
| 

extensions

 | 

DOMString

 | 

服务器选定的扩展。目前这个属性只是一个空字符串，或者是一个包含所有扩展的列表。

 |
| 

protocol

 | 

DOMString

 | 

一个表明服务器选定的子协议名字的字符串。这个属性的取值会被取值为构造器传入的 protocols 参数。

 |
| 

readyState

 | 

unsigned short

 | 

连接的当前状态。取值是 Ready state constants 之一。只读。

 |
| 

url

 | 

DOMString

 | 

传入构造器的 URL。它必须是一个绝对地址的 URL。只读。

 |

4、常量
----

### Ready state 常量

| 
常量

 | 

值

 | 

描述

 |
| --- | --- | --- |
| 

CONNECTING

 | 

0

 | 

连接还没开启。

 |
| 

OPEN

 | 

1

 | 

连接已开启并准备好进行通信。

 |
| 

CLOSING

 | 

2

 | 

连接正在关闭的过程中。

 |
| 

CLOSED

 | 

3

 | 

连接已经关闭，或者连接无法建立。

 |

**三、webSocket 与 HTTP**
======================

webSocket 和 http 同为协议，大家心里肯定会想它俩之间有什么联系，当然，我也好奇，所以就有了下面的研究结果，呵呵呵呵~~

大家都知道，webSocket 是 H5 的一种新协议（这样看来和 http 是没什么关系），本质是通过 http/https 协议进行握手后创建一个用于交换数据的 TCP 连接，服务端与客户端通过此 TCP 连接进行实时通信。也就是说，webSocket 是 http 协议上的一种补充。

相对于 HTTP 这种非持久的协议来说，Websocket 是一个持久化的协议。

以 php 的生命周期为例：

在 http1.0 中，一个 request，一个 response，一个周期就结束了。

在 http1.1 中，有了 keep-alive，可以发送多个 Request，接收多个 Response。但在 http 中永远是一个 request 对应一个 response。而且这个 response 是被动的，不能主动发起。

这时候 webSocket 就派上用场了。

**四、webSocket 原理**
==================

首先，先来看一张 http 的 Request Headers：

![](https://ask.qcloudimg.com/http-save/yehe-1729928/3blb8417h5.png?imageView2/2/w/1620)

再看一张 webSocket 的：

![](https://ask.qcloudimg.com/http-save/yehe-1729928/xko1nvurqg.png?imageView2/2/w/1620)

以及 webSocket 的 Response Headers：

![](https://ask.qcloudimg.com/http-save/yehe-1729928/0bxjgmwjuk.png?imageView2/2/w/1620)

I guess，无论熟不熟悉 http，想必都看出了区别，哈哈哈。接下来就要对这些东西进行讲解啦：

### （1）Upgrade 和 Connection

```
Upgrade: websocket
Connection: Upgrade

```

这个就是 webSocket 的核心，告诉 Apache、ngix 等服务器：注意啦，我发起的是 webSocket 协议，快点帮我找到对应的助理处理~ 不是那个老土的 http。

### （2）Sec-WebSocket-Key、Sec-WebSocket-Extensions 和 Sec-WebSocket-Version

```
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Extensions: chat, superchat
Sec-WebSocket-Version: 13

```

这个很好理解啦，首先，**Sec-WebSocket-Key** 是一个 Base64 encode 的值，这个是浏览器随机生成的，告诉服务器：尼好，我是 webSocket，这是我的 ID 卡，让我过去吧。

然后，**Sec-WebSocket-Extensions**：协议扩展， 某类协议可能支持多个扩展，通过它可以实现协议增强

最后，**Sec-WebSocket-Version** 是告诉服务器所使用的 webSocket Draft（协议版本）。喏，我是小喵 4.1 版本哆啦 A 梦，哈哈哈哈哈哈哈哈。

然后只要服务器返回了上面我放的那一系列 balabala 的东西，就代表已经接受请求，webSocket 建立成功啦！

### （3）Sec-WebSocket-Accept 和 Sec-WebSocket-Extensions

请求时，webSocket 会自带加密过的 ID 卡过来让服务端验证；

对应的，接受请求之后，服务端也得搞一个安全卡（Accept 头域的值就是 Key 的值，是由浏览器发过来的 Sec-WebSocket-Key 生成的）来证明是我同意你通过的，而不是什么肯蒙拐骗的坏银 ->

就这样，原理部分就说完啦，握手成功！

**五、webSocket 的作用**
===================

说 webSocket 之前，先说一下 ajax 轮询和 long poll。

1、ajax 轮询：
----------

ajax 轮询很简单，就是让浏览器隔个几秒就发送一次请求，询问服务器是否有新信息。

```
客户端：hello hello，有没有新信息(Request)

服务端：没有（Response）

客户端：hello hello，有没有新信息(Request)

服务端：没有。。（Response）

客户端：hello hello，有没有新信息(Request)

服务端：你好烦啊，没有啊。。（Response）

客户端：hello hello，有没有新消息（Request）

服务端：有啦有啦，here you are（Response）

客户端：hello hello，有没有新消息（Request）

服务端：。。没。。。。没。。。没。。。。（Response）

```

2、long poll
-----------

long poll 和 ajax 轮询原理很像，不过 long poll 是阻塞模型，简单来说，就是一直给你打电话，直到你接听为止。

```
客户端：hello hello，有没有新信息，没有的话就等有了再返回给我吧（Request）

服务端：额。。。     （。。。。等待到有消息的时候。。。。）     有了，给你（Response）

```

很明显，ajax 轮询和 long poll 弊大于利：

### （1）被动性

上面这两种方式都是客户端先主动消息给服务端，然后等待服务端应答，要知道，等待总是难熬的，如果服务端能主动发消息多好，这也就是缺点之一：被动性。

### （2）非常消耗资源

ajax 轮询 需要服务器有很快的处理速度和资源（速度）；

long poll 需要有很高的并发，也就是说同时接待客户的能力（场地大小）。

so，当 ajax 轮询和 long poll 碰上 503（啊啊啊啊啊，game over）

这时候，神奇的 webSocket 又派上用场了。

3、webSocket
-----------

### （1）被动性

首先，解决被动性：

```
客户端：hello hello，我要建立webSocket协议，扩展服务：chat，Websocket，协议版本：17（HTTP Request）

服务端：ok，确认，已升级为webSocket协议（HTTP Protocols Switched）

客户端：麻烦你有信息的时候推送给我噢。。

服务端：ok，有的时候会告诉你的。

服务端：balabalabalabala

服务端：balabalabalabala

服务端：哈哈哈哈哈啊哈哈哈哈

服务端：笑死我了哈哈哈哈哈哈哈

```

就这样，只需要一次 http 请求，就会有源源不断的信息传送了，是不是很方便。

### （2）消耗资源问题

首先，了解一下，我们所用的程序是要经过两层代理的，即 http 协议在 Nginx 等服务器的解析下，然后再传送给相应的 Handler（PHP 等）来处理。简单地说，我们有一个非常快速的接线员（Nginx），他负责把问题转交给相应的客服（Handler） 。

本身接线员基本上速度是足够的，但是每次都卡在客服（Handler）了，老有客服处理速度太慢，导致客服不够。

webSocket 就解决了这样一个难题，建立后，可以直接跟接线员建立持久连接，有信息的时候客服想办法通知接线员，然后接线员再统一转交给客户。

这样就可以解决客服处理速度过慢的问题了。

同时，在传统的方式上，要不断的建立，关闭 HTTP 协议，由于 HTTP 是非状态性的，每次都要重新传输鉴别信息，来告诉服务端你是谁。

虽然接线员很快速，但是每次都要听这么一堆，效率也会有所下降的，同时还得不断把这些信息转交给客服，不但浪费客服的处理时间，而且还会在网路传输中消耗过多的流量 / 时间。

但是 webSocket 只需要一次 http 握手，所以说整个通讯过程是建立在一次连接 / 状态中，也就避免了 http 的非状态性，服务端会一直知道你的信息，直到你关闭请求，这样就解决了接线员要反复解析 http 协议，还要查看 identity info 的信息。

**六、Socket.io**
===============

既然说到了 webSocket，就难免扯到 socket.io。

有人说 socket.io 就是对 webSocket 的封装，并且实现了 webSocket 的服务端代码。可以这样说，但不完全正确。

在 webSocket 没有出现之前，实现与服务端的实时通讯可以通过轮询来完成任务。Socket.io 将 webSocket 和轮询（Polling）机制以及其它的实时通信方式封装成了通用的接口，并且在服务端实现了这些实时机制的相应代码。也就是说，webSocket 仅仅是 Socket.io 实现实时通信的一个子集。

下面直接上一个用 socket.io 做的小小聊天室吧。

（1）首先你得有 node，然后安装 socket.io。

```
$ npm install socket.io

```

（2）服务器端（index.js）

```
'use strict';
module.exports = require('./lib/express');

var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);

app.get('/', function(req, res){
    res.sendFile(__dirname + '/index.html');
});

io.on('connection', function(socket){
    socket.on('message',function(msg){
        console.log(msg);
        socket.broadcast.emit('chat',msg);    //广播消息
    })
});

http.listen(3000);

```

（3）客户端

先引入 js 文件：

```
<script src="/socket.io/socket.io.js"></script>

```

交互代码（index.html）：

```
<!DOCTYPE html><html><head>
<meta charset="UTF-8">
<title>聊天室</title>
<style>
body,div,ul,li{margin: 0;padding: 0;list-style: none;}
.auto{margin: auto;}
.l{text-align: left;}
.r{text-align: right;}
.flex{display: box;display: -webkit-box;display: -moz-box;display: -ms-flexbox;display: -webkit-flex;display: flex;-webkit-box-pack: center;-webkit-justify-content: center;-moz-justify-content: center;-ms-justify-content: center;-o-justify-content: center;justify-content: center;-webkit-box-align: center;-webkit-align-items: center;-moz-align-items: center;-ms-align-items: center;-o-align-items: center;align-items: center;}
.chat-box{background: #f1f1f1;width: 56vw;padding:2vw;height:36vw;border:1px solid #ccc;margin-top: 2vw;}
.chat-li{display:inline-block;margin-top: 5px;background: #5CB85C;border-radius: 5px;padding: 3px 10px;color: #fff;}
.other-chat-li{background: #fff;color: #333;}
.send-box{width: 60vw;border:1px solid #ccc;justify-content: space-between;border-top: 0;}
.send-text{width: 50vw; border: none; padding: 10px;outline:0;}
.send{width: 10vw;background: #5cb85c; border: none; padding: 10px;color: #fff;cursor: pointer;}
.chat-name{color: #f00;}
.other-box,.self-box{width: 50%;height:100%;}
</style>
</head>
<body>
    <div>
        <ul></ul>
        <ul></ul>
    </div>
    <div>
        <input type="text">
        <button type="button">发送</button>
    </div>
</body>
<script src="/socket.io/socket.io.js"></script>
<script src="https://code.jquery.com/jquery-1.11.1.js"></script>
<script>
    $(function(){  
        var socket = io();
        
        $(".send").click(function(){
            var msg = $(".send-text").val();
            if(msg != ""){
                socket.send(msg);
                $('.self-box').append('<li>'+ msg +'<li>');
                $(".send-text").val("");
            }else{
                return false;
            }
        })
        
        $(".send-text").keydown(function(event){
            if(event.keyCode == 13){
                var msg = $(".send-text").val();
                if(msg != ""){
                    socket.send(msg);
                    $('.self-box').append('<li>'+ msg +'<li>');
                    $(".send-text").val("");
                }else{
                    return false;
                }
            }
        })

        socket.on("chat",function(msg){
            $('.other-box').append('<li>'+ msg +'<li>');
        })
    })
</script>
</html>

```

（4）运行代码：

```
$ node index.js

```

然后打开两个浏览器页面（**http://localhost:3000/**），就可以聊天啦，至于聊天名称呀、聊天头像呀什么的，可以自己去研究罗~~~

下面是效果图：

![](https://ask.qcloudimg.com/http-save/yehe-1729928/6ir0axkcht.png?imageView2/2/w/1620)

到底为止啦，感觉好像裹脚布，so long~~~~~~~