## 一 简单的路由实现
```JavaScript
const http = require('http');
const url = require('url');
const fs = require('fs');
let server = http.createServer();
server.on('request', function (req, res) {
    console.log('有用户请求了');
    let urlStr = url.parse(req.url);
    switch (urlStr.pathname) {
        case '/':           //首页 
            res.writeHead(200, {'content-type': 'text/html;charset=utf-8'});
            res.end('<h1>这是首页</h1>');
            break;
        case '/user':       //用户页面 
            sendData(__dirname + '/html/' + '1.html', req, res);
            break;
        default:
            res.writeHead(404, {'content-type': 'text/html;charset=utf-8'});
            res.end('<h1>页面不存在</h1>');
            break;
    }
});

function sendData(filePath, req, res) {
    fs.readFile(filePath, function (err, data) {
        if (err) {
            res.writeHead(404, {'content-type': 'text/html;charset=utf-8'});
            res.end('<h1>页面不存在</h1>');
        } else {
            res.writeHead(200, {'content-type': 'text/html;charset=utf-8'});
            console.log(String(data));
            res.end(data);
        }
    });
}

server.listen(8081, 'localhost');
```
## 二 静态资源管理
#### 2.1 简单实现
```JavaScript
const http = require('http');
const fs = require('fs');
const url = require('url');
const path = require('path');   //用来判断请求的文件的扩展名

http.createServer((req,res)=>{
    let pathname = url.parse(req.url,true).pathname;
    if(pathname == '/'){
        res.end('hello world');
    } else if(pathname.substring(0,8) == '/public/'){
        fs.readFile('.' + pathname,(err,data)=>{
            if(err){
                res.writeHead(404,{'Content-Type':'text/html;charset=UTF8'});
                res.end('404');
            } else {
                let mime = getMime(path.extname(pathname));
                res.writeHead(200,{'Content-Type':mime});
                res.end(data);
            }
        });
    } else {
        res.end('other page');
    }
}).listen(3000);

function getMime(extname) {
    switch (extname){
        case '.html':
            return 'text/html';
            break;
        case '.jpg':
            return 'image/jpg';
            break;
        case '.png':
            return 'image/png';
            break;
    }
}

```
#### 2.2 静态资源缓存
每次请求服务器的静态资源，都会造成IO上的浪费，那么我们可以使用缓存来优化性能。当浏览器中有缓存副本时，不确定该副本是否有效，会生成一个get请求，在该请求的header中包含一个if-modified-since时间参数。如果服务器端文件在这个时间参数后修改过了，服务器发送全部文件给客户端，如果没有，则返回304状态码，并不发送整个文件。
如果确定该副本有效，客户端不会发送GET请求。判断有效的方法是：服务端响应头上带有expires头。
Expires：是一个毫秒值，如果该值小于当前时间，则不缓存。
```JavaScript
const http = require('http');
const url = require('url');
const fs = require('fs');

http.createServer(function (req,res) {
    let pathname = url.parse(req.url).pathname;
    if(pathname == '/favicon.ico'){
        return;
    } else {
        dealStatic(req,res,pathname);
    }
}).listen(80);


function dealStatic(req,res,pathname) {
    console.log('pathname=' + pathname);
    let realPath = __dirname + '\\public\\' + pathname.toString().substr(1);
    console.log('realPath=' + realPath);
    if(pathname == '/' || pathname == '/index'){
        res.writeHead(200);
        res.end('hi');
    } else {
        fs.exists(realPath,function (exists) {
            if(!exists){                //文件不存在
                res.writeHead(404,{'Content-Type':'text/plain'});
                res.end('404');
            }else {

                let mimeString = pathname.substring(pathname.lastIndexOf('.') + 1);
                console.log('mimeString=' + mimeString);
                let mimeType = null;
                switch (mimeString){
                    case 'css': mimeType = 'text/css';
                        break;
                    case 'png': mimeType = 'image/png';
                        break;
                    default: mimeType = 'text/plain';
                }

                let fileInfo = fs.statSync(realPath);
//获取服务器文件最后修改时间
                let lastModified = fileInfo.mtime.toUTCString();  
                //设置7天缓存存在时间
let CACHETIME = 60*60*24*7;                       
                /*
                客户端请求时间 大于 Expires（date值）发送新请求
                客户端请求时间 小于 Expires（date值）读取本地缓存
                 */
                let date = new Date();
//当前时间+缓存时间
                date.setTime(date.getTime() + CACHETIME*1000);   

                if(req.headers['if-modified-since'] && lastModified == req.headers['if-modified-since']){
                    console.log('执行了读取本地缓存');
                    res.writeHead(304,'Not Modified');
                    res.end('304');
                } else {
                    fs.readFile(realPath,function (err,file) {
                        if(err){
                            res.writeHead(500);
                            res.end(err);
                        } else {
                            //没有缓存，设置缓存
                            console.log('执行了发送服务器文件');
                            res.setHeader('Expires',date.toUTCString());
                            res.setHeader('Cache-Control','max-age=' + CACHETIME);
                            res.setHeader('Last-Modified',lastModified);
                            res.writeHead(200,{'Content-Type':mimeType});
                            res.end(file);
                        }
                    });
                }
            }
        });
    }
}

```
## 三 处理请求
#### 3.1 表单的提交方式
application/x-www-form-urlencoded		默认提交方式，即url编码方式
text/plain								用的很少，纯文字提交
multipart/form-data					提交文件
#### 3.2 GET请求
```JavaScript
const http = require('http');
const url = require('url');

http.createServer((req,res)=>{
//true代表直接将结果解析为json
    let {pathname,query} = url.parse(req.url,true);
    console.log(pathname);      //输出请求地址 /get
    console.log(query);         //参数对象  {name:'lisi'}
    res.end('hello');
}).listen(8000);
```
#### 3.3 POST请求
```JavaScript
const http = require('http');
const querystring = require('querystring');
http.createServer((req,res)=>{
    let str = '';
    req.on('data',data=>{
         str += data;
    });
    req.on('end',()=>{
        let postData = querystring.parse(str);
        console.log(postData);  // 输出json格式
    });
    res.end();
}).listen(8000);
```
## 四  文件上传
#### 4.1 原生文件上传
在POST请求说明中，使用了字符串变量str来接收，如果上传的是文本文件，则仍然是正确的，但是如果上传的是图片、音频等，这里就无法实现了，我们可以使用数组的形式保存：
```JavaScript
const http = require('http');
let server = http.createServer(function (req,res) {
    let arr = [];
    req.on('data',function (data) {
        arr.push(data);
    });
    req.on('end',function () {
        let buf = Buffer.concat(arr);
        console.log(buf.toString());
        res.end();
    });
});
server.listen(8000,function () {
    console.log('server is started...');
});
```
#### 4.2 第三方模块 formidable 
```JavaScript
const url = require('url');
const http = require('http');
const path = require('path');

const formidable = require('formidable');

http.createServer(function (req,res) {
    let pathname = url.parse(req.url).pathname;
    if (pathname == '/upload' && req.method.toLowerCase() == "post") {
        uploading(req,res);
    }
}).listen(3000);

//图片上传的方法
function uploading(req,res) {
    let form = new formidable.IncomingForm();
    form.uploadDir = path.join(__dirname, './upload');
    form.parse(req, function (err, fields, files) {
        if (err) throw err;
        res.writeHead(200, {'content-type': 'text/plain;charset=utf8'});
        res.end('上传成功');
    });
}
```

