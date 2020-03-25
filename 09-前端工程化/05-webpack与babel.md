## 一 webpack配置babel

转译ES6，ES7需要babel-loader加载器。

第一步：安装 babel-loader
```
webpack 4.x | babel-loader 8.x | babel 7.x
npm i -D babel-loader @babel/core @babel/preset-env 
webpack 4.x | babel-loader 7.x | babel 6.x
npm i -D babel-loader@7 babel-core babel-preset-env
```

第二步：添加babel-loader加载器
```
{
	test: /(\.jsx|\.js)$/,
	use: {
		loader: "babel-loader"
	},
	exclude: /node_modules/
}
```

第三步：此时babel-loader不知道用哪种babel格式转译，配置 .babelrc
```
{
	"presets": ["@babel/preset-env"]
}
```
注意：如果是babel6，此处配置为：["babe-preset-env"]  

也可以不添加.babelrc文件，在babel-loader插件内直接配置：
```
	use: {
		loader: "babel-loader",
		options: { 
			presets: ["@babel/env"] 
		}
	},
```

## 二 babel转换函数

### 2.0 babel无法转换的语法

虽然babel把ES6解析为了ES5，但是仍然有许多变量在低版本不支持，比如：Promise,Set,Symbol,Array.from,async等。  

解决办法：
```
# 安装poyfill
npm install -S @babel/polyfill

# 配置preset，意思是在第三步中引入的pollyfill不再将所有的ES6语法实现，即只有用到了map，才会加载map的实现，不会加载没用到的promise实现。
presets: [
  ["@babel/preset-env", {useBuiltIns: "usage}]
]

# 在业务源码的顶部加入以下代码：
import "@babel/pollyfill"     # 如果配置了useBuiltIns，则无需引入，会自动引入
```

但是如果我们当前书写的不是业务代码，而是自己书写一些插件，框架，那么pollyfill的方式不推荐，因为pollyfill生成的map，promise都是以全局变量的形式存在，会污染框架的环境，此时推荐的配置方式:
```
# 注释掉import pollyfill象关的代码

# 安装transform-runtime
npm i -D @babel/plugin-transform-runtime
npm i -S @babel/runtime
npm i -S @babel/runtime-corejs2   

# 删除presets配置，添加plugins
options: {
  "plugins":[
    ["@babel/plugin-transform-runtime",{
      "corejs": 2,
      "helpers": true,
      "regenerator": true,
      "useESModules": false
    }]
  ]
}
```

### 2.1 原理解读

```js

// 原生代码如下：
const foo = (a, b) => {
    return Object.assign(a, b);
};

//经过babel编译后为：
"use strict";
var foo = function foo(a, b) {
    return Object.assign(a, b);
};
```

Object.assign作为ES6语法被编译成了普通函数，而不是我们理想的结果：
```js
(Object.assign||function() { /*...*/}) 
```

这样编译为了保证正确的语义，只能转换语法而不是去增加或修改原有的属性和方法。  

所以 babel 不处理 Object.assign 反倒是最正确的做法。而处理这些方法的方案则被称为 polyfill。  

babel针对每个API都提供了对应的转换插件，如上述案例需要安装的插件是：
```
# 安装
npm i - S @babel/plugin-transform-object-assign	(注意旧版写法)

# .babelrc配置：
 "plugins": ["@babel/transform-object-assign"]
```

此时编译后的代码为：
```js
var _extends = Object.assign || function (target) { for (var i = 1; i < arguments.length; i++) { var source = arguments[i]; for (var key in source) { if (Object.prototype.hasOwnProperty.call(source, key)) { target[key] = source[key]; } } } return target; };

var foo = exports.foo = function foo(a, b) {
  return _extends(a, b);
};
```
此时如果遇到下面的场景：
```js
// another.js
export const bar = (a, b) => Object.assign(a, b);

// index.js
import { bar } from './another';

export const foo = (a, b) => Object.assign(a, b);
```
编译后的结果是：
```js
/***/ 211:
/***/ (function(module, exports, __webpack_require__) {

"use strict";

Object.defineProperty(exports, "__esModule", {
  value: true
});
exports.foo = undefined;

var _extends = Object.assign || function (target) { for (var i = 1; i < arguments.length; i++) { var source = arguments[i]; for (var key in source) { if (Object.prototype.hasOwnProperty.call(source, key)) { target[key] = source[key]; } } } return target; };

var _another = __webpack_require__(212);

var foo = exports.foo = function foo(a, b) {
  return _extends(a, b);
};

/***/ }),

/***/ 212:
/***/ (function(module, exports, __webpack_require__) {

"use strict";

Object.defineProperty(exports, "__esModule", {
  value: true
});

var _extends = Object.assign || function (target) { for (var i = 1; i < arguments.length; i++) { var source = arguments[i]; for (var key in source) { if (Object.prototype.hasOwnProperty.call(source, key)) { target[key] = source[key]; } } } return target; };

var bar = exports.bar = function bar(a, b) {
  return _extends(a, b);
};

/***/ })
```
transform 的引用是 module 级别的，这意味着在多个 module 使用时会带来重复的引用，这在多文件的项目里可能带来灾难。另外，你可能也并不想一个个的去添加自己要用的 plugin，如果能自动引入该多好。

