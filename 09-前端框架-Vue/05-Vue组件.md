## 一 Vue组件创建

### 1.1 组件化和模块化的不同
+ 模块化 —— 是从代码逻辑的角度进行划分的，方便代码分层开发，保证每个功能模块的职能单一；
+ 组件化 —— 是从UI界面的角度进行划分的，方便UI组件的重用；

贴士：模板中的元素必须只有一个根元素（外围包裹元素）。



### 1.2 创建方式一:extend

注意：

如果组件名称使用了驼峰命名，在引用组件的时候，需要把大写的驼峰改为小写的字母，同时两个单词之间使用‘-’连接；

如果组件名不使用驼峰，则直接拿名称来使用即可。

```html

    <div id="app">
        <my-com1></my-com1> <!-- 驼峰需要使用-连接 -->
    </div>

    <script>
        // 创建全局组件
        Vue.component('myCom1', Vue.extend({
            template: '<h3>h3组件</h3>'         // 通过template属性，指定结构
        }));                                     // 这里使用驼峰命名

        let vm = new Vue({
            el: "#app"
        });
    </script>
```

### 1.3 创建组件方式二：component

```html

    <div id="app">
        <my-com2></my-com2>                      <!-- 驼峰需要使用-连接 -->
    </div>

    <script>
        Vue.component('myCom2', {
            template: '<h3>1111</h3>'
        })
        let vm = new Vue({
            el: "#app"
        });

    </script>
```

### 1.4 创建组件方式三：template标签
```html

    <div id="app">
        <my-com3></my-com3>                      <!-- 驼峰需要使用-连接 -->
    </div>

    <template id="tmp1">
        <h3>组件</h3>
    </template>

    <script>

        Vue.component('myCom3', {
            template: '#tmp1'
        })

        let vm = new Vue({
            el: "#app"
        });

    </script>
```

### 1.5 创建私有组件

```html
    <div id="app">
        <my-com4></my-com4>                      <!-- 驼峰需要使用-连接 -->
    </div>

    <script>

        let vm = new Vue({
            el: "#app",
            components: {
                myCom4: {
                    template: '<h3>com4</h3>'
                }
            }
        });

    </script>
```

当然这里的template也可以引入一个template标签。

## 二 组件的数据

### 2.1 组件内使用数据

组件内也可以又data属性，但是该属性是个函数，且必须返回一个对象：
```html
    <div id="app">
        <my-com4></my-com4>                      <!-- 驼峰需要使用-连接 -->
    </div>

    <script>

        let vm = new Vue({
            el: "#app",
            components: {
                myCom4: {
                    template: '<h3>{{msg}}</h3>',
                    data: function(){
                        return {
                            msg: '组件内部数据'
                        }
                    }
                }
            }
        });

    </script>
```

### 2.2 组件切换

除了可以使用自定义true/false方式来切换组件外，vue本身也提供了组件切换机制：

vue提供了 component 里展示对应名称的组件

component 是一个占位符， :is 属性 可以用来指定要展示的组件的名称  

```html
    <div id="app">
        <span @click="coMname= 'com1'">显示组件com1</span>
        <span @click="coMname= 'com2'">显示组件com2</span>
        <component :is="'coMname'"></component>
    </div>

    <script>

        Vue.component('com1', {
            template: '<h3>111</h3>'
        });

        Vue.component('com2', {
            template: '<h3>222</h3>'
        });

        let vm = new Vue({
            el: "#app"
            data:{
                coMname:'com1'
            }
        });

    </script>
```

### 2.3 父组件向子组件传值

**2.3.1 传值的步骤**

1、父组件，可以在引用子组件的时候，通过属性绑定（v-bind:）的形式，把需要传递给子组件的数据，以属性绑定的形式，传递到子组件内部，供子组件使用。

2、把父组件传递过来的属性，先在props数组中定义一下才能使用这个属性值（传递过来的数据）

```html
    <div id="app">
        <son v-bind:parent-msg="msg"></son>
    </div>

    <script>

        let vm = new Vue({
            el: "#app",
            data: {
                msg: "father..."
            },
            components: {
                son: {
                    props: [            // 父组件通过属性传递过来的数据需要在这里再次定义才能使用
                        'parentMsg'     // 下滑线对应大写     
                    ],
                    template: '<h4>子组件得到的父组件数据:{{parentMsg}}</h4>'
                }
            }
        });

    </script>
```

注意：
- data中的数据是子组件私有的，父组件的数据并没有传递给子组件的data，子组件想要获取父组件的数据是通过props传递

- data中一般放置请求到的数据，props里面放置的是传递过来的数据。
- data中的数据都是可读写的，而props中的数据是只读的



**2.3.2 父组件向子组件传递方法**

步骤：

1、父组件向子组件传递方法，使用的是事件绑定机制（v-on）；自定义了一个事件属性sonfunc，父组件的方法传递给此属性。

2、子组件自定义一个方法myclick，在myclick方法中，通过 this.$emit()去触发sonfunc： 

    this.$emit('sonfunc');

    this代表子组件实列；    
    emit 代表触发、调用的意思；

每当触发子组件的myclick事件的时候，都会通过this.$emit()来触发父组件传递过来的sonfunc，从而触发父组件的方法show。


`<son @sonfunc="show"></son>`

注意：

引用子组件的时候，传递的show方法不能带括号，如果带括号就表示把show方法调用的结果传递给sonfunc，而不是把方法的引用传递给sonfunc

```html
    <div id="app">
        <son @sonfunc="show"></son>
    </div>

    <script>

        let vm = new Vue({
            el: "#app",
            data: {
                msg: "father..."
            },
            methods: {
                show: function(){
                    console.log("show...")
                }
            },
            components: {
                son: {
                    props: [            // 父组件通过属性传递过来的数据需要在这里再次定义才能使用
                        'parentMsg'     // 下滑线对应大写     
                    ],
                    methods: {
                        myclick: function(){
                            this.$emit('sonfunc');
                        }
                    },
                    template: '<button @click="myclick">点击触发父组件传递过来的事件</button>'
                }
            }
        });

    </script>
```


向父组件传参：

父组件方法：show: function(形参){}

子组件调用：this.$emit('sonfunc',实参);    


### 2.4 子组件向父组件传值

**2.4.1 子组件向父组件传数据**

通过上面父组件向子组件传递方法的时候，我们可以通过传参的形式，达到子组件向父组件传递数据的效果。

```html
<div id="app">
    <son @sonfunc="show"></son>
</div>

<script>

    let vm = new Vue({
        el: "#app",
        data: {
            arr: ""
        },
        methods: {
            show: function(arg){
                console.log(arg);
                this.arr = arg;
            }
        },
        components: {
            son: {
                data(){
                    return {
                        sonArr:[
                            {
                                id:1,
                                name:'aa'
                            },
                            {
                                id:2,
                                name:'bb'
                            }
                        ]
                    }
                }
                template: '<button @click="myclick">点击触发父组件传递过来的事件</button>'

                methods: {
                    myclick: function(){
                        this.$emit('sonfunc',this.sonArr);
                    }
                },
            }
        }
    });

</script>
```
