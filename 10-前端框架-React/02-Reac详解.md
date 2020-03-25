## 一 视图相关概念
### 1.1 常用名词解释
- props：属性，就是element上的attrs，换个名字property，变成复数，即props。从父组件传递进来的props属性，都可以被监听到！
- state：按钮有三态，Normal，Highlight，Selected，包括extjs，jquery里的大部分ui框架都是有状态的
- event：其实还应该算一个dom事件，下面的例子就把onChange的handler编译后的handleChange方法
### 1.2 props：父组件数据传递给直接子组件
从父组件传递属性到子组件：
```JavaScript
//父组件：<Hello name="lisi" />>
//子组件中获取name的方式
let { name } = this.props;
```
### 1.3 context:父组件数据传递给跨级子组件
props传递数据只能一个组件一个组件传递，context可以实现跨组件传递。
注意：使用context必须进行属性验证。
```JavaScript
//父组件中定义数据
class App extends Component {
    getChildContext(){
        return {
            msg: 'test'
        }
    }
    render(){
        return (
            <div><Father />></div>
        )
    }
}
App.childContextTypes = {
    msg: PropTypes.string
}
//我们可以跨过父组件Father，直接将数据传递给子组件Son
import React,{Component} from 'react';
import PropTypes from 'prop-types';
let contextTypes = {
    msg: PropTypes.string
}
export default class Son extends Component {
    render(){
        console.log(this.context) 
    }
}=
Son.contextTypes = contextTypes;
```
### 1.4 state
state在构造函数中定义，类似Angular中的$scope:
```JavaScript
import React, {Component} from 'react';

export default class Hello extends Component {
    constructor(props, context) {
        super(props, context);
        this.state = {name: "lisi", isShowLoading: true, movieDetailData: []}
    }

    render() {
        console.log(this.props);
        return (
            <div>
            <h1>我是Hello组,{this.state.name}</h1>
            </div>
        )
    }
}
```
如果要给state里的属性赋值，需要使用this.setState
### 1.5 this指向
在react绑定this指向推荐的做法是在构造函数中绑定：
```JavaScript
class App extends Component{

    constructor(props){
        super(props);
        this.changeView = this.changeView.bind(this);
    }
    changeView(view){
    }
}
```
## 二 事件处理
### 2.1 绑定事件
```JavaScript
import React, {Component} from 'react';
import {render} from 'react-dom';

export default class LinkButton extends Component {
    constructor(props) {
        super(props);
        this.state = {liked: false};
    }

    handleClick(e) {
        this.setState({liked: !this.state.liked});
    }

    render() {
        const text = this.state.liked ? 'like' : 'haven\'t liked';
        return (<p onClick={this.handleClick.bind(this)}> You {text} this. Click to toggle. </p>);
    }
}
```
### 2.2 参数传递
ES6写法：给事件处理函数传递额外参数的方式：bind(this, arg1, arg2, ...)
```
render: function() {
    return <p onClick={this.handleClick.bind(this, param1,param2,param3)}>;
},
handleClick: function(param1,param2,param3, event) {
    // handle click
}

```
## 三 DOM操作
###  3.1 获取组件
 ReactDOM.render 组件返回的是对组件的引用也就是组件实例（对于无状态状态组件来说返回 null），JSX 返回的不是组件实例，它只是一个 ReactElement 对象。
 当组件加载到页面上之后（mounted），你都可以通过 react-dom 提供的 findDOMNode() 方法拿到组件对应的 DOM 元素。（但是过于老旧，不推荐）
 ```
 //findDOMNode() 不能用在无状态组件上。
componentDidMound() {
  constel = findDOMNode(this);
}

 ```
