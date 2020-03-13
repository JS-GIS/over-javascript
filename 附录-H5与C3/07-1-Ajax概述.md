## 前言 传统的请求
- 同步请求
- iframe局部更新
- flash请求
- ajax异步请求
## 一 Ajax与异步编程
异步与同步：
```
异步：某段程序执行时不会阻塞其它程序执行，其表现形式为程序的执行顺序不依赖程序本身的书写顺序，相反则为同步。
同步：必须等待前面的任务完成，才能执行后面的任务；
现实生活中的一个例子：银行排队取款是同步，排队时玩手机是异步。
```
AJAX即 Asynchronous Javascript And XML（异步JavaScript和XML）。
当我们访问一个普通的网站时,当浏览器加载完:HTML,CSS,JS以后网站的内容就固定了.如果网站内容发生更改必须刷新页面才能够看到更新的内容。
异步更新的实际情况是：我们在访问新浪微博时,当你看到一大半了,会自动帮我们加载更多的微博,同时页面并没有刷新。
AJAX的作用就是：在浏览器中,我们也能够不刷新页面,通过ajax的方式去获取一些新的内容,类似网页有微博,朋友圈,邮箱等。
AJAX 不是一门的新的语言，而是对现有持术的综合利用，本质是在HTTP协议的基础上通过异步的方式与服务器进行通信。
## 二 Ajax案例
```javascript
// 1-创建Ajax实例 
let request;
if(XMLHttpRequest){     
    request = new XMLHttpRequest();
} else {    //兼容IE6
    request = new ActiveXObject("Microsoft.XMLHTTP");
}  

// 2-设置请求行：第三个参数可选，默认true异步
request.open('get', './test.php', true);    

// 3-如果是POST请求则需设置请求头
request.setRequestHeader('Content-Type','application/x-www-form-urlencoded');

// 4-设置请求体，仅用于POST请求，GET请求时可以不写send，或者参数为null
request.send(null);     //参数是字符串：'name=zs&age=10'

// 5-接受服务器端响应
request.onreadystatechange = function () {  

    console.log(request.readyState);

    //判断服务器是否正确响应
    if (request.readyState == 4 && request.status == 200) {
        console.log(request.responseText);
    }
}


```
ajax的五步步骤：
- 1 建立XMLHTTPRequest对象
- 2 注册回调函数：当服务器回应我们了,我们想要执行什么逻辑
- 3 使用open方法设置和服务器端交互信息：提交的网址,数据
- 4 设置发送的数据，开始和服务器端交互，发送数据
- 5 更新界面：在注册的回调函数中,获取返回的数据,更新界面
## 三 AjaxAPI
request.statusText; // 获取状态信息
request.getAllResponseHeaders(); //获取全部响应头
request.getResponseHeader(); 		//获取MIME类型响应头
request.responseText;			//获得字符串形式的响应数据
request.responseXML;			//获得XML形式的响应数据

每当readyState属性改变，就会调用onreadystatechange函数:
```
//readyState值
0:XMLHTTPRequest对象创建完成
1:发送请求准备完毕
2:已经完成发送
3:服务器已经返回数据
4:服务器返回的数据已经可以使用
```
## 四 Ajax请求相关问题
### 4.1 GET缓存问题
 如果每次访问同一个url，那么后面的访问都会从缓存中获取，如果后台数据变化了，用户无法获取到最新的信息。
 注意：POST请求没有缓存问题。
 解决办法：AJAX请求链接中加入时间戳
 ```javascript
 let url = './test.php';
 url += "?" + data.data + "&_t=" + new Date().getTime();
 ```
 ### 4.2 上传文件前端表单
 ```html
 <form action="file_big.php" method="post" enctype="multipart/form-data">
   <input type="file" name="upFile"  >
   <input type="submit" >
</form>
```
### 4.3 解析JSON
```
json字符串:     let jsonStr = ‘{“name”:”三国”,”category”:”文学”}’;
json对象:	    let jsonObj = JSON.parse(jsonStr);
json对象转字符串:let s = JSON.stringify(jsonObj);
```
JSON.parse()方法:将JSON字符串转化为JavaScript对象
JSON.stringify()方法:将JavaScript对象,转化为JSON字符串
注意：IE8以下没有JSON对象，使用第三方框架JSON2.js
## 五 Ajax封装
```javascript
let option = {
    url: "",
    method: "",
    params: {

    },
    error: null,
    success: null
};
function ajax(option){

    let xmlRequest;

    if (XMLHttpRequest) {

        xmlRequest = new XMLHttpRequest();

    }else{

        xmlRequest = new ActiveXObject("Microsoft.XMLHTTP");

    }

    if (option.method.toUpperCase() == "POST") {
        xmlRequest.setRequestHeader("Content-type","application/x-www-form-urlencoded");
    }

    // 格式化传入的数据为name=fox&age=18这样的格式
    let formatStr = "";
    for(let item in option.data){
        formatStr += item;  // 属性名
        formatStr += "=";   //等号
        formatStr+=option.data[item];//属性值
        formatStr+="&";//分隔符
    }
    // 去除最后一个&
    formatStr = formatStr.slice(0,-1);

    // open方法 如果是get方法,那么url之后需要添加数据
    if(option.method == "get"){
        option.url = option.url + "?" + formatStr;
        option.params = null;
    }

    xmlRequest.send(option.params);

    xmlRequest.onreadystatechange = function () {
        // 这步为判断服务器是否正确响应
        if (xhr.readyState == 4 && xhr.status == 200) {
            option.success(ajax获取的数据); 
        } 
    };
}
```
## 六 jQuery中的ajax
```javascript
$.ajax({
    type: "post",
    dataType: "html",
    url: '',
    data: dataurl,
    success: function (data) {

    }
});
```
