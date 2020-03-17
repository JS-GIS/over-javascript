## 一 RESTful概念

REST中文含义是表现层状态转换，符合REST规范的设计，都可以称呼为RESTful。RESTful的思想是将服务端提供和的内容实体看做资源，与URL进行一一对应，依据请求方式不同分别有增、删、改、查等方法。  

比如一个用户地址：`/user/lisi`，代表了 lisi 这个用户在服务器上的个人信息相关的资源，对该资源进行操作，则请求为：
```
增加用户lisi    POST        /user/lisi
删除用户lisi    DELETE      /user/lisi
修改用户lisi    PUT         /user/lisi
查询用户lisi    GET         /user/lisi
```

而在过去，新增一个用户，往往其接口为：`/user/saveUser`。  

在RESTful设计中，资源的具体格式由请求报头中的Accept字段和服务端的支持情况决定，比如如果客户端同时接收JSON和XML格式的响应，则其Accept字段的值如下：
```
Accept: application/json,application/xml
```

## 二 设计Node的RESTful路由器

在上一节中，如果use方法是对所有请求方法的处理，那么在RESTful中，需要区分请求方法进行设计：
```js
var routes = {
    'all': []
}

var app = {};

app.use = function(path, action){
    routes.all.push(pathRegexp(path), action);
}

var funcArr = ['get', 'put', 'delete', 'post'];
funcArr.forEach(function(method){
    routes[method] = [];
    app[method] = function(path, action){
        routes[method].push({
            regexp: pathRegexp(path),
            action: action
        });
    }
});
```

上面的代码添加了get()，put()，delete()，post()四个方法后，现在开始完成路由映射，并能达到类似下列的请求方式：
```js
app.post('/user/:username', addUser)
app.get('/user/:username', infoUser)
// ...
```

这样的路由能够识别请求方法，并将业务进行分发，为了让分发更简洁，必须将匹配的部分抽取为match()方法：
```js
function match(pathname, routes){

    for(var i = 0; i < routes.length; i++){

        // 正则匹配
        var matched = red.exec(pathname);
        if(!matched){
            return false;
        }

        var reg = routes[i].regexp;
        var keys = route[i].keys;

        var params = {};
        for(var j = 0; j < kyes.length; j++){
            var value = matched[j + 1];
            if (value){
                params[keys[i]] = value;
            }
        }

        req.params = params;
        var action = routes[i].action
        action(req, res);
        return true;
                
    }

}
```

分发部分的改进：
```js
function(req, res){

    var pathname = url.parse(req.url).pathname;
    var method = req.method.toLowerCase();

    if(routes.hadOwnPerperty(method)){

        // 根据请求方法分发
        if(match(pathname, routes[method])) {
            return
        } else {
            // 如果路径没有匹配成功，尝试让all()来处理
            if (match(pathname, routes.all)){
                return
            }
        }

    } else {
            if (match(pathname, routes.all)){
                return
            }
    }

    // 404处理
}
```