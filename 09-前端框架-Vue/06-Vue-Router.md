
## 前言

VueRouter是vue可以渐进引入的路由模块，但是既然已经引入了路由模块，项目已经具备了工程化概念，不妨直接使用webpack方式构建vue项目

## 一 vue-cli 

vue-cli是vue的脚手架（为vue提供初始化环境的工具）。

安装vue-cli：
```
npm i vue-cli -g
vue                         # 查看是否安装成功
```

生成基础项目：
```
vue init webpack example    # 使用webpack模板生成项目example
```

main.js中的`/* eslint-disable no-new */`用户忽略eslint的检查（因为这里new Vue未被使用）。

## 二 vue-router

### 2.1 前端路由

前端路由即：根据url分配对应的处理程序。vue-router中，通过管理url，实现组件与url的对应，通过url进行组件之间的切换。  

**使用步骤：**

 + 安装路由；

    npm i vue-router --save

    如果用的是vue-cli就不需要单独安装了

  + 引入路由模块；
  
    import VueRouter from 'vue-router'

  + 作为vue的插件；

    Vue.use(VueRouter)

  + 创建路由实例对象；
  
    new VueRouter({
      ...配置参数
    })

  + 注入vue选项参数；
    new Vue({
      router
    })

  + 告诉路由渲染位置：
    ``` html
    <router-view> </router-view>
    ```
### 2.2 helloworld

在main.js中配置vue-router，
```js
import VueRouter from 'vue-router'

// 启用vue-router
Vue.use(VueRouter)
```

创建一个组件HomeComponent，并在main中引入，完整代码如下
```js
import Vue from 'vue'
import VueRouter from 'vue-router'
import App from './App'
import Home from '@/components/Home'

// 启用vue-router
Vue.use(VueRouter)

Vue.config.productionTip = false

let router = new VueRouter({
  routes: [ // 一个路径对应一个组件 
    {
      path: '/home',
      component: Home
    }
  ]
})

/* eslint-disable no-new */
// 变量名和属性名一致可以直接写为：router
new Vue({
  el: '#app',
  router: router,
  components: { App },
  template: '<App/>'
})

```

在根组件App.vue上展示该组件
```js
  <div id="app">
    <router-view></router-view>
  </div>
```

访问：http://localhost:8080/#/home

### 2.3 router.js

由于路由信息很多，推荐路由信息脱离main.js，单独配置。  

src下新建文件夹router，新建index.js：
```js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '@/components/Home'

// 启用vue-router
Vue.use(VueRouter)

/* eslint-disable no-new */
let router = new VueRouter({
  routes: [
    {
      path: '/home',
      component: Home
    }
  ]
})
export default router
```


## 三 路由的常见配置

### 3.1 路由模式

路由模式有两种：hash模式和history模式；

路由的默认模式是hash模式，


```js
let router = new VueRouter({
  mode: 'history',                  // 默认是hash，此时设置为history模式
  routes: [
    {
      path: '/home',
      component: Home,
    }
  ]
})
```

哈希模式时，路由地址这样书写：
```js
  <div id="app">
    <a href="#/home">点击渲染home组件</a>
    <router-view></router-view>
  </div>
```

修改为history模式后，路由地址就可以直接书写为： `<a href="/home">`  

### 3.2 router-link标签

采用history模式会带来新的问题：该模式下，a连接会引起页面刷新。vue为了解决这个问题，提供了router-link标签。
```js
  <div id="app">
    <router-link to="/home">显示home</router-link>
    <router-view></router-view>
  </div>
```

使用router-link替代a标签的好处：
- router-link在被解析仍然是a标签，但是其默认行为(刷新页面)被阻止了。  
- hash模式下，使用router-link也无需修改跳转地址为#/home。  

router-link的其他设置：  
- 跳转地址也支持动态绑定形式：` :to="home" `，此时home是该组件导出的数据对象
- 也支持对象形式配置：` :to="{path: '/home'}" `
- router-link默认生成的是a标签，也可以生成div、p等标签，添加属性：` tag="div" `，此时div会自动拥有监听点击事件的功能
- 默认的触发组件事件是点击事件，也可以修改为别的事件：添加属性：` event="mouseover" `
- 添加` exact `属性会让样式渲染变为不包含形式（精确匹配）。

**贴士1：router-link可以改成制定的标签：**

很多导航中，使用导航标签既包含图标又包含文字，router-link可以这样配置：
```js
<router-link :to="home" tag="li">
    <i><img src=""></i>
    <span>首页</span>
</router-link>
```  

**贴士2：router-link 配置当前激活状态的class名**

