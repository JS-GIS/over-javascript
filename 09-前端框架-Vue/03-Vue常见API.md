## 一 常见API

#### 1.2 过滤器
过滤器可以用在两个地方：mustache插值和v-bind表达式；过滤器应该被添加在js表达式的尾部，由“管道符”指示;

语法：
```js
  Vue.filter("过滤器的名称",function(data,arg){
    // 对传过来的数据进行处理；
  })
```
+ data 是由过滤器 管道符前面(如下面案列中的msg) 传递过来的数据 ;
+ arg 这个是参数(可以传多个)，可在过滤器调用的时候灵活赋值，如：
  ```html
  <div id="app">
    {{count | myFormat(3,1) }} 
  </div>

  <script>
    Vue.filter("myFormat", function (count,num1，num2) {
        return count + num1 - num2;
    });

    new Vue({
        el: "#app",
        data: {
            count: 2
        }
    });
  </script>
    
  ```

过滤器可多次调用：{{count | myFormat(3,1) | test }} 

多个过滤器调用时：先把最原始的值 count 交给第一个过滤器 myFormat 来处理，处理的结果再交给第二个过滤 test 器去处理...，直到最后一个过滤器调用完毕，把最终的结果放在{{}}里
```HTML
 <div id="app">
    {{count | myFormat(3,1) | test }} 
  </div>

  <script>

    Vue.filter("myFormat", function (count,num1，num2) {
        return count + num1 - num2;
    });
    Vue.filter("test", function (count) {
        return count + 9;
    });

    new Vue({
        el: "#app",
        data: {
            count: 2
        }
    });

  </script>

```

全局过滤器：
```js
    <div id="app">
        {{msg | myFormat }}     <!-- msg 经过 myFormat处理后的结果 -->
    </div>

    Vue.filter("myFormat", function (msg) {
        return msg.replace("h","x");
    });

    new Vue({
        el: "#app",
        data: {
            msg: "helloworld"
        }
    });
```

私有过滤器：
```js
new Vue({
    filters:{
        myFormat: funtion(){}
    }
});
```

如果出现了同名过滤器，将会依据就近原则调用（私有为准）

#### 1.2 按键修饰符
+ 按键别名

    .enter \ .tab \ .delete \ .esc \ .space \ .up \ .down \ .left \ .right

```js
<input type="text" @keyup.enter="add">  
```

+ 自定义按键修饰符

113键（F2）按下去的时候执行add方法
```js
<input type="text" @keyup.113="add">  
```

但是键盘码不方便记忆，可以自定义：
```js
<input type="text" @keyup.f2="add"> 


Vue.config.keyCode.f2 = 250; // 自定义按键修饰符
```

js里键盘事件对应值：https://www.cnblogs.com/wuhua1/p/6686237.html

#### 1.3 自定义指令

定义一个全局指令：v-focus，用于获取焦点

    使用directive()定义全局的指令，

    其中：
    
    参数1---指令的名称，（注意，在定义的时候，指令的名称前面不需要添加 v- 前缀；但是在调用的时候必须在指令前面加上 v- 前缀）；

    参数2：是一个对象，这个对象身上有一些指令相关的函数，这些函数可以在特定的阶段执行相关的操作；


```js
<input v-focus>         
Vue.directive('focus', {
    bind: function(el){           //每当指令绑定到元素上的时候，会立即执行bind函数，且只执行一次，一般绑定样式相关操作
        // el.focus()                // 此时元素还未插入，focus没有作用
    },         
    inserted: function(el){      // 元素插入DOM时，会执行该函数，只会触发一次
        el.focus();
    },    
    updated: function(el){}       // 当VNode更新时，执行，可能会触发多次
});
```

绑定值：
```js
<input v-size="'30px'">         
Vue.directive('focus', {
    bind: function(el,binding){           //每当指令绑定到元素上的时候，会立即执行bind函数，且只执行一次，一般绑定样式相关操作
        // el.focus()                // 此时元素还未插入，focus没有作用
        console.log(binding.value)      //30px
    },         
    inserted: function(el){      // 元素插入DOM时，会执行该函数，只会触发一次
        el.focus();
    },    
    updated: function(el){}       // 当VNode更新时，执行，可能会触发多次
});
```

