## 一 Node的IO模型

#### 1.1 回调

理解回调关系需要了解一个前提：
- 在任务完成之前，CPU在任何情况下都不会暂停或者停止执行，CPU如何执行和同步、异步、阻塞、非阻塞没有必然关系
- 操作系统始终保证CPU处于运行状态，这是通过系统调度来实现的，具体一点就是通过在不同进程/线程间切换实现

回调其实是将一个函数作为参数传递给了另外一个函数，并且作为参数的函数可以被执行，本质上是一个高阶函数。常见的案例是map：
```js
[1,2,3].map(function(val){
    console.log(val)                // 依次输出 1 2 3
})
```
![](/images/node/huidiao1.png)

回调方法和主线程处于同一层级，假设主线程发起了一个底层调用，那么操作系统转而去执行这个系统调用，当调用结束后，又回到主线程上调用其后的方法，这也是为什么会被称为回调。  

但是回调函数具体何时执行是没有要求的，可以是同步的，如map中的function，也可以是异步，如setTimeout中的function。

#### 1.2 异步过程中的回调

单线程运行的语言在设计时都会遇到一个问题：如果遇到一个耗时操作，例如IO，要不要等待操作完成后再执行下一步操作。  

有的语言选择了等待，比如PHP，而Node在发起一个调用后继续向下执行，耗时操作完成后，再执行对应的回调函数（异步），虽然代码运行在单线程环境中，但依靠异步+回调的方式，也能实现高并发支持。  

示例：
```js
let fs = require("fs");
let callback = function(err, data){}
fs.readFile("./foo.txt", callback);
console.log("后续步骤")
```
示例中，readFile发起了系统调用，没有等待调用结果，直接执行了console。

#### 1.3 同步与异步

同步和异步其实是线程/进程的调用方式:
- 同步：进程/线程发起调用后，一直等待调用返回后才继续执行下一步操作。注意：这并不代表CPU在这段时间内也会一直等待，操作系统多半会切换到另一个进程/线程上去，等待调用返回后再切换回原来的进程/线程。  
- 异步：与同步相反，发起调用后，进程/线程继续向下执行，当调用返回后，通过某种手段来通知调用者。

注意：同步和异步中的调用返回指内核进程将数据复制到调用进程(linux环境下)。

我们常说JS是一门天生支持异步的语言，但是ECMAScript中并没有异步规范，JS的异步更多的是依赖于浏览器的运行时环境内部的其他线程来实现的，并非JS本身的功能。

#### 1.4 阻塞与非阻塞

阻塞与非阻塞是针对IO状态而言的，关注程序在等待IO调用返回这段时间的状态。Node在官网的介绍中并没有提及异步的概念，只是使用了非阻塞。 

阻塞与否与异步与否完全是两组概念，它们之间没有任何必然联系。

#### 1.5 IO模型

IO调用的结果是如何返回给调用的进程/线程呢？  

通过内核复制给进程，在Linux中，用户进程没办法直接访问内核空间，通常是内核调用copy_to_user方法来传递数据，大致的流程是：IO的数据会先被内核空间读取，然后内核将数据复制给用户进程。（当然也有一种内存共享技术，内核进程和用户进程共享一块内存地址，这样可以避免内存复制）。  

IO通常分为数据准备和返回结果两个阶段，IO的编程模型即操作系统在这2个阶段中采取的处理方式，常见的IO模型有四种（都是Linux下的）：
- 阻塞IO(blocking IO):进程发出一个系统调用后，等待IO两个阶段完成，拿到结果再继续运行
- 非阻塞IO(nonblocking IO):进程发起一个系统调用后，如果数据没有准备就绪，就会马上返回一个结果告诉进程现在还内有就绪，此时用户进程会不断同步查询内核状态
- 事件驱动IO(IO multiplexing/Event Driven):也是以轮询的方式查询内核的执行状态，和非阻塞IO区别是一个进程可能会管理多个IO，当某个IO调用有了结果之后就会返回对应结果。
- 异步IO：当进程发出调用后，内核立即返回结果，进程会继续做其他的事情，直到操作系统返回数据，给用户发送一个信号，这里也可以看出异步没有任何回调的概念。

那么此时，Node官网为何没有标榜自己是异步IO，而是写成非阻塞IO呢？因为Node中的异步IO是通过libuv模拟出来的，而非阻塞是实打实的。

#### 1.6 IO原理

Node的异步调用，其背后机制是Linux的Epoll，在windows下，则采用的是完成端口IOCP。这两种方式的共同点是没有启动任何其他线程。
Linux下，Node启动时，会维护一个线程池，Node使用这个线程池的线程，同步读取文件，在windows下利用IOCP通知机制，把代码交给操作系统的线程池运行。

