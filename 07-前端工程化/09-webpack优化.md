## 一 源码性能优化

### 1.0 代码分析

使用 http://webpack.github.com/analyse 可以进行代码分析。

使用步骤：
```
# 在package.json中添加
"dev-build": "webpack --profile --json > stats.json --config webpack.config.dev.js"

# 此时打包会在根目录生成  stats.json文件，将该文件传递到上述网址中即可
```

chrome亚也提供了代码覆盖率分析工具,位于谷歌开发者工具关闭按钮左边三个小点的more tools中，名字是 coverage。

### 1.1 异步加载

webpack推荐使用异步代码，所以在代码分割中，默认配置是：`async`。  

因为这里涉及代码利用率的问题，比如首屏加载，会加载很多js，有一部分是无用的，会降低代码的利用率。 

写法：
```js
document.addEventListener('click', ()=>{
    import('./click.js').then(()=>{
        
    }
})
```

贴士：缓存能够提升的性能非常少，但是异步加载提升空间非常大。

### 1.2 后台加载

当前台需要展示的部分已经加载完毕，我们还希望前端在空闲时间时，内部悄悄地加载异步组件，以供用户触发事件时立即展示，可以这样做：
```js
//异步组件中添加如下魔法
document.addEventListener('click', ()=>{
    import(/* webpackPrefetch: true */ './click.js').then(()=>{

    }
})
```

## 二 webpack打包性能优化