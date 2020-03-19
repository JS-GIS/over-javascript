## 一 webpack初识

### 1.1 webpack简介

网页引入过多的静态资源会造成网页加载速度变慢（多次请求），且需要处理复杂的依赖关系。一般采用合并、压缩、精灵图等方式来解决上述问题，比如require.js、sea.js。但是ES6已经原生支持模块化了，使用原生开发，然后编译、打包更能体验ES6带来的便捷。
但是ES6、Less却不能直接被浏览器支持！WebPack可以看做是模块打包机，可以分析你的项目结构，找到JavaScript模块以及其它的一些浏览器不能直接运行的拓展语言（Scss，TypeScript等），并将其转换和打包为合适的格式供浏览器使用。  

webpack的优点： 
- 1. 对 CommonJS 、 AMD 、ES6的语法规范都做了兼容
- 2. 对js、css、图片等资源文件都支持打包
- 3. 串联式模块加载器以及插件机制，使用更加灵活，扩展性更强
- 4. 有独立的配置文件webpack.config.js
- 5. 可以将代码切割成不同的chunk，实现按需加载，降低了初始化时间
- 6. 支持 SourceUrls 和 SourceMaps，易于调试
- 7.  webpack使用异步 IO 并具有多级缓存。这使得 webpack 很快且在增量编译上更快。

推荐课程：DellLee老师的https://coding.imooc.com/class/316.html

### 1.2 webpack与gulp区别  

Gulp 的定位是 Task Runner, 用来跑一个一个任务，但是没有解决 js module 的问题。其工作方式是：指明对某些文件进行类似编译、组合、压缩等任务的具体步骤，之后gulp工具可以自动替你完成这些任务。
![](/images/JavaScript/webpack-01.png)
Webpack工作方式：把项目当做一个整体，通过一个给定的主文件（如index.js），Webpack将从这个文件开始找到项目的所有依赖文件，使用loaders处理它们，最后打包为一个（或多个）浏览器可识别的JavaScript文件。
![](/images/JavaScript/webpack-02.png)  

### 1.3 webpack安装
```
在项目根目录下 npm init 后：
npm i -D webpack webpack-cli 			# 注意：只有webpack4版本才需要安装webpack-cli

# 查看安装的webpack版本，默认会安装最新版，可以使用 npm i -D webpack@3 确立版本
npx webpack -v							  
```

贴士：很多人使用全局安装webpack的方式，这样在命令行直接就可以启用webpack命令了，（无需像1.5章节中使用.\node_modules\.bin\webpack test.js --mode production）。笔者不推荐，因为不同的项目可能使用的webpack版本不同，全局安装后会影响对不同版本项目的支持。

### 1.4 工程目录
![](/images/JavaScript/webpack-03.png)

### 1.5 使用webpack打包
在工程根目录下输入打包命令：  
```
# 方式一
.\node_modules\.bin\webpack test.js 	# webpack4 需要带上参数：--mode production

# 方式二
npx webpack test.js
```

打包完毕后会在根目录下生成`dist`文件夹，内部包含一个`test.js`文件被打包后生成的`main.js`文件

webpack常用命令参数：
```
--open				打包后自动打开浏览器
--port 				设置端口  
--contentBase 		打开目的文件目录
--hot				浏览器异步更新  主要针对样式的更改
--config a.js		指定配置文件，默认指定根目录下的 webpack.config.js
```

### 1.6 npm脚本运行
反复输入上述命令很麻烦，可以配置一个npm脚本来替代：
```
//注意 webpack3及其以下版本不需要 --mode 参数
"dev": "webpack --mode development"

//配置完成后启动webpack打包命令
npm run dev
```

### 1.7 webpack.config.js

在实际开发中，webpack的打包命令需要配置大量的命令参数，这些参数如果都写在npm脚本中也会引起npm脚本的臃肿，我们可以指定一个webpack的配置文件，让npm脚本运行的webpack命令去该配置文件查找webapck的命令行参数：

```
"dev": "webpack --config webpack.config.js --mode development"
```

在项目根目录下创建webpack.config.js配置文件：
```js
const path = require('path');

module.exports = {
	entry:path.resolve(__dirname,'src/index.js'), 	 //入口
	output: {
		path: path.resolve(__dirname, 'dist'),      //输出文件夹
		filename: 'bundle.js'          				 //输出文件名
	}
};
```

贴士：在webpack4.0时，打包需要设置mode，默认值为production，也可以设置为development，二者分别用于生产环境（会压缩）和开发环境