router-link生成的li元素，被点击的li上会带有router-link-active的class属性，可以配置激活状态的样式。如果觉得这个class名字很长也可以自己配置新名字：

配置当前激活状态的class名有两种方式：

```js
// 第一种：全局配置方式：在router里面修改，linkActiveClass:“” ， 所有生成的元素的router-link-active都被修改
let router = new VueRouter({
    linkActiveClass: 'isActive',            // 此时class名为isActive
})

// 第二种：局部修改：在router-link标签上添加属性 active-class=“” ，只支持当前组件
<router-link to="/home" active-class="isActive">显示home</router-link>
```

### 3.3 router-view 标签的其他配置

router-view是组件渲染的地方，他也可以设置很多配置：

**同时给子组件添加相同的类名**

在router-view标签添加class="center"，那么所有子组件渲染的时候，外层根节点标签就会自动添加class="center"；

```js
<router-view class="center"></router-view>      <!-- 配置样式 -->
```

### 3.4 redirect重定向

重定向与别名设置：

```js
let router = new VueRouter({
  routes: [
    {
      path: '/home',
      component: Home,
      name: 'Home',                 // 可选项，给路由起个名字Home，Home就代表当前的路由，可以用这个name来指向这个路由，或是用这个name来做其他的事情；
      alias: '/index'               // 别名：访问/index渲染Home
    },
    {
      path: '*',                    // 配置在最后，当所有路由都未被匹配到则如何处理
      // component: Error,          // 可以直接调转到错误提示页

      // redirect会替换掉浏览器中的地址，alias不会
      // redirect: '/index'         // 也可以配置重定向
      // redirect: {path: '/home'} // 重定向方式二
      // redirect: {name: 'Home'}   // 重定向方式三，name是路由的名字
       redirect: (to)=>{          // 重定向方式四,动态设置重定向的目标；
      //     console.log(to);       // to目标路由对象，就是访问的路径的路由信息
      //     return '/home'         // return值也可以写为  {path:} 或 {name:}


          //除了可以直接return重定向的路由外，还可以通过 path\hash\query 等判断，动态设置重定向的目标路由：

          if(to.path === "/123"){
             return '/home' 
          }else if(to.path === "/456"){
             return {path: '/document'}
          }else{
            return {name: 'about'}
          }

       }

    }
  ]
})
```


### 3.5 路由中的name属性

路由中的name属性是个可选项，

作用是 给路由起个名字Home，Home就代表当前的路由，可以用这个name来指向这个路由，或是用这个name来做其他的事情；

name的使用场景一：
```js
let router = new VueRouter({
  routes: [
    {
      path: '/home',
      component: Home,
      name: 'Home',                 // 可选项
      alias: '/index'               // 别名：访问/index渲染Home
    },
    {
      path: '*',
      redirect: {name: 'Home'}      
    }
    
  ]
})
```

name的使用场景二：
```html
<router-link :to="{name:'路由的名字'}"></router-link>
```


### 3.6 嵌套路由

**3.6.1 嵌套步骤：**

+ 1、html的书写：
    
    点击about的时候，设置默认匹配到study的话，router-link的to属性值就不需要写成 to="/about/study" 而是需要写成： to="/about"

  
+ 2、路由的书写：
  
    a、路由配置的时候，path的值不需要设置：path: ''；
    
    b、嵌套路由时，父路由不要使用name，name应该设置在默认子路由上 

```html
<ul class="nav">
  <!-- -->
  <router-link to="/about" tag="li">
    <a>study</a>
  </router-link>
  <router-link to="/about/work" tag="li">
    <a>work</a>
  </router-link>
  <router-link to="/about/tel" tag="li">
    <a>tel</a>
  </router-link>
</ul>
```

```js
{
    path: '/about',
    component: about,
    children: [
        {                           
            path: '',          // 当访问 /about也会默认渲染children第一个
            component: study,
            name: about       // 父路由的name要放在默认的子路由上
        },
        {                          
            path: 'work',          // 匹配地址为： /about/work
            component: work,
            name: work
        },
        {
            path: 'tel',          // 匹配地址为： /about/tel
            component: tel,
            name: tel
        }
    ]
}
```

**3.6.2 exact表示精确的匹配**

路由的嵌套里，有个全包含匹配关系，需要设置精确匹配才能去掉全包含匹配的激活状态样式，从而精确到只显示点击的当前匹配样式。

