## 一 关于OAuth2.0授权简介
![](/images/JavaScript/node-02.png)
第一步：引导用户前往授权页面：https://www.test.com/oauth2/authorize?client_id=***&response_type=code&redirect_uri=***  
第二步：用户在授权时如果点击了同意授权，则跳转到该地址，且url中会添加参数?code=AUTHOTIZATION_CODE(用于获取access_token的code)  
第三步：利用拿到的code获取access_token。请求地址为：https://www.test.com/oauth2/access_token?client_id=**&client_secret=**&grant_type=authotization_code&redirect_uri=***&code=**
第四步：拿到access_token后在请求业务API时，添加参数?source=当前appkey&access_token=**
说明：  
https://www.test.com/oauth2/authorize:是授权服务器提供的授权页面  
client_id:是API提供方给当前应用分配的app_key  
response_type:返回的授权码类型，一般为code  
redirect_uri：授权成功后的回调地址  
一般通过code换取到的access_token结果为：
```
{
    "access_token":"ASDSFDSFSDF",
    "remind_in":3600,
    "expires_in":3600
}
```
access_token是本次授权所获得的授权码，在执行后续的API请求时需要同时提供此token以验证当前用户身份。  
remind_in和expires_in为此token的有效期信息，超过时间限制，需要重新向用户申请授权。  
## 二 接口开发
#### 2.1 所需接口与环境
```
OAuth2/authorize        引导用户授权token
OAuth2/access_token     获取被授权的access_token
OAuth2/get_token_info   授权信息查询接口
OAuth2/revokeoauth2     授权回收接口
```
环境：
```js
const path = require("path");

const express = require("express");
const ejs = require("ejs");

let app = express();
app.engine('html',ejs.__express);
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'html');

//测试接口
app.get("/test",(req, res)=>{
    return res.render("test");
});

//统一校验中间件
app.use((req, res, next)=>{
    //检查用户是否登录
    //检查参数是否正确：client_id,redirect_uri
    next();
});

app.listen(3000);
```
#### 2.2 OAuth2/authorize
```js
//引导用户授权接口
app.get("/OAuth2/authorize", (req, res)=>{

    return res.render("authorize");

});

//用户确认授权
app.post("/OAuth2/authorize", (req, res)=>{

    let uid = req.body.uid;
    let client_id = req.body.client_id;
    let redirect_uri = req.body.redirect_uri;
    let appkey = req.body.appkey;

    //加密生成一个code，这里省略直接书写一个假code
    let code = "3456";

    //将code,uid,client_id,appkey保存入数据库

    //跳转页面
    return res.redirect(redirect_uri + `?code=${code}`);

});
```
#### 2.3 换取access_token
```js
//换取access_token接口
app.post("/OAuth2/authorize", (req, res)=>{

    let client_id = req.body.client_id;
    let client_secret = req.body.client_secret;
    let redirect_uri = req.body.redirect_uri;
    let appkey = req.body.appkey;
    let code = req.body.code;

    //去数据库查询该code是否正确,并得到code对应的uid
    let uid = "lisi";
    
    //根据得到的uid和client_id生成access_token
    
    //换取access_token成功后删除原有的token

    //返回access_token
});

```
#### 2.4 access_token过期处理
由于用户量大后，经常需要处理access_token，我们可以在access_token后加入时间戳，先依据时间戳来判断是否过期，再查看校验具体的access_token。
#### 2.5 安全问题
大部分安全问题来自于app_sercret的泄露，可以通过重置该秘钥解决，如果用户的access_token泄露，则可以删除该token。  
OAuth2采用https认证，不存在中间人攻击。  