推荐使用refs属性获取元素：
在要引用的 DOM 元素上面设置一个 ref 属性指定一个名称，然后通过 this.refs.name 来访问对应的 DOM 元素。
如果 ref 是设置在原生 HTML 元素上，它拿到的就是 DOM 元素，如果设置在自定义组件上，它拿到的就是组件实例，这时候就需要通过 findDOMNode 来拿到组件的 DOM 元素。
因为无状态组件没有实例，所以 ref 不能设置在无状态组件上，一般来说这没什么问题，因为无状态组件没有实例方法，不需要 ref 去拿实例调用相关的方法，但是如果想要拿无状态组件的 DOM 元素的时候，就需要用一个状态组件封装一层，然后通过 ref 和 findDOMNode 去获取。
注意事项：
- 你可以使用 ref 到的组件定义的任何公共方法，比如 this.refs.myTypeahead.reset()
- Refs 是访问到组件内部 DOM 节点唯一可靠的方法
- Refs 会自动销毁对子组件的引用（当子组件删除时）
- 不要在 render 或者 render 之前访问 refs
- 不要滥用 refs，比如只是用它来按照传统的方式操作界面 UI：找到 DOM -> 更新 DOM
## 四 react中的表单
### 4.1 状态属性
表单元素有这么几种属于状态的属性：
- value，对应 input 和 textarea 所有
- checked，对应类型为 checkbox 和 radio 的 input 所有
- selected，对应 option 所有
在 HTML 中 textarea 的值可以由子节点（文本）赋值，但是在 React 中，要用 value 来设置。
表单元素包含以上任意一种状态属性都支持 onChange 事件监听状态值的更改。
针对这些状态属性不同的处理策略，表单元素在 React 里面有两种表现形式。
### 4.2 受控组件
对于设置了上面提到的对应“状态属性“值的表单元素就是受控表单组件，比如：
```JavaScript
render: function() {
    return <input type="text" value="hello"/>;
}
```
一个受控的表单组件，它所有状态属性更改涉及 UI 的变更都由 React 来控制（状态属性绑定 UI）。比如上面代码里的 input 输入框，用户输入内容，用户输入的内容不会显示（输入框总是显示状态属性 value 的值 hello），这有点颠覆我们的认知了，所以说这是受控组件，不是原来默认的表单元素了。
如果你希望输入的内容反馈到输入框，就要用 onChange 事件改变状态属性 value 的值：
```JavaScript
getInitialState: function() {
    return {value: 'hello'};
},
handleChange: function(event) {
    this.setState({value: event.target.value});
},
render: function() {
    let value = this.state.value;
    return <input type="text" value={value} onChange={this.handleChange} />;
}
//使用这种模式非常容易实现类似对用户输入的验证，或者对用户交互做额外的处理，比如截断最多输入140个字符：
handleChange: function(event) {
    this.setState({value: event.target.value.substr(0, 140)});
}
```
### 4.3 非受控属性
和受控组件相对，如果表单元素没有设置自己的“状态属性”，或者属性值设置为 null，这时候就是非受控组件。
它的表现就符合普通的表单元素，正常响应用户的操作。
同样，你也可以绑定 onChange 事件处理交互。
如果你想要给“状态属性”设置默认值，就要用 React 提供的特殊属性 defaultValue，对于 checked 会有 defaultChecked，<option> 也是使用 defaultValue。
### 4.4 为什么要有受控组件
引入受控组件不是说它有什么好处，而是因为 React 的 UI 渲染机制，对于表单元素不得不引入这一特殊的处理方式。
在浏览器 DOM 里面是有区分 attribute 和 property 的。attribute 是在 HTML 里指定的属性，而每个 HTML 元素在 JS 对应是一个 DOM 节点对象，这个对象拥有的属性就是 property（可以在 console 里展开一个 DOM 节点对象看一下，HTML attributes 只是对应其中的一部分属性），attribute 对应的 property 会从 attribute 拿到初始值，有些会有相同的名称，但是有些名称会不一样，比如 attribute class 对应的 property 就是 className。（详细解释：.prop，.prop() vs .attr()）
回到 React 里的 <input> 输入框，当用户输入内容的时候，输入框的 value property 会改变，但是 value attribute 依然会是 HTML 上指定的值（attribute 要用 setAttribute 去更改）。
React 组件必须呈现这个组件的状态视图，这个视图 HTML 是由 render 生成，所以对于
```JavaScript
render: function() {
    return <input type="text" value="hello"/>;
}
```
在任意时刻，这个视图总是返回一个显示 hello 的输入框。
### 4.5 <select>的处理
在 HTML 中 <select> 标签指定选中项都是通过对应 <option> 的 selected 属性来做的，但是在 React 修改成统一使用 value。
所以没有一个 selected 的状态属性。
```JavaScript
<select value="B">
    <option value="A">Apple</option>
    <option value="B">Banana</option>
    <option value="C">Cranberry</option>
</select>
```
你可以通过传递一个数组指定多个选中项：<select multiple={true} value={['B', 'C']}>
## 五 循环插入子元素
如果组件中包含通过循环插入的子元素，为了保证重新渲染 UI 的时候能够正确显示这些子元素，每个元素都需要通过一个特殊的 key 属性指定一个唯一值。为了内部 diff 的效率。
```JavaScript
let TodoList = React.createClass({
    render: function () {
        let createItem = function (item) {
            return <li key={item.id}>{item.text}</li>;
        };
        return <ul>{this.props.items.map(createItem)}</ul>;
    }
});
module.export = TodoList
```
（1）当 React 校正带有 key 的子级时，它会确保它们被重新排序（而不是破坏）或者删除（而不是重用）。 务必 把 key 添加到子级数组里组件本身上，而不是每个子级内部最外层 HTML 上。
（2）也可以传递 object 来做有 key 的子级。object 的 key 会被当作每个组件的 key。但是一定要牢记 JavaScript 并不总是保证属性的顺序会被保留。实际情况下浏览器一般会保留属性的顺序，除了 使用 32位无符号数字做为 key 的属性。数字型属性会按大小排序并且排在其它属性前面。一旦发生这种情况，React 渲染组件的顺序就是混乱。可能在 key 前面加一个字符串前缀来避免：
```
render: function() {
    let items = {};

    this.props.results.forEach(function(result) {
      // 如果 result.id 看起来是一个数字（比如短哈希），那么
      // 对象字面量的顺序就得不到保证。这种情况下，需要添加前缀
      // 来确保 key 是字符串。
      items['result-' + result.id] = <li>{result.text}</li>;
    });

    return (
      <ol>
        {items}
      </ol>
    );
  }
```
组件标签里面包含的子元素会通过父元素的props.children 传递进来。
HTML 元素会作为 React 组件对象、JS 表达式结果是一个文字节点，都会存入 Parent 组件的 props.children。
props.children 通常是一个组件对象的数组，但是当只有一个子元素的时候，props.children 将是这个唯一的子元素，而不是数组了
```
let NotesList = React.createClass({
  render: function() {
    return (
      <ol>
      {
        React.Children.map(this.props.children, function (child) {
          return <li>{child}</li>;
        })
      }
      </ol>
    );
  }
});

ReactDOM.render(
  <NotesList>
    <span>hello</span>
    <span>world</span>
  </NotesList>,
  document.body
);
```

上面代码的 NoteList 组件有两个 span 子节点，它们都可以通过 this.props.children 读取。

这里需要注意， this.props.children 的值有三种可能：如果当前组件没有子节点，它就是 undefined ;如果有一个子节点，数据类型是 object ；如果有多个子节点，数据类型就是 array 。所以，处理 this.props.children 的时候要小心。
React 提供一个工具方法 React.Children 来处理 this.props.children 。我们可以用 React.Children.map 来遍历子节点，而不用担心 this.props.children 的数据类型是 undefined 还是 object。更多的 React.Children 的方法，请参考官方文档。