#### 1.7 Node中的单线程模型

使用node开发编写的代码运行在单线程环境中，但是node底层的实现并非单线程的，libuv会通过类似线程池的实现来模拟不同的操纵系统下的异步调用，对开发者来说，这些底层都是不可见的。  

libuv由Node作者开发，是一个跨平台的异步IO库，针对不同操作系统，底层使用了Linux的libev和win下的IOCP。Node中的非阻塞IO以及事件循环都是由libuv实现的。  

#### 1.8 并行与并发

以两队人排队取火车票为例：
- 并发：只有一个取票机，第一队排头取票后，第二队的排头取票，依次循环
- 并行：开放两个取票机，两队同时向前移动

并发和并行应对了两种需求：做更多的事（同时处理多个队列），更快的做事。所以并行程序其实是并发程序的一种设计体现。

## 二 事件循环

#### 2.1 事件循环的概念

简单的事件循环例子：用户点击了页面上行的按钮或者进行其他操作时，会产生相应的事件，这些事件被加入到一个队列中，然后主循环会逐个处理他们。  

每个事件都有自己对应的事件处理函数，如果处理函数是类似readFile这样的IO耗时操作，那么势必影响整个循环的效率，采用异步加回调的方式解决该问题。

#### 2.2 Node中的事件循环

Node将事件循环分成了6个阶段，每个阶段都维护了一个回调函数队列，在不同的阶段，事件循环处理不同类型的事件，事件循环的阶段依次是：
- Timers:用来处理setTimeout和setTimeInterval的回调
- IO callbacks:大多数的回调方法在这个阶段执行，除了timers、close、setImmediate事件的回调
- idle,prepare:仅仅Node内部使用
- Poll:轮询，不断检查有没有新的IO事件，事件循环可能会在这里阻塞
- Check:处理setImmediate事件回调
- close.callback:处理一些close相关的事件，如：socket.on("close",...)

上述的处理用伪代码展示：
```js
while (true) {
    uv_run_timers();
    uv_run_pending(loop);
    uv_run_idle();
    uv_io_poll();
    uv_run_check();
    uv_run_closeing_handles();
}
```
上述代码中，每个方法代表一个阶段，假设事件循环现在进入了某阶段（即开始执行删哪个面其中一个方法），即使在这期间有其他队列中的事件就绪，也会将当前阶段队列里的全部回调方法执行完毕后，再进入到下个阶段。

#### 2.3 深入理解Node的事件监听（重要！）

在Node中，事件队列不止一个，定时器相关的事件和磁盘IO产生的事件需要不同的处理方式。如果把所有的事件都放在一个队列里，势必要增加许多类似switch/case的代码，还不如直接归类到不同的事件队列，然后一层层遍历。  

示例：
```js
let fs = require("fs");

let startTime = Date.now();

//setTimeout 的异步
setTimeout(function() {
    let delay = Date.now() - startTime;
    console.log(delay + " 毫秒后才开始执行setTimeout回调");
}, 100);

//文件读取的异步：假设耗时95ms
fs.readFile("./foo.js", function(err, data){
    let beginCallbackTime = Date.now();
    while(Date.now() - beginCallbackTime < 10) {       // 使用while阻塞10ms
        console.log("阻塞中");
    }
});
```
上述代码中，存在readfile和timer两个异步操作，启动文件后，运行时开始轮询：
- 首先检查timers，timers对应的事件队列目前还为空，因为100ms后才会有事件产生
- 进入poll阶段，没有事件出现，代码中也没有setImmediate操作，事件循环便在此一直等待新的事件出现
- 直到95ms读取文件完毕，产生了一个事件，加入poll队列中，此时事件循环将该队列的事件去除，准备执行之后的callback，readFile的回调方法什么都没做，只是暂停了10ms。
- 事件循环本身也被阻塞10ms，按照通常的思维，95+10=105>100，timers队列的事件已经就绪，应该先执行对应的回调方法才对，但是由于事件循环本身是单线程运行，因为也会被停止10ms，如果readFile中出现了一个死循环，那么整个事件循环都会被阻塞，setTimeout回调永远不会执行。
- readFile的回调完成后，事件循环切换到了timers阶段，接着取出timers队列中的事件执行对应的回调方法

#### 2.4 process.nextTick

process.nextTick就是定义一个异步动作，并且让这个动作在事件循环当前阶段结束后执行：
```js
process.nextTick(function(){
    console.log("tick...");
});
console.log("console...");
```
打印结果：
```
console...
tick...
```

