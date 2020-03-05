## 一 offset
#### 1.1 offset简介
offset用于获取元素的位置、尺寸。
主要属性包括：
```
offsetWidth     获取宽度（width + padding + boder），与他人无关
offsetHight     获取高度（height + padding + boder），与他人无关
offsetTop       距离带定位父盒子（都没有则以body为准）上边的位置，且从父亲的padding（没有body）开始算  
offsetLeft      同上
offsetParent    返回父级盒子中带有定位的父盒子节点
                如果当前元素的父级元素没有定位，offsetParent为body。
                如果当前元素的父级元素中有定位，offsetParent取最近的那个父级元素
```

```
注意：元素.style.height 是无法获取高度的，因为元素还会有内容撑开，但是元素使用内嵌式设置高度可以获取:
<div style=”height:50px;”>
```
#### 1.2 offset系列与style系列区别
offset系列可以返回没有定位盒子的距离上/左的位置（四舍五入取整），style.不可以（因为只有定位的盒子才有left、top之类的值）。
同样：style.left只能是行内样式的值才可以被获取。
offset系列 返回的是数字，而 style.返回的是字符串+单位px；
offset系列只可获取值，而 style.系列 还可以赋值；
如果没有给 HTML 元素指定过 类似top 样式，则类似style.top 返回的是空字符串。
## 二 事件对象属性page与client
#### 2.1 常见事件对象属性
![](/images/JavaScript/JavaScript-01.png)
```
pageY/pageX: 
    以文档的左上角为基准点，很类似绝对定位,IE6/7/8不支持。
screenY/screenX: 
    以电脑屏幕为基准点；鼠标位于屏幕的上方和左侧的距离。
clientX/clientY: 	
    以可视区为基准点，很类似固定定位。鼠标位于浏览器的左侧和顶部的距离。（浏览器大小和位置）
```
pageY和pageX的兼容写法:
```
在页面位置就等于 = 看得见的+看不见的
pageX = event.clientX + scroll().left
pageY = event.clientY + scroll().top
```
## 三 scroll
scrollWidth和scrollHeight：盒子内容的宽高
IE567可以比盒子小。 IE8+火狐谷歌不能比盒子小。
scrollTop：被卷去的头部，当滑动滚轮浏览网页的时候，网页隐藏在屏幕上方的距离；
scrollLeft：被隐藏的左边部分距离。
兼容问题：
```
一、未声明 DTD（谷歌只认识他）
        document.body.scrollTop
二、已经声明DTD（IE678只认识他）
   		document.documentElement.scrollTop
三、火狐/谷歌/ie9+以上支持的
   		window.pageYOffset
```
兼容写法：
```javascript
let aaa = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop || 0;

let aaa = document.documentElement.scrollTop + document.body.scrollTop;
```

