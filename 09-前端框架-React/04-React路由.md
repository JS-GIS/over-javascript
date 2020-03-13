## 一 React路由
### 1.1 路由依赖
React路由依赖包是： react-router-dom；经常使用的部分是：
```JavaScript
import {HashRouter, Route, Link} from 'react-router-dom';
```
HashRouter：	表示路由的根容器；
Route：		表示路由规则，拥有2个重要属性path，component
Link：			表示一个路由的链接
### 1.2 完整案例展示
注意：以下代码使用jsx，webpack中需要有对应的文件配置。
项目结构在helloworld基础上配置如下：
![](/images/JavaScript/react-02.png)

index.js:
```JavaScript
import React from 'react';
import ReactDOM from 'react-dom';

import App from './App.jsx';
ReactDOM.render(<App/>, document.getElementById('app'));
```

App.jsx:
```JavaScript
import React from 'react'
import {HashRouter, Route, Link} from 'react-router-dom';

import Home from './components/home.jsx';
import Content from './components/content.jsx';
import About from './components/about.jsx';
/*
* <HashRouter>在一个项目中最好只使用一次，启用后，网址中会出现 #
*/
export default class App extends React.Component {
	constructor(props){
		super(props)
		this.state = {
		}
	}
	render(){
		return <HashRouter>
			<div>
				<nav>
					<Link to="/home">首页</Link>&nbsp;&nbsp;
					<Link to="/content">内容</Link>&nbsp;&nbsp;
					<Link to="/about">关于</Link>
				</nav>
				<Route path="/home" component={Home}></Route>
				<Route path="/content" component={Content}></Route>
				<Route path="/about" component={About}></Route>
			</div>
		</HashRouter>
	}
}

```
### 1.3 路由参数传递
默认情况下，路由中的规则，是模糊匹配的，如果路由可以部分匹配成功，就会展示这个路由对应的组件，
如果要匹配参数，可以在匹配规则中，使用：修饰符，表示这个位置匹配到的是参数。
```
<Route path="/movie/:type/:id component={movie} exact></Route>
```
在组件中获取参数的方式：this.props.match.params，如果觉得这样写过长，可以如下封装：
```
export default class Movie extends React.Component {
    constructor(props){
        super(props)
        this.state = {
            routeParams: props.match.params
        }
    }
}
```