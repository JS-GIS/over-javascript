## 一 Node的异步并发按控制

Node虽然很容易实现并发，但是也必须对并发做出限制：
```js
for (var i = 0; i < 10000; i++>) {
    fs.readFile()
}
```

上述代码瞬间对文件系统发起大量并发调用，操作系统的文件描述符数量会被瞬间用光：
```js
Error: EMFILE, too many open files
```
这是异步I/O和同步I/O的最大编程体验差距，同步I/O中的调用时一个接一个的，不会出现文件描述符耗尽的情况，但是性能低下，而异步I/O并发实现简单，但是需要提供一定的过载保护！  

一般可以通过队列来控制并发量：
- 如果当前活跃（调用发起但未执行回调）的异步调用量小于限定值，从队列中取出执行
- 如果活跃调用达到限定值，调用暂时存放到队列中
- 每个异步调用结束时，从队列中取出新的异步调用执行

参见实现：https://github.com/JacksonTian/bagpipe

async库也有解决方案：
```js
async.parallelLimit([
    function (callback) {
        fs.readFile('file1.txt', 'utf-8', callback);
    },
    function (callback) {
        fs.readFile('file2.txt', 'utf-8', callback);
    }
], 1, function (err, results) {
    // TODO
});
```

parallelLimit()方法与parallel()类似，但多了一个用于限制并发数量的参数，使得任务只能同时并发一定数量，而不是无限制并发。  

parallelLimit()方法的却显示是无法动态增加并行任务，async的queue()方法可以，比如用来遍历文件目录等操作十分有效：
```js
var q = async.queue(function (file, callback) {
    fs.readFile(file, 'utf-8', callback);
}, 2);

q.drain = function () {
    // 完成了对了队列中的所有任务
};

fs.readdirSync('.').forEach(function (file) {
    q.push(file, function (err, data) {
    // TODO
    });
});
```

queue()的却显示接收参数固定，丢失了parallelLimit()方法的灵活性。  