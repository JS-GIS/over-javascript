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

用户访问`http://localhost/pathname`时，如果服务端发现查询字符串中没有session_id参数，就会将用户调转到 `http://localhost/pathname?session_id=12344567` 类似这样的地址，如果浏览器收到302状态码和Location报头，就会重新发起新的请求：
```
< HTTP/1.1 302 Moved Temporarily
< Location: /pathname?session_id=12344567
```

这样，新的请求到来时就能通过Session的检查。虽然该方案可以应对Cookie被禁用的情况，但是只要将地址栏中的地址发给另外一个人，他就会拥有和你相同的身份，风险更大！


## 三 集中式Session管理

在上述案例中Session都是存储在一个Node进程的变量中的。这会引起两个问题：
- 状态过多，如登录用户数目极大，会突破Node进程的内存限制，引起频繁GC扫描，造成性能问题
- Node多进程中不共享内存，Session就会出现错乱
- 在负载均衡状态下，多个服务器共同协作，用户的请求可能被不同的服务器执行，这时候其中一个服务器保存了session，那么用户下次的请求在别的服务器上，将如何获取？

通常Session不会被考虑直接存储在业务进程中，一般将session保存在缓存服务器中，如redis、memcache。


## 四 Session安全

Session的口令仍然是存储在Cookie中的，同样存在口令盗用的情况。通常可以对口令进行私钥加密签名，提升伪造成本。在服务端，只需要在响应数据时将口令和签名进行对比，如果签名非法，则服务端的数据立即过期即可：
```js
// 将值通过私钥签名，由 . 分割原值和签名
var sign = function (val, secret) {
    return val + '.' + crypto
        .createHmac('sha256', secret)
        .update(val)
        .digest('base64')
        .replace(/\=+$/, '');
};
```

在响应时，设置session值到Cookie中或者跳转URL中：
```js
var val = sign(req.sessionID, secret);
res.setHeader('Set-Cookie', cookie.serialize(key, val));
```

接收请求时，检查签名，对比用户提交的值：
```js
// 取出口令部分进行签名，对比用户提交的值
var unsign = function (val, secret) {
    var str = val.slice(0, val.lastIndexOf('.'));
    return sign(str, secret) == val ? str : false;
};
```

这样一来，即使攻击者知道口令中 . 号的前的值是服务端Session的ID的值，只要不知道私钥的值，就无法伪造签名信息。