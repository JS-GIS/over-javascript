## 一 跨文档通信
在过去，跨源或者跨窗口之间的通信往往是与服务端进行数据交互来实现的，并且需要借助轮训或者Connect技术来监听消息。  
H5提供了PostMessages实现安全的跨源通信：
```javascript
otherWindow.postMessage(message, targetOrigin, [transfer]);

// otherWindow:其他窗口的引用，比如iframe中的contentWindow属性，执行widnow.open返回的窗口对象。
//三个参数：
//message:将要发给其他窗口的数据
// targetOrigin:通过创窗口的origin属性来指定哪些窗口能接收到消息事件，其值可以是字符“*”（表示无限制）或者一个url
// transfer:可选参数，是一串和message同时传递的Transferable对象，这些对象的所有权将被转译给消息的接收方，而发送乙方将不再保有所有权
```
iframe应用实例：
```html
    <button id="btn">点击发送消息给iframe</button>
    <iframe src="http:127.0.0.1/iframe.html"></iframe>
    <script>
        let btn = document.querySelector("#btn");
        let data = ["周一","周二","周五"];
        btn.onclick = function(){
            alert("执行发送数据给iframe？");
            window.parent.postMessage(data, "http:127.0.0.1/iframe.html");
        }
    </script>
```
iframe接受数据：
```javascript
    <script>
        window.addEventListener("data", e =>{
            console.log("origin=", e.origin);
            console.log("data=", e.data);
        });
    </script>
```
## 二 Ajax2.0新特性
设置HTTP请求超时时间
```javascript
xhr.timeout = 3000;
xhr.ontimeout = function(e){}
```
使用FormData表单对象管理表单数据：
```javascript
let formData = new FormData();
formData.append("username", "zs");
formData.append("id", 2331);
xhr.send(formData);
```
ajax2.0上传文件
```javascript
for (let i = 0; i < files.length; i++) {            //files是一个选择文件的表单元素
    formData.append("files[]", files[i]);
}
```
全新的跨域请求，需要服务端配合。  
获取服务端的二进制数据：
```javascript
xhr.open("GET", "...");
xhr.responseType = 'blob';                      //表示服务器传回的数据是二进制对象
```
获取数据传输的进度信息
```javascript

```
## 三 Server Sent Event 服务器主动通信
像邮件这样的应用，服务端发送了邮件后，需要主动通知客户端。传统的做法是，客户端不断的去轮训服务端，这种做法十分不优雅！  
H5提供了Server Sent Event技术解决上述问题：
- 简单轻量
- 单向数据传送（服务端向客户端传送）
- 基于HTTP协议
- 默认支持断线重连接
- 自定义发送数据类型
```javascript
          if (typeof (EventSource) !== "undefined") {

                let source = new EventSource("/test");

                source.onmessage = function (e) {
                    div.innerHTML = e.data;
                }
                
            } else {
                alert("浏览器不支持服务器发送事件");
            }
```
