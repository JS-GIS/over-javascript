## 一 Events模块简介

Node的Events模块只定义了一个雷，即：EventEmitter，该类被Node本身以及第三方模块大量使用，是Node重要的类库之一。  

Node程序中的对象会产生一系列的事件，被称为事件触发器(event emiiter)，例如一个HTTPServer会在每次有新连接时触发一个事件。  


## 二 Events模块使用

```js
let EventEmitter = require("events");

let e = new EventEmitter();

//注册事件
e.on("test", function(){
    console.log("test");
});

console.log(e.eventNames())     //[ 'test' ]  查看注册了哪些事件

//触发事件
e.emit("test");
```

注意：
- 可以注册多个同名事件，触发都会触发
- 同名事件在eventNames数组中不会重复出现




