## 一 网页基础

### 1.1 网页标准

网页的标准：
- 结构：HTML，即 `Hyper text markup language` ，超文本标记语言，能够实现网页跳转的文本	
- 表现：CSS，即 `Cascading Style Sheets`，层叠样式表			
- 行为：由JavaScript编程语言控制

HTML标准由W3C组织制定，经过多年发展，其标准已经到达了HTML5，当然其兼容性仍然较差，市面上现有的HTML标准有：
- HTML4：被大多数浏览器所兼容，推荐使用
- XHTML：是HTML4与XML的过渡版本，不推荐使用
- XML：可以自定义标签，但是未被浏览器标准接受，被广泛应用于通信协议领域
- HTML5：具备划时代意义的HTML版本，已经被Chrome、FireFox等浏览器接受，但是大多数场合兼容度较低

### 1.2 标准HTML4网页结构
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

### 1.3 标准HTML5网页结构
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

## 二 常见标签

### 2.0 标签分类

网页的内容都是由大量的HTML标签构成，按照标签的开合可以将标签分为分类：
- 单标签：只有开始没有结束，如：`<hr>  <!-- -->  <br>  <hr>`
- 双标签：有开始有结束，如：`<p></p>   <em></em> `

### 2.1 图片标签img与路径

图片标签img示例：
```html
<img src="1.jpg" atl="我是图片" title="图片标题" width="200" height="300">
```

img标签常用属性：
- alt：当图片无法显示时显示alt内的文字
- src：图片路径，可以是绝对路径（/开头）或者相对路径（./开头，可以省略./）

### 2.2 a链接

a链接示例：
```html
<a href="http://www.baidu.com/">百度</a>
```

a标签常用属性：
- href：值为#时不能跳转，值为一个文件时则可以实现下载功能  
- title：鼠标划过时显示的文字
- target：网页打开方式，常见打开方式有
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

### 2.3 特殊字符

Html中的特殊字符需要使用转义字符书写：  

| 特殊字符 | HTML中的表示 |
| ------ | ------ |
| 空格 | `&nbsp;` |
| < | `&lt;` |
| > | `&gt` |
| & | `&amp;` |

### 2.4 meta标签

meta标签用来设置字符集、关键字、描述、重定向等：
```html
<meta charset="utf-8">
<meta name="keywords" content="流行资讯">
<meta name="description" content="最新服装设计流行信息">
<meta http-equiv="refresh"  content="2; url=1.html">        <!-- 2秒后跳转到1.html -->
```

### 2.5 link标签

link标签必须放在head中，用来引入外部样式表和网页标题小图标：
```html
<link rel="stylesheet" href="1.css">
<link rel="icon" href="favicon.ico">
```

### 2.6 背景音乐标签 embed

引入音乐文件：
```html
<embed src="1.mp3" hidden="true"></embed>
```

### 2.7 滚动标签 marquee

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

### 2.8 注释

```html
<!-- 这里书写注释 -->
```

### 2.9 列表标签

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

### 4.1 table组成

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

### 4.2 合并单元格

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