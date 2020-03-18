## 一 cluster模块概述

child_process模块虽然提供了大多数多进程架构所需的功能，但是构建一个健壮的单机集群，仍然需要很多细节处理，如考虑上一节中的自动重启、负载均衡等。  

cluster模块在Node v0.8版本推出，解决了child_process的问题，既能解决多核CPU利用率问题，也提供了完善的API，解决程序健壮性问题。  

使用cluster实现一个Node进程集群：
```js
var cluster = require('cluster');
var http = require('http');
var os = require('os');

var numCPUs = os.cpus().length;

if (cluster.isMaster) {

    for (var i = 0; i < numCPUs; i++) {
        cluster.fork();
    }

    cluster.on('exit', function(worker, code, signal) {
        console.log('worker ' + worker.process.pid + ' died');
    });

} else {

    http.createServer(function(req, res) {
        res.writeHead(200);
        res.end("hello world\n");
    }).listen(8000);
}
```

在进程中判断是主进程还是工作进程，取决于环境变量中是否有`NODE_UNIQUE_ID`：
```js
cluster.isWorker = ('NODE_UNIQUE_ID' in process.env);
cluster.isMaster = (cluster.isWorker === false);
```

## 二 cluster事件

cluster为健壮性提供了很多事件：
- fork：复制一个工作进城后触发该事件
- online：复制一个工作进程后，工作进程主动发送一条online消息给主进程，主进程收到消息后，触发该事件
- listening：工作进程中调用listen()，发送一条listening消息给主进程，主进程收到消息后，触发该事件
- disconnect：主进程和工作进程之间IPC通道断开后回触发该事件
- exit：有工作进程退出时触发该事件
- setup：cluster.setupMaster()执行后触发该事件

## 三 cluster工作原理

cluster模块本质上就是child_process和net模块的组合，cluster在启动时，会在内部启动TCP服务器，cluster.fork()时，将这个TCP服务器端的socket文件描述符发送给工作进程。如果进程是通过cluster.fork()出来的，那么其环境变量存在NODE_UNIQUE_ID，如工作进程中存在listen()真挺网络端口的地爱用，它将拿到该文件描述符，并通过SO_REQUSEADDR端口重用，从而实现多个子进程共享端口。  

使用cluster方式没有child_process灵活，因为自己通过child_process操作子进程时，可以隐式创建多个TCP服务器,使得子进程可以共享多个服务端的socket。  

