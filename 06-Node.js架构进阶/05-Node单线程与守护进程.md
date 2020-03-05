## 一 单线程Node

单线程Node服务往往会因为一些异常而被中断，其实只要我们处理了下列异常即可：
```js
process.on('uncaughtException', err => {
    console.log("uncaughtException:", err)
});
```

当然我们也可以使用try/catch，但是要注意try/catch在异步中的使用问题。

## 二 企业级进程管理工具pm2

针对上一节中Node的问题，笔者推荐使用进程管理工具pm2，pm2可以有效的监控服务器状况，并让node服务器自动重启，支持守护进程。  

安装pm2：
```
npm install pm2 -g
```

使用pm2：
```
开启：pm2 start ***.js		                    # node关闭窗口仍然运行，且会自动重启
参数启动：pm2 start **.js --name=”test” 		 # 启动应用程序并命名为 "api"
关闭：pm2 stop **.js
重启：pm2 restart **.js
```

pm2可以监控并管理多个应用程序，并对其进行日志监控。  

进程监控命令：`pm2 list`，监控界面：
![](/images/node/pm2list.png)

日志监控：`pm2 logs 项目名/id名`，监控界面：
![](/images/node/pm2logs.png)