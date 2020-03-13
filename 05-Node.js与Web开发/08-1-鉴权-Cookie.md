## 一 Cookie简介

http是无状态协议，即使在同一个网站，从一个页面跳转到另外一个页面时，服务器和浏览器都没有任何内在的方法认识到他们的关联，所有的http请求都要包含所有必要的信息，服务器才能满足这个请求。

cookie是浏览器端的技术，用来实现类似 登录、记忆用户喜好 等功能。cookie的处理步骤：
- 服务端向客户端发送Cookie
- 浏览器将收到的Cookie保存
- 之后的每次请求，浏览器都会将Cookie信息发送给服务端

cookie包括：名字、值、过期时间、路径、域，只有4k大小。路径与域一起构成cookie的作用范围。  

cookie过期时间如果不设置，关闭浏览器，cookie就会消失，这种生命期为浏览器会话期间的cookie被称为会话cookie。会话cookie一般保存在内存中，这种行为是不规范的。若设置了过期时间，则保存信息到硬盘上。  

cookie特点：
- cookie是不加密的，用户可以自由看到；
- 用户可以删除cookie，或者禁用它；
- cookie可以被篡改、攻击（XSS攻击）；
- cookie存储量很小，滥用会被用户注意到，如果可以选择，会话要优于cookie，未来要被localStorage替代。

## 二 Node中使用Cookie

构造一个携带Cookie的请求：
```
curl -v -H "Cookie: foo=bar;baz=val" "http://127.0.0.1:3000"
```

Node解析请求中的Cookie：
```js
function parseCookie(cookie) {

    var cookies = {};
    if (!cookie) {
        return cookies;
    }

    var list = cookie.split(';');

    for (var i = 0; i < list.length; i++) {
        var pair = list[i].split('=');
        cookies[pair[0].trim()] = pair[1];
    }

    return cookies;
};
```

在业务逻辑代码执行之前，可以将cookie挂载在req对象上，让业务代码可以直接访问：
```js
function (req, res) {
    req.cookies = parseCookie(req.headers.cookie);
    hande(req, res);
}

function hande(req, res) {

    res.writeHead(200);

    if (!req.cookies.isVisit) {
        res.end('࣌ᆓڼᅃْઠڟ动࿿ᇴ ');
    } else {
        // TODO
    }
};
```

响应的Cookie值在Set-Cookie字段中，其格式如下：
```
Set-Cookie: name=value; Path=/; Expires=Sun, 23-Apr-23 09:01:35 GMT; Domain=.domain.com;
```

其中name=value是必须包含的部分，其余部分皆是可选参数：
- path：表示这个Cookie影响到的路径，当前访问的路径不满足该匹配时，浏览器则不发送这个Cookie
- Expires和Max-Age分别用来告知浏览器Cookie何时过期，多久后过期。如果不设置该选项，则浏览器一旦关闭，Cookie直接就会消失。
- HttpOnly：告知浏览器不允许通过脚本`document.cookie`去更改这个Cookie只，事实上，设置HttpOnly之后，这个值在`document.cookie`中不可见，但是在Http请求过程中，依然会发送这个Cookie到服务端
- Secure：该值为true时，只有在HTTPS中才有效，表示创建的Cookie只有在HTTPS连接中才会被传输

cookie序列化的方法：
```js
function serialize(name, val, opt) {

    var pairs = [name + '=' + encode(val)];

    opt = opt || {};

    if (opt.maxAge) pairs.push('Max-Age=' + opt.maxAge);
    if (opt.domain) pairs.push('Domain=' + opt.domain);
    if (opt.path) pairs.push('Path=' + opt.path);
    if (opt.expires) pairs.push('Expires=' + opt.expires.toUTCString());
    if (opt.httpOnly) pairs.push('HttpOnly');
    if (opt.secure) pairs.push('Secure');
    
    return pairs.join('; ');
};
```

用户状态的判断业务代码：
```js
function handle(req, res) {
    if (!req.cookies.isLogin) {
        res.setHeader('Set-Cookie', serialize('isVisit', '1'));
        res.writeHead(200);
        res.end('欢迎登陆');
    } else {
        res.writeHead(200);
        res.end('请先登录');
    }
};
```

报头中也可能存在多个字段：
```js
res.setHeader('Set-Cookie', [serialize('foo', 'bar'), serialize('baz', 'val')]);
```

## 三 Cookie的性能问题

由于网络传输需要额外传输Cookie信息，一旦设置的Cookie过多，将会导致报头较大，而且大多数Cookie并不需要每次都用上，同样会造成带宽的浪费。  

应该：
- 减小Cookie大小
- 为静态组件使用不同的域名，避免不必要的Cookie
- 减少DNS查询，现代浏览器都支持DNS缓存，可以削弱DNS查询带来的影响。


## 四 Express中使用cookie

### 4.1 中间件 cookie-parser

```JavaScript
const express = require('express');
const cookieParser = require('cookie-parser');

let app = express();
app.use(cookieParser());

app.get('/',function(req,res){
    res.cookie('name','lisi',{maxAge:900000,httpOnly:true});
    res.send(req.cookies);
});

app.get('/other',function (req,res) {
    //当用户访问首页缓存了cookie后，再访问other，在这个页面我们就可以获取cookie
   let cookieName = req.cookies.name;
   res.send(cookieName);

});
```

### 4.2 cookie加密

cookie的加密方式有两种：
- 方式一：保存的时候加密
- 方式二：cookie-parser 里面，signed属性设置为true，且需要在调用中间件时传递 任意加密字符串。
```JavaScript
const express = require('express');
const cookieParser = require('cookie-parser');

let app = express();

app.use(cookieParser('sss'));

app.get('/',function (req,res) {
    //获取加密的cookie
    console.log(req.signedCookies);
    res.send('hi');
});

app.get('/set',function (req,res) {
    res.cookie(
        'info',
        'lisi',
        {
            maxAge: 60000,
            signed: true
        }
    );
    res.send('set cookie');
});

app.listen(3000);

```
### 4.3 删除cookie

删除cookie：`res.clearCookie(‘secret’);`

### 4.4 cookie的其他设置

cookie的其他配置：
```
domain：将cookie分配给特定的子域名，但是不能分配给和服务器所用域名不同的域名
path：控制应用该cookie的路径，默认是 /  应用到所有页面上，如果参数是 /foo，则会应用到 /foo /foo/bar等路径上
maxAge：cookie有效期，单位是毫秒
secure：cookie只通过https发送
httpOnly：设置为true，表明cookie只能由服务器修改，可以防范XSS攻击。
singend：设为true会对cookie签名，此时只能通过res.signedCookies获取，而不是res.cookies。此时被串改的签名cookie会被服务器拒绝，并且cookie会被重置为原始值。
```