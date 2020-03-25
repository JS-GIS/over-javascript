## 一 模块简介

在项目开发中，对不同的项目功能需要做文件切割，以更好的产出可维护性代码，对分割后的不同功能代码的引入便需要编程需要编程语言的模块机制支持。  

JavaScript由于诞生之初只是一个浏览器脚本语言，并未支持大型项目要求的模块化需求。为了适应现在的发展，JavaScript先后诞生了各种模块规范，如AMD、CMD。随着ECMAScript标准的逐步更新，这些标准的混战也接近尾声，目前主流的两个模块化应用方案是：
- CommonJS：NodeJS当前所使用的模块方案
- ES Module：ES6所使用的模块方案，由于NodeJS遵循ECMASCript规范，也在渐渐支持ES Module。

## 二 CommonJS

###  2.1 CommonJS规范

CommonJS认为模块即文件，所以每个文件都可以依据模块规则进行导入，导出。  

CommonJS中定义一个模块：
```js
// 创建math.js文件
function add(num1, num2) {
    return num1 + num2;
}

module.exports = add
```

在同目录下其他文件引入该模块：
```js
var add = require('./math');

console.log(add(5, 4));         // 输出 9
```

一个JavaScript文件，可以向外exports无数个变量、函数。但是require的时候，仅仅需要require这个JS文件一次。使用的它的变量、函数的时候，用点语法即可。所以，CommonJS的模块导出、导入机制可以让用户完全不必考虑变量污染问题。    

###  2.2 exports的导出导入

exports导出变量：
```js
// 新建foo.js，导出 msg1 和 msg2 这两个变量
let msg1 = "你好";
let msg2 = "hello world";
exports.zhText = msg1;
exports.enText = msg2;
```

变量导入并使用：
```js
//新建一个main.js，输入以下代码
let foo = require("./foo.js");      // 核心模块（node本身的模块）在导入时无须路径，如 let http = require('http');
console.log(foo.zhText);
console.log(foo.enText);
```

###  2.3 module.exports 与 exports.变量 区别

**mdoule.exports** 直接导出了变量本身，而 **exports.变量**则是将该导出的变量重新挂载到`=`后的变量上。  

在本质上，`exports`只是`module`对象下的一个属性。所有`exports`都是通过`module.exports`传递的，类似于每个模块头部都有：`var exports = module.exports;`。如图所示：  

![](../images/node/01-02.svg)   

贴士：
- 多个`module.exports`，只会导出最后一个，前面的都会被忽略
- `module.exports`和`exports`不能共用
- require并不依赖于exports，可以加载一个没有暴露任何方法的模块，这相当于执行一个模块内部的代码

推荐在导出一个整体对象的时候使用`module.exports`，因为exports是通过形参方式传入的，直接赋形参会改变形参的引用。  

## 三 模块加载机制

###  3.1 文件类型朝招机制

如果加载模块文件时，没有写后缀的话，找模块的时候会按照`.js`，`.json`，`.node`这三种后缀的优先级分别去找，查找过程如下：
- 首先按照加载的模块的文件名进行查找；
- 没找到，在模块文件名后加上 .js 后缀进行查找；
- 没找到，在文件名后加上 .json 后缀查找；
- 没找到，在文件名后加上 .node 后缀查找；
- 抛出错误。

贴士：在引入.node和.json文件时，带上标识名，会加快一点速度。

###  3.2 文件加载机制

在Node中，模块分为两大类：
- 核心模块：Node官方默认提供的模块，如fs、http，引入方式：`require('fs')`，无需指定路径。
- 文件模块：用户编写的模块，引入方式：`require('./mymodule/math')`，也可以使用绝对路径。
- 自定义模块：笔者认为可以称为第三方模块，也无需路径直接引入：`require('express')`

模块在被引入时大致要经过三个步骤：路径分析-->文件定位-->编译执行，但是在Node中，核心模块与文件模块加载时机制完全不同：
- 核心模块：这些模块在Node源码编译时候，已经编译进了Node的二进制可执行文件中，Node程序启动时，这些模块就会被直接加载进内存。所以核心模块在引入时只有编译执行一步，其加载速度也是最快的！
- 文件模块：该部分模块在运行时动态加载，需要完整执行上述三步，速度较慢

当然还有一种情况，用户引入了第三方模块，这种模块位于Node默认提供的`node_modules`文件夹中，而这个文件夹在Node的项目目录、Node安装目录中都存在着！！！由于require时没有指定路径，那么就需要Node进行逐级向上查找：
- 首先从当前项目目录下的node_modules中查找
- 从父目录下的node_modules中查找，依次类推
- 直到找到Node本身跟目录下的node_modules文件夹

该模块随着项目目录的深入，会造成大量耗时！！

###  3.3 文件缓存机制

已经加载的模块会被缓存，所以，多次require同一个文件的时候不会重新加载，只会加载一次；`require`根据文件的路径去判断是否是同一个文件，路径一样的话，加载多次，用的也是第一次加载的内容。  

所在Node在加载文件时的消耗上整体是：从缓存中加载-->加载核心模块-->加载文件模块。  

注意：require从缓存中加载文件是基于文件路径的，这表示即使有两个完全相同的文件，但她们位于不同的路径下，也会在缓存中维持两份。  

查看缓存代码：
```js
console.log(require.cache);
```

当调用require加载一个模块时，模块内部的代码都会被调用！！！！，有时候这可能会带来隐藏的bug。   

module.js：
```js
function test() {
    setInterval(function(){
        console.log("test");
    },1000);
}
test();

module.exports = test;
```

main.js:
```js
let test = require("./module");
```

main.js只是加载了module文件，但是仍然每隔1秒输出了test字符串，且main.js的进程始终没有退出！！这在生产环境中极其造成内存泄漏。所以使用模块时要留意该情况。

###  3.4 文件目录分析机制

在查找require的文件时，没有找到相应文件，却找到了对应require参数的目录，此时Node会将目录当做一个包来处理，此时Node会查找到包内的哪个文件呢？
- 每个Node项目都带有package.json文件，该json文件内部有个key指定了包的入口文件`"main":"app.js"`，此时Node会加载该包内的app.js
- 如果package.json文件不存在，或者内部的main指定的文件名错误，则Node会依次查找包中：index.js、index.json、index.node
- 如果没找到，则抛出查找失败异常

###  3.5 重复引用问题

Node无须关心重复引用问题，因为Node先从缓存中加载模块，一个模块被第一次加载后，就会在缓存中维持一个副本，如果遇到重复加载的模块会直接提取缓存中的副本，也就是说在任何情况下每个模块都只在缓存中有一个实例。

## 四 前端和后端模块化要求的不同

前端标准的模块化规范：
- AMD：代表技术是require.js，CommonJS规范是AMD规范的延伸。
- CMD：代表技术是seajs
- ES Module：ECMAScript官方模块化规范，笔者相信AMD和CMD都会慢慢退出舞台

服务端的模块化方案常用的是CommonJS，其代表是Node。  

前端和后端模块规范的主要区别是：
- 前端异步加载（import）：前端加载的js等文件，要经过服务器、通过网络、传输到浏览器端，代码才能执行；
- 后端同步加载（require）：服务端访问的文件位于本地，且会缓存已经加载的文件，IO开销几乎可以忽略，所以无需像前端那样考虑异步加载，且服务端很少出现重启情况，也无须担心启动时的时间开销