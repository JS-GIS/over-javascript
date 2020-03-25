## 一 生命周期
### 1.1 生命周期概念
组件的创建、加载、销毁等过程，总是伴随着各种各样的事件，这些特定的时期，以及触发的事件，被称为生命周期。
组件的生命周期分为三部分：
```
	创建：创建阶段的生命周期函数只会执行一次；
	运行：运行阶段会根据组件的state和props的改变，有选择的触发一些函数；
	销毁：销毁阶段的生命周期函数只会执行一次。
```
### 1.2 创建阶段
第一步： static defaultProps = {}
初始化默认的props属性值，可以防止组件中某些必须的属性在外界没有传递时报错。
第二步： this.state = {}
初始化state值，即初始化组件的私有数据，该数据定义在构造函数中，所以该数据会在第一时间初始化
第三步： componentWillMount函数
这是组件即将被挂载函数。
执行时间：组件的虚拟DOM元素，将要挂载到页面上的时候执行。此时虚拟DOM还没有被创建，也没有被挂载到页面上，因为虚拟DOM是在render函数中创建的，所以此时根本还没有创建虚拟DOM。
第四步： render函数
渲染虚拟DOM的函数，当进入到render函数中执行时，已经开始渲染虚拟DOM，render执行完，虚拟DOM在内存中创建好了，但是，此时虚拟DOM没有挂载到真正的页面上。
第五步： componentDidMount函数
表示组件已经完成了挂载，state上的数据、内存中的虚拟DOM、浏览器的页面 都已经保持了一致。组件进入到了运行阶段。
### 1.3 运行阶段
触发更新：
属性props的改变，状态state的改变都可以触发组件的更新。
![](/images/JavaScript/react-03.png)
对应函数：
```
componentWillReceiveProps:	
    组件将要接收新属性，此时，只要这个方法被触发，就证明父组件给当前子组件传递了新的属性值
shouldComponentUpdate：	
    组件是否需要被更新，此时，组件尚未被更新，但是 state和props 肯定是最新的
componentWillUpdate：		
    组件将要被更新，此时，尚未更新，内存中的虚拟DOM树还是旧的
render：						
    重新根据最新的state和props重新渲染DOM树于内存中，render调用完毕后，内存中旧DOM树被新的								DOM树替换了，此时页面还是旧的
componentDidUpdate：		
    此时页面又被重新渲染了，state、虚拟DOM、页面保持了同步

```
### 1.4 销毁阶段
对应函数：
```
componentWillUnmount：		组件将要被销毁，但是还能正常使用。

```
### 1.5 使用
封装组件时，组件内部，有些数据是必须的，即使用户没有传递一些相关的启动参数，这时，组件内部尽量自己提供一个默认值：
```JavaScript
//react中使用静态的defaultProps属性，来设置组件的默认属性值：
static defaultProps = {
    initcount: 0
}
```
如果用户传递了数据，但是数据的类型不是自己期望的，那么在封装组件时，需要进行类型的校验。propType静态对象内部用来书写类型校验，但是需要安装react提供的类型校验包：prop-types。（注意：v15之前的版本无需安装）
```JavaScript
import ReactTypes from 'prop-types';
static propTypes = {
    initcount: ReactTypes.number
}
```
