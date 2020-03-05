## 一 开发工具

笔者推荐的NodeJS开发工具是VScode/WebStorm。  

如果是使用VSCode开发，推荐安装插件：
- Code Runner：用于运行node代码
- Search node_modules：用于node包快速提示

如果使用WebStorme（或者IDEA+Node插件），则需要配置下node环境：  
- 打开WebStorme，在设置中界面中选择：`Languages&Frameworks`，对比下图进行勾选：  
![](/../images/node/idea.jpg)

## 二 HelloWorld

#### 1.1 普通函数的helloworld

新建`helloworld.js`，编写代码如下：
```js
console.log("hello world");
```

此时可以在开发工具中直接右键运行，也可以在当前目录打开命令行，输入以下命令：
```
node helloworld.js      # 查看输出结果
```

注意：文件名不要以node.js来命名。

#### 1.2 服务端helloweb

新建`helloweb.js`，编写代码如下：

```js
let http = require('http');
let server = http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/plain;charset=UTF8'});
    res.end('hello web');
}).listen(8000);
```
对该js文件执行:`node helloweb.js`，此刻即启动了一个基于node的web服务器，访问`localhost:8000`，即可看到网页输出`hello web`
