## 一 http模块

### 1.1 简单的web程序性

```JavaScript
const http = require('http');
let server = http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/plain;charset=UTF8'});
    res.end('hello world');
});
server.listen(8000);
```

http.createServer()方法返回的是http模块封装的一个基于事件的http服务器。  

req，res分别是http.IncomingMessage和http.ServerResponse的实例。  

http.Server的主要事件有：
- request：客户端发起请求时，被处罚，提供req，res参数
- connection：TCP建立连接时候处罚，提供一个scoket参数，是net.Socket的实例。
- close：服务器关闭时，触发。


http.createServer()方法其实就是添加了一个Reuqest事件监听，如下所示：
```JavaScript
const http = require('http');
let server = http.createServer();
server.on('error', function (err) {
    console.log(err);
});
server.on('request', function (req, res) {
    console.log('有用户请求了');
    console.log(req);
});
server.listen(8081, 'localhost');
```  

### 1.2 Node没有web容器

在Java、php等语言中，还需要一个web容器来存放web资源（如Java的Tomcat，PHP的Apache），而Node没有这样的概念。
所以，Node很难提供一个静态文件服务，也就是说，node.js中，如果看见一个网址是：127.0.0.1:3000/fang，不一定有fang这个文件夹。
即：URL和真实物理文件，是没有关系的，URL是通过了Node的顶层路由设计，呈递某一个静态文件的。

### 1.3 http模块常见api

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

## 二 简单的路由实现

```JavaScript
const http = require('http');
const url = require('url');
const fs = require('fs');
let server = http.createServer();
server.on('request', function (req, res) {
    console.log('有用户请求了');
    let urlStr = url.parse(req.url);
    switch (urlStr.pathname) {
        case '/':           //首页 
            res.writeHead(200, {'content-type': 'text/html;charset=utf-8'});
            res.end('<h1>这是首页</h1>');
            break;
        case '/user':       //用户页面 
            sendData(__dirname + '/html/' + '1.html', req, res);
            break;
        default:
            res.writeHead(404, {'content-type': 'text/html;charset=utf-8'});
            res.end('<h1>页面不存在</h1>');
            break;
    }
});

function sendData(filePath, req, res) {
    fs.readFile(filePath, function (err, data) {
        if (err) {
            res.writeHead(404, {'content-type': 'text/html;charset=utf-8'});
            res.end('<h1>页面不存在</h1>');
        } else {
            res.writeHead(200, {'content-type': 'text/html;charset=utf-8'});
            console.log(String(data));
            res.end(data);
        }
    });
}

server.listen(8081, 'localhost');
```