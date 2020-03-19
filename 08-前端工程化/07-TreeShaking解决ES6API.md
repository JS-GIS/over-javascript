## 一 TreeShaking简介

webpack从版本2开始引入了TreeShaking，即引入一个文件后，该文件内未被使用的方法，将不会被打包。  

TreeShaking只支持ESModule。  

development模式默认不开启Treeshaking，可以通过配置如下方式修改：
```js
//webpakc添加配置，在dev模式下也启用
optimization: {
    usedExports: true
}

//package.json修改以下配置，默认为false，treeshaking对所有模块生效
"sideEffects":["@babel/polly-fill"]   //防止import polyfill这样的文件不被打包

//一般这样配置：
"sideEffects":[
    "*.css"
]
```

注意：
- 开发环境下即使开启了treeshaking，所有源码依然会被打包，只是会提示**未被使用
- 生产环境下默认开启了treeshaking，是不需要配置optimization的，但是仍然需要配置package.json

