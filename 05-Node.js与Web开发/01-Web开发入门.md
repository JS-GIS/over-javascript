## 一 Web开发的概念

### 1.1 HTTP协议的请求响应

HTTP是一种无状态的协议，客户端与服务端之间的链接是基于一种请求应答的模式。即：客户端和服务端建立一个链接，客户端提交一个请求，服务端收到该请求后返回一个响应，然后二者就会断开连接。  

![](../images/node/01-001.svg)  

不过HTTP协议是基于TCP协议的上层协议，在请求响应模型中，仍然遵循三次握手、四次挥手的规范！  

### 1.2 CS和BS架构

CS架构是典型的请求响应模型，客户端（client）发出请求，服务端（server）返回响应。日常生活中常见的客户端，如：QQ，迅雷，手机视频播放器等。  

BS架构是互联网兴起后开始流行的软件架构，由浏览器（Browser）作为客户端发起请求，并接收服务端的响应。  

CS架构与BS架构的异同主要体现在安装方式上：
- 安装方式：CS架构都有自己的独立客户端，需要独立安装，且每次有新版本更新都要进行下载安装包升级。BS架构的系统只需要一个浏览器，使用网址访问即可，每次系统更新，用户无需更新，刷新浏览器即可。

CS与BS架构各有优缺点，适合在不同的场景中，如需求多变的互联网系统，较多采用BS架构，对界面要求性能高、交互性强则可以使用CS架构。  

### 1.3 输入url到网页展示的过程

假设我们点击了某网页上的一个链接，指向一个新浪体育上一个詹姆斯扣球新闻，一般网页地址为：`http://www.sina.com/sport/nba/20200203-zms.html`。 

这个访问过程大致如下：
- 第一步：查找DNS服务
- 第二步：根据DNS服务查找该网页所在服务器IP
- 第三步：浏览器与网页所在服务器建立连接
- 第四步：浏览器发出获取文件请求，服务端响应请求返回文件内容
- 第五步：浏览器与服务端双方断开连接
- 第六步：浏览器将获取到的数据通过解析引擎展示给用户

### 1.4 web开发中的常见概念

报文：HTTP协议中有两类报文，请求报文、响应报文。HTTP是面向文本的，所以报文中的每一个字段都是一些ASCII码串，因而各个字段的长度都是不确定的。HTTP请求报文和响应报文都是由三个部分组成。这两种报文格式的区别就是开始行不同。  
- 开始行：用于区分是请求报文还是响应报文。在请求报文中的开始行叫做请求行，而在响应报文中的开始行叫做状态行。
- 首部行：用来说明浏览器、服务器或报文主体的一些信息。首部可以有好几行，但也可以不使用。
- 实体主体：在请求报文中一般都不用这个字段，而在响应报文中也可能没有这个字段

示例：
```
GET/sample.jspHTTP/1.1 
Accept:image/gif.image/jpeg,/ 
Accept-Language:zh-cn 
Connection:Keep-Alive 
Host:localhost 
User-Agent:Mozila/4.0(compatible;MSIE5.01;Window NT5.0) 
Accept-Encoding:gzip,deflate
```

请求方法：客户端和服务器之间交互会使用不同的方法。常见的请求方法有：
- OPTIOON：请求一些选项的信息
- GET：请求读取由URL所标记的信息
- HEAD：请求读取由URL所标记的信息的首部
- POST：向服务器提交信息
- PUT：在指明的URL下存储一个文档
- DELETE：删除指明的URL所标记的资源
- TRAC：用来进行环回测试的请求报文
- CONNECT：用于代理服务器

响应的状态码通常是三位数字：
- 1xx 表示通知信息，如请求收到或正在进行处理
- 2xx 表示成功，如接受或知道了
- 3xx 表示重定向，如要完成请求还必须采取进一步的行动
- 4xx 表示客户的差错，如请求中有错误的语法或不能完成
- 5xx 表示服务器的差错，如服务器失效无法完成请求

## 二 简单的Node Web程序

```JavaScript
var http = require('http');

var server = http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/plain;charset=UTF8'});
    res.end('hello world');
});

server.listen(8000);
```

以上简单的几行代码就能开启一个web服务，访问”localhost:8000“，即可看到页面输出8000端口。  

## 三 http模块

### 1.1 createServer

`http.createServer()`方法返回的是http模块封装的一个基于事件的http服务器。  

req，res分别是http.IncomingMessage和http.ServerResponse的实例。  

http.Server的主要事件有：
- request：客户端发起请求时，被处罚，提供req，res参数
- connection：TCP建立连接时候处罚，提供一个scoket参数，是net.Socket的实例。
- close：服务器关闭时，触发。


http.createServer()方法其实就是添加了一个Reuqest事件监听，如下所示：
```JavaScript
var http = require('http');

var server = http.createServer();

server.on('error', function (err) {
    console.log(err);
});

server.on('request', function (req, res) {
    console.log('有用户请求了');
    console.log(req);
});

server.listen(8081, 'localhost');
```  

### 1.2 http模块常见api

http.IncomingMessage是http请求信息，提供了3个事件：
- data：当请求数据到来时触发；
- end：当请求体数据传输完毕时候触发；
- close：当用户请求结束时候触发。

http.IncomingMessage提供的属性有：
- method：请求方式
- headers：请求头
- url：请求路径
- httpVersion：http版本

http.ServerResponse是返回客户端的信息，主要方法有：
- res.writeHead(statusCode,[headers];	//向请求的客户端发送响应头
- res.write(data,[encoding]);	        //向请求发送内容
- res.end([data],[encoding);            //结束请求