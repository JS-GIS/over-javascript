Vue的自定义插件是放在Vue的原型身上的；

### 9.1.1 定义插件的方式
+ 第一种就是直接在prototype身上定义；
+ 第二种，可以定义一个对象为插件,然后在use()里调用；

**第一种方式：**

```js
   
    Vue.prototype.$aaa = '这是定义的插件';
```

**第二种方式：**
```js
    let obj = {
        install:function(Vue,opt){
            Vue.prototype.$aaa = '这是定义的插件';
        }
    }

    Vue.use(obj, {a:1})； 
```

用对象作为插件时必须遵循一个原则：

key值install对应的是一个函数， 这个函数可以接收两个参数；

第一个参数是构造函数Vue；拿到这个构造函数就可以在它身上扩展属性和方法了。

第二个参数是使用插件的时候传过来的参数： "Vue.use(obj, {a:1})" 中的 “{a:1}”；

```js
    let obj = {
        install : function(Vue,option){ // 页面一刷新的时候，就可以获取到这两个参数：Vue,option

        }
    }
```


### 9.1.2 Vue插件案列

案列： 获取和设置 localStorage 的存储
    
```js
    // 对象文件：utils.js
    let local = {
        save(key,value){
            localStorage.setItem(key, JSON.stringify(value))
        },
        fetch(){
            return JSON.parse(localStorage.getItem(key) || {})
        }
    };

    export default{ 
        install:function(vm){
            vm.prototype.$local = local // local对象挂载到Vue原型上
        }
    }
   
```

    
```js
    // 使用utils插件的文件：

    import Utile from './lib/utils';

    Vue.use(Utile)；

    // 一旦作为一个插件使用之后，就可以在每个组件里面通过 this 访问到 local对象了
```

**可以用一个文件写很多的对象，把想要暴露出去的对象挂载到Vue原型身上就行；如：**

```js

let obj1 = {};
let obj2 = {};
let obj3 = {};

export default {
  install: function (vm) {
    vm.prototype.$obj1 = obj1;
    vm.prototype.$obj2 = obj2;
    vm.prototype.$obj3 = obj3;
  }
}

```