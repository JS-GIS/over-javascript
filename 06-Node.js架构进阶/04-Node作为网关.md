## 一 微服务中的网关

TODO

## 二 Node作为网关

网关有两个基础功能：请求转发与跨域支持。   

原本的app.js:
```js
const http = require("http");

const app = http.createServer( (req, res) => {

    if ( req.url != '/proxy' ) {
        res.writeHead(200, {'Conten-Type': 'text/plain'});
        return res.end('hello world!');
    }

    // 进行代理转发
    let options = {
        host: req.host,
        port: 5000,
        headers: req.headers,
        path: '/proxy',
        agent: false,
        method: 'GET'
    }
    // 模拟出一个新的代理请求
    let httpProxy = http.request(options, response => {
        response.pipe(res);  // 将代理请求的结果返回给当前的res       
    });

    // 执行代理请求
    req.pipe(httpProxy);

});

app.listen(3000, () => {
    console.log("监听端口:", app.address().port);
});
```

被代理的appProxy.js
```js
const http = require("http");

const app = http.createServer( (req, res) => {

    if ( req.url != '/proxy' ) {
        res.writeHead(200, {'Conten-Type': 'text/plain'});
        return res.end('hello new world!');
    }

    res.writeHead(200, {'Conten-Type': 'text/plain'});
    return res.end('hello proxy!');

});

app.listen(5000, () => {
    console.log("监听端口:", app.address().port);
});
```