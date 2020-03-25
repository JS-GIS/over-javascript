## 一 函数式编程

在JavaScript中，可以将函数作为参数、返回值来进行传递，这样的做法称为高阶函数。如下所示：
```js
function foo(x) {
    reuturn funtion() {
        return x;
    }
}
```

高阶函数可以形成一种后续传递风格（Continuation Passing Style）的结果的接收方式，而非单一的返回值形式，大量的业务代码可以这样依次传递下去：
```js
function foo(x, fn1) {
    x++;
    return fn1(x);
}
```
JS中通过高阶函数机制实现了很多方法，如：数组的forEach、reduce()等。高阶函数也在Node中的异步编程中被广泛使用，如事件的回调函数。  

## 二 回调函数编程难点

###  2.1 回调地狱

异步编程最大的痛点也来自于回调函数，很容易形成回调地狱：
```js
fs.readdir(source, function (err, files) {
  if (err) {
    console.log('Error finding files: ' + err)
  } else {
    files.forEach(function (filename, fileIndex) {
      console.log(filename)
      gm(source + filename).size(function (err, values) {
        if (err) {
          console.log('Error identifying file size: ' + err)
        } else {
          console.log(filename + ' : ' + values)
          aspect = (values.width / values.height)
          widths.forEach(function (width, widthIndex) {
            height = Math.round(width / aspect)
            console.log('resizing ' + filename + 'to ' + height + 'x' + height)
            this.resize(width, height).write(dest + 'w' + width + '_' + filename, function(err) {
              if (err) console.log('Error writing file: ' + err)
            })
          }.bind(this))
        }
      })
    })
  }
})
```

回调地狱的解决方案见下一节。

###  2.2 错误处理

回调函数在异常处理上也会产生问题：
```js
try {
    // 执行业务
} catch(e) {
    // 捕获异常后处理
}
```
上述代码在异步编程中并不适用，Node的异步I/O分为两个阶段：提交请求和处理结果，两个阶段之间通过事件循环进行调度，彼此互不关联。异步方法通常在第一阶段提交请求后就会立即返回，异常通常不会在该阶段产生，try/catch则无法捕获异常了，如下所示：
```js
// 定义一个异步函数
function asyncFN(callbck){
    process.nextTick(callback);
}

// 执行该异步函数
try {
    asyncFN(callback);
} catch (e) {

}
```

这里只能捕获到asyncFN的异常，却无法捕获callback的异常，因为callback会被存放起来，直到下一个事件循环到达才会取出来执行。  

Node的解决方案是将异常作为回调函数的第一个参数传回:
```js
asyncFN(function(err, results){

})
```

同时也要注意捕获asyncFN本身的异常不能这样处理：
```js
try {
    // 业务处理
    callback()
} catch (e) {
    // 错误处理
    callback(e)
}
```
这时候，如果回调函数和原函数都产生异常，则回调函数会执行2次，正确写法如下：
```js
try {
    // 业务处理
    callback()
} catch (e) {
    // 错误处理
    return callback(e);
}
callback();
```

###  2.3 代码阻塞

Node没有sleep()线程睡眠函数，也导致了阻塞代码执行变得困难，如果使用下面方式，将会引起灾难：
```js
// 模拟 sleep(1000)
var start = new Date();
while(new Date() - start < 1000) {
    // TODO
}
```
上述代码完全破坏了Node的事件循环调度，会持续占用CPU进行判断，导致整个应用阻塞，与线程睡眠差距极大。  