## 一 jQuery事件
#### 1.1 jQuery事件机制
jQuery的事件机制：指jQuery对JS操作DOM事件的封装：绑定、解绑、触发。
常见jQuery事件：
```
click(handler) 				单击事件
blur(handler) 				失去焦点事件
mouseenter(handler) 		鼠标进入事件
mouseleave(handler)		鼠标离开事件
dbclick(handler) 			双击事件
change(handler)          改变事件，如：文本框值改变，下来列表值改变等
focus(handler) 				获得焦点事件
keydown(handler) 			键盘按下事件
```
jQuery的事件绑定发展历程：
```
1 简单事件绑定：类似click()
2 bind绑定：1.7后被on取代
    $("p").bind("click mouseenter", function(e){});
3 delegate绑定：
    $(".parentBox").delegate("p", "click", function(){
        //为 .parentBox下面的所有的p标签绑定事件
    });
4 on方式
```
#### 1.2 on方式绑定事件
```javascript
//on绑定事件书写方式一
$('div').on('click mouseover',function(){
    alert(123);
});

//on绑定事件书写方式二
$('div').on({
    'click' : function(){
        alert(123);
    },
    'mouseover' : function(){
        alert(456);
    }
});

//注册委托事件：让子元素li执行事件
$('div').on('click','li',function(){
    alert(123);
});

//绑定一次性事件
$( "p" ).one( "click", function() {
alert( $( this ).text() );
});

```
如果一个元素被父级绑定了委托事件，自己也绑定了普通事件，优先执行委托事件。
注意：on还有一个可选参数data，在事件函数内，data的访问方式是：e.data
#### 1.3 jQuery移除事件绑定

bind与delegate事件的解绑：
```javascript

$(selector).unbind();           //解绑bind所有的事件
$(selector).unbind(“click”);    //解绑bing绑定的指定事件

$( selector ).undelegate();     //解绑所有的delegate事件
$( selector).undelegate( “click” ); //解绑所有的click事件

```
on事件解绑：
```javascript
$(selector).off();              // 解绑匹配元素的所有事件
$(selector).off("click");       // 解绑匹配元素的所有click事件
$(selector).off( "click", "**" );   // 解绑所有代理的click事件，元素本身的事件不会被解绑 
```
#### 1.4 jQuery事件对象
jQuery的事件对象ev已经是兼容的。常见属性：
```
ev.pageX----相对于文档
ev.clientX----相对于可视区
以上同理还有Y轴对应方法。
 
ev.which:keycode

event.data 					传递给事件处理程序的额外数据
event.currentTarget 			等同于this，当前DOM对象
event.pageX 					鼠标相对于文档左部边缘的位置
event.target 					触发事件源，不一定===this
event.stopPropagation()；	阻止事件冒泡
event.preventDefault(); 	阻止默认行为
event.type 					事件类型：click，dbclick…
event.which 					鼠标的按键类型：左1 中2 右3
event.keyCode				键盘按键代码
```
#### 1.5 事件触发
事件触发是指在某些场景下，让某个元素去执行事件。
```javascript
$(selector).click(); 						//简单事件触发：触发 click事件
$(selector).trigger("click");				//让元素触发click事件，和上述相同
$(selector).triggerHandler("click");	    //此方式不触发浏览器默认行为
```
#### 1.6 阻止冒泡与默认行为
```javascript
event.stopPropagation()		//阻止事件冒泡
event.preventDefault(); 		//阻止默认行为
// 如果：return false 则直接阻止全部
```
#### 1.7 节流阀
当类似onkeydown事件触发时，用户不停的按按按键，会反复触发，为了保证只触发一次，需要添加节流阀：
```javascript
//按下1-9这几个数字键，能触发对应的mouseenter事件
$(document).on("keydown", function (e) {
  if(flag) {
    flag = false;
    //获取到按下的键
    let code = e.keyCode;
    if(code >= 49 && code <= 57){
      //触发对应的li的mouseenter事件
      $(".nav li").eq(code - 49).mouseenter();
    }
  }
 
});

$(document).on("keyup", function (e) {
  flag = true;
  
  //获取到按下的键
  let code = e.keyCode;
  if(code >= 49 && code <= 57){
    //触发对应的li的mouseenter事件
    $(".nav li").eq(code - 49).mouseleave();
  }
});
```
## 二 $ 方法
$下的方法大多数为工具类方法，不仅可以给jQuery使用，也可以给原生JS使用。
```javascript
type()      //判断类型，比如时间对象返回Date，而typeof返回的都是Object。

trim()      // 去除空白

inArray()   //类似indexOf();
let arr = ['a','b','c','d'];
console.log( $.inArray('b',arr) );      //输出 1

proxy()     //改变this指向

    function show(n1,n2){
        console.log(n1);
        console.log(n2);
        console.log(this);
    }
    // show(3,4);      //输出 3 4 Window
    let showNew = $.proxy(show , document);  //也可以直接传入参数
    showNew(3,4);       //输出 3 4 #document

parseJSON()   //将字符串数据转换成json对象
    let str = '{"name":"hello"}';
    alert($.parseJSON( str ).name);

makeArray()     //将类数组转换成真正的数组
    let aDiv = document.getElementsByTagName('div'); //类数组
    $.makeArray(aDiv).push();

map()函数
    $.map(arry,function(object,index){})        //返回一个新的数组
    $("li").map(function(index, element){})     //注意参数的顺序是反的

    let newArr1 = $.map($("li"), function(i, e) {
        return $(e).text() + i;//每一项返回的结果组成新数组
    });

    let newArr2 = $("li").map(function(e, i){
        retrun i;
    });
```



