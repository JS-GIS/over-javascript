## 一 闭包
#### 1.1 立即执行函数
JS中的沙箱模式基本模型：
```javascript
(function(){

})();
```
上述基本模型，也称为立即执行函数IIFE，该模式不会向外界暴露任何全局变量，形成了一个封闭的空间。
如果直接使用上述沙箱模式，那么类似jQuery这样的库就无法向外提供可调用的API了，我们可以考虑将想被外界使用的属性和方法加到window全局对象上去。
但是window全局对象不可以直接引用，这样破坏了沙箱原则，所以jQuery中，使用传参形式将window对象传入沙箱内，此时就不需要使用全局的window对象，而是沙箱内部定义的形参。
```javascript
(function(win){
    var Obj= {
        getEle:function () {
        }
    }
    win.Obj= win.$ = Obj;
})(window)
```
注意：参数如下理解：
```js
(function(形参){

})(实参)
```
沙箱模式主要用于书写框架、插件等，主要的原理是：利用函数构建独立作用域。
#### 1.2 闭包使用
闭包其实就是函数，闭包的原理就是作用域访问原则：上级作用域无法访问下级作用域中的变量  
要解决的问题是：让函数外部的成员能够间接访问函数内部的成员  
闭包：可以通过闭包返回的函数来实现对函数内部数据操作
```js

 //实现修改数据
 function foo() {
    var num = 123;
    return function (a) {
            num = a;
            console.log(num);
     }
 }
 foo()(456);         //这样就修改了函数内部的数据
 ```
 ```js
 //使用对象设置、获取多个数据

function foo(){
    var name = "zs";
    var age = 18;
    return{
        setName:function (val) {
            name = val;
            return name;
        },
        getName:function () {
            return name;
        },
        getAge:function () {
            return age;
        }
    }
}

var obj = foo();
obj.getAge();
```
## 二 函数调用 apply call bind
```javascript
    var name = "Lisi";

    function sayHi() {
        console.log(this.name);
    }

    var obj = {
        name: "zs"
    }

    sayHi();            //输出Lisi
    sayHi.apply(obj);   //输出zs，修改了this指向
    sayHi.call(obj);    //输出zs，修改了this指向
```
apply与call的使用：
```
函数名.apply(绑定对象,函数参数列表数组);
函数名.call(绑定对象,函数参数1,参数2,参数3....);
```
案例：
```javascript
    var name = "Lisi";

    function sayHi(a, b) {
        console.log(this.name + "拥有" + (a*b) + "个项目");
    }

    var obj = {
        name: "zs"
    }

    sayHi(3,4);                 //输出 Lisi拥有12个项目
    sayHi.apply(obj,[3,4]);     //输出 zs拥有12个项目
    sayHi.call(obj,3,4);        //输出 zs拥有12个项目
```
apply和call使用场景：
- apply用于函数的形参个数不确定的情况；
- call用于确定了函数的形参有多少个的时候使用；
- apply和call的第一个参数都为null时，表示为函数调用模式，即this指向window
使用案例一：求数组最大值
```javascript
var arr = [9,1,4,10,7];
var max1 = Math.max(9,1,4,10,7);
var max2 = Math.max.apply(null,arr);
console.log(max1);      //输出10
console.log(max2);      //输出10
```
使用案例二：伪数组
```javascript
//obj是个伪数组，无法使用obj.0获取属性，也无法像数组那样用obj[0]获取
var obj = {
    0: "a",
    1: "b",
    2: "c",
    length: 3
};
// [].concat(1,2,3)会产生数组[1,2,3]
var arr = [].concat.apply([], obj);
console.log(arr);                       //输出['a','b','c']
```
bind：bind函数的返回值是函数本身
```js
function fn(x){
    console.log(x + '---' + this);
}

var f1 = fn.bind(); //bind是复制的意思，参数可以在此时传入，也可以在复制后调用时传入
f1(3);   //3

var f2 = fn.bind(null);
f1(3);  //3

```

