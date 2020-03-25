## 一 TCP简介

TCP即传输控制协议，在OSI七层模型中属于传输层协议，大多应用层协议都是基于TCP构建而来，如：HTTP/SMTP/IMAP等，可以说TCP协议是现代网络编程最重要的基石之一。  

TCP是面向连接的协议，其显著的特征是在传输之前需要进行3次握手形成会话： 

![](../images/node/node&tcp-01.svg)  

当会话形成后，服务器和客户端之间分别提供一个套接字，这2个套接字形成一个连接，服务端和客户端通过套接字实现了二者的连接操作。  

## 二 Node开发TCP程序

### 2.1 TCP服务端

Node使用net模块的createServer方法构建TCP服务器。
```js
var net = require('net');

var server = net.createServer(function (socket) {               // 创建一个套接字socket，作为参数传递给回调函数

    console.log('Connect...');                                  // 有访问就会输出

    socket.write("hello");

    socket.on('data', function(data){
        socket.write("hi");
    });
   
    socket.on('end', function(){
        console.log("connect shut down");
    });

});

server.listen(3000,function () {
    console.log('Listen...');   //服务器启动就会输出
});
```

TCP服务端还可以这样构建：
```js

var net = require('net');

var server = net.createServer();

server.on("connection", function(socket){
    console.log("new connection comming");
})

server.listen(3001,function () {
    console.log('Listen...');                              
});
```

### 2.2 TCP客户端

Node创建TCP客户端只需要创建一个连接TCP客户端的socket对象即可，示例如下：
```js
var net = require('net');

//新建的client对象（就是socket）用来支持数据交互
var client = new net.Socket();

client.connect(3000,'localhost',function () {

    console.log('connect...');                      //运行后输出该句，证明回调已经执行

    client.write('msg of client');                  //发送数据

});

//接收数据
client.on('data',function (data) {
    console.log('the data come from server is ' + data.toString());
});
```


创建client时候可以传入一个json对象，这个json对象有以下属性：
```
fd：置顶一个存在的文件描述，默认为null
readable：是否允许在这个socket上读，默认为false
writeable：是否允许在这个socket上写，默认为false
allowHalfOpen：该属性为false时，TCP服务器接收到客户端发送的一个FIN包后将会回发一个FIN包，该属性为true时，TCP服务器接收客户端发送的一个FIN包后不会回发FIN包。
``` 