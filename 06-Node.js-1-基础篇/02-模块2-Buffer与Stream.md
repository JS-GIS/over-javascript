## 一 Buffer

#### 1.1 Buffer的使用

可以将Buffer暂且看作Node内置的一种数据类型，类似String/Number这样直接使用。  

Buffer类用于操作二进制数据，类内的参数用于分配长度，分配后不能更改。  
 

Buffer的创建：
```js
let bf = Buffer.from('test','utf-8');                // 新版node已经废弃了new Buffer()创建方式，因为不安全     
console.log(bf);                                    // 输出 <Buffer 74 65 73 74>
for(let i = 0; i < bf.length; i++){                 //此length长度和字符串的长度有区别，指buffer的bytes大小
    console.log(bf[i].toString(16));                // buffer[index]:获取或设置在指定index索引未知的8位字节内容
    console.log(String.fromCharCode(bf[i]));        //依次输出 t e s t
    console.log(bf.toString());                     //输出 test  ，可选参数是 [encoding, start, end]，默认使用UTF-8
}
```

Node在文件、网络操作中，如果没有显示声明编码格式，默认返回的数据类型都是Buffer，比如readFile回调中的data。

注意：在ES6中增加了ArrayBuffer类型，Node中可以直接使用

#### 1.2 Buffer的实例方法

```js
buf.write(string,[offset],[length],[encoding]):根据参数offset，将参数string数据写入buffer
buf.toString([encoding],[length]):返回一个解码的string类型
buf.toJSON():返回一个JSON表示的Buffer实例，JSON.stringify将会默认调用来字符串序列化这个Buffer实例
buf.slice([start],[end]):返回一个新的buffer，这个buffer和老的buffer引用相同的内存地址
buf.copy(targetBuffer,[targetStart],[sourceStart],[sourceEnd])：进行buffer的拷贝，拷贝不会影响老的buffer。
```

#### 1.3 Buffer的静态方法

```JavaScript
Buffer.isBuffer(buf);		            // 判断是不是Buffer
Buffer.byteLength(str);	                // 获取字节长度，第二个参数为字符集，默认utf8
Buffer.concat(list[, totalLength])	    // Buffer的拼接
```

## 二 Stream流

#### 2.1 Stream流简介

使用readFile方法是极其占用内存的（一次加载大文件），读取完毕后再发送，读和发没有同时进行，这样不但影响性能，也会让网络应用、磁盘应用之间永远只有一个在疯狂活动，应该读取一点，发送一点，这样网络、磁盘都在积极使用。  

使用流可以解决上述问题：
- 读取流：fs.createReadStream(req)
- 写入流：fs.createWriteStream(res)

在Node有四种基础的stream类型：
- Readable:可读流
- Writeable:可写流
- Duplex:读写流
- Transform:操作写入的数据，然后读取结果，通常用于输入数据和输出数据不要求匹配的场景，如zlib.createDeflate()

#### 2.2 Stream流的使用

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

#### 1.3 自定义Stream

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

案例：https://www.cnblogs.com/jkko123/p/10240574.html

## 三 乱码问题

#### 3.1 乱码的出现原因

在web开发中，经常使用下面的方式拼接Buffer：
```js
let body = "";
req.setEncoding("utf8");
req.on("data", function(chunk){
    body += chunk;
});
req.on("end", function(){});
```
在上述拼接过程中，包含了一个隐式转换，`body+=chunk`相当于`body+=chunk.toString()`。如果上传的字符全是英文则没有问题。当字符串中包含中文或者其他语言，由于toString默认使用utf-8编码，这时就会出现乱码。 

新建一个文本test.txt，内部书写：`八百标兵奔北坡，北坡炮兵并排跑`;
```js
let rs = require("fs").createReadStream("./test.txt",{highWaterMark: 10});

let data = "";

rs.on("data", function(chunk){
    data += chunk;
});

rs.on("end", function(){
    console.log(data);
});
```
输出结果：
```
八百标���奔北��，北坡炮兵并���跑
```  

解释：  

highWaterMark即最高水位线，表示内部缓冲区最多能容纳的字节数，如果超过这个值，则停止读取资源文件，默认是64KB。  
假设文件是100KB，当读取到64KB时，就会触发data事件，然后接着读取剩下的36KB，并再次触发data、end事件。  
如果highWaterMark值过小，则会频繁产生系统调用，影响性能。  
在utf8中，1个汉字占据3个字节，highWaterMark被设置为10个字节后，前三个汉字不会受到影响，第四个汉字就会被截断，在调用toString时候出现乱码。

#### 3.2 乱码的解决

上述代码已经被舍弃，官方推荐使用push方法来拼接Buffer：
```js
let rs = require("fs").createReadStream("./test.txt",{highWaterMark: 10});

let dataArr = [];

rs.on("data", function(chunk){
    dataArr.push(chunk);
});

rs.on("end", function(){
    let buf = Buffer.concat(dataArr);
    console.log(buf.toString());
});
```