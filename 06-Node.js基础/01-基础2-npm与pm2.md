## 一 npm的使用  

#### 1.1 npm简介  

npm是Node官方的包管理器，用来下载各种node第三方模块，在安装node时候自带了npm。  

npm官网：https://www.npmjs.com/  

我们可以在npm官网搜索自己想要的包。  

#### 1.2 使用npm初始化一个Node项目

步骤一：初始化项目配置
```
mkdir demo      # 创建一个 demo项目文件件
cd demo
npm init        # 初始化Node项目环境，会在当前项目目录下生成核心配置文件  package.json
```

步骤二：安装开发包
```
npm install express     # install会执行安装express开发包的命令，默认安装最新版本。 install可以简写为 i

# 安装完毕后，我们会在 package.json文件中发现该依赖，当前目录会生成文件夹 node_modules，express包便被安装在此目录中
```

步骤三：使用开发包
```js
// 安装了express包后，现在就可以使用express包了，新建一个文件 app.js，代码如下：
let express = require("express");

let app = express();

app.get("/", function(req, res){
    return res.send("hello world");
});

app.listen(3000);

```

步骤三：测试。打开浏览器，访问 `http://localhost:3000/`，就会看到我们使用express包开发这个项目输出结果了。

#### 1.3 npm的一些使用

npm在安装包时候可以设定其安装为生产环境还是开发环境，如下所示：

```
npm i express -g    # 全局安装
npm i express -S    # 以生产依赖形式本地安装
npm i express -D    # 以开发依赖形式本地安装
```

#### 1.4 npm镜像与cnpm

由于一些国内原因，npm安装包速度很慢，可以设置npm镜像：
```
npm config rm proxy
npm config rm https-proxy
npm config set registry https://registry.npm.taobao.org
```

也可以直接使用淘宝开发的cnpm来代替npm：
```
npm install cnpm -g
cnpm install jQuery
```

恢复npm镜像办法：
```
npm config set proxy=http://127.0.0.1:1080
npm config set registry=http://registry.npmjs.org
```