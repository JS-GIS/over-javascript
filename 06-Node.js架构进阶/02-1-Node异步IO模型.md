## 一 Node的IO模型

### 1.0 前情提要

通过前面的介绍，在多线程开发中，在创建线程、执行线程上下文切换时会有巨大开销，且多线程编程需要应对锁、同步等问题，开发体验很差。  

Node采用单线程避免了多线程上述缺点，同时单线程也不意味着性能的降低，利用异步I/O模型，单线程也能实现较大并发，缺点是无法利用多核CPU。  

Node中的异步支持库为libuv，其在Linux中使用Node作者的libuv库实现，在windows中则采用win平台的IOCP。  

注意：虽然强调Node是单线程的，只是指JS执行在单线程中，其内部完成I/O任务则是依赖一个线程池。  

### 1.1 事件循环

在Node项目进程启动时，会创建一个类似于 `while(true)` 的死循环，每执行一次循环体的过程可以称呼为 Tick。  

每轮Tick中，都在查看是否有事件在等待处理，如果有，则取出事件，并执行器关联的回调函数，没有关联的回调函数则不执行。  

查看是否有事件等待处理利用了观察者模式：每个事件循环中有一到多个观察者，只需要向观察者循环是否有要处理的事件即可。   

事件循环是一个典型的生产者、消费者模型，异步I/O、网络请求等是事件的生产者，产生的事件被传递到对应的观察者，事件循环则从观察者取出事件并处理。  

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

### 1.2 回调的产生

在普通开发中，回调函数一般由开发者自己调用，如：
```js
var forEach = funtion(list, cb) {
    for (var i = 0; i < list.length; i++>) {
        cb(list[i], i, list);
    }
}
```

但是在Node中，回调函数并不是有开发者调用。JS发起调用后，一直到内核完成I/O的过程中，会产生一个名为 请求对象 的中间产物，示例：
```js
fs.open = function(path, flags, callback){
    // ...
    binding.open(pathModule._makeLong(path), stringToFlags(flags), mode, callback);
}
```
`fs.open()`的作用是根据指定路径和参数去打开文件得到其描述符，其调用步骤是：JS代码-->Node核心模块-->C++内建模块-->libuv进行系统调用。最终调用的是libuve内部的`_uv_fs_open()`方法，在此方法中，会创建一个FSReqWrap请求对象，所有JS代码中的参数、方法都会封装在该请求对象中，回调函数会被设置在对象的`oncomplete_sym`属性上：
```c
req_wrap->object_->Set(oncomplete_sym, callback);
```

对象包装完毕后，则会其推入线程池中等待执行。也就是说当前的I/O操作在线程池中开始等待，不管是否阻塞，都不会影响JS线程的后续执行，达到了异步目的。  

### 1.3 回调的执行

线程池中I/O操作调用完毕后，会将获取的结果存储在req->result属性上，然后通过一些方法通知完成，即取出请求对象的result属性作为参数，取出oncomplete_sym属性作为方法，然后调用执行，以此达到调用JS中传入的回调函数的目的。  

## 二 定时器

### 2.1 深入理解Node的事件监听

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

### 2.2 process.nextTick

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

### 2.3 setImmdiate和nextTick

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

### 2.4 setImmediate与setTimeout

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