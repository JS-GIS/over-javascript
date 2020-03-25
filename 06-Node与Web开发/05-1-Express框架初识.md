## 一 Node开发框架

原生的Node开发，会出现很多需要解决的问题，如：
- 呈递静态页面很不方便，需要处理每个HTTP请求，还要考虑304问题
- 路由处理代码不直观清晰，需要写很多正则表达式和字符串函数

框架可以解决很多原生开发中的问题，常见的Node Web框架有：
- Express:TJ所著，用户量最大的框架
- Koa：TJ所著，号称下一代Node的Web开发框架
- Nest：号称与Java的著名框架Spring一样

## 二 Express

安装方式：
```
# 初始化一个项目
mkdir myproject
cd myproject
npm init -y
npm install express -S
```

使用express启动一个web服务：
```js
var express = require('express');

var app = express();

app.get("/", function(req, res){
    res.send('hello world!');
});

app.listen(3000);
```

## 三 Express常见技术使用

静态服务：
```JavaScript
app.use(express.static("./public"));
//一句话，就将public文件夹变为了静态服务文件夹。当然也可以指定访问路由名称：
app.use('/test',express.static('public'));		//test是虚拟路径
```

模板引擎：
```JavaScript
app.set("view engine","ejs");
app.get("/",function(req,res){
    res.render("haha",{
        "news" : ["我是小新闻啊","我也是啊","哈哈哈哈"]
    });
}); 
//但是上述模板引擎定义后，我们使用的模板依然需要修改后缀为设定的引擎后缀，进行下列设定，无需修改html后缀：
app.engine('html',ejs.renderFile);
app.set('view engine','html');
```

错误处理：一般情况下，普通的错误，重定向或者展示一个错误页面即可，express内部在执行路由时候，其实也是用到了tyr catch。但是这些并不是真正的捕获了异常。如下无法捕获：
```JavaScript
app.get('/test', function (req, res) {
    process.nextTick(function () {
        throw new Error('error');
    });
});
```
该错误被推迟到Node空闲时执行，但是当Node空闲时，上下文已经不存在了，所以只能关闭服务器！！！出现未捕获异常，唯一能做的是关闭服务器，但是我们要保证服务器的正常关闭，且有故障转移，最容易的故障转移机制是集群。  

正常关闭服务器办法，node有2个：uncaughtException事件和域，严重推荐后者。域是一个执行上下文，会捕获在其中发生的错误。  

每个请求都在域中处理，能达到很好的错误处理，封装一个放在最前的中间件：
```JavaScript
const express = require('express');

let app = express();

app.use(function (req, res,next) {

    let domain = require('domain').create();    //为这个请求创建一个域
    domain.on('error',function (err) {
        console.log("domain err " + err.stack);
    });

    try  {

        setTimeout(function () {         //5秒内故障保护关机
            console.log('server shutdown');
        },5000);

        let worker = require('cluster').worker;
        if (worker) worker.disconnect();    //从集群断开

        server.close();             //停止接收新请求

        try {
            next(err);  //尝试使用express错误路由
        } catch (e) {
            console.log("express error failed " +e.stack);      //如果express错误路由失效
            res.statusCode = 500;
            res.setHeader('content-type','text/plain');
            res.end('Server error');
        }
    } catch (error) {
        console.log('Unable to send 500 ', error.stack);
    }

    domain.add(req);        //向域中添加请求、响应对象
    domain.add(res);
    domain.run(next);
});

app.get('/test', function (req, res) {
    process.nextTick(function () {
        throw new Error('error');
    });
});

app.listen(3000);
```
上面的代码中，一旦出现未捕获错误，就会调用该函数，
