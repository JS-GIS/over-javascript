## 一 HelloWorld

###  1.1 普通函数 hello world

新建`helloworld.js`，编写代码如下：
```js
console.log("hello world");
```

此时可以在开发工具中直接右键运行，也可以在当前目录打开命令行，输入以下命令：
```
node helloworld.js      # 查看输出结果
```

注意：文件名不要以node.js来命名。

### 1.2 服务端 hello web

新建`helloweb.js`，编写代码如下：

```js
let http = require('http');
let server = http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/plain;charset=UTF8'});
    res.end('hello web');
}).listen(8000);
```

运行：
```
# 启动web服务器，启动后可以访问：localhost:8000
node helloweb.js
```

## 二 全局对象

###  2.0 Node全局对象汇总

Node提供了一些全局对象：
- global：用于挂载全局变量
- __filename：用于获取当前文件路径
- __dirname：用于获取当前目录路径
- process：用于获取系统运行信息
- buffer：缓存模块
- exports：用于支持模块导出
- module：用于支持文件模块化
- require()：该函数用于导入模块

###  2.1 全局对象global

在原生js中定义全局变量，浏览器中可以通过window顶层对象访问（即全局变量是顶层对象的属性）：
```js
let a = 100;
console.log(window.a);
```

Node没有window这个顶层对象，Node自身的顶层对象是 global，定义全局变量方式：
```js
a = 3;            // 第一种定义全局变量方式，如果使用var a = 3; global是无法访问到的。
global.b = 3;     // 第二种定义全局变量方式
```

###  1.2 模块属性 __filename  __dirname  

- __filename：返回当前模块的文件的解析后的绝对路径（包含文件名），该属性并非全局的（不能global.__filename），而每个模块都有该属性，所以直接可以使用。
- __dirname：返回当前模块文件所在目录解析后的绝对路径（不包含文件名），返回的是文件夹。同__filename一样，其也非全局对象，而是每个模块都有的属性。  