注意：需要设置渲染在哪个位置，即填写router-view标签
```js
<ul class="nav">
    <!-- 默认渲染 about/work, exact表示精确的匹配，exact如果不加，则会模糊匹配，点击about/tel时，第一个link的样式也是被点击状态 -->
    <router-link to="about" exact tag="li">         
        <a>work</a>
    </router-link>
    <router-link to="about/tel" exact tag="li">
        <a>tel</a>
    </router-link>
</ul>
```

**3.6.3 路由的路径不嵌套但组件嵌套的时候的设置**

+ 组件和路由路径都嵌套的写法：localhost:3000/about/work

+ 组件嵌套，但路径不嵌套的写法： localhost:3000/tel

  path值前面添加斜杠 '/'，表示相对于根路径的

```js
{
    path: '/about',
    component: about,
    children: [
        {                           
            path: '',          
            component: study,
            name: about      
        },
        {                          
            path: 'work',          // 匹配地址为： /about/work
            component: work,
            name: work
        },
        {
            path: '/tel',          //path值前面添加斜杠 '/' ，匹配地址为： /tel  
            component: tel,
            name: tel
        }
    ]
}
```

**3.6.4 命名视图**

一个路径里面展示多个同级组件

步骤
+ 1、为了明确同级组件的渲染地方，在router-view渲染处需要取名字来区别，如：name="slider"；

+ 2、在路由里配置对应的组件: 一个路径对应多个组件 -- 使用components 而不是 component

    components: {
        default:document,
        slider:slider    
      }     

```html
<router-view name="slider"></router-view>  
<router-view class="center"></router-view>  
```
```js
let router = new VueRouter({
  routes: [
    {
      path: '/document',
      components: {    // components
        default:document,   // 默认的
        slider:slider   
      }            
    }
  ]
})


        // 设置一个默认的组件。 当进入路由的时候，遇到default，就会把这个默认的组件渲染到 没有取名字的 router-view 中。 
        //（<router-view class="center"></router-view>  ）


        // 键slider 是上面的 name="slider" 中的 slider   
        // 值slider 是引进来的组件slider: import slider from '@/components/slider'
```

### 3.7 scrollBehavior()

跳转组件后，返回上一次组件界面时，定位到上次滚动条的位置

+ 方法一：

   通过 scrollBehavior() 方法里的savePosition参数记录滚动条坐标,来定位前进后退后的视窗位置

```js
let router = new VueRouter({
  mode: 'history',                  
  scrollBehavior(to,from,savePosition){   // 点击浏览器前进后退、切换导航时触发
    // to 要进入的目标路由对象                
    // from 要离开的路由对象
    // savePosition 记录滚动条坐标,点击前进后退的时候记录值       
    if(savePosition){
      return savePosition;
    }else {
      return {x:0,y:0}
    }
  },
  routes: [
    {
      path: '/home',
      component: Home,
    }
  ]
})
```
+ 方法二：在 scrollBehavior()方法里 利用hash来定位前进后退后的视窗位置
   
步骤一： 设置跳转的hash值  
```html
<router-link to="/about#abc" ></router-link>

```
步骤二：在about组件里添加id的属性值为abc。
```html
<div>
<p id="abc">定位到这个元素</p>
</div>

```
步骤三：在路由里判断hash值
```js
let router = new VueRouter({
  mode: 'history',                  
  scrollBehavior(to,from,savePosition){   
    // to 要进入的目标路由对象                
    // from 要离开的路由对象
    //  利用hash记录坐标    
    if(to.hash){
      return {
        selector: to.hash
      }
    }  
  },
  routes: [
    {
      path: '/home',
      component: Home,
    }
  ]
})
```   

## 四 路径与参数



### 4.1 动态参数

什么时候用到动态路径？

匹配到所有路由，全部映射到同一个组件的时候。
    
    比如：
    在一个网站中，进入到个人信息的时候，每个信息页面一样，但是不同的用户展示的信息不一样，此时的路径就可以写成： user/:userId ，userId 为动态路径参数；

    访问的是同一个组件，但是userId根据不同用户展示数据不一样；我们通过获取到userId就可以拿到不同用户的信息

**获取参数：路由信息对象的params**

用户数据
```js
let dataUser = [
  {
    uid:1,
    name:zhangsan,
  },
  {
    uid :2,
    name:lisi,
  }
]
```
路由配置：
```js
 routes: [
    {
      path: '/user/:userId?',  
      component: user,
    }
  ]

  // userId只是个变量，当访问对应具体的路由的时，此变量的值就是对应的具体值；
  // userId后面的‘？’是个正则，作用是判断匹配：当userId没有值的时候，直接访问的是‘/user’，当userId有值的时候，直接访问的是 ‘/user/所传值’ ，
``` 
数据插入对应路由
```html
  <div>
    <router-link 
      style="padding-right: 50px;" 
      :to="'/user/' + item.uid " 
      :key="index" 
      v-for="(item,index) in userList"
    > 
      第 {{ index + 1 }} 个用户： {{ item.name }}
    </router-link>
  </div>

```



