## 一 Express路由

### 1.1 路由展示

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

### 1.2 常见路由API

```js
app.get("网址",function(req,res){	});      //get请求
app.post("网址",function(req,res){  });   //post请求
app.all("网址",function(){  });          //处理这个网址的任何类型请求
```
注意：
- 所有的GET参数，? 后面的都已经被忽略，锚点#也被忽略，路由到/a ， 实际/a?id=2&sex=nan 也能被处理；  
- 正则表达式中，未知部分用圆括号分组，然后可以用req.params[0]、[1]得到，req.params是一个类数组对象。

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

### 1.3 参数获取

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

### 1.4 错误处理与端口设置
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