---
title: 原生 JS 实现 Ajax 及 Ajax 的跨域请求
---
<table border="0"><tbody><tr><td><strong>一、&nbsp;JQuery 的 Ajax</strong></td></tr></tbody></table>
首先，先回忆下 JQuery 的 Ajax 写法：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
$.ajax({
    url: ,
    type: '',
    dataType: '',
    data: {
          
    },
    success: function(){
         
    },
    error: function(){
          
    }
 })

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

<table border="0"><tbody><tr><td><strong>二、原生 JS 实现 Ajax</strong></td></tr></tbody></table>
[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
// 第一步： 获得XMLHttpRequest对象
            var ajax = new XMLHttpRequest();
            // 第二步： 设置状态监听函数
            ajax.onreadystatechange = function(){
                console.log(ajax.readyState);
                console.log(ajax.status);
                // 第五步：在监听函数中，判断readyState=4 && status=200表示请求成功
                if(ajax.readyState==4 && ajax.status==200){
                    // 第六步： 使用responseText、responseXML接受响应数据，并使用原生JS操作DOM进行显示
                    console.log(ajax.responseText);
                    console.log(ajax.responseXML);// 返回不是XML，显示null
                    console.log(JSON.parse(ajax.responseText));
                    console.log(eval("("+ajax.responseText+")"));
                }
            }
            // 第三步： open一个链接
            ajax.open("GET","h51701.json",false);//true异步请求，false同步
            
            // 第四步： send一个请求。 可以发送对象和字符串，不需要传递数据发送null
            ajax.send(null);

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")


注释：

1. open(method, url, async) 方法需要三个参数:

　 method：发送请求所使用的方法（GET 或 POST）；与 POST 相比，GET 更简单也更快，并且在大部分情况下都能用；然而，在以下情况中，请使用 POST 请求：

*   无法使用缓存文件（更新服务器上的文件或数据库）
*   向服务器发送大量数据（POST 没有数据量限制）
*   发送包含未知字符的用户输入时，POST 比 GET 更稳定也更可靠

　url：规定服务器端脚本的 URL(该文件可以是任何类型的文件，比如 .txt 和 .xml，或者服务器脚本文件，比如 .asp 和 .php （在传回响应之前，能够在服务器上执行任务）)；

　async：规定应当对请求进行异步（true）或同步（false）处理；true 是在等待服务器响应时执行其他脚本，当响应就绪后对响应进行处理；false 是等待服务器响应再执行。

2. send() 方法可将请求送往服务器。

3. onreadystatechange：存有处理服务器响应的函数，每当 readyState 改变时，onreadystatechange 函数就会被执行。

4. readyState：存有服务器响应的状态信息。

*   0: 请求未初始化
*   1: 服务器连接已建立
*   2: 请求已接收
*   3: 请求处理中
*   4: 请求已完成，且响应已就绪

5. responseText：获得字符串形式的响应数据。

 **eval() 和 JSON.parse() **
---------------------------

另外，给大家介绍两种解析字符串的方法：

**eval() :**

　　 ** eval 函数用于将字符串中的 JS 代码解析出来并执行！！       
   　  当使用 eval 函数解析 JSON 字符串时，需 要在函数内部将 JSON 字符串用 () 拼接  
   　　   例如：  eval("("+json1+")")  
  　　　　  表示 eval 函数中的字符串不是用于执行，而是要进行字符串解析  
  　　   即：  
   　　　　 eval("("+json1+")") = JSON.parse(json1；**

**JOSN.parse() :**

**　　纯粹的将 JSON 字符串解析为数组或对象；**

<table border="0"><tbody><tr><td><strong>四、&nbsp;Ajax 的跨域请求</strong></td></tr></tbody></table>
首先，我们得知道 为什么会有跨域请求这回事，以及什么情况下会有跨域请求？

1  **同源策略**
-----------

同源策略限制从一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的关键的安全机制。

它的定义是：

　　一段脚本向后台请求数据，只能读取属于同一协议名、同一主机名、同一端口号下的数据；

　　所以，请求不同协议名、不同端口号、不同主机名下面的文件时，  
       将会违背同源策略，无法请求成功，需要进行跨越处理!!

2 **解决跨域请求的方法**
---------------

**方法一：**后台 PHP 进行设置****

前台无需任何设置，在后台被请求的 PHP 文件中，写入一条 header。

   header("Access-Control-Allow-Origin:*");

   --- 表示允许哪些域名请求这个 PHP 文件，* 表示所有域名都允许

JS 代码：

**　　　　　　　　　　![](https://images2017.cnblogs.com/blog/1206202/201711/1206202-20171104202603029-1699177757.png)**

注释：

　　其中，url 为 PHP 文件的路径;

PHP 代码：

　　　　　　　　　　![](https://images2017.cnblogs.com/blog/1206202/201711/1206202-20171104205632779-1059622645.png)

结果：

　　![](https://images2017.cnblogs.com/blog/1206202/201711/1206202-20171104203310529-263998610.png)

**方法二 ：**使用 SRC 属性 + jsonp 实现跨域****


**实现步骤：　　　**

**　　　　1、原有 src 属性的标签子带跨域功能；所以可以使用 script 标签的 src 属性请求后台数据**

** 　　　　　　 <script src="http://127.0.0.1/json.php">< /script>**

**  　　　 2、用于 src 在加载数据成功后，会直接将加载的内容放到 script 标签中；**

**　　　　　　   所以，后台直接返回 JSON 字符串将不能在 script 标签中解析。**

**  　　　　　　 因此，后台应该返回给前台一个回调函数名，并将 JSON 字符串作为参数传入。**

**  　　　　　　　　后台 PHP 文件中返回： echo "callback({$json})";**

**  　　   3、前台接收到返回的回调函数，将直接在 script 标签中调用。因此，需要声明这样一个回调函数，作为请求成功的回调**

[?](#)

<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td>1234567</td><td><code>function callback(data){</code>&nbsp;<code>alert("请求成功!!");</code>&nbsp;<code>console.log(data);</code>&nbsp;<code>}</code></td></tr></tbody></table>
JS 代码：

　　　　　　　　　　　　![](https://images2017.cnblogs.com/blog/1206202/201711/1206202-20171104204359670-1159434907.png)

PHP 文件：

　　　　　　![](https://images2017.cnblogs.com/blog/1206202/201711/1206202-20171104204620545-357175626.png)　　　　　　

结果：

　　　　　　　　　　　　![](https://images2017.cnblogs.com/blog/1206202/201711/1206202-20171104203747138-8801060.png)

****方法三 ：JQuery 的 Ajax 实现 jsonp****

**　　1、在 ajax 请求时，设置 dataType 为 "jsonp"；**

**       2、后台返回时，依然需要返回回调函数名，但是，ajax 在发送请求时，会默认使用 get 请求将回调函数名发给后台，**

**  　　　　 后台 $_GET['callback'] 取出函数名：**

**   　　　　　　---   echo "{$_GET['callback']}({$str})";**

**       3、后台返回以后，前台就可以使用 ajax 的 success 函数作为成功的回调**

** 　　　　　　  ---    success : function(data){}**

 js 代码：

　　　　　　　　![](https://images2017.cnblogs.com/blog/1206202/201711/1206202-20171104203628591-1068515119.png)

**PHP 文件：　　**　　　　　　　

　　　　　　　　　　![](https://images2017.cnblogs.com/blog/1206202/201711/1206202-20171104203724795-578907116.png)

结果：

　　![](https://images2017.cnblogs.com/blog/1206202/201711/1206202-20171104205045310-1140197937.png)

当然，后台也可以随便返回一个函数名，前台只要请求成功，就会自动调用这个函数。类似第二条的②、③步，而不需要本方法的第③步

 PHP 返回： echo "callback({$str})";

  JS 代码：  function callback(data){

　　　　　　　　console.log(data);

　　　　　　}

**js 代码：**

**　　　　　　　　![](https://images2017.cnblogs.com/blog/1206202/201711/1206202-20171104205327701-867851017.png)**

**PHP 文件：**

　　　　　　　　　　![](https://images2017.cnblogs.com/blog/1206202/201711/1206202-20171104205407420-1920299817.png)

结果：

　　　　　　　　![](https://images2017.cnblogs.com/blog/1206202/201711/1206202-20171104205441685-177870530.png)

虽然，影子是一名 web 前端工程师，但是，影子中的觉得关于数据交互这一块，对我们这一群人来说，要用的地方还是，比较多的；况且，就算是用不到，多一技傍身也是，不错的；

好了，今天，影子的分享，就到这里结束了，感谢大家的支持!!!!