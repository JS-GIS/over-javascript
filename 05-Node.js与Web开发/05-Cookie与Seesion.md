## 一 Cookie
### 1.1 Cookie简介
http是无状态协议，即使在同一个网站，从一个页面跳转到另外一个页面时，服务器和浏览器都没有任何内在的方法认识到他们的关联，所有的http请求都要包含所有必要的信息，服务器才能满足这个请求。
为了实现‘登录’、记忆用户喜好等功能，就有了cookie和session（会话）。
cookie的想法很简单：服务器发送一点信息，浏览器在一段可配置的时期内保存它。
cookie的原理：
访问一个页面，服务器将用户信息发送给下行HTTP报文，命令浏览器存储一个字符串；
再次访问该页面所载的域，浏览器会把该字符串携带到上行HTTP请求中，服务器发现了该字符串，则认为校验通过。
cookie包括：名字、值、过期时间、路径、域，路径与域一起构成cookie的作用范围；cookie只有4k大小。
过期时间如果不设置，关闭浏览器，cookie就会消失，这种生命期为浏览器会话期间的cookie被称为会话cookie。会话cookie一般保存在内存中，这种行为是不规范的。若设置了过期时间，则保存信息到硬盘上。
第一次访问一个服务器，不可能携带cookie，必须是服务器得到这次请求，在下行响应报头中，携带cookie信息，此后每次浏览器往这个服务器发送的请求，都会携带这个cookie。

cookie特点：
- cookie是不加密的，用户可以自由看到；
- 用户可以删除cookie，或者禁用它；
- cookie可以被篡改、攻击（XSS攻击）；
- cookie存储量很小，滥用会被用户注意到，如果可以选择，会话要优于cookie，未来要被localStorage替代。

### 1.2 中间件 cookie-parser
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
### 1.3 cookie加密
方式一：保存的时候加密
方式二：cookie-parser 里面，signed属性设置为true，且需要在调用中间件时传递 任意加密字符串。
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

删除cookie：res.clearCookie(‘secret’);
cookie的其他配置：
domain：将cookie分配给特定的子域名，但是不能分配给和服务器所用域名不同的域名
path：控制应用该cookie的路径，默认是 /  应用到所有页面上，如果参数是 /foo，则会应用到 /foo /foo/bar等路径上
maxAge：cookie有效期，单位是毫秒
secure：cookie只通过https发送
httpOnly：设置为true，表明cookie只能由服务器修改，可以防范XSS攻击。
singend：设为true会对cookie签名，此时只能通过res.signedCookies获取，而不是res.cookies。此时被串改的签名cookie会被服务器拒绝，并且cookie会被重置为原始值。
## 二 session案例
### 2.1 session简介
cookie保存在浏览器中，session保存在服务端。
session的使用过程：
用户第一次访问服务器，服务器创建session对象，生成一个类似key，value的对象，然后将key返回给浏览器，以cookie的形式保存该key。
用户再次访问时，携带了该key，会得到对应的值。
```JavaScript
const express = require('express');
const session = require('express-session');

let app = express();

app.use(session({
//任意一个字符串，作为session的签名
    secret: 'sss',  
    name: 'session_id', //session在本地cookie的名字，可以不设置
    //强制保存session，即使它没有变化，默认为true，建议为false
resave: false,  
//强制将为初始化的session存储，默认是true
saveUninitialized: true,    
cookie: {
   //不设置，那么关闭浏览器就过期，
//设置了即使在浏览页面，50000内没有操作也会过期
        maxAge: 50000
    },
    // secure https下才能访问cookie
//每次请求时强行设置cookue，将重置cookuie过期时间，默认false
    rolling: true   
}));

app.get('/',function (req,res) {
    console.log(req.session.info);
    res.send('index');
});
app.get('/set',function (req,res) {
    req.session.info = 'lisi';
    res.send('set..');
});

app.listen(3000);
```
### 2.2 session销毁
没有设置maxAge，那么session在浏览器关闭时候被销毁，但是有时候用户即使仍在访问，我们也需要主动销毁session，比如用户不关闭浏览器切换账户登录。
销毁方法一：req.session.cookie.maxAge = 0;
销毁方法二：req.session.destroy(function(err){})
### 2.3 多服务器负载均衡共享session
在负载均衡状态下，多个服务器共同协作，用户的请求可能被不同的服务器执行，这时候其中一个服务器保存了session，那么用户下次的请求在别的服务器上，将如何获取？
这时候，需要将session保存在数据库中，比如redis。


