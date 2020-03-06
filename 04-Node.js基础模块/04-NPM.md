## 一 npm的使用  

### 1.1 npm简介  

npm是Node官方的包管理器，用来下载各种node第三方模块，在安装node时候自带了npm。  

npm官网：https://www.npmjs.com/  

我们可以在npm官网搜索自己想要的包。  

### 1.2 使用npm初始化一个Node项目

步骤一：初始化项目配置
```
mkdir demo      # 创建一个 demo项目文件件
cd demo
npm init        # 初始化Node项目环境，会在当前项目目录下生成核心配置文件  package.json，添加 -y 参数可以自动生成
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

## 二 描述文件

在使用 `npm init`时会生成一个包描述文件 `package.json`，该描述文件详细解释了整个包的情况。

## 三 搭建局域NPM仓库

NPM是Node拥有当前生态的重要推动因素之一，在实际开发中，项目代码托管在git仓库中，项目模块发布在NPM中，二者相辅相成。不过在企业中，安全性至为重要，往往需要私建git仓库与npm仓库。   

目前主流的私有npm仓库方案是 sinopia：使用文件系统作为存储，仅保存用户需要的包，如果本地仓库没有对应的包，则从指定的registry下载，默认为npmjs.org，可以改成淘宝的镜像。  

安装Sinopia：
```
# 需要具备 python2.7环境和环境变量
npm install -g sinopia

# 如果出现crypt3问题，则可以删除sinopia安装目录node_modules里的 fs-ext 和 crypt3 相关的包，包括：0.2.0@crypt3，.0.6.0@fs-ext，crypt3和fs-ext以及sinopia-htpasswd\node_modules下的crypt3和fs-ext
```

启动：
```
npx sinopia         # 启动后访问 http://localhost:4873/
```

切换npm源：
```
npm config set registry http://xxx.xx.xx.xx:4873/       
//xxx.xx.xx.xx 为自己的ip 
```

建议使用nrm：
```
npm install -g nrm # 安装nrm
nrm add name http://XXXXXX:4873 # 添加本地的npm镜像地址
nrm use name # 使用本址的镜像地址     name为你要增加的地址
```

新建用户：
```
npm adduser
Username: test
Password: test
Email: (this IS public) xxx@xxxx
```

发布包：
```
npm publish     // 在自己要发布的包 路径打这个命令
```

配置文件：
```
#
# This is the default config file. It allows all users to do anything,
# so don't use it on production systems.
#
# Look here for more config file examples:
# https://github.com/rlidwka/sinopia/tree/master/conf
#

# path to a directory with all packages
storage: ./storage    # npm包存放的路径

auth:
  htpasswd:
    file: ./htpasswd
    # Maximum amount of users allowed to register, defaults to "+inf".
    # You can set this to -1 to disable registration.
    # max_users: 1000
    max_users: 1000     # 默认为1000，改为-1，禁止注册

# a list of other known repositories we can talk to
uplinks:
  npmjs:
    url: http://registry.npm.taobao.org/  # 默认为npm的官网，由于国情，修改 url 让sinopia使用 淘宝的npm镜像地址

packages:
  '@*/*':
    # scoped packages
    access: $all
    publish: $authenticated

  '*':
    # allow all users (including non-authenticated users) to read and
    # publish all packages
    #
    # you can specify usernames/groupnames (depending on your auth plugin)
    # and three keywords: "$all", "$anonymous", "$authenticated"
    access: $all

    # allow all known users to publish packages
    # (anyone can register by default, remember?)
    publish: $authenticated

    # if package is not available locally, proxy requests to 'npmjs' registry
    # proxy: npmjs   #这个去掉的话,sinopia 将不去下载依赖包   如果只是要放自己资源仓库的话就去掉      
    # 

# log settings
logs:
  - {type: stdout, format: pretty, level: http}
  #- {type: file, path: sinopia.log, level: info}

# you can specify listen address (or simply a port) 
listen: 0.0.0.0:4873  # 默认没有，只能在本机访问，添加后可以通过外网访问。
```

注意：
- 配置文件 是在你的用户名 如：本人就admin 找到目录就可以了 （win)
- storage: 仓库保存的路径
- htpasswd: 保存密码信息 只有新建用户后才 有这个文件
- config.yaml: 这个是本地的 配置文件 信息（改这个）