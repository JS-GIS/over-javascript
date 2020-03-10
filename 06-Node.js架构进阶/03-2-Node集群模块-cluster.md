## 四 Cluster

child_process的重要使用场景是创建多进程服务来保证服务稳定运行。Node为了统一创建多进程的方式，在0.6版本后增加了Cluster模块，内部封装了child_process。  

Cluster模块的显著优点是可以共享同一个socket连接，这代表可以使用Cluster模块实现负载均衡。  

```js
const cluster = require("cluster");
const http = require("http");
const numCPUs = require("os").cpus().length;

console.log("numCPUs = ", numCPUs);

if (cluster.isMaster) {

    console.log("Master process id is ", process.pid);

    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }

    cluster.on('listening',function(worker,address){
        console.log('listening: worker ' + worker.process.pid +', Address: '+address.address+":"+address.port);
    });

    cluster.on("exit", function(worker, code, singal){
        console.log("worker process died, id: ", worker.process.pid);
    });

} else {

    //Worker可以共享同一个TCP连接，这里的例子是一个http服务器
    http.createServer(function(req, res){
        res.writeHead(200);
        res.end("hello world\n");
    }).listen(3000);

    console.log("Worker started, process id: ", process.pid);
}
```

在上述方式中，cluster的fork与process的fork其实是同一个方法。  

Cluster模块采用的是经典的主从模型，由master进程来管理所有的子进程，master进程只负责调度和管理，worker进程负责处理具体的任务。





## 五 Node使用多核CPU

Node采用了单线程模型，但是并不能说明Node只能运行在单核CPU下。Node原生已经支持集群特征。
制作一个简单HelloWorld示例，我们将应用程序部署在单台服务器的多核CPU上，从而搭建集群环境。也就是说：每个CPU内核都会运行一个Node进程，这样的集群只能在单台服务器上运行。
```JavaScript
const http = require('http');
const cluster = require('cluster');
const os = require('os');

let PORT = 8000;
let CPUS = os.cpus().length;     //获取cpu内核书

if(cluster.isMaster){   //当前进程为主进程
    for(let i = 0; i < CPUS; i++){
        cluster.fork();
    }
} else {    //当前进程为子进程:只有在子进程才执行相关代码
    let app = http.createServer(function (req,res) {
        res.writeHead(200,{'Content-Type':'text/plain'});
        res.end('hello world');
    }).listen(PORT,function () {
        console.log('server is running at' + PORT);
    });
}
```
实际上，程序内部是通过主进程去调度各个子进程的。即：cpu核心数越多，单台服务器上可创建的子进程越多，支持的并发越大，从而整个应用程序的吞吐率就越高。

## 六 Node执行环境设置

Express等框架中，可以使用app.set(‘env’,’production’);指定执行环境，但是不建议，因为应用程序会一直运行在该环境中，推荐使用NODE_ENV指定运行环境。调用app.get(‘env’); 让它报告运行在哪个模式下：
```js
const http = require('http');
const express = require('express');
let app = express();
http.createServer().listen(app.get('port'),function () {
    console.log('Express started on ' + app.get('env'));        //直接启动会是在 development
});
```
在生产环境执行：export NODE_ENV=production  
如果是Unix系系统：NODE_ENV=production node app.js  
注意：Express在生产环境中默认会启动视图缓存。
Node启动程序的模块化：
```js
const http = require('http');
const express = require('express');

function startServer() {
    let app = express();
    http.createServer().listen(app.get('port'),function () {
        console.log('Express started on ' + app.get('env'));        //直接启动会是在 development
    });
}

if (require.main === module) {
    startServer();  //应用程序直接运行
} else {            //作为一个模块 导出
    module.exports = startServer;
}
```
这样做的好处是，这个文件既可以直接启动，也可以作为模块导出。直接运行时，module.main === module 是true，如果是false，证明是另外一个脚本require进来的。
此时创建一个新脚本：
```js


const cluster = require('cluster');

function startWoker() {
    let worker = cluster.fork();
    console.log('cluster worker %d started',worker.id);
}

if (cluster.isMaster) {

    require('os').cpus().forEach(function () {
        startWoker();
    });

    //记录所有断开的工作线程，断开应该退出
    cluster.on('disconnect', function (worker) {
        console.log('cluster worker %d disconnect',worker.id)
    });

    //当有工作线程死掉，则创建一个线程替代它
    cluster.on('exit', function (worker,code,signal) {
        console.log('cluster worker %d died',worker.id);
        startWoker();
    });
} else {        //在工作线程上启动服务器
    require('./app.js')();
}
```

该JS文件在执行时，要么在主线程的上下文中（node app.js直接运行），要么在工作线程的上下文中（被node集群系统执行），属性cluster.isMaster和cluster.isWoker决定了在哪个上下文中。运行脚本时，是在主线程下执行的，并使用cluster.for为系统的每个cpu启动了一个工作线程，在else语句中处理工作线程。
此时可以在中间件中查看当前线程：
```js
let cluster = require(‘cluster’)
if (cluster.isWorker) {
require(‘cluster’).worker.id;
next();
}
```