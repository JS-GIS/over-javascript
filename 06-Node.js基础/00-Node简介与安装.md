## 一 Node.js 简介

#### 1.1 node.js是什么？

Node.js由 Ryan Dahl 于2009年发布，RY原本的目的是为了开发高性能web服务器，由于基于异步事件驱动的想法与js本身的异步特性相像，且JS在web服务端领域一片空白，Node.js横空出世。  

node.js是javascript运行时环境，目前JS常见的运行时环境有两个： 
- node.js：封装了操作网络、文件的api
- 浏览器：封装了操作DOM和BOM的api

他们的共同点，都追随ECMAScript的标准 ，内置了V8引擎（当然火狐等浏览器为主）。 

#### 1.2 Node.js的主要特点：

- 单线程：

    在Java、PHP或者.net等服务器端语言中，会为每一个客户端连接创建一个新的线程。而每个线程需要耗费大约2MB内存。也就是说，理论上，一个8GB内存的服务器可以同时连接的最大用户数为4000个左右。要让Web应用程序支持更多的用户，就需要增加服务器的数量，而Web应用程序的硬件成本当然就上升了。
- 非阻塞I/O：

    当在访问数据库取得数据的时候，需要一段时间。在传统的单线程处理机制中，在执行了访问数据库代码之后，整个线程都将暂停下来，等待数据库返回结果，才能执行后面的代码。也就是说，I/O阻塞了代码的执行，极大地降低了程序的执行效率。
    
    由于Node.js中采用了非阻塞型I/O机制，因此在执行了访问数据库的代码之后，将立即转而执行其后面的代码，把数据库返回结果的处理代码放在回调函数中，从而提高了程序的执行效率。
- 事件驱动：

    在Node中，客户端请求建立连接，提交数据等行为，会触发相应的事件。在Node中，在一个时刻，只能执行一个事件回调函数，但是在执行一个事件回调函数的中途，可以转而处理其他事件（比如，又有新用户连接了），然后返回继续执行原事件的回调函数，这种处理机制，称为“事件环”机制。

    Node.js底层是C++（V8也是C++写的）。底层代码中，近半数都用于事件队列、回调函数队列的构建。

注意：Node10版本中提供了开启线程的api。

#### 1.3 适合开发什么？

+ web服务器后台
+ 命令行工具

Node善于I/O，不善于计算。即CPU密集型Node不擅长，Node擅长的是IO密集型。  

因为Node.js最擅长的就是任务调度，如果你的业务有很多的CPU计算，实际上也相当于这个计算阻塞了这个单线程，就不适合Node开发。  

当应用程序需要处理大量并发的I/O，而在向客户端发出响应之前，应用程序内部并不需要进行非常复杂的处理的时候，Node.js非常适合。Node.js也非常适合与web socket配合，开发长连接的实时交互应用程序，比如：用户表单收集、聊天室、考试系统、图文直播、提供RestfulAPII。  

当然Node从10版本开始支持worker_threads，也能强有力的支持CPU密集型运算。

#### 1.4 Node结构

![](/images/JavaScript/node-01.png)
如图所示，binging一层是JS与底层C++沟通的关键，前者通过bindings调用后者，相互交换数据，libuv为Node提供了跨平台、线程池、事件池、异步IO能力，是Node核心。

## 二 Node安装

#### 2.0 安装说明

偶数位版本为稳定版，奇数位版本为非稳定版。初学者可以在Node官网下载安装包，下一步下一步安装即可，并（可能）需要配置环境变量。  

nvm是一款可以管理node版本的工具，所以在企业级开发中，我们可以使用nvm来安装node，这样可以方便我们控制node的版本。  

贴士：nvm安装也可以有效避免直接安装可能会出现的权限问题，笔者极力推荐使用nvm安装Node。

#### 2.1 安装nvm

win版nvm下载地址：https://github.com/coreybutler/nvm-windows/releases  

```
# 安装步骤：下载后直接下一步下一步即可

# 安装后查看：
nvm version
```

Linux与Mac安装地址：https://github.com/creationix/nvm  

```
# 执行安装：
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash

# 配置环境变量
vim ~/.bash_profile
export NVM_DIR="${XDG_CONFIG_HOME/:-$HOME/.}nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
source ~/.bash_profile

# 安装后查看：
nvm version
```

#### 2.2 安装node

设置nvm下载镜像：
```
# 设置node下载镜像地址
nvm node_mirror https://npm.taobao.org/mirrors/node/  

# 设置node的第三方包下载镜像地址
nvm npm_mirror https://npm.taobao.org/mirrors/npm/         
```

安装Node：
```
# 安装最新版
nvm install latest
# 安装指定版本
nvm install 8.11.3
# 切换版本
nvm use 8.11.3
# 查看当前使用版本
nvm use ls
# 指定默认版本
nvm alias default 8.5.0
```

安装完node后，可以使用 `node -v` 查看node版本；  

安装完node后，打开webstorme，会自动识别node路径，如果没有识别：
可以打开Settings-搜索node-配置；如果此时webstorme 仍然没Node的代码提示功能，解决步骤：File-setting-Languages&Frameworks-Node。