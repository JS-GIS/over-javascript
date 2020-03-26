## 一 jQuery初步使用
### 1.1 jQuery简介
原生JS痛点：
- window.onload事件只能出现一次，多次出现会覆盖之前的事件
- 兼容性复杂
- 简单功能原生实现复杂（比如各种循环，jQuery隐式迭代帮我们做了）
jQuery的API都是方法，即要加小括号()，小传入的参数不同，功能不同。
```
版本一：1.x版本，兼容IE6/7/8
版本二：2.x版本，不兼容IE6/7/8
版本三：3.x版本，更精简，不再兼容低版IE
```
### 1.2 jQuery使用步骤
```javascript
<script src="./jQuery-1.11.3.min.js"></script> <!-- 引包 -->
<script>
    $(document).ready(function () { //入口函数
        //代码
    });
    //也可以简写为：
    $(function(){
        $('#div').click(function () {
            console.log(1)
        });
    });
</script>
```
注意：
- \$实际上表示的是一个函数，即jQuery函数：  jQuery ===$;
- jQuery事件不带on

### 1.3 jQuery入口函数与JS入口函数
JS的入口函数是：window.onload = function() { };
jQuery的入口函数是：$(function(){ });
区别：
- JS入口函数只能出现一次，出现多次会存在事件覆盖的问题，jQuery入口函数可以书写多次，没有覆盖问题。
- S入口函数是在所有的 文件资源 加载完成后才执行，jQuery入口函数在 文档 加载完后执行，即无需外部资源加载，DOM树加载完成就执行
### 1.4 jQuery对象和DOM对象
DOM对象：使用JS操作DOM返回的对象；
jQuery对象：使用jQuery操作DOM获得的对象，内部是一个对DOM对象进行包装后的伪数组
jQuery对DOM对象封装后，就不需要大量重复的遍历，且能更好的实现兼容问题。
DOM对象与jQuery对象转换：
```
DOM对象转换为jQuery对象方式：
$(DOM对象);
jQuery对象转换为DOM对象：
方式一：var btn = jQuery对象[0];		//推荐使用该方式
方式二：var btn = jQuery对象.get(0);

```
## 二 jQuery选择器
### 2.1 基本选择器
```
id选择器        $('#btn');
类选择器        $('.btn');
标签选择器      $('div');
选择器的交集    $(.div,.green')		选择class为div，或class为green的元素
选择器的并集    $('.div.green')		选择class为div，且class为green的元素
```
### 2.2 层级选择器
```
后代选择器（空格）：	$('#ul li')		选择id为ul的元素的所有后代li 
子代选择器（>）：		$('#ul > li')	选择id为ul的元素的直系后代li
```
### 2.3 过滤选择器
```
:eq(index)	选择匹配索引的元素		    $('li:eq(2)')选择索引号为2的li
:odd		选择匹配奇数索引元素		$('li:odd')
:even		选择匹配偶数索引元素		$('li:even')
```
### 2.4 筛选选择器
```
查找所有后代元素:find(selector)
$(“#j_wrap”).find(“li”).css(“color”, “red”);

查找直接子代元素:children()
$(“#j_wrap”).children(“ul”).css(“color”, “red”);

查找所有兄弟元素:siblings()	（不包括自己）
$(“#j_liItem”).siblings().css(“color”, “red”);

查找所有兄弟节点：
nextAll()：查找下一个所有兄弟节点
nextUntil()：作用同上，可以传入参数，查找到指定位置  
prevAll()：查找上一个所有的兄弟节点
prevUntil()：作用同上，可以传入参数，查找到指定位置 

查找父元素:parent()	
$(“#j_liItem”).parent(“ul”).css(“color”, “red”);    //选择id为j_liItem的父元素

所有祖先节点:parents()  //传入参数具备筛选功能（只有复合参数的祖先节点）

获取有定位的父级:offsetParent() 

查找指定元素的第index个元素:eq(index)	
$(“li”).eq(2).css(“color”, “red”);   //选择所有li元素中的第二个

slice(start,end):返回选择元素集合从第start-end位置的元素
```
## 三 jQuery的DOM操作
### 3.1 jQuery操作样式
```javascript
// 注意点：操作类样式的时候，所有的类名都不带点

//获取样式
$(selector).css(“font-size”);	
//设置样式:可以设置单个、多个										
$(selector).css({“color”: “red”, “font-size”: “30px”});	

//添加样式
$(selector).addClass(“liItem”);

//移除样式：无参表示移除被选中元素的所有类名
$(selector).removeClass(“liItem”);

//判断有没有类样式：返回true或false
$(selector).hasClass(“liItem”);

//切换类样式：该元素有类则移除，没有指定类则添加
$(selector).toggleClass(“liItem”);

```
### 3.2 节点操作
```javascript

//创建元素 $() 或者 节点.html()
let $spanNode = $("<span>我是一个span元素</span>");
let node = $("#box").html（"<li>我是li</li>"）；	

//添加子元素 append()
$(selector).append($node);				//追加传入jQuery对象
$(selector).append('<div></div>');		//直接传入html片段
appendTo(s)          //添加到s元素最后面，原生没有
prepend()           //添加到子元素最前面，类似原生的appendChild()

//添加兄弟元素
after()		  	    //添加到自己后面（作为兄弟）
before()		    //添加到自己前面（作为兄弟）

//获取到的元素剪切到某个位置
nsertBefore()  	    //原生JS这里是选中后复制到某个位置
insertAfter()		//原生JS没有insertAfter()

//html() val() text
html():	没有参数是获取包含标签的内容，有参数是插入内容,
        设置内容时，如果是html标记，与原生 innerHTML相同
text():	没有参数获取不包含标签的内容(字符串)，有参数是插入内容，
        设置内容时，类似原生innerText（火狐使用textContent获取），无视HTML标记插入纯文本，但是text()不存在兼容问题
val():  获取匹配元素的值，只匹配第一个元素，
        有参数时设置所有匹配到的元素的值

//删除与清空元素
$(selector).empty();        // 清空参数所有子元素，会清除事件，推荐使用
$(selector).html("");	    // 同上，但元素事件不会被清空,会出现内存泄露
$(selector).remove();	    // 删除元素与事件，包括自己，返回被删除的元素
$(selector).detach();	    // 同上，但是会保留事件

//复制元素 clone()
$(selector).clone();        //复制匹配的元素，返回值为复制的新元素
$(selector).clone(true);	//同时复制操作行为

```
### 3.3 属性操作
使用style获取的都采用css()方法设置，在标签里书写的样式，使用attr()获取。
```
attr():
$(selector).attr(“title”, “jQeury简介”);	//设置属性，设置多个传入对象
$(selector).attr(“title”);				//获取属性
$(selector).removeAttr(“title”); 		//移除属性

prop():
注意：checked、selected、disabled要使用.prop()方法。(1.7版本之前仍然可以使用attr)。
prop方法通常用来影响DOM元素的动态状态，而不是改变的HTML属性。例如：input和button的disabled特性，以及checkbox的checked特性。


```
### 3.4 快速操作表单-数据串联
```html
<form>
    <input type="text" name="a" value="1">
<input type="text" name="b" value="2">
<input type="text" name="c" value="3">
</form>
<script>
    $(function(){

        let str = $('form').serialize());
        let arr = $('form').serializeArray();

        /*
        str = a=1&b=2&c=3
        arr = [
            { name : 'a' , value : '1' },
            { name : 'b' , value : '2' },
            { name : 'c' , value : '3' }
        ]
        */
        
    });

```
### 3.5 操作尺寸位置
```javascript
//设置宽高 weight height
$(selector).height();       //获取高度
$(selector).height(200);	//设置高度						
//注意：使用css()获取的宽高是string类型，带px后缀，而height()是数字型

innerWidth()		//获取width+左右padding	
outerWidth()	    //获取width+左右padding+左右边框宽度
outerWidth(true)    //获取width+左右padding+左右边框宽度+左右margin
//注意：原生的outerWidth无法获取隐藏元素的值，而jQquery可以。所以jQuery中可以获取到隐藏元素的一些属性值。

//  offset():获取或设置元素相对于文档的位置
$(selector).offset();
$(selector).offset({left:100, top: 150});
// 注意点：设置offset后，如果元素没有定位(默认值：static)，则被修改为relative

// position() 获取相对于其最近的具有定位的父元素的位置,只能获取不能设置
$(selector).position();		// 返回值为对象：{left:num, top:num}

// scrollTop() :	获取或者设置元素垂直方向滚动的位置
$(selector).scrollTop(100);     //无参数表示获取偏移,有参数表示设置偏移

// scrollLeft() :	获取或者设置元素水平方向滚动的位置
$(selector).scrollLeft(100);

//总结：
$(“div”).offset();		    // 获取或设置坐标值，设置值后变成相对定位
$(“div”).position();		// 获取坐标值 子绝父相 	只能读取不能设置

$(“div”).scrollTop();		// 被卷曲的高度，即相对于滚动条顶部的偏移
$(“div”).scrolllLeft();	    // 被卷曲的宽度，即相对于滚动条左部的偏移
/*
垂直滚动条位置 是可滚动区域 在 可视区域上方的 被隐藏区域的高度。
如果滚动条在最上方没有滚动 或者 当前元素没有出现滚动条，那么这个距离为0
*/

```
### 3.6 filter has not
```javascript
$('div').filter('#div1').css('background','red');
$('div').has('span').css('background','green');

filter():	过滤
not()：		filter的反义词
has()：		has查看的是当前元素是否包含，filter过滤的是所有同级元素
```
### 3.7 常见案例 全选/反选
```js
$(selector).prop("checked", true);          //全选
$(selector).prop("checked", false);         //全不选
```
