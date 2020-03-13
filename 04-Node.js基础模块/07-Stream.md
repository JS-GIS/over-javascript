## 一 Stream流

###  1.1 Stream流简介

使用readFile方法是极其占用内存的（一次加载大文件），读取完毕后再发送，读和发没有同时进行，这样不但影响性能，也会让网络应用、磁盘应用之间永远只有一个在疯狂活动，应该读取一点，发送一点，这样网络、磁盘都在积极使用。  

使用流可以解决上述问题：
- 读取流：fs.createReadStream(req)
- 写入流：fs.createWriteStream(res)

在Node有四种基础的stream类型：
- Readable:可读流
- Writeable:可写流
- Duplex:读写流
- Transform:操作写入的数据，然后读取结果，通常用于输入数据和输出数据不要求匹配的场景，如zlib.createDeflate()

###  1.2 Stream流的使用

可读流示例：
```js
let fs = require("fs");

let readStream = fs.createReadStream("./test.txt", "utf-8");

readStream.on("data", function(data){
    console.log(data);                  // 输出test文本
});

readStream.on("close", function(){
    console.log("close");               // 接着输出close
});

readStream.on("error", function(error){
    console.log("error:", error);
});
```

可读流和可写流配合示例：
```JavaScript
const fs = require('fs');
let rs = fs.createReadStream('./1.jpg');        //读取流
let ws = fs.createWriteStream('./2.jpg');       //写入流
rs.pipe(ws);
```

在服务器上应用：
```js
const http = require('http');
const fs = require('fs');
http.createServer((req,res)=>{
    let rs = fs.createReadStream('./1.jpg');
    rs.pipe(res);
}).listen(8000);
```
注意：上述案例中，读取1.jpgg，写入到2.jpg，流是有方向的，从读取端流向写入端流的结束事件是finish不是end。其实req，res也是流，因为只有流才有on事件。

###  1.3 自定义Stream

当原生的stream无法满足要求时，可以自定义stream：
```js
const {Readable} = require('stream');
 
//这里我们自定义了一个用来读取数组的流
class ArrRead extends Readable {
    constructor(arr, opt) {
        //注意这里，需调用父类的构造函数
        super(opt);
        this.arr = arr;
        this.index = 0;
    }
 
    //实现 _read() 方法
    _read(size) {
        //如果当前下标等于数组长度，说明数据已经读完
        if (this.index == this.arr.length) {
            this.push(null);
        } else {
            this.arr.slice(this.index, this.index + size).forEach((value) => {
                this.push(value.toString());
            });
            this.index += size;
        }
    }
}
 
let arr = new ArrRead([1, 2, 3, 4, 5, 6, 7, 8, 9, 0], {
    highWaterMark: 2
});
 
//这样当我们监听 'data' 事件时，流会调用我们实现的 _read() 方法往缓冲区中读取数据
//然后提供给消费者
arr.on('data', function (data) {
    console.log(data.toString());
});
```

## 二 fs.createReadStream

fs.createReadStream(path, opts) 读取文件时可以传入一些参数，如highWaterMark。该方法工作方式是在内存中准备一段Buffer，然后在fs.read()读取时逐步从聪攀中将字节复制到Buffer中，完成一次读取时，则从这个Buffer中通过slice()方法取出部分数据作为一个小Buffer对象，再通过data事件传递给调用方。如果Buffer用完，则重新分配一个，如果还有剩余，则继续使用，下面为分配一个新的Buffer的操作：
```js
var pool;
function allocNewPool(poolSize) {
    pool = new Buffer(poolSize);
    pool.used = 0;
}
```

在理想的情况下，每次读取的长度就是用户指定的highWaterMark，但是有可能读到了文件末尾，或者文件本身没有指定的highWaterMark那么大，这个预先指定的Buffer对象就会有部分剩余，不过该部分内存可以分配给下次读取时使用。pool是常驻内存的，只有pooo单元剩余数量小于128（kMinPoolSpace）字节时，才会重新分配一个新的Buffer对象。  

highWaterMark如果设置过小，可能会导致系统调用次数过多，该值越大，文件的读取速度越开（但是内存开销也会随着增大）。  