## 三 作用域与变量提升
#### 3.1 作用域分类
- 块级作用域：代码块级别的作用域。JavaScript中没有块级作用域。但是ES6中提供了 let const 等支持块级类似块级作用域。
- 词法作用域：在代码写好的那一刻，变量的作用域已经确定。JS支持词法作用域！！！与词法作用域对应的是动态作用域。
```js
      var a = 123;
      function f1(){
        console.log(a);
      }

      function f2(){
        var a = 456;
        f1();
      }
      f2();                      //得到结果123---这里是词法作用域，执行f1()，直接进入f1()内部查找变量，找不到，去全局查找
                                  //如果是动态作用域,执行f1(),应该先在f1所处环境找，结果是456
 ```
 在JavaScript中只有函数能产生作用域！！！
 #### 3.2 预解析变量提升
JavaScript代码在预解析阶段，会对以var声明的变量名，和function开头的语句块，进行提升操作（hoisting）
变量提升：
- 首先将以var声明的变量名
- 以function开头的函数进行提升
- 执行
伪代码：
```js

预解析函数同名：后面的函数会替换前面的函数
func1();
function func1(){
    console.log("第一个func1被执行了");
}
func1();
function func1(){
    console.log("第二个func1被执行了");
 }

 预解析结果：
 function func1(){
 console.log("第一个func1被执行了");
 }
 function func1(){
 console.log("第二个func1被执行了");
 }
 func1();
 func1();
 第一个func1被顶替了，输出的结果全部是：第二个func1被执行了
 ```
变量和函数同名：在提升的时候，如果有变量和函数同名，会只提升函数
```js
alert(foo);         //输出函数体
function foo(){}
var foo = 2;
alert(foo);         //输出2

预解析提升后的代码
 function foo(){}
 var foo;
 alert(foo);
 foo = 2;
 alert(foo);
 ```
变量的提升是分作用域的:
```js
 console.log(a);            //undefined
 var a = 123;

 function test1(){
 console.log(num);          //undefined
 var num = 10;
 }


var num = 456;
 function test2(){
    console.log(num);          //undefined
    var num = 10;
 }
//  提升结果：
 var num;
 num = 456;
 function test2(){
    var num;
    console.log(num);          //undefined
    num = 10;
 }


var num = 456;
function f1(){
    console.log(num);
    num = 10;
}
console.log(num);               //10
f1()                            //456
// 提升结果：
var num;
function f1(){
    console.log(num);
    num = 10;
}
num = 456;
console.log(num);
f1();
```

函数表达式不会被提升:
```js
func();                     //报错
var func = function(){
    console.log("func执行了！！");
}

预解析后代码
var func;
func();
func = function(){
 console.log("func执行了！！");
}
```
函数可以创造作用域，函数中又可以再创建函数，函数内部的作用域就可以访问外观的作用域
如果有多个函数嵌套，就会构成作用域链

```js
//条件式函数声明是否会被提升，取决于浏览器的不同。

foo();                      //报错，标准浏览器不支持这样的变量提升
if(true){
    function  foo() {
        console.log(123);
    }
}
```
## 四 JS中的线程
线程：一个线程只能处理一件事情。
JS是单线程，可以用alert阻塞的方式进行演示。
JS中，分了三个线程
- 1-渲染
- 2-执行JS代码
- 3-事件处理任务
## 五 深拷贝与浅拷贝
浅拷贝：
```js
function extend(oldObj, newObj) {
    for(var key in oldObj) {
        newObj[key] = oldObj[key];
    }
}
var obj = {
    age:10,
    sex:1
}

var newObj = {};
extend(obj,newObj);
console.log(newObj.sex);        //1
```
深拷贝：
```js
function extend(oldObj, newObj) {
    for(var key in oldObj) {
        var item = oldObj[key];
        if(item instanceof Array) {
            newObj[key] = [];
            extend(item, newObj[key]);
        } else if(item instanceof Object) {
            newObj[key] = {};
            extend(item, newObj[key]);
        } else {
            newObj[key] = item;
        }
    }
}
var obj = {
    age:10,
    sex:1,
    sonObj: {
        name: "lisi"
    }
}

var newObj = {};
extend(obj,newObj);
console.log(newObj.sonObj);        //{ name: 'lisi' }
```