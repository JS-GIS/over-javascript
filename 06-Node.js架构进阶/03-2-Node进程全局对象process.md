## 一 全局对象process

process是Node默认提供的一个全局对象，常见方法有：
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

## 二 标准流

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

## 三 信号

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

## 四 nextTick

异步方法：`process.nextTick();` 该方法比setTimeout效率高很多。
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