**关于路由的两个对象：**
- $router: 通过 $router 拿到router的实例对象 
- $route: 当前激活的路由信息对象，每个组件实例都会有
      
    每访问一个路径，都会对应一个路由，此时 $route 里存放的就是当前路由的信息对象。所以可以通过它来获得用户id了。



获取路由中的参数：
```js
// 以 /user/:uid  方式访问某个子组件，根据uid的不同，该子组件渲染不同的数据
  <div>
    <router-link 
      style="padding-right: 50px;" 
      :to="'/user/' + item.uid " 
      :key="index" 
      v-for="(item,index) in userList"
    > 
      第 {{ index + 1 }} 个用户： {{ item.name }}
    </router-link>
  </div>


// 子组件中获取数据
  export default {
    name: 'User.vue',
    watch: {                      // 监控
      $route(){
        console.log(this.$route.params)
      }
    },
    created () {
      console.log(this.$route.params)
    }
  }

  // 生成一个组件之后，再在这个组件里面操作，这个组件不会重新生成，而是复用之前生成的组件； 

  //组件刚生成一次的时候，会去执行一下生命周期的钩子函数， 如果下次没有让组件重新生成，钩子函数就不会执行了，也就是说钩子函数只会执行一次；此时， 在组件里面切换的时候，路径发生变化，匹配到一个路由，就可以得到一个路由信息对象 $route；

  //如果地址栏一旦发生变化，$route 就会重新生成一个路由信息对象，$route 重新赋值， 通过 watch 监控路由信息对象 $route；当 $route 改变的时候，就可以拿到对应的信息了。

```

贴士：
- 传输多个参数的方式：:to="'/user/1/1'   类型参数：是否是vip，uid参数 1
- 路由配置：  path: '/user/:isvip?/:uid?'   
- 子组件中的打印结果： {type: 1, uid: 1}

获取查询字符串：
```js
<router-link exact :to="{path:'', query:{uid:'1'}}">
<div>
{{$route.query}}      <!-- 这就是上述query内的json -->
</div>

// query对象里可以多个元素 ： query:{uid:'1', 'infor':'alow', ...} 
```
## 五 编程式导航

之前使用router-link 生成标签去切换组件；编程式导航也可以切换组件。

编程式导航 其实是借助于 router 实例的一些方法，通过调用这些方法，编写代码来实现导航的切换；

**router实例提供的方法：**
+ back 回退一步
+ forward 前进一步
+ go 指定 前进/回退 步数
+ push 导航到不同的 url，向 history栈 添加一个新的记录
+ replace 导航到不同的 url，替换 history栈 中当前记录

```html
<div>
  <input type="button" value="后退" @click="backHandle">
  <input type="button" value="前进" @click="forwardHandle">
  <input type="button" value="前进/后退 到指定步数" @click="goHandle">
  <input type="button" value="控制指定的导航 push" @click="pushHandle">
  <input type="button" value="控制指定的导航 replace" @click="replaceHandle">
</div>

<script>
  export default{
    methods: {
      backHandle(){
        this.$router.back();
      },
      forwardHandle(){
        this.$router.forward();
      },
      goHandle(){
        this.$router.go(-2);  // 负数是后退，正数是前进，0是刷新当前页面，超过步数的话没有效果；
      },
      pushHandle(){
         this.$router.push('/document');  // 目标链接 -- 字符串形式
         this.$router.push({path:'/document', query:{uid:'1'}});  // 目标链接 -- 对象形式
      },
      replaceHandle(){
        this.$router.replace('/document');  // 目标链接 -- 字符串形式
        this.$router.replace({path:'/document', query:{uid:'1'}});  // 目标链接 -- 对象形式
      }
    }
  }

</script>
```

## 六 导航的钩子函数

导航发生变化的时候，导航钩子主要用来拦截导航，让它完成跳转或取消；

### 6.1 执行钩子函数的位置

+ router全局：

     router实例身上：beforeEach(to, from, next)、afterEach;(to, from)

+ 单个路由：
  
     单个路由中：beforeEnter(to, from, next)

+ 组件中：
  
     组件内的钩子：
     
     beforeRouteEnter(to, from, next)、
     
     beforeRouteUpdate(to, from, next)、
     
     beforeRouteLeave(to, from, next)


### 6.2 导航钩子函数的案列


