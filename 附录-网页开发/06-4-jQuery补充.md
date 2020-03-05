## 一 jQuery特点
链式编程：
链式编程原理是 return this;通常情况下，只有设置操作才能把链式编程延续下去。因为获取操作的时候，会返回获取到的相应的值，无法返回 this。
end();操作结束当前链最近的一次过滤操作，并且返回匹配元素之前的状态。
隐式迭代：
在方法的内部会为匹配到的所有元素进行循环遍历，执行相应的方法；而不用我们再进行循环，简化我们的操作，方便我们调用。
如果获取的是多元素的值，大部分情况下返回的是第一个元素的值。
但是有时候我们需要对获取的元素集合中每个元素做不同的处理，可以使用each()方法：
```javascript
   $('li').each(function(i,elem){ //i：下标 elem : 每个元素
        $(elem).html(i);
    });
```
## 二 jQuery多库共存
多库共存指的是：jQuery占用了$ 和jQuery这两个变量。当在同一个页面中引用了jQuery这个js库，并且引用的其他库（或者其他版本的jQuery库）中也用到了$或者jQuery这两个变量，那么，要保证每个库都能正常使用，这时候就有了多库共存的问题。
```javascript
// 模拟另外的库使用了 $ 这个变量，就与jQuery库产生了冲突
let $ = { name : “itecast” };
//noConflict() 可以让jQuery释放对$的控制权，让其他库能够使用$符号，此后，只能使用jQuery来调用jQuery提供的方法 
let myQuery = $.noConflict();
let $ = 10;
myQuery(function(){  
    myQuery('body').css('background','red');
});
```
## 三 jQuery插件机制
通过插件的方式，可以扩展jQuery的功能：
```
$.extend: 		                扩展工具方法下的插件形式 
$.fn.函数名 = function(){}:  	扩展到JQ对象下的插件形式 （fn是 prototype的简写）
```
案例：
```javascript

<script> 
$.extend({
    leftTrim: function(str){
        return str.replace(/^\s+/,'');
    },
    rightTrim: function(){
        //方法体
    }
});
</script>

<script>
let str = ' hello ';
console.log($.leftTrim(str));
</script>

```
## 四 jQuery缓存机制
#### 4.1 斐波那契数列性能问题案例
```javascript
//斐波那契数列实现：f(n) = f(n-1) + f(n - 2)
let count = 0;
function fib(n){
    count ++;
    if(n <= 2){
        return 1;
    }
    return fib(n - 1) + fib(n - 2);
}
fib(5);
console.log(count);     //9
```
这里会有一个严重的性能问题：数值越大，计算的速度越慢。可以使用缓存优化策略：
- 1.先从cache数组中去取想要获取的数字
- 2.如果获取到了，直接使用
- 3.如果没有获取到，就去计算，把计算结果存入cache，然后将结果返回
```javascript
let cache = [];

function fib(n) {
    //1.从cache中获取数据
    if (cache[n] !== undefined) {
        //如果缓存中有 直接返回
        return cache[n];
    }
    //如果缓存中没有 就计算
    if (n <= 2) {
        //把计算结果存入数组
        cache[n] = 1;
        return 1;
    }
    let temp = fib(n - 1) + fib(n - 2);
    //把计算结果存入数组
    cache[n] = temp;
    return temp;
}
console.log(fib(6));
```
当然暴露了缓存变量是不安全的，上述代码可以采取闭包的办法。
```javascript
let count = 0;

function createFib() {
    let cache = [];

    function fib(n) {
        count++;
        //1.从cache中获取数据
        if (cache[n] !== undefined) {
            //如果缓存中有 直接返回
            return cache[n];
        }
        //如果缓存中没有 就计算
        if (n <= 2) {
            //把计算结果存入数组
            cache[n] = 1;
            return 1;
        }
        let temp = fib(n - 1) + fib(n - 2);
        //把计算结果存入数组
        cache[n] = temp;
        return temp;
    }
    return fib;
}

let fn = createFib();
fn(6);
console.log(count);
```
#### 4.2 jQuery缓存分析
```javascript
function createCache() {
    let keys = [];

    function cache(key, value) {
        // 使用(key + " ") 是为了避免和原生（本地）的原型中的属性冲突
        if (keys.push(key + " ") > 3) {
            // 只保留最新存入的数据
            delete cache[keys.shift()];
        }
        // 1 给 cache 赋值
        // 2 把值返回
        return (cache[key + " "] = value);
    }
    return cache;
}

let typeCache = createCache();
typeCache("monitor");
console.log(typeCache["monitor" + " "]);
typeCache("monitor", "张学友");
console.log(typeCache["monitor1" + " "]);
typeCache("monitor", "刘德华");
console.log(typeCache["monitor2" + " "]);
typeCache("monitor3", "彭于晏");
console.log(typeCache["monitor3 "]);
// console.log(typeCache["monitor "]);
```
#### 4.3 完整的jQuery缓存处理
```javascript
//eleCache
//typeCache
//classCache
//eventCache

function createCache() {
    //cache对象中以键值对的形式存储我们的缓存数据
    let cache = {};
    //index数组中该存储键，这个键是有顺序，可以方便我们做超出容量的处理
    let index = [];
    return function (key, value) {
        //如果传了值，就说名是设置值
        if (value !== undefined) {
            //将数据存入cache对象，做缓存
            cache[key] = value;
            //将键存入index数组中，以和cache中的数据进行对应
            index.push(key);

            //判断缓存中的数据数量是不是超出了限制
            if (index.length >= 50) {
                //如果超出了限制
                //删除掉最早存储缓存的数据
                //最早存入缓存的数据的键是在index数组的第一位
                //使用数组的shift方法可以获取并删除掉数组的第一个元素
                let tempKey = index.shift();
                //获取到最早加入缓存的这个数据的键，可以使用它将数据从缓存各种删除
                delete cache[tempKey];
            }
        }
        //如果没有传值，只穿了键，那就是获取值
        // else{
        // return cache[key];
        // }
        return cache[key];
    }
}

let eleCache = createCache();
eleCache("name", "高金彪");
console.log(eleCache("name"));
let typeCche = createCache();
```
#### 4.4 斐波那契数列最终版
```javascript
// 创建缓存容器

// function createCache(){
// let cache = {};
// return function (key, value) {
// //如果传了值，就说名是设置值
// if(value !== undefined){
// cache[key] = value;
// return cache[key];
// }
// //如果没有传值，只穿了键，那就是获取值
// else{
// return cache[key];
// }
// }
// }

let count = 0;

function createFib() {
    let fibCache = createCache();

    function fib(n) {
        count++;
        //1.从cache中获取数据
        if (fibCache(n) !== undefined) {
            //如果缓存中有 直接返回
            return fibCache(n);
        }
        //如果缓存中没有 就计算
        if (n <= 2) {
            //把计算结果存入数组
            fibCache(n, 1);
            return 1;
        }
        let temp = fib(n - 1) + fib(n - 2);
        //把计算结果存入数组
        fibCache(n, temp);
        return temp;
    }
    return fib;
}
```



