## 一 TCP
### 1.1 基本构建
Node使用net模块的createServer方法构建TCP服务器。
```JavaScript
const net = require('net');
let server = net.createServer(function (socket) {
    console.log('Connect...');  //有访问才会输出
    server.maxConnections = 3;  //可以用来设置服务器最大连接数
    server.getConnections(function (err,count) {    //count为当前连接数
        console.log(count);
    });
});
server.listen(3000,function () {
    console.log('Listen...');   //服务器启动就会输出
});
```
除了listening事件外，还支持connection，close，error事件。
当创建了一个TCP服务器后，可以通过server.address()方法来查看这个TCP服务器监听的地址，并返回一个JSON对象，该JSON对象的属性有：
port：TCP服务器监听的端口号；
family：说明TCP服务器监听的地址是PVV6还是IPV4；
address：TCP服务器监听的地址；
因为这个方法返回的是TCP服务器监听的地址信息，所以这个方法应该在使用了server.listen()方法或者绑定事件listening中的回调函数中使用。
### 1.2 与客户端进行数据交互
createSever()方法的回调函数参数是一个net.Socket对象（服务器所监听的端口对象），这个对象同样也有一个address()方法，用来获取TCP服务器绑定地址，同样也是返回一个含有port family address 属性的对象。
socket对象可以用来data事件接收客户端发送的流数据，支持的事件还有：connect，end，error，timeout。
利用socket.write()可以使TCP服务器发送数据，这个方法只有一个必须参数，就是需要发送的数据：第二个参数为编码格式，可选。同时，可以为这个方法设置一个回调函数。当有用户连接TCP服务器的时候，将发送数据给客户端：
```JavaScript
const net = require('net');
let server = net.createServer(function (socket) {
    //获取地址信息
    let address = server.address();
    let message = 'server address is ' + JSON.stringify(address);
    socket.write(message,function () {
        let writesize = socket.bytesWritten;
        console.log(message + ' has end');
        console.log('the size of message is ' + writesize);
    });
    socket.on('data',function (data) {
        console.log('data is ' + data.toString());
        let readsize = socket.bytesRead;
        console.log('the size of data is ' + readsize);
    });
}).listen(3000);
```

可以使用telnet与服务器进行交互。上述代码中用到socket对象的bytesWritten和bytesRead属性，分别代表发送数据的字节数和接收数据的字节数。socket还有以下属性：
socket.localPort：本地端口的地址
socket.localAddress：本地IP地址
socket.remotePort：远程端口地址
socket.remoteFamily：远程IP协议地址
socket.remoteAddress：远程的IP地址
### 1.3 使用node创建TCP客户端
Node创建TCP客户端只需要创建一个连接TCP客户端的socket对象即可，创建时候可以传入一个json对象，这个json对象有以下属性：
fd：置顶一个存在的文件描述，默认为null
readable：是否允许在这个socket上读，默认为false
writeable：是否允许在这个socket上写，默认为false
allowHalfOpen：该属性为false时，TCP服务器接收到客户端发送的一个FIN包后将会回发一个FIN包，该属性为true时，TCP服务器接收客户端发送的一个FIN包后不会回发FIN包。
```JavaScript
let net = require('net');
let client = new net.Socket();
client.connect(3000,'localhost',function () {
    console.log('connect...');  //运行后输出该句，证明回调已经执行

    //发送数据
    client.write('msg of client');

});
//新建的client对象（就是socket）用来支持数据交互

//接收数据
client.on('data',function (data) {
    console.log('the data come from server is ' + data.toString());
});

```
## 二 UDP
### 1.1 简介
TCP数据传输是一种可靠的数据传输方式，在数据传输之前必须建立客户端与服务端之间的连接。UDP是一种面向非连接的协议，所以其传输速度比TCP更加快速。
Node使用dgram模块中的createSocket()方法创建一个UDP服务器，这个方法接收一个必须参数和一个可选参数，必须参数是一个标识UDP协议的类型，可指定为udp4或者udp6，可选参数是一个回调函数，即UDP服务器接收数据时触发的回调函数，回调函数有2个参数，一个是接收的数据，一个是存放发送者信息的对象。
```JavaScript
const dgram = require('dgram');
let socket = dgram.createSocket('udp4',function (msg,rinfo) {
    // code
});
socket.bind(3000,'localhost',function () {
    console.log('bind 3000....');
});
```
这里与TCP的createServer一样，返回的都是socket对象，且回调函数就是监听message事件，因此使用createServer也可以不指定回调函数，直接显示的监听message事件就可以。
```JavaScript
const dgram = require('dgram');
let socket = dgram.createSocket('udp4');
socket.bind(3000,'localhost',function () {
    console.log('bind 3000....');
});

socket.on('message',function (msg,rinfo) {
    console.log(msg.toString());
});
```
###  2.2 与客户端数据交互
socket对象主要有以下方法：
bind();	绑定端口号
send();	发送数据
address();	获取该socket端口对象相关的地址信息；
close();		关闭socket对象
```JavaScript
const dgram = require('dgram');
let socket = dgram.createSocket('udp4',function (msg,rinfo) {
    console.log(msg.toString());
    let message = new Buffer('hello,I am server');
    socket.send(message,0,message.length,rinfo.port,rinfo.address,function (err,bytes) {
        if(err){
            console.log('error is ' + err);
        } else {
            console.log('send ' + bytes + ' message');
        }
    });
}).bind(3000,'localhost');

```

send方法有很多参数：
buf：需要发送的消息，可以是缓存对象或者字符串
offset：是个整数，代表消息在缓存中的偏移量
length：是个整数，代表消息的比特数

### 2.3 UDP客户端
```JavaScript
const dgram = require('dgram');
let message = new Buffer('msg from client');
let socket = dgram.createSocket('udp4');
socket.send(message,0,message.length,3000,'localhost',function (err,bytes) {
    if(err){
        console.log(err);
        return;
    } else {
        console.log('client send ' + bytes + 'message')
    }
});
socket.on('message',function (msg,rinfo) {
    console.log('msg from server');
});

```
