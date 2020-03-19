## 一 表单标签

### 1.1 表单标签简介

表单用来收集信息，构成包含：表单域（form标签）与表单控件（input等），form 标签即代表包裹了一个表单域：
```Html
<form action="" method="">  <!--  action:用来处理表单数据 method:表单提交得方式 -->
</form>
```

### 1.2 表单控件

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

### 1.3 表单优化写法

表单元素的描述文字应该使用label标签包裹，并且使用for属性指向表单元素，从而达到点击描述文字可以聚焦文本框的效果：(注意下列的username是id)
```html
<label for="username">用户名</label>
<input type="text" id="username">
```