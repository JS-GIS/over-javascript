## 一 Node的进程管理

Node由于单线程运行的原因，潜在的错误会导致线程崩溃，进程也随之退出。同时，单进程也无法充分利用当前多核CPU。  

Node从0.1版本开始提供了child_process模块，提供多进程支持，并暴露了全局对象process。

## 二  child_process模块

### 2.1 创建进程的方法

child_process模块提供了四种创建子进程的方法：
- spawn
- exec
- fork
- execFile

注意：后三个API基本都是借助于spawn方式实现的。  

### 2.2 spawn 创建进程

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

### 2.3 fork 创建进程

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

### 2.4 exec和execFile

在一个庞大的系统中，不同的模块可能使用不同的技术开发，比如web服务使用Node开发，消息队列使用Java开发，二者之间通信通常使用进程通信方式实现，但是有时候开发者认为该方式过于复杂，采用折中的方式，例如通过shell调用目标服务，拿到结果。  

spawn、exec、execFile都可以在shell中执行命令，运行文件：
- spawn：spawn会调用系统的shell工具，使用流式处理方式，子进程产生数据时，主进程可以通过监听事件获取消息
- exec：exec会根据当前环境初始化一个shell工具，将所有返回的信息放在stdout里一次性返回，如果返回的数据大小超过了exec方法参数maxBuffer，报错
- execFile：execFile是exec的内部实现方式，更底层，无须启动一个shell，效率更高


## 三 全局对象process

process是一个全局对象。
```js
process.cwd();		            # 获取程序当前目录，即完整路径
process.chdir(‘/usr/local’’);	# 改变程序当前目录，注意：参数必须是完整路径
process.pid;			        # 获取程序进程
process.title;			        # 获取进程名称
process.version		            # 获取node版本号
process.versions		        # 获取版本属性
process.config			        # 查看node配置
process.execPath		        # 获取当前进程可执行文件绝对路径
process.argv			        # 获取当前进程命令行参数数组
process.platform		        # 获取系统信息
process.arch			        # 获取CPU架构
process.env			            # 获取当前shell环境变量参数
```

标准输出流：node的console其实是通过Process模块的stdout.write()方法来实现的。
```js
process.stdout.write(‘hello world’);  // console.log与stdout.write()的区别是多了换行符
```

标准错误流：
```js
process.stderr.write(‘err’);
```

标准输入流：
```js
process.stdin.setEncoding('utf8');              //控制台接受输入
process.stdin.on('readable', function () {
    let chunk = process.stdin.read();
    if(chunk != null){
        process.stdout.write(chunk);
    }
});

process.stdin.on('end',function () {            //控制台结案数输入
    process.stdout.write('end');
});
```

杀死进程：
```js
process.on('SIGHUP',function () {
    console.log('get SIGHUP');
});
setTimeout(function () {
    process.exit(0);    //真正的杀死进程
},1000);
process.kill(process.pid,'SIGHUP'); //kill方法只是发送一个sighup信号
```

异步方法：process.nextTick(); 该方法比setTimeout效率高很多。
```js
console.time('timeout---');
setTimeout(function () {
    console.log('test timeout');
},0);
console.timeEnd('timeout---');

console.time('timeout---');
process.nextTick(function () {
    console.log('test nextTick');
});
console.timeEnd('timeout---');
```

