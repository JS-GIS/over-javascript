## 一 Node的进程管理

Node由于单线程运行的原因，潜在的错误会导致线程崩溃，进程也随之退出。同时，单进程也无法充分利用当前多核CPU。  

Node从0.1版本开始提供了child_process模块，提供多进程支持，并暴露了全局对象process。  


新建一个文件：worker.js
```js
var http = require('http');

http.createServer(function(req, res){
    res.writeHead(200, {'Content-Type': 'text/plian'});
    res.end('hello world\n');
}).listen();
```

新建一个文件：master.js
```js
var child_process = require('child_process');
var os = require('os');

var cpuNum = os.cpus().length;
console.log("当前CPU数量为：", cpuNum);

for(var i = 0; i < cpuNum; i++){
    child_process.fork('./worker.js');
}
```
启动并查看node进程数量(和cpu数量相等)：
```
node master.js

# 新开启一个命令行查看进程数
ps aux|grep worker.js
```

这时候Node就以多进程的方式启动了，这是著名的主从模式（Master-Worker），整个系统进程为了两部分：
- 主进程：不负责具体业务，而是负责调度和管理工作进程
- 工作进程：负责具体的业务处理

![](../images/node/process-01.svg)  

通过fork()复制的进程都是一个独立的进程，每个进程中都包含着一个独立、全新的V8实例，需要至少30毫秒启动时间，至少10MB的内存。   

fork()是非常消耗资源的，但是Node本身的单线程非阻塞I/O已经解决了大并发问题，fork()的作用仅仅是为了充分利用CPU，而不是像以前的高并发架构中描述的使用多进程应对并发。


## 二  child_process模块

### 2.0 创建进程的方法

child_process模块提供了四种创建子进程的方法：
- spawn()：启动一个子进程来执行命令
- exec()：与spawn()类似，但是该函数额外有一个回调函数可以获知子进程状况
- fork()：与spawn()类似，但是创建的紫禁城只需要执行指定的JS文件模块即可
- execFile()：启动一个子进程来执行可执行文件

四者在执行第一节中worker的代码分别如下：
```js
var cp = require('child_process');

cp.spwan('node', ['worker.js']);
cp.exec('node worker.js', function(err, stdout, stderr){})
cp.execFile('worker.js', function(err, stdout, stderr){})
cp.fork('./worker.js')
```

注意：
- exec，execFile、fork基本都是借助于spawn方式实现的。  
- exec()，execFile()创建子进程时，可以指定子进程的运行时间（timeout）。
- execFile()执行JS文件时，其首行内容必须添加：`#!/usr/bin/env node` 


### 2.1 spawn 创建进程

spawn会使用指定的command生成一个新的进程，执行完对应的command后子进程自动退出。  

示例：
```js
let spawn = require("child_process").spawn;

let ls = spawn("ls", ['-lh', '/usr']);      // win需要修改为： ("powershell",["dir"])

ls.stdout.on("data", function(data){
    console.log("stdout:", data.toString());
});

ls.stderr.on("data", function(err){
    console.log("stderr:", err.toString());
});

ls.on("close", function(code){
    console.log("exited with code:", code);
});
```

### 2.2 fork 创建进程

在Linux环境中，创建一个新进程的本质是复制一个当前继承，用户调用fork后，操作系统会为该进程分配新的空间，然后将父进程的数据原样复制一份过去（除了少数值不复制，如进程标识符PID）。  

在Node环境中，fork并不会复制父进程，父子进程都有独立的内存空间和V8实例，父子进程之间唯一联系是用来进程间通信的IPC Channel。  

fork接收的第一个参数是文件名，相当于在命令行调用了`node **.js`，父子进程之间通过process.send通信。  

fork内部会通过spawn调用process.executePath，即node可执行文件地址，生成一个node实例，然后用这个实例来执行for方法的modulePath参数。  

master.js:
```js
let child_process = require("child_process");

let worker = child_process.fork("worker.js", ["args1"]);

// 父进程向子进程发送信息
worker.send(
    {
        hello: "child"
    }
);

worker.on("message", function(msg){
    console.log("父进程接收的消息：", msg);
});

worker.on("exit", function(){
    console.log("child process exit");
});
```

worker.js:
```js
let begin = process.argv[2];
console.log("子进程启动:", begin);

// 子进程向父进程发送信息
process.send(
    {
        hello: "parent"
    }
);

process.on("message", function(msg){
    console.log("子进程接收的消息：",msg);
    process.exit();
});
```

注意：
- fork创建的进程运行完成后不会自动退出，需要process.exit()
- 这些派生的node进程是全新的V8实例，假设每个node进程大致需要至少30毫秒启动时间和10M内存，也就是说不能创建成百上千的实例。
- node本身存在多个线程，但是运行在V8上的JS是单线程的。 
- node中的exit和close事件虽然都是退出、结束的意思，但是exit事件后，标准输入输出流仍然在开启，而close事件后是在一个子进程中所有的标准输入、输出流被终止时触发，因为多进程有时候会共享一个标准输入、输出流。
- 在Node中，正常退出时，退出码定义为0，非正常为1-3之间的数字。

### 2.3 exec和execFile

在一个庞大的系统中，不同的模块可能使用不同的技术开发，比如web服务使用Node开发，消息队列使用Java开发，二者之间通信通常使用进程通信方式实现，但是有时候开发者认为该方式过于复杂，采用折中的方式，例如通过shell调用目标服务，拿到结果。  

spawn、exec、execFile都可以在shell中执行命令，运行文件：
- spawn：spawn会调用系统的shell工具，使用流式处理方式，子进程产生数据时，主进程可以通过监听事件获取消息
- exec：exec会根据当前环境初始化一个shell工具，将所有返回的信息放在stdout里一次性返回，如果返回的数据大小超过了exec方法参数maxBuffer，报错
- execFile：execFile是exec的内部实现方式，更底层，无须启动一个shell，效率更高