## 一 DOM性能优化
#### 1.1 减少DOM操作方式1
浏览器会将DOM与JS分开独立实现，如果使用JS操作DOM就像从一个区域跳转到了另外一个区域，每次跳转都会消耗性能，所以要尽量减少这种操作。
案例：
```js
//第一种操作DOM方式：
var div = document.getElementById('div');
console.time('hi');
//5000次来实现添加html
for(var i = 0; i < 5000; i++){
    div.innerHTML += 'a';
}
console.timeEnd('hi');//chrome下输出hi: 241.7109375ms

//将innerHTML移除：
var div = document.getElementById('div');
console.time('hi');
var html = '';
for(var i = 0; i < 5000; i++){
    html += 'a';
}
div.innerHTML = html;
console.timeEnd('hi');//chrome下输出hi: 7.304931640625ms
```
#### 1.2 减少DOM操作方式2：使用节点克隆
创建多个相同节点时，推荐使用clone，因为clone不是新创建节点，所以性能更高。
原本方式：
```js
console.time('hi');
var oUl = document.getElementById('ul');
for(var i = 0; i< 5000; i++){
    var oLi = document.createElement('li');
    oLi.innerHTML = 'li';
    oUl.appendChild(oLi);
}

console.timeEnd('hi');//chrome下输出hi: 28ms
```
使用clone方式优化：
```js
console.time('hi');
var oUl = document.getElementById('ul');
var oLi = document.createElement('li');
oLi.innerHTML = 'li';
for(var i = 0; i< 5000; i++){
    var newLi = oLi.cloneNode(true);
    oUl.appendChild(newLi);
}

console.timeEnd('hi');//chrome下输出hi: 8ms
```
#### 1.2 重排与重绘
浏览器重排：通过DOM操作改变了页面元素的位置、形状。  
浏览器重绘：重排后的内容显示出来的过程叫做重绘。  
注意：改变背景颜色不会触发重排，只会触发重绘，重排与重绘都会影响性能。  
优化措施：尽量在appendChild前添加重排、重操作。
```js
//旧方式
console.time("hi");
for(var i = 0;i < 5000; i++) {
    var liObj = document.createElement("li");
    ulObj.appendChild(liObj);
    liObj.innerHTML = 'li';
}
console.timeEnd("hi");      //750ms

//新方式
console.time("hi");
for(var i = 0;i < 5000; i++) {
    var liObj = document.createElement("li");
    liObj.innerHTML = 'li';
    ulObj.appendChild(liObj);
}
console.timeEnd("hi");      //440ms
```


