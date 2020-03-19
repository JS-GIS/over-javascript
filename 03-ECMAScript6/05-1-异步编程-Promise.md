## Promise/A

### 1.1 Promise/A方案介绍

Promise/A方案包括2部分：Promisee和Deferred。  
- Promise操作只会处在3种状态的一种：未完成状态、完成状态、失败状态
- Promise的状态必须遵从这样的转换方向：未完成态-->完成态/失败态
- Promise的状态一旦转化，将不能被更改

在API上，Promise/A方案要求一个Promise对象只具备then()防范即可，对then()方法的要求如下：
- then()方法只接受function对象，其余对象会被忽略
- then()方法必须继续返回Promise对象，以实现链式调用
- then()必须接收完成态、错误态的回调方法，以让操作在完成或者出错时，调用对应方法
- 可选支持progeress事件回调作为第三个方法

示例：
```js
then(successHandler, errorHandler, progressHandler)
```

### 1.2 使用Node.js实现Promise对象

利用NodeJS来实现Promise对象：
```js
// 继承Node的events模块，创建一个Promise对象
function Promise() {
    EventEmitter.call(this);
};
util.inherits(Promise, EventEmitter);

// 添加圆形方法
Promise.prototype.then = function (fulfilledHandler, errorHandler, progressHandler) {

    if (typeof fulfilledHandler === 'function') {
        // 利用once()方法，保证成功回调只执行一次
        this.once('success', fulfilledHandler);
    }

    if (typeof errorHandler === 'function') {
        // 利用once()方法，保证异常回调只执行一次
        this.once('error', errorHandler);
    }

    if (typeof progressHandler === 'function') {
        this.on('progress', progressHandler);
    }

    return this;
};
```

这里then()方法所做的事情是将回调函数存放了起来，还需要触发执行这些回调函数的地方，实现这些功能的对象通常被称为Deferred，即延迟对象：
```js
function Deferred() {
    this.state = 'unSuccessed';
    this.promise = new Promise();
}

// 完成态
Deferred.prototype.resolve = function(obj) {
    this.state = 'successed';
    this.promise.emit('success', obj);
}

// 失败态
Deferred.prototype.reject = function(err) {
    this.state = 'failed';
    this.promise.emit('error', err);
}

Deferred.prototype.progress = function(data) {
    this.promise.emit('progress', data);
}
```

利用上述对象，现在可以对下面典型的响应对象进行封装：
```js
res.on('data', function(chunk) {
    console.log('Body:' + chunk);
});

res.on('end', function() {
   console.log('done!');
});

res.on('error', function(err) {
   console.log('error:' + err);
});
```

利用自制的Promise，Deffered对象进行封装：
```js
function promisify(res){

    var deferred = new Deferred();
    var result = '';

    res.on('data', function(chunk) {
        result += chunk;
        deferred.progress(chunk);
    });

    res.on('end', function() {
        deferred.resolve(result);
    });

    res.on('error', function(err) {
        deferred.reject(err);
    });

    return deferred.promise
}
```


封装后用户使用的代码：
```js
res.then(
    function(){
        console.log('done!');
    },
    function(err){
        console.log('error:' + err);
    },
    function(chunk){
         console.log('Body:' + chunk);
    }
)
```

### 1.3 Promise对象实现多异步协作

Promise解决了单个异步操作问题，如果现在需要多个异步操作，如何处理？  

多异步协作简单实现原型
```js
Deferred.prototype.all = function(promise){

    var count = promises.length;
    var that = this;
    var results = [];

    promises.forEach(function (promise, i) {

        promise.then(
            function (data) {
                count--;
                results[i] = data;
                if (count === 0) {
                    that.resolve(results);
                }
            }, 
            function (err) {
                that.reject(err);
            });
        });

    return this.promise;
}
```

以下示例为多次文件读取场景，all方法将两个单独的Promise重新抽象组合成一个新的Promise：
```js
var promise1 = readFile("foo.txt", "utf-8");
var promise2 = readFile("bar.txt", "utf-8");
var deferred = new Deferred();
deferred.all([promise1, promise2]).then(
    function (results) {
    // TODO
    }, 
    function (err) {
    // TODO
    }
);
```

### 1.4 Promise序列执行队列

原生的回调函数：
```js
obj.api1(function(data1){
    obj.api2(data1, function(data2){
        obj.api3(data2, function(data3){
            obj.api4(data3, function(data4){
                callback(data4);
            });
        });
    });
}){

}
```

普通解耦：
```js
var fn1 = function(data1) {
    obj.api2(data1, fn2);
}

var fn2 = function(data2) {
    obj.api3(data2, fn3)
}

var fn3 = function(data3){
    obj.api4(data3, fn4)
}

var fn4 = function(data4){
    callback(data4)
}

obj.api1(fn1)
```

使用事件解耦：
```js
var emitter = new event.Emitter();

emitter.on('step1', function(){
    obj.api1(function(data1){
        emitter.emit('step2', data1);
    });
});

emitter.on('step2', function(data1){
    obj.api2(data1, function(data2){
        emitter.emit('step3', data2);
    });
});

emitter.on('step3', function(data2){
    obj.api3(data2, function(data3){
        emitter.emit('step4', data3);
    });
});

emitter.on('step4', function(data3){
    obj.api1(data3, function(data4){
        callback(data4);
    });
});

emitter.emit("step1");
```

