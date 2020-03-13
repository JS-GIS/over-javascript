## 一 querystring模块

该模块有2个主要方法：
- querystring.parse()：将查询字符串反序列化为一个对象，类似JSON.parse()
- querystring.stringify()：将一个对象序列化为一个字符串对象，类似JSON.stringify()

```JavaScript
const querstring = require('querystring');
let str1 = 'username=lisi&password=123';
let obj1 = querstring.parse(str1);    //转换为了对象
let obj2 = {username: 'zs', password: '456'};
let str2 = querstring.stringify(obj2);  //重新转换为字符串
```

用户提交的网址一般包含一定的数据，比如 username=lisi 这样的参数，我们可以通过url.query获取到这些字符串数据，Node提供了querystring对象来对这个字符串数据进行实例化处理：
```JavaScript
let querystring = require('querystring');
urlStr = url.parse(req.url);
console.log(querystring.parse(urlStr));
```