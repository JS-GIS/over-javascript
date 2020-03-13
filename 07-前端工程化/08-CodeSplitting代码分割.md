## 一 CodeSplitting代码分割入门

### 1.1 手动代码分割

比如现在有一个业务文件：index.js
```js
import _ from "lodash";
console.log(_.join(['a','b','c'],"***"));
```

此时，打包后的结果是，lodash文件和当前的index.js都被打到了输出文件main.js内，一般理想化的情况是lodash本身被打包为一个js文件。此时需要做的步骤如下：
```js

// 在lib目录下创建lodash.js文件
import lodash from 'lodash';
window.lodash = lodash;
window._ = lodash;

//webpack配置
entry: {  
    "lib/lodash": path.resolve(__dirname, './src/lib/lodash.js'),                                                              
    main: path.resolve(__dirname, './src/index.js')
},

//main中使用
// import _ from "lodash";                      //此时已经不需要该步骤了
console.log(_.join(['a','b','c'],"***"));
```

### 1.2 使用webpack4代码分割工具

在第一章，其实还是做了人工代码分割，在webpack4中，内部自带了自动代码分割的插件。  

```js

//无需手动创建lodash.js文件了

//webpack配置：entry回到最初状态，添加optimization配置
entry: {                                                        
    main: path.resolve(__dirname, './src/index.js')
},
optimization: {
    splitChunks: {
        chunks: 'all'
    }
},

//main中使用
import _ from "lodash";                    
console.log(_.join(['a','b','c'],"***"));
```

打包后生成`vendors~main.js`，并被html文件引入。  


### 1.3 加载异步模块（懒加载） 

上述代码中，import是同步加载了一个文件，有些情况下会引用一些异步组件
```js
//一个异步js文件内的内容:其中注释内容代表打包后的文件名
function testAsync() {
    return import(/* webpackChunkName:"lodash" */'lodash').then(({defaylt: _}) => {
        console.log("test async");
    })
}
testAsync().then(()=>{
    console.log("then...);
});

//此时如果main引入了该文件，则该文件也会被自动打包，但是需要安装异步插件支持
// cnpm i -D @babel/plugin-syntax-dynamic-import

//还需要添加.babelrcl配置:
{
    presets:[...]
    plugins:["@babel/plugin-syntax-dynamic-import"]
}
```

使用异步组件的原因：在大多数场景中，只有触发了事件（点击事件），才会加载模块（lodash），这样能有效提升项目加载速度。这便是懒加载的概念。

## 二 常见配置

```js
splitChunks: {              //该值如果为{}，则走默认配置
    chunks: 'all',          //默认为async（只分割异步代码），其他值有all，initial（只分割同步代码）,其实只需要这一个配置即可，其他地方全部不需要
    minSize: 30000,         //引入模块大小大于30KB才做代码分割
    // maxSize: 50000,         //一般不配置
    minChunks: 1,           //至少引入了多少次才做分割
    maxAsyncRequests: 5,    //只打包前5个需要分割的文件（防止太多被分割造成请求多）
    maxInitialRequest: 3,   //入口文件被分割数量
    automaticNameDelimiter: '~',    //打包后文件名的连接符
    name: true,             //是否让cacheGroups内名字生效
    cacheGroups: {          //打包同步代码将会走这一步      
        vendors: {          //默认为false,指定打包后文件名是否有前缀
            test: /[\\/]node_modules[\\/]/, //引入文件位于node_modules时打包方式
            priority: -10,  //vendors和default打包优先级
            //filename: 'vendors.js'  //强行指定打包后文件名 
        },
        default: {          //默认打包配置，如果不符合vendors要求，如下打包
            priority: -20,
            reuseExistingChunk: true    //当前模块之前已经被打包，是否直接引用之前被打包的模块
            //filename: 'default.js'  //强行指定打包后文件名 
        }
    }
}

```

## 三  CSS代码分割

### 3.1 入口文件与引入文件打包的不同

```js
entry:{
    main: './src/index.js'
},
output: {
    filename: '[name],js',              // entry中的文件被打包后的名字main
    chunkFilename: '[name].chunk.js'    // 非入口，被入口文件引入的模块文件
}
```

### 3.2 CSS代码分割插件MiniCssExtractPlugin

当直接在js文件中使用import直接引入css文件时，webpack的`css-loader`将会采取`css in js`的方式打包，即不会生成该js文件。  

如果我们不希望webpack这样打包，而是仍然像传统的打包方式（html，css，js依次分开），


MiniCssExtractPlugin插件暂时不支持热更新，需要手动刷新浏览器，所以我们推荐将该插件配置在生产环境中，而不是开发环境.

```js
// 安装
cnpm i -D mini-css-extract-plugin

// 配置 webpack.config.prod.js
// 第一步:移除base配置中的css相关配置,并拷贝到dev环境中
// 第二步:配置生产环境中css
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
  module: {
    rules: [
      {
        test: /\.(css|scss)$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: "[name].css",            // 此时import方式引入的css走此步打包,并生成一个link加入html中,以前是直接以style形式追加
      chunkFilename: "[id].css"
    })
  ]
//此时打包生成了css:main.css
```

注意1:由于TreeShaking的存在,CSS文件往往不会被打包,需要配置package.jso如下:
```js
sideEffects: [
    "*.css",
    "*.scss"
]
```

注意2: index.js内引入了2个css文件,那么这2个CSS文件都会被打包到main.css文件中!!!!  

注意3: 打包的css需要压缩,需要使用插件:optimize-css-assets-webpack-plugin  

注意4: 如果不同的入口的css要打包到不同的css文件,额可以查看插件官网,配置cacheGroups

注意5: 在生产环境中,打包的文件名,即filename和chunkFilename应该这样书写:`[name].[contentHash].css`,即打包后的名字会带一个由内容产生的hash,内容不变(源码未改),则会产生缓存,内容改变后,该打包文件名也会改变,则重新请求.

注意6:旧版的webpack的hash值仍然会变化,需要添加配置
```
optimization: {
    runtimeChunk: {
        name: 'runtime'
    }
}
```