##  一 Session简介

cookie的数据有严重的安全问题，很容易被篡改、伪造，比如在上一节案例中，开发者直接发一个请求中带有 isLogin 的字段，服务端就判断其为登录状态了而且Cookie对敏感数据的保护是无效的。  

Session是专门为了解决Cookie数据安全性而提出来的技术，其实现基于Cookie。  

一般Session的实现步骤：
- 用户第一次访问服务器，服务器创建session对象，生成一个类似key，value的对象，然后将key返回给浏览器，以cookie的形式保存该key。
- 用户再次访问时，携带了该key，会得到对应的值。

## 二 Session的实现

### 2.1 第一种 基于Cookue实现用户和数据的映射

可以将Session的口令存储在Cookie中，一旦口令被篡改，就会丢失映射关系，无法修改服务端数据了，而且Session有效期很短，通常为20分钟，也就是说，20分钟内客户端与服务端之间没有交互产生，服务端就会将该Cookie中的口令数据删除。  

```js
var sessions = {};
var key = 'session_id';
var EXPIRES = 20 * 60 * 1000;

function generate() {
    var session = {};

    session.id = (new Date()).getTime() + Math.random();
    session.cookie = {
        expire: (new Date()).getTime() + EXPIRES
    };

    sessions[session.id] = session;

    return session;
};
```

每个请求到来时，检查Cookie中的口令与服务端的数据，如果过期，就重新生成：
```js
function (req, res) {
    var id = req.cookies[key];
    if (!id) {
        req.session = generate();
    } else {
        var session = sessions[id];
        if (session) {
            if (session.cookie.expire > (new Date()).getTime()) {
                // 更新超时时间
                session.cookie.expire = (new Date()).getTime() + EXPIRES;
                req.session = session;
            } else {
                // 超时了，删除旧的数据，并重新生成
                delete sessions[id];
                req.session = generate();
            }
        } else {
            // 如果session过期或口令不对，重新生成session
            req.session = generate();
        }
    }
    handle(req, res);
}
```

此时还需要在响应给客户端时设置新的值，以便下次请求时能够对应服务端的数据，这里重新实现writeHead()方法，在内部注入设置Cookie的逻辑：
```js
var writeHead = res.writeHead;
res.writeHead = function () {
    var cookies = res.getHeader('Set-Cookie');
    var session = serialize('Set-Cookie', req.session.id);
    cookies = Array.isArray(cookies) ? cookies.concat(session) : [cookies, session];
    res.setHeader('Set-Cookie', cookies);
    return writeHead.apply(this, arguments);
};
```

业务逻辑使用session：
```js
function handle(req, res) {
    if (!req.session.isLogin) {
        res.session.isLogin = true;
        res.writeHead(200);
        res.end('欢迎登陆');
    } else {
        res.writeHead(200);
        res.end('请先登录');
    }
};
```

该方案依赖了Cookie的实现，也是大多Web系统中Session的实现方案，如果客户端禁止Cookie，则本方案将会失效。

### 2.2 第二种 通过查询字符串来实现浏览器和服务端数据对应

该方案原理是检查请求的查询字符串，如果没有值，则会先生成新的带值的URL，禁用cookie时可以采用该方案：
```js
function getURL(_url, key, value) {
    var obj = url.parse(_url, true);
    obj.query[key] = value;
    return url.format(obj);
};
```
然后形成跳转，让客户端重新发起请求：
```js
function (req, res) {

    var redirect = function (url) {
        res.setHeader('Location', url);
        res.writeHead(302);
        res.end();
    };

    var id = req.query[key];

    if (!id) {
        var session = generate();
        redirect(getURL(req.url, key, session.id));
    } else {
        var session = sessions[id];
        if (session) {
            if (session.cookie.expire > (new Date()).getTime()) {
                // 更新超时时间
                session.cookie.expire = (new Date()).getTime() + EXPIRES;
                req.session = session;
                handle(req, res);
            } else {
                // 超时了，删除旧的数据，并重新生成
                delete sessions[id];
                var session = generate();
                redirect(getURL(req.url, key, session.id));
            }
        } else {
            // 如果session过期或口令不对，重新生成session
            var session = generate();
            redirect(getURL(req.url, key, session.id));
        }
    }
}
```


## 三 负载均衡下的session与redis

在负载均衡状态下，多个服务器共同协作，用户的请求可能被不同的服务器执行，这时候其中一个服务器保存了session，那么用户下次的请求在别的服务器上，将如何获取？这时候，需要将session保存在数据库中，比如redis。

## 四 Express中使用Session

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

没有设置maxAge，那么session在浏览器关闭时候被销毁，但是有时候用户即使仍在访问，我们也需要主动销毁session，比如用户不关闭浏览器切换账户登录。
销毁方法一：req.session.cookie.maxAge = 0;
销毁方法二：req.session.destroy(function(err){})

