
二 第三方库引用
场景一：
通过npm安装jquery，或者通过远程地址引入的jquery，利用providePlugin插件：

安装jquery：npm install jquery -S
配置$:
new webpack.ProvidePlugin({
	$: 'jquery',
	jQuery: 'jquery'
})

在index.js中直接使用jquery：
$('#box').css("background-color","red");


场景二：
通过本地目录引入的方式，在上述配置中，当文件使用jquery时候会去node_modules中查找jquery模块，但是jquery是在本地，所以需要额外告知webpack去哪查找：
 
qjeury$带$的意思是解析到文件下，而不是解析到目录。

当然也可以使用import-loader 替代上述插件。
三 抽离与压缩
2.1抽离公共JS webpack4版
场景一：有个公用JS文件module.js同时被pageA.js  pageB.js引用，打包后，需要将该公共JS文件也被打包为一个文件。
	上述无需配置，是webpack4的默认行为，不过需要注意，默认配置只有多入口文件才可以打包公共代码。
    optimization: {
        splitChunks: {
            cacheGroups: {
                commons: {
                    name: "module",
                    chunks: "initial",
                    minChunks: 2        //出现多少次将会被打包
                }
            }
        }
    },

	minChunks:出现多少次会被打包，不打包则直接写入pageA.js这样的文件中。

场景二：

2.2 抽离公共JS webpack3版
场景一：
	
	一个js文件被多次引用，希望被被打包在一个公共的JS文件中；
	 
	name：打包后的额外生成的文件名
	minChunks：出现多少次将会被打包为额外的JS

	场景二：在一个js文件中引入了jquery，那么打包时，jquery也被打包进了bundle文件中，这不是我们想要的，我们需要的是第三方的包都放在类似libs这样的文件夹中。
首先在入口处配置：
entry: {
    	app :   path.resolve(__dirname,'src/js/app.js'),
    	vendors:['jquer']
},

	插件处配置为：
plugins: [
    		new webpack.optimize.CommonsChunkPlugin({
			name: 'vendors', 
			filename: 'vendors.js'
		}),
]

	这时候我们在dist文件中的index.html文件添加：
	<script src="vendors.js"></script>
执行命令：npm  run  publish

直接右键打开html页面就可以看到部署后的效果。vebdors.js就是被部署的入口文件。

注意：如果vendors.js引入顺序错误，会报错：
Uncaught ReferenceError: webpackJsonp is not defined。
因为我们需要调整index文件内JS的顺序如下：
<script src=”vendors.js”></script>
<script src=”bundle.js”></script>

场景三：如果我们希望jquery与自己的公共代码都被打包到同一个文件中：
 
那么打包后，lodash包以及开发者自己的公共JS都被打包到了vendor.bundle.js中



场景：如果项目中有 pageA.js pageB.js 二者同时引用了loadsh，且内部也有公共代码，我们不希望公共代码和lodash打包在一起，想要将公共代码打包为一个文件，lodash本身打包为一个文件：
 
 
注意：图上的后2个插件可以写为：
 
1.2 页面src引入文件的方式，改成用script标签嵌入的方式，减少http请求( 提高加载性能）每个页面加载的时候，都会去加载公共代码，这里值得优化。我们可以引入的script文件直接作为代码插入html中，则可以解决。
利用插件：html-webpack-inline-chunk-plugin
 
代码分割与异步加载

1.3 JS压缩
 
但是webpack4不需要上述第二行的插件配置
1.4 html压缩
 
Webpack4指定了生产环境后，无需上述配置
1.5 抽取CSS
 
加上css/前缀表示公共css放置位置
1.6 压缩抽取出来的css
 