### 2.2 解决方案
前面提到问题主要在于方法的引入方式是内联的，直接插入了一行代码从而无法优化。鉴于这样的考虑，babel 提供了 babel-plugin-transform-runtime，从一个统一的地方 core-js自动引入对应的方法。
```
npm i -D @babel/plugin-transform-runtime
npm i -S @babel/runtime

# .babelrc
{
  "plugins": ["@babel/transform-runtime"]
}
```
但是这里引入了新的问题，因为自动化的操作往往不精准：
```
export const foo = (a, b) => Object.assign(a, b);

export const bar = (a, b) => {
    const o = Object;
    const c = [1, 2, 3].includes(3);
    return c && o.assign(a, b);
};
```
我们会发现上述 o.assign(a, b)被原模原样的编译了。同时，因为 babel-plugin-transform-runtime 依然不是全局生效的，因此实例化的对象方法则不能被 polyfill，比如 [1,2,3].includes 这样依赖于全局 Array.prototype.includes 的调用依然无法使用。

### 2.3 babel-polyfill

上面两种 polyfill 方案共有的缺陷在于作用域。因此 babel 直接提供了通过改变全局来兼容 es2015 所有方法的 babel-polyfill，安装 babel-polyfill 后你只需要在所有代码的最前面加一句 import 'babel-polyfill' 便可引入它，如果使用了 webpack 也可以直接在 entry 中添加 babel-polyfill 的入口。
```js
import '@babel/polyfill';

export const foo = (a, b) => Object.assign(a, b);
```
加入 babel-polyfill 后，打包好的 pollyfill.js 一下子增加到了 251kb（未压缩），（建议感兴趣的同学把代码拉下来运行一下，之后提到的所有方式也都可以看到打包结果）搜索一下 polyfill.js 不难找到这样的全局修改:
```js
//polyfill
`$export($export.S + $export.F, 'Object', {assign: __webpack_require__(79)});
```
babel-polyfill 在项目代码前插入所有的 polyfill 代码，为你的程序打造一个完美的 es2015 运行环境。babel 建议在网页应用程序里使用 babel-polyfill，只要不在意它略有点大的体积（min 后 86kb），直接用它肯定是最稳妥的。值得注意的是，因为 babel-polyfill 带来的改变是全局的，所以无需多次引用，也有可能因此产生冲突，所以最好还是把它抽成一个 common module，放在项目 的 vendor 里，或者干脆直接抽成一个文件放在 cdn 上。

如果你是在开发一个库或者框架，那么 babel-polyfill 的体积就有点大了，尤其是在你实际使用的只有一个 Object.assign 的情况下。更可怕的是对于一个库来说，改变全局环境是使不得的。谁也不希望使用了你的库，还附带了一家老小的 polyfill 改变了全局对象。这时不污染全局环境的 babel-plugin-transform-runtime 才是最合适的。

以上内容出自：https://zhuanlan.zhihu.com/p/27777995

### 2.4 babel-preset-env

回到应用开发。通过自动识别代码引入 polyfill 来优化看来是不太靠谱的，那是不是就无从优化了呢？并不是。还记得 babel 推荐使用的 babel-preset-env 么？它可以根据指定目标环境判断需要做哪些编译。而在张克炎大神的建议下，babel-preset-env 也支持针对指定目标环境选择需要的 polyfill 了，只需引入 babel-polyfill，并在 babelrc 中声明 useBuiltIns，babel 会将引入的 babel-polyfill 自动替换为所需的 polyfill。
```
# .babelrc
{
  "presets": [
    ["env", {
      "targets": {
        "browsers": ["IE >= 9"]
      },
      "useBuiltIns": true
    }]
  ]
}
```
### 2.5 总结
解决上述问题有两种方案。
方案一：
```js
// npm i -D @babel/polyfill
//在你入口.js顶部将 polyfill 引入进来,确保它在任何其他代码/依赖声明之前被调用！
//es module ： import '@babel/polyfill';
require('@babel/polyfill');
```
方案二：
```
npm i -D @babel/plugin-transform-runtime
npm i -S @babel/runtime

# .babelrc
{
  "plugins": ["@babel/transform-runtime"]
}
```
babel-polyfill是直接在全局作用域里进行垫片，所以会污染全局作用域
babel同时提供了babel-plugin-transform-runtime这一插件，它的好处在于：
- 需要用到的垫片，会使用引用的方式引入，而不是直接替换，避免了垫片代码的重复
- 由于使用引用的方式引入，所以不会直接污染全局作用域。这就对于库和工具的开发带来了好处
但是babel-plugin-transform-runtime仍然不能单独作用。因为有一些静态方法，如"foobar".includes("foo")仍然需要引入babel-polyfill才能使用。

## 三 配置react

安装react：
```
npm i -S react react-dom
```

安装babel：和配置babel一样，注意版本，额外安装react相关语法插件即可
```
# webpack4 版本
npm i -D babel-preset-react@6
# webpack3 版本
npm i -D @babel/preset-react
```


配置webpack：和配置babel一样
```
{
	test: /(\.jsx|\.js)$/,
	use: {
		loader: "babel-loader"
	},
	exclude: /node_modules/
}
```

同样babel配置需要写在.babelrc中：
```
{
	"presets": ["@babel/env", "@babel/react"]	//老版为["env", "react"]
}
```