process.nextTick并不是事件循环的一部分，但是他的回调方法是由事件循环调用的，该方法定义的回调方法被加入到名为nextTickQueue的队列中。在事件循环的任何阶段，如果nextTikcQueue不为空，都会在当前阶段操作结束后优先执行nextTickQueue中的回调函数，执行完该队列中的回调后，事件循环才会继续向下执行。  

注意：多个nextTick在一起，则会按照顺序执行，且会依次阻塞后面的nextTick

#### 2.5 setImmdiate和nextTick

setImmdiate是Node独有的标准，不接受时间作为参数，他的事件总是在当前事件循环的结尾触发，对应的回调方法会在当前时间循环末尾（check）阶段执行，但是由于nextTick会在当前操作完成后立刻执行，因此总会在setImmdiate前执行。  

有一个笑话：nextTick和setImmdiate的行为和名称含义其实是反过来的
```js
setImmediate(function(args){
    console.log("executing immediate",  args);
},"so immediate");

process.nextTick(function(){
    console.log("tick...");
});
console.log("console...");
```
依次输出：
```
console...
tick...
executing immediate so immediate
```

注意：node限制了bextTickQueue队列的大小，如果使用很大的循环来产生该队列，则会抛出错误，而setImmediate不会出现该问题，因为它不会生成call stack。

下列代码不会报错：
```js
function recurse(i, end) {
    if (i > end) {
        console.log("Done!");
    } else {
        console.log(i);
        setImmediate(recurse, i+1, end);
    }
}

recurse(0, 9999999999);
```

下列代码报错：Maximum call stack size exceeded
```js
function recurse(i) {
    while(i < 9999) {
        process.nextTick(recurse(i++));
    }
}

recurse(0);
```

#### 2.6 setImmediate与setTimeout

setImmediate方法会在poll阶段结束后执行，而setImmediate会在规定时间到期后执行，由于无法预测执行代码时时间循环处于哪个阶段，因此当代码中同时存在这个两个方法时，回调函数的执行顺序不是固定的：
```js
setTimeout(function(){
    console.log("setTimeout");
},0);
setImmediate(function(){
    console.log("setImmediate");
});
```

但是如故将二者放在一个IO操作的callback中，则永远是setImmediate先执行：
```js
require("fs").readFile("./foo.js", function(){
    setTimeout(function(){
        console.log("setTimeout");
    },0);
    setImmediate(function(){
        console.log("setImmediate");
    });
});
```

## 三 Node的模块机制

#### 3.1 与JS有关的模块规范

- CommonJS：NodeJS采用的模块机制，即require，exports
- AMD：RequireJS采用的模块机制，即异步模块加载机制，依赖这个模块的代码定义在一个回调函数中，等待加载完成后，这个回调函数才会运行
- ES6Module：ES6的模块机制

#### 3.2 重复引用问题

Node无须关心重复引用问题，因为Node先从缓存中加载模块，一个模块被第一次加载后，就会在缓存中维持一个副本，如果遇到重复加载的模块会直接提取缓存中的副本，也就是说在任何情况下每个模块都只在缓存中有一个实例。

#### 3.3 require同步加载问题

require加载模块为什么是同步而非异步？
- CommonJS的标准中，require是同步加载（这没有多少意义，标准也是可以推翻的）
- Node服务器端，会缓存已经加载的模块，且访问的是本地文件，IO开销几乎可以忽略，所以无需像前端那样考虑异步加载，且服务端很少出现重启情况，也无须担心启动时的时间开销

#### 3.4 require缓存策略

require从缓存中加载文件是基于文件路径的，这表示即使有两个完全相同的文件，但她们位于不同的路径下，也会在缓存中维持两份。  

查看缓存代码：
```js
console.log(require.cache);
```

#### 3.5 require缓存隐患

当调用require加载一个模块时，模块内部的代码都会被调用！！！！，有时候这可能会带来隐藏的bug。   

module.js：
```js
function test() {
    setInterval(function(){
        console.log("test");
    },1000);
}
test();

module.exports = test;
```

main.js:
```js
let test = require("./module");
```

main.js只是加载了module文件，但是仍然每隔1秒输出了test字符串，且main.js的进程始终没有退出！！这在生产环境中极其造成内存泄漏。所以使用模块时要留意该情况。

## 四 Node异常处理机制

Node是单线程的，为了保证服务器不崩溃，必须保证项目的高容错性。  

例如：require一个文件时，如果文件不存在，那么会出现一些可能的突发情况，正确做法：
```JavaScript
http.createServer(function (req,res) {
    try{
        const test = require('test');
    } catch (e){
        console.log(e);
    }
    res.end('hello world');
}).listen(80);

```


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