也可以定义私有指令：
```js
new Vue({
    directives: {
        'focus': {}
    }
});
```

也可以直接简写
```js
directives: {
    'focus': function(el, bingding){}       // 相当于同时绑定到了bind和inserted
}
```

## 二 常见第三方包


## [vue-resource 实现 get, post, jsonp请求](https://github.com/pagekit/vue-resource)
除了 vue-resource 之外，还可以使用 `axios` 的第三方包实现实现数据的请求
1. 之前的学习中，如何发起数据请求？
2. 常见的数据请求类型？  get  post jsonp
3. 测试的URL请求资源地址：
 + get请求地址： http://vue.studyit.io/api/getlunbo
 + post请求地址：http://vue.studyit.io/api/post
 + jsonp请求地址：http://vue.studyit.io/api/jsonp
4. JSONP的实现原理
 + 由于浏览器的安全性限制，不允许AJAX访问 协议不同、域名不同、端口号不同的 数据接口，浏览器认为这种访问不安全；
 + 可以通过动态创建script标签的形式，把script标签的src属性，指向数据接口的地址，因为script标签不存在跨域限制，这种数据获取方式，称作JSONP（注意：根据JSONP的实现原理，知晓，JSONP只支持Get请求）；
 + 具体实现过程：
 	- 先在客户端定义一个回调方法，预定义对数据的操作；
 	- 再把这个回调方法的名称，通过URL传参的形式，提交到服务器的数据接口；
 	- 服务器数据接口组织好要发送给客户端的数据，再拿着客户端传递过来的回调方法名称，拼接出一个调用这个方法的字符串，发送给客户端去解析执行；
 	- 客户端拿到服务器返回的字符串之后，当作Script脚本去解析执行，这样就能够拿到JSONP的数据了；
 + 带大家通过 Node.js ，来手动实现一个JSONP的请求例子；
 ```
    const http = require('http');
    // 导入解析 URL 地址的核心模块
    const urlModule = require('url');

    const server = http.createServer();
    // 监听 服务器的 request 请求事件，处理每个请求
    server.on('request', (req, res) => {
      const url = req.url;

      // 解析客户端请求的URL地址
      let info = urlModule.parse(url, true);

      // 如果请求的 URL 地址是 /getjsonp ，则表示要获取JSONP类型的数据
      if (info.pathname === '/getjsonp') {
        // 获取客户端指定的回调函数的名称
        let cbName = info.query.callback;
        // 手动拼接要返回给客户端的数据对象
        let data = {
          name: 'zs',
          age: 22,
          gender: '男',
          hobby: ['吃饭', '睡觉', '运动']
        }
        // 拼接出一个方法的调用，在调用这个方法的时候，把要发送给客户端的数据，序列化为字符串，作为参数传递给这个调用的方法：
        let result = `${cbName}(${JSON.stringify(data)})`;
        // 将拼接好的方法的调用，返回给客户端去解析执行
        res.end(result);
      } else {
        res.end('404');
      }
    });

    server.listen(3000, () => {
      console.log('server running at http://127.0.0.1:3000');
    });
 ```
5. vue-resource 的配置步骤：
 + 直接在页面中，通过`script`标签，引入 `vue-resource` 的脚本文件；
 + 注意：引用的先后顺序是：先引用 `Vue` 的脚本文件，再引用 `vue-resource` 的脚本文件；
6. 发送get请求：
```
getInfo() { // get 方式获取数据
  this.$http.get('http://127.0.0.1:8899/api/getlunbo').then(res => {
    console.log(res.body);
  })
}
```
7. 发送post请求：
```
postInfo() {
  let url = 'http://127.0.0.1:8899/api/post';
  // post 方法接收三个参数：
  // 参数1： 要请求的URL地址
  // 参数2： 要发送的数据对象
  // 参数3： 指定post提交的编码类型为 application/x-www-form-urlencoded
  this.$http.post(url, { name: 'zs' }, { emulateJSON: true }).then(res => {
    console.log(res.body);
  });
}
```
8. 发送JSONP请求获取数据：
```
jsonpInfo() { // JSONP形式从服务器获取数据
  let url = 'http://127.0.0.1:8899/api/jsonp';
  this.$http.jsonp(url).then(res => {
    console.log(res.body);
  });
}
```