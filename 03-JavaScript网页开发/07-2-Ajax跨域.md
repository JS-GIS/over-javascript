## 一 跨域
### 1.1 跨域的产生
如果一个域下的文件请求了和它不一样域（IP、端口、协议、二级域名）的资源，就会产生跨域请求。
跨域解决方案：
- iframe：包含跨域的文件，但无法对其内部进行dom操作、处理数据
- 后台请求：比如让本地php请求跨域资源，然后ajax访问本地的php；
- Flash
- -JSONP：json with padding
### 1.2 解决跨域问题方式一 JSONP
script标签具备跨域的特性，即可以获取不同域下的文件。
比如使用script标签引入cdn上的jquery文件就是利用了script标签的跨域功能。
这时候如果直接使用script标签还会遇到问题：
```
服务端返回数据：res.send("你好")
前端script标签加载该请求地址拿到数据，此时数据虽然拿到了，但是无法使用变量接收，也就自然无法使用。

如果服务器这样返回：res.send("let str = '你好'");
此时前端直接输出 console.log(str) 我们发现可以拿到结果了！！！

但是这时候我们需要防止 script标签的异步加载问题，而且要注意顺序问题，所以我们一般动态的创建script标签来获取数据。
```
动态创建script标签方式：
```js
let script = document.createElement('script);
script.src = "http://127.0.0.1:3000/userinfo";
document.getElementsByTagName("head")[0].appendChild("script");
```
随之而来的新问题：
```js
//按照上述方式动态创建script标签，发送的请求是异步的，无法获取请求结果！！！

//解决办法：服务端返回一个函数调用即可

//服务端
res.send("foo("你好")");


//前端
function foo(data) {
    console.log(data);
}

```

上述已经完全实现了jsonp的原理，但是还有可以优化的地方：服务端返回的函数名被限制为了foo。实际开发中，我们肯定需要这里很灵活：
```js
//前端的请求地址中添加一个参数
script.src = "http://127.0.0.1/userinfo?callback=cb"
cb(data);
//后端处理
let fnName = req.query.callback;
res.send(fnName + "(" + "你好" + ")" );
```

### 1.3 jQuery中的跨域
jQuery已经封装好了跨域配置，可以直接使用
```js
$.ajax({
    type: 'get',
    url: 'http://127.0.0.1/userinfo',
    dataType:'jsonp',
    success: function(data){}
})
```
