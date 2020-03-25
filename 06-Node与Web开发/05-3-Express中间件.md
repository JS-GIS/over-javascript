## 一 中间件
中间件是在管道中执行的，在Express中，使用app.use()向管道中插入中间件。  

中间件讲究顺序，匹配上第一个之后，就不会往后匹配了，next函数才能够继续往后匹配。  

模糊意义上讲，app.get() app.post()等方法也属于中间件。  

app.use()也是一个中间件。与get、post不同的是，他的网址不是精确匹配的，使用user(path,fn())时候，user内的路由，将匹配/path /path/images/  /path/images/1.png 等路由情况。  

如果写一个 / 实际上就相当于"/"，就是所有网址，也可以直接不写该地址。  

大坑：express的中间件函数，不需要传入req，res，他是在中间件函数执行回调的时候自动传入。创建中间件：
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