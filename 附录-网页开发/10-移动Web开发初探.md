## 一 移动Web开发的问题
#### 1.1 视口
设计师在设计网页时，一般会按照固定的宽度设计，如PC端是1000px或者1200px，移动端是640px或者750px，然而这样的网页在移动设备上浏览时，局限于移动设备屏幕大小，会显示不完整，设备的宽度远远不够，以iPhone6为例，视口宽度是375像素。  
为了弥补这个问题，移动设备的浏览器会把视口放大，一般是980px或者1024px，但是这样浏览器就会出现滚动条。因为设备的实际可视区要比浏览器自设的宽度小很多。  
为了解决上述问题，引入了视口属性viewport
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0,maximum-scale=1.0,user-scaleable=0">
```
上述代码的作用是让当前视口的宽度等于设备的宽度，同时不允许用户手动缩放。
#### 1.2 媒体查询
CSS3引入了一个新的操作表达式，称为媒体查询，允许开发者基于设备的不同特性来应用不同的样式申明。  
其中，通过对视口宽度的判断，对网页输出不同的展示效果，比如在iPhone7下默认字号为12px，而在iPhone7Plus下默认为14px，如果是iPad，则采用多列布局。


## 移动开发常用css
```
/*=======reset css========*/
*,
*::before,
*::after{
    /*所有的标签，和伪元素都选中*/
    margin: 0;
    padding: 0;
    /*移动端常用布局是非固定像素*/
    box-sizing: border-box;
    -webkit-box-sizing: border-box;
    /*点击高亮效果的清除*/
    tap-highlight-color: transparent;
    -webkit-tap-highlight-color: transparent;
}
body{
    font-size: 14px;
    font-family: "Microsoft YaHei",sans-serif;
    color: #333;
}
ul,ol{
    list-style: none;
}
a{
    text-decoration: none;
    color: #333;
}
input,textarea{
    border: none;
    outline: none;
    /*不允许改变尺寸*/
    resize: none;
    /*元素的外观  none没有任何样式*/
    -webkit-appearance: none;
}
/*=======common css========*/
.f_left{
    float: left;
}
.f_right{
    float: right;
}
.clearFix::before,
.clearFix::after{
    content: "";
    display: block;
    visibility: hidden;
    height: 0;
    line-height: 0;
    clear: both;
}
.m_l10{
    margin-left:10px;
}
.m_r10{
    margin-right:10px;
}
.m_t10{
    margin-top:10px;
}
.m_b10{
    margin-bottom:10px;
}
/*使用精灵图的公用样式*/
[class^="icon_"],[class*=" icon_"]{
    background-repeat: no-repeat;
    background-image: url("../images/sprites.png");
    background-size: 200px 200px;
}
```