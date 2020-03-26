## 一 函数调用 apply call bind
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