**6.2.1 导航钩子函数的参数**
+ to -- 目标导航的路由信息对象；
+ from -- 离开的路由信息对象；
+ next -- 是否要进入导航，如果需要进入导航就执行 next();

**6.2.2 router实例身上的钩子函数案列**

+ beforeEach():
```js
  let router= new VueRouter();

  router.beforeEach((to, from, next)=>{
    console.log('beforeEach');
    next(); 
  })

  // 只要切换不同的导航，beforeEach 这个导航钩子就会被执行
  
```


next 里面可以接收参数：next(false) -- 取消导航，不会进入导航内；比如 登录功能，如果没有登录，就重定向到登录页面；

```js
  // 路由配置：
  routes: [
    {
      path: '/user',  
      component: user,
      meta:{
        login:true     // 添加 login 标识
      }
    }
  ]

  // 钩子函数判断：
  router.beforeEach((to, from, next)=>{ 
    if(to.meta.login){
      next('/login');  // 如果需要登录就进入登录页
    }else {
      next();          // 如果不需要登录就进入导航页
    }
  })
```
+ afterEach()

afterEach没有参数 next;

当进入导航之后触发；
```js

  //案列： 当渲染不同的导航时，页面的 title 对应的改变

  // 路由配置：
  routes: [
    {
      path: '/user',  
      component: user,
      meta:{
        title: 'user'     // 添加 title 标识
      }
    }
  ]

  // 钩子函数判断：
  router.afterEach((to, from)=>{ 
    if(to.meta.title){
      window.document.title = to.meta.title;  // 如果title存在，就改变页面的title为to.meta.title
    }else {
      window.document.title = '给个固定的title';        
    }
  })

  // 钩子函数内部是不能直接访问document的，需要通过window去访问，所以不能直接写成：document.title
  
```


**6.2.3 单个路由中的钩子函数案列**

+ beforeEnter

给某个具体的路由设置钩子函数：
```js

  routes: [
    {
      path: '/user',  
      component: user,

      beforeEnter(to, from, next){
       // 当访问 /user 这个导航的时候，执行这个钩子函数
        console.log('beforeEnter');

        next();
      },
      
      meta:{
        login:true   
      }
    }
  ]

```

**6.2.4 组件内的钩子函数案列**
+ beforeRouteEnter
  
  beforeRouteEnter执行的时候，组件还没有创建，所以在beforeRouteEnter里面是不能直接用this来获取这个组件的数据的；

**进入组件的时候，执行的第一个生命周期函数是 beforeCreate, 那么是路由的钩子函数先执行还是组件的钩子函数先执行？**

下面的案列  先打印 beforeRouteEnter，后打印beforeCreate，所以是路由的钩子函数先执行，再执行组件里的钩子函数，创建组件实例；

路由的钩子函数执行的时候，组件的实例还没有创建，所以this是undefuad,所以直接用this就访问不到 test了;

**那么要怎么在 beforeRouteEnter 里去修改数据呢？**

可以通过 beforeRouteEnter 的参数 next(),里面传递一个回调函数，这个回调函数会在进入导航之后执行，同时这个回调函数会接收一个参数，这个参数就是当前的组件实例 vm，此时就可以获取到实例了，从而获取到 test ；

```html
<!-- 案列： 进入导航的时候，在beforeRouteEnter里把渲染的数据修改了  -->

<div>
  测试：{{test}}
</div>

<script>
  export default{

    data(){
      return {
        test: '改变前' 
      } 
    },
    beforeCreate(){
       console.log('beforeCreate');
    },
    // 进入组件之前执行 beforeRouteEnter
    beforeRouteEnter(to, from, next){
      console.log('beforeRouteEnter');
      next((vm)=>{
        vm.test = '改变了';
      });
    }
  }

</script>
  
```
+ beforeRouteUpdate
  
  在一个组件里面，点击切换二级导航的时候，导航改变更新了，就会触发钩子函数 beforeRouteUpdate；
```js
export default{
  // 二级导航导航改变更新的时候执行 beforeRouteUpdate
  beforeRouteUpdate(to, from, next){
    console.log('beforeRouteUpdate');
    next();
  }
}
```

+ beforeRouteLeave
  
  离开当前组件的时候，就会触发钩子函数 beforeRouteUpdate；
```js
export default{
  beforeRouteLeave(to, from, next){
    console.log('beforeRouteLeave');
    next();   // 控制是否离开当前组件
  }
}
```


**当访问一个子路由的时候，不仅仅把自己的路由信息对象存在 matched 里面，同时把父路由父信息对象也存在 matched 里面；**