从上述看出，使用事件的方式，需要预先设定执行流程，这是由发布订阅模式的机制决定的。但是使用事件解耦明显增加了代码量。  

理想的代码应该是链式调用的：
```js
promise()
    .then(obj.api1)
    .then(obj.api2)
    .then(obj.api3)
    .then(obj.api4)
    .then(function (value4) {
        // Do something with value4
    }, function (error) {
        // Handle any error from step1 through step4
    })
    .done();
```

改造Deferred代码实现链式调用：
```js
function Deferred () {
    this.promise = new Promise();
};

// 完成态
Deferred.prototype.resolve = function (obj) {

    var promise = this.promise;
    var handler;

    while ((handler = promise.queue.shift())) {
        if (handler && handler.fulfilled) {
            var ret = handler.fulfilled(obj);
            if (ret && ret.isPromise) {
                ret.queue = promise.queue;
                this.promise = ret;
                return;
            }
        }
    }
};

// 失败态
Deferred.prototype.reject = function (err) {
    var promise = this.promise;
    var handler;
    while ((handler = promise.queue.shift())) {
        if (handler && handler.error) {
            var ret = handler.error(err);
            if (ret && ret.isPromise) {
                ret.queue = promise.queue;
                this.promise = ret;
                return;
            }
        }
    }
};

// 生成回调函数
Deferred.prototype.callback = function () {
    var that = this;
    return function (err, file) {
        if (err) {
            return that.reject(err);
        }
        that.resolve(file);
    };
};

// Promise
var Promise = function () {
    // 队列用ᇀ存ځئ执行的回调函数
    this.queue = [];
    this.isPromise = true;
};


Promise.prototype.then = function (fulfilledHandler, errorHandler, progressHandler) {
    var handler = {};
    if (typeof fulfilledHandler === 'function') {
        handler.fulfilled = fulfilledHandler;
    }
    if (typeof errorHandler === 'function') {
        handler.error = errorHandler;
    }
    this.queue.push(handler);
    return this;
};
```

这时候读取文件的例子中，假设读取第二个依赖第一个：
```js
var readFile1 = function (file, encoding) {
    var deferred = new Deferred();
    fs.readFile(file, encoding, deferred.callback());
    return deferred.promise;
};

var readFile2 = function (file, encoding) {
    var deferred = new Deferred();
    fs.readFile(file, encoding, deferred.callback());
    return deferred.promise;
};

readFile1('file1.txt', 'utf8').then(function (file1) {
    return readFile2(file1.trim(), 'utf8');
}).then(function (file2) {
    console.log(file2);
});
```

由上可见，Promise如果要支持链式调用，主要基于：
- 回调都存储在队列中
- Promise完成时，逐个执行回调，一旦检测到返回了新的Promise对象，停止执行，然后将当前Deferred对象的promise引用改变Wie新的Promise对象，并将队列中余下的回调转交给它


## 二 尾触发

除了事件和Promise外，有些方法需要手动调用才能持续执行后续方法，该类方法称为尾触发，如Express中的基础：
```js
app.use(connect.staticCache());
app.use(connect.static(__dirname + '/public'));
app.use(connect.cookieParser());
app.use(connect.session());
app.use(connect.query());
app.use(connect.bodyParser());
app.use(connect.csrf());
app.listen(3001);
```

在通过use()方法注册号一系列中间件后，监听端口上的请求，中间件利用了尾触发机制：
```js
function (req, res, next) {
// 中间件
}
```

每个中间件传递请求对象、响应对象、尾触发函数，通过队列形成一个处理流：
![](./temp-01.png)  

connect 的核心实现如下：
```js
function createServer() {

    function app(req, res){ 
        app.handle(req, res); 
    }

    utils.merge(app, proto);
    utils.merge(app, EventEmitter.prototype);
    app.route = '/';

    app.stack = [];
    for (var i = 0; i < arguments.length; ++i) {
        app.use(arguments[i]);
    }

    return app;
};
```

上述代码的核心关键是stack，是内部维护的中间件队列，通过调用use()方法将中间件放进队列中：
```js
app.use = function(route, fn){
    // some code
    this.stack.push({ route: route, handle: fn });
    return this;
};
```

此时处理模型建立好了，实现监听函数即可：
```js
app.listen = function(){
    var server = http.createServer(this);
    return server.listen.apply(server, arguments);
};
```

最终回到app.handle()方法，每一个监听到的网络请求都从这里开始：
```js
app.handle = function(req, res, out) {
    // some code
    next();
};
```

next() 的原理：
```js
function next(err) {
    // some code
    // next callback
    layer = stack[index++];
    layer.handle(req, res, next);
}
```

值得注意的是：中间件尾触发模式并没有要求中间件方法是异步的，即使每个步骤都是异步，其实也是串行化的处理，没办法通过并行的异步调用来提升业务处理效率。并行逻辑处理仍然需要搭配事件或Promise来完成。  

