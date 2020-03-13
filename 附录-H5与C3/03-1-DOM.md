## 一 DOM


浏览器js主要构成：
- ECMAScript：js基础语法规范
- DOM：`Document Object Model`，JavaScript操作网页元素的API
- BOM：`Browser Object Model`，JavaScript操作浏览器部分功能的API

贴士：  

### 1.1 DOM解析过程与入口函数
HTML加载完毕，渲染引擎会在内存中把HTML文档，生成一个DOM树，getElementById是获取内中DOM上的元素节点。然后操作的时候修改的是该元素的属性。
DOM（document）即文档对象模型的一部分，DOM是一个复合的数据类型。

入口函数如下：
```javascript
window.onload = function(){ 

}
```
该函数解释：页面的结构、样式、节点等加载完毕后才去执行函数体。

在HTML当中一切都是节点，整个文档是一个文档节点，每一个HMTL标签都是一个元素节点，标签中的文字则是文字节点，标签的属性是属性节点。
### 1.2 获取节点
```javascript

document.getElementById("demo");            //通过 id 找到 HTML 元素
document.getElementsByTagName("div")[0];    //通过标签名找到 HTML 元素	获得的是标签伪数组
document.getElementsByClassName("a");       //通过类名找到 HTML 元素，在部分IE中无效

节点.parentNode             //获取父节点，一个节点只有一个父节点

节点.nextSibling            //获取下一个节点（IE678中指下一个元素节点，标准浏览器包括空文档和换行节点）	
节点.nextElementSibling     //获取下一个节点（不支持IE678）	

节点.previousSibling        //获取前一个节点（用法同nextSibling）
节点.previousElementSibling //获取前一个节点（用法通nextElementSibling）

父节点.firstChild           //获取第一个子节点（IE678中指获取第一个子元素节点），同理 lastChild
父节点.firstElementChild    //获取第一个子元素节点（不支持IE678）

节点.parentNode.children[index];    //随意得到兄弟节点

父节点.childNodes;          //标准属性，获取所有子节点数组，包括：文本、换行
父节点.children;            //非标准属性，但用的最多，只返回HTML节点
/* 
注意：虽然不是标准的DOM属性，但它和innerHTML方法一样，得到了几乎所有浏览器的支持。children在IE6/7/8中包含注释节点。
*/
```
### 1.3 获取节点属性
获取：getAttribute(名称)
设置：setAttribute(名称, 值)
删除：removeAttribute(名称)
注意：IE6、7不支持。
### 1.4 节点的增、删、复制
```javascript
//创建节点
document.write("节点与内容");     //会抹掉页面已加载的元素
对象.innerHTML = "节点及内容";
document.createElement();

//插入节点
父节点.appendChild(新节点);         //父节点的最后插入一个新节点
父节点.insertBefore(新节点,参考节点); //在参考节点前插入;

//删除节点
父节点.removeChild（子节点);      //必须指定要删除的子节点

//复制节点
新节点=要复制的节点.cloneNode(参数); //参数可选复制节点

```
### 1.5 节点类型
```
nodeType  ==  1  表示元素节点(标签)      
nodeType  ==  2  表示属性节点   
nodeType  ==  3  表示文本节点
```
## 二 事件
### 2.1 JS常见事件
JS是以事件驱动为核心的一门语言。
事件的三要素是：事件源、事件、事件驱动程序。
常见事件：
```
onclick     鼠标单击
ondbclick   鼠标双击
onmouseover 鼠标悬停
onmouseout  鼠标移除
onchange    文本内容或下拉菜单中的选项发生改变
onkeyup     按下并释放键盘上的一个健
onfocus     获得焦点
obblur      失去焦点
onload      网页文档加载事件
onunload    关闭网页时
onsubmit    表单提交事件
onreset     重置表单时
```
注意：
mouseover/mouseout事件，鼠标经过的时候会触发多次，每遇到一个子元素就会触发一次。
mouseenter/mouseleave事件，鼠标经过的时候只会触发一次。

### 2.2 事件对象
当一个事件发生时，事件的详细信息都会保存到一个对象中，这个事件对象即event对象。
ie：内置了event对象，默认值是null
chrome：内置了全局event对象，默认值是undefined
火狐：没有内置，只能通过事件函数第一个参数定义
兼容写法：
```javascript
function fn(ev) {
    var e = ev || event;
}
```
### 2.3 事件冒泡
事件冒泡：多个元素嵌套，都注册了相同的事件，那么如果里面的元素触发了事件，则外面的元素都会自动触发该事件。
即：当一个元素接收到事件的时候，会把接收到的所有事件传播给父级，一直到顶级window。
事件冒泡与元素的显示界面无关，只与元素之间的结构关系有关。
冒泡机制：通常我们给一个元素添加了事件，只是添加了事件处理函数。并不是没有添加事件，就不能接收该事件。同理，冒泡事件中，子级触发了一个点击事件，而父级没有写点击事件的处理函数，那么父级仍然是接收了该事件，只是不知道要如何处理而已。
阻止冒泡：
```
event.cancelBubble = true;  //IE事件函数没有event，所以不支持，需要上述兼容的e写法。
//同时兼容火狐IE写法：
window.event? window.event.cancelBubble = true : e.stopPropagation();
```
### 2.4 事件绑定
第一种绑定方式：
```
obj.onclick = fn1;
obj.onclick = fn2;  //fn2会覆盖fn1
```
第二种事件绑定：
```
IE: 没有捕获事件，事件名有on,绑定多事件执行顺序为正序，this为window
obj.attachEvent('onclick', fn1);   
标准:有捕获，事件名没有on，执行顺序倒序，this为触发该事件的对象,false：冒泡方式；true：捕获方式，默认冒泡
obj.addEventListener(‘click’, fn1, false);
```
由于IE下，事件函数内部的this指向的是window，这是不能容忍的，需要使用call()修改指向。
```javascript
function bindEvent(element,eName,fn) {
    if (element.addEventListener) {
        element.addEventListener(eName,fn,false);
    } else if (element.attachEvent) {
        element.attachEvent('on' + eName,function () {
            fn.call(element);
        });
    } else {
        element["on" + eName] = fn;
    }
}
```
### 2.5 事件取消
第一种绑定取消方式：节点.onclick = null;
第二种绑定取消方式：
- IE下：节点.detachEvent(事件名称,事件函数);
- 标准：节点.removeEventListener(事件名称,事件函数,是否捕获默认false); 
取消的兼容写法：
```javascript
function removeEvent(element,eName,fn) {
    if (element.removeEventListener) {
        element.removeEventListener(eName, fn, false );
    } else if (element.detachEvent) {
        element.detachEvent("on" + eName, fn);
    } else {
        element["on" + eName] = null;
    }
}
```
### 2.6 事件默认行为
当一个事件发生时，浏览器会默认做一些事情，如何阻止？
该行为是哪个事件触发，就在这个事件处理函数中使用 return false
但是retrun false只能阻止传统的obj.on事件名=fn形式触发的默认行为，如果事件是以addEventListner绑定，那么取消该行为需要使用：
event.preventDefault();