## 一 Node开发框架
原生的Node开发，会出现很多需要解决的问题，如：
- 呈递静态页面很不方便，需要处理每个HTTP请求，还要考虑304问题
- 路由处理代码不直观清晰，需要写很多正则表达式和字符串函数
Node的开元贡献者们提供了各种框架来解决原生开发的问题，常见的框架有：
Express:TJ所著，用户量最大的框架
Koa：TJ所著，号称下一代Node的Web开发框架
nest：号称与Java的著名框架Spring一样
本章介绍express，安装方式：
```
npm install express --save
```
## 二 路由与中间件
### 2.1 路由展示
```JavaScript
const express = require('express');
let app = express();

app.get('/',function (req,res) {
    res.send('hi');
});

app.get('/showname',function () {
    res.send('name si lisi');
});

app.get(/^\/student\/([\d]{10})$/,function(req,res){
    res.send("学生信息，学号" + req.params[0]);
});

app.get("/teacher/:gonghao",function(req,res){
    res.send("老师信息，工号" + req.params.gonghao);
});

app.listen(3000);

```
注意：express使用res.set和res.status取代了原生的res.writeHead()
### 2.2 常见路由API
```
app.get("网址",function(req,res){	});      //get请求
app.post("网址",function(req,res){  });   //post请求
app.all("网址",function(){  });          //处理这个网址的任何类型请求
```
注意：
所有的GET参数，? 后面的都已经被忽略，锚点#也被忽略，路由到/a ， 实际/a?id=2&sex=nan 也能被处理；
正则表达式可以被使用。正则表达式中，未知部分用圆括号分组，然后可以用req.params[0]、[1]得到，req.params是一个类数组对象。
如果get、post回调函数中，没有next参数，那么就匹配上第一个路由，就不会往下匹配了。如果想往下匹配的话，那么需要写next()：
```JavaScript
app.get("/",function(req,res,next){
    console.log("1");
    next();
});

app.get("/",function(req,res){
    console.log("2");
});
```
### 2.3 参数获取
GET请求的参数在URL中，在原生Node中，需要使用url模块来识别参数字符串。在Express中，不需要使用url模块了。可以直接使用req.query对象。
POST请求的参数在express中不能直接获得，必须使用body-parser模块。使用后，将可以用req.body得到参数。但是如果表单中含有文件上传，那么还是需要使用formidable模块。
冒号参数引起的路由问题：

```JavaScript
app.get("/:username/:id",function(req,res){    //express推荐使用冒号获得参数
    console.log("1");
    res.send("用户信息" + req.params.username);
});

app.get("/admin/login",function(req,res){
    console.log("2");
    res.send("管理员登录");
});

```
上面两个路由，感觉没有关系,但是实际上冲突了，因为admin可以当做用户名 login可以当做id。
```JavaScript
//解决方法1 交换位置。 也就是说，express中所有的路由（中间件）的顺序至关重要，匹配上第一个，就不会往下匹配了。 具体的往上写，抽象的往下写。
app.get("/admin/login",function(req,res){
    console.log("2");
    res.send("管理员登录");
});

app.get("/:username/:id",function(req,res){
    console.log("1");
    res.send("用户信息" + req.params.username);
});

//解决方法2： 
app.get("/:username/:id",function(req,res,next){
    let username = req.params.username;
    //检索数据库，如果username不存在，那么next()
    if(检索数据库){
        console.log("1");
        res.send("用户信息");
    }else{
        next();
    }
});

app.get("/admin/login",function(req,res){
    console.log("2");
    res.send("管理员登录");
});
```
### 2.4 错误处理与端口设置
```JavaScript

const express = require('express');

let app = express();

app.set('port', process.env.PORT || 9001);

app.get('/',function (req, res) {
    res.send("hello world!");
});


app.use(function (req, res) {
    res.type('text/plain');
    res.status(404);
    res.send('404 Not Found');
});                                      //404

app.use(function (err, req, res, next) {        500
    res.type('text/plain');
    res.status(500);
    res.send('Server-Error:' + err.stack);
});



app.listen(app.get('port'), function() {
    console.log('Express server listeni.ng on port ' + app.get('port'));
});

```
## 三 中间件
中间件是在管道中执行的，在Express中，使用app.use()向管道中插入中间件。
中间件讲究顺序，匹配上第一个之后，就不会往后匹配了，next函数才能够继续往后匹配。
模糊意义上讲，app.get() app.post()等方法也属于中间件。
app.use()也是一个中间件。与get、post不同的是，他的网址不是精确匹配的，使用user(path,fn())时候，user内的路由，将匹配/path /path/images/  /path/images/1.png 等路由情况。
如果写一个 / 实际上就相当于"/"，就是所有网址，也可以直接不写该地址。
大坑：express的中间件函数，不需要传入req，res，他是在中间件函数执行回调的时候自动传入：
创建中间件：
```JavaScript
module.exports = function (req, res, next) {
    //中间件函数在这里调用
    next();     //记得使用next()或者 next(route)
};
```
使用中间件：
```JavaScript
const test2 = require('./test1');
app.use(test2);

app.get('/',function (req, res) {//每次访问/调用一次中间件
    res.send("hello world!");
});

```
创建可配置的模块化中间件：
```JavaScript
module.exports = function (config) {    
    if (!config) config= {}
    return function (req, res, next) {
        next();     //除非这个中间件是终点，否则需要next
    }
};
```
输出多个相互关联的中间件：
```JavaScript
module.exports = function (config) {
    if (config)  config= {}
    return {
        m1: function (req, res, next) {
            next();     //除非这个中间件是终点，否则需要next
        },
        m2: function (req, res, next) {
            next();     //除非这个中间件是终点，否则需要next
        }
    }
};

```
使用互相关联的中间件：
```JavaScript
const test2 = require('./test1')({option:'test'});
app.use(test2.m1);
app.use(test2.m2);

```
也可以将处理函数挂载在一个对象的原则上：
```JavaScript
//创建中间件
function Stuff(config) {
    this.config = config || {};
}

Stuff.prototype.m1 = function (req, res, next) {
    //注意这里的this最好别用
};
module.exports = Stuff;
//使用中间件
const test2 = require('./test1');
let stuff = new test2({option:"test"});
app.use(stuff.m1);
```
## 四 常用技术
### 4.1 静态服务
```JavaScript
app.use(express.static("./public"));
//一句话，就将public文件夹变为了静态服务文件夹。当然也可以指定访问路由名称：
app.use('/test',express.static('public'));		//test是虚拟路径
```
### 4.2 模板引擎使用
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
### 4.3 服务器安全
响应报头会包含大量服务端信息，在express中可以直接禁止：
```JavaScript
app.disable(‘x-powered-by);
```
### 4.4 express正确处理ajax
```JavaScript
app.post('/',function (req, res) {
    if (req.xhr || req.accept ('json,html') === 'json') {   //判断
        //一系列操作
        res.send({status:"1"});
    } else {
        res.redirect('303','/303');
    }
});
```
### 4.5 错误处理
一般情况下，普通的错误，重定向或者展示一个错误页面即可，express内部在执行路由时候，其实也是用到了tyr catch。但是这些并不是真正的捕获了异常。如下无法捕获：
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


