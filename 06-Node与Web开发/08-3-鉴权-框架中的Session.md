## 一 Express中使用Session

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

