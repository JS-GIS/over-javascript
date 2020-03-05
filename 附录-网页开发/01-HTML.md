## 一 网页基础

网页的标准：
- 结构：HTML，即 `Hyper text markup language` ，超文本标记语言，能够实现网页跳转的文本	
- 表现：CSS，即 `Cascading Style Sheets`，层叠样式表			
- 行为：由JavaScript编程语言控制

HTML标准由W3C组织制定，经过多年发展，其标准已经到达了HTML5，当然其兼容性仍然较差，市面上现有的HTML标准有：
- HTML4：被大多数浏览器所兼容，推荐使用
- XHTML：是HTML4与XML的过渡版本，不推荐使用
- XML：可以自定义标签，但是未被浏览器标准接受，被广泛应用于通信协议领域
- HTML5：具备划时代意义的HTML版本，已经被Chrome、FireFox等浏览器接受，但是大多数场合兼容度较低

标准HTML4网页结构：
```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
        "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <title></title>
</head>
<body>

</body>
</html>
```

标准HTML5网页结构：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

</body>
</html>
```

贴士：H5部分将会在03章讲解。

## 二 常见标签

#### 2.0 标签分类

网页的内容都是由大量的HTML标签构成，按照标签的开合可以将标签分为分类：
- 单标签：只有开始没有结束，如：`<hr>  <!-- -->  <br>  <hr>`
- 双标签：有开始有结束，如：`<p></p>   <em></em> `

#### 2.1 图片标签img与路径

图片标签img示例：
```html
<img src="1.jpg" atl="我是图片" title="图片标题" width="200" height="300">
```

img标签常用属性：
- alt：当图片无法显示时显示alt内的文字
- src：图片路径，可以是绝对路径（/开头）或者相对路径（./开头，可以省略./）

#### 2.2 a链接

a链接示例：
```html
<a href="http://www.baidu.com/">百度</a>
```

a标签常用属性：
- href：值为#时不能跳转，值为一个文件时则可以实现下载功能  
- title：鼠标划过时显示的文字
- target：网页打开方式，有两种值
  - "_self"：默认值，在当前页打开
  - "_blank"：在新窗口中打开

锚链接位置跳转演示：
```html
<p id="test">你好</p>
<a href="#test">查看问候语</a>
```

a链接特殊写法：可以制造点击不执行，一般在tab栏切换中使用
```html
<a href=”javascript:;”></a>  
<a href=”javascript:void(0);”></a> 
```

#### 2.3 特殊字符

Html中的特殊字符需要使用转义字符书写：  

| 特殊字符 | HTML中的表示 |
| ------ | ------ |
| 空格 | `&nbsp;` |
| < | `&lt;` |
| > | `&gt` |
| & | `&amp;` |

#### 2.4 meta标签

meta标签用来设置字符集、关键字、描述、重定向等：
```html
<meta charset="utf-8">
<meta name="keywords" content="流行资讯">
<meta name="description" content="最新服装设计流行信息">
<meta http-equiv="refresh"  content="2; url=1.html">        <!-- 2秒后跳转到1.html -->
```

#### 2.5 link标签

link标签必须放在head中，用来引入外部样式表和网页标题小图标：
```html
<link rel="stylesheet" href="1.css">
<link rel="icon" href="favicon.ico">
```

#### 2.6 背景音乐标签 embed

引入音乐文件：
```html
<embed src="1.mp3" hidden="true"></embed>
```

#### 2.7 滚动标签 marquee

```html
<marquee behavior="slide" direction="up" width="280" height="300" bgcolor="blue">
    <img src="1.jpg">
</marquee>
```

中间的内容可以是文字、图片，也可以是由程序生成的文字、图片，behavior可以设置滚动方式：
- slide：		一端滚到另一端不会重复
- scroll：		一端滚到另一端会重复
- alternate：	两端之间来回滚动
- direction：	设置滚动方向，值分别有left、right、up、down
- loop：		设置滚动次数，-1为一直滚动下去

#### 2.8 注释

```html
<!-- 这里书写注释 -->
```

#### 2.9 列表标签

无序列表 ul：
```html
<ul>新闻1</ul>
<ul>新闻2</ul>
<ul>新闻3</ul>
```

无序列表，默认无任何显示样式，但是可以添加一些样式：
- type="square"：列表样式为小方块
- type="circle"：列表样式为小圆圈

有序列表 ol:
```html
<ol type="a" start="3">
    <li>新闻1</li>
    <li>新闻2</li>
</ol>
```
有序列表的样式：
- A,a：分别以A或者a字幕顺序排序
- I,i ：分别以大小写罗马数字排列
- start="3" ：li前面的显示从第几个开始计数

自定义列表 dl：
```html
<dl>
    <dt>新闻汇总</dt>
    <dd>新闻1</dd>
    <dd>新闻2</dd>
</dl>
```

## 三 标签语义化

标签语义化概念：根据内容的结构化（内容语义化），选择合适的标签（代码语义化）。  

标签语义化可以让网页结构更加合理，且利于SEO。  

标签语义化方式：
- 尽可能少的使用无语义的标签div和span；
- 在语义不明显时，既可以使用div或者p时，尽量用p, 因为p在默认情况下有上下间距，对兼容特殊终端有利；
- 不要使用纯样式标签，如：b、font、u等，改用css设置。
- 需要强调的文本，可以包含在strong或者em标签中strong默认样式是加粗（不要用b），em是斜体（不用i）；

## 四 表格标签table

#### 4.1 table组成

```Html
<table border="1" width="300" height="100" cellspacing="0" cellpadding="5" align="center" bgcolor="pink">
    <tr>
        <th>姓名</th>
        <th>性别</th>
    </tr>
    <tr>
        <td>111</td>
        <td>122</td>
    </tr>
    <tr>
        <td>211</td>
        <td>222</td>
    </tr>
</table>
```

表格结构:thead（表头）、tbody（主体）、tfoot（结尾）	
- cellspacing：用来设置单元格与单元格的距离（td），默认值为2
- cellpadding：设置内容距边框的距离（文字距离td左侧）
- align：设置对齐方式，包括left、right、center

注意：
- table标签的标题标签是caption，不能使用title
- td内容垂直对齐使用valign
```Html
<!-- valign的其他值有 top middle bottom -->
<td valign="bottom">123</td>
```

#### 4.2 合并单元格

- 横向合并colspan：设置td的横向合并
- 纵向合并rowspan：设置td的纵向合并

```Html
<table width="300" height="200" cellspacing="0" border="1">
    <tr>
        <td colspan="2">111</td>
        <td>122</td>
        <td>133</td>
    </tr>
    <tr>
        <td>211</td>
        <td>222</td>
        <td rowspan="2">233</td>
    </tr>
    <tr>
        <td>311</td>
        <td>322</td>
        <td>333</td>
    </tr>
</table>
```

## 五 表单标签

#### 5.1 表单标签简介

表单用来收集信息，构成包含：表单域（form标签）与表单控件（input等），form 标签即代表包裹了一个表单域：
```Html
<form action="" method="">  <!--  action:用来处理表单数据 method:表单提交得方式 -->
</form>
```

#### 5.2 表单控件

只有位于表单域中的数据才能被提交，这些数据存放于input标签（表单控件）中。  

input标签的type可以指定控件的类型为文本、密码、单选框等多种类型，最常见的表单空间是文本输入框：
```html
<input type="text" maxlength="6" disabled="disabled">
<!-- 
    maxlength="6"    		限制输入字符长度
    readonly=”readonly”  	将输入框设置为只读状态（不能编辑）
    disabled="disabled"     输入框未激活状态
    name="username"   	    输入框的名称
    value="大前端"           为当前控件设置默认值，将输入框的内容传给处理文件 
    placeholder 			提示信息属性 
-->
```

常用表单控件：
```html
<!-- 密码框： -->
<input type="password">  <!-- 密码输入框的属性与文本输入框一致 -->

<!-- 单选框：表单控件一般都有name属性，单选框如果没有name，就不能实现单选，checked="checked"，表示默认选中-->
<input type="radio" name="a">男
<input type="radio" name="a">女

<!-- 下拉列表： 属性 multiple="multiple"可以实现多选-->
<select>
    <option>河北</option>
    <option selected="selected">河南</option>
</select>
<!-- 下拉还可以有更深的子级嵌套： -->
<select>
    <optgroup label="河南">
        <option>南阳</option>
        <option>洛阳</option>
    </optgroup>
</select>

<!-- 多选框 -->
<input type="checkbox"  checked="checked">喝酒  <!--checked代表默认选中-->

<!-- 多行文本域 cols  控制输入字符的长度，rows  控制输入的行数-->
<textarea ></textarea> 

<!-- 按钮 -->
<input type="submit">              <!--提交按钮，用来完成内容提交-->
<input type="button" vulue="登录"> <!--普通按钮，长与JavaScript配合使用-->

<!-- 重置按钮 -->
<input type="reset">  <!-- 该按钮将页面中的表单控件中的值恢复到默认值 -->

<!-- 图片提交按钮 -->
<input type="image" src="1.jpg">

<!-- 分组控件 -->
<fieldset>
    <legend> 用户注册信息</legend>
</fieldset>
```

#### 5.3 表单优化写法

表单元素的描述文字应该使用label标签包裹，并且使用for属性指向表单元素，从而达到点击描述文字可以聚焦文本框的效果：(注意下列的username是id)
```html
<label for="username">用户名</label>
<input type="text" id="username">
```