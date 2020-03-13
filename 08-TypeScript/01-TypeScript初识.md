## 一 HelloWorld
TypeScript是由微软开发的一款开源编程语言，是JavaScript的超集，遵循ES6与ES5规范，扩展了原生JS语法，适合开发大型企业项目。
环境配置：
```
npm install -g typescript
```
helloworld 案例：
```ts
//新建一个 hello.ts
export class Hello {
    run() {
        console.log("hello world!");
    }
}
(new Hello()).run();
```
该代码不能在浏览器、Node环境中直接运行，需要编译：
```
//编译后产生的 hello.js 可以直接运行
tsc hello.ts
```
每次书写完的ts文件需要编译是很痛苦的，通过配置可以实时编译，（webstorme支持自动编译），VSCode配置步骤如下：
第一步：tsc --init 生成配置文件，可以生成一个tsconfig.json文件；
第二步：点击 菜单-任务-运行任务-tsc:监视-tsconfig.json。
## 二 数据类型
### 2.1 九种类型
```
布尔类型：	boolean
数字类型：	number
字符串类型：string
数组类型：	array
元组类型：	tuple
枚举类型：	enum
任意类型：	any
void类型：	void
never类型： 包括null和undefined
```
### 2.2 类型校验
TypeScript引入了类型校验，生命一个变量必须同时声明变量的类型：
```ts
var flag:boolean = true;
flag = 1;                   //已经报错
```
### 2.3 数组 array
在typescript中可以直接使用es5的定义语法，也可以指定数组中元素类型：
```ts
var arr1 = [1,2,'str']; 				//ES5原生数组
var arr2:number[] = [1,2,3]; 			//数组元素只能是数字
var arr3:Array<number> = [1,2,3]; 	    //数组元素只能是数字
```
### 2.4 元组类型 tuple
大多数语言的数组内所有元素的类型都是相同的，我们也推荐在js中这样使用，元组也是一种数组，可以在数组内添加不同类型的元素。
```ts
let arr1:[number,string] = [1,'aaa'];
let arr2:[number,string] = [1,2,'aaa']; //报错 二个元素应该是字符串且长度不对
```
### 2.5 枚举 enum
```ts
enum Flag {
    success = "成功",
    error = "失败"
}
let f:Flag = Flag.success;
console.log(f);     //成功
```
案例：
```ts
enum Color {
    blue, 
    red, 
    green=3,
    '红色'
}
let a:Color = Color.blue;
let b:Color = Color.red;
let c:Color = Color.green;
let d:Color = Color.红色;
console.log(a); //输出0 如果标识符没有赋值，它的值就是下标
console.log(b); //输出1
console.log(c); //输出3
console.log(d); //输出4
```
### 2.6 任意类型 any
在前端开发中通过getElementById()获取到的元素是个object类型，而ts没有该类型，直接赋值就会报错，可以使用 any 类型。
```ts
let num:any = 123;
console.log(typeof num);
num = false;
console.log(typeof num);
num = 'str';
console.log(typeof num);
```
### 2.7 其他类型 never
never类型除了never修饰外，undefined和null也是never类型的子类型。

```ts
var num1:number
console.log(num1);       //声明没有赋值报错

var num2:undefined
console.log(num2);      //undefined类型直接输出不会报错

var num3:number | undefined
console.log(num3);      //不会报错

var num:null;
num = null;		//正确
num = 123;		//报错，定义一个变量为null时，变量的值只能是null

```

never类型是任何类型的子类型，可以赋值给任何类型；
然而，除了never本身之外，没有类型可以赋值给never类型（包括any）。
通常表现为抛出异常或无法执行到终止点（例如无线循环）。
```ts
let x: never;

// 运行正确，never 类型可以赋值给 never类型
x = (()=>{ throw new Error('exception')})();

// 返回值为 never 的函数可以是抛出异常的情况
function error(message: string): never {
    throw new Error(message);
}

// 返回值为 never 的函数可以是无限循环这种无法被执行到的终止点的情况
function loop(): never {
    while (true) {}
}
```
## 三 函数
```ts
//声明
function run(name:string, age:number) :boolean {
    return false;
}

/*
ts 也支持匿名函数
void 类型表示没有返回值
*/

//参数可以设置默认参数
function test(a:string,b:string = 'hi'){
    console.log(a);
    console.log(b);
}

//ES5中，方法的实参和形参可以不一样，但是ts中必须一样，如果要求二者不一样，ts中可以配置可选参数。
function test(a:string,b?:string,c:string = 'hi'){
    console.log(a);
    console.log(b);
    console.log(c);
}

//剩余参数
function fn(a:any,...args:any) {
    args.forEach((arg:any)=>{
        console.log(arg)
    });
}

fn(1,2,3);      //输出2 3

//无需书写小括号的场景：字符串模板作为参数
fn`test ${name}`

```
函数重载：ts中的函数重载与Java中的重载不太一样。
```ts
function getInfo(name :string) :string;
function getInfo(age :number) :string;

function getInfo(str :any) :any {
    if ( typeof str === 'string') {
        return "我叫：" + str;
    } else {
        return "我的年龄是：" + str;
    }
}
console.log(getInfo("张三"));
console.log(getInfo(12));
```
## 四 类 class
### 4.1 类的定义
```ts
class Person {

    name :string; 
    age :number; 
    
    constructor(a:string, b:number){ 
        this.name = a;
        this.age = b;
    }
    
    info() :void {
        console.log(this.name + "的年龄是：" + this.age);
    }
}
    
let p = new Person('张三',40);
p.info();
```
注意：TS一样支持 ES6中的 extends static 关键字
### 4.2 属性修饰符
```
public：		默认的修饰符，可以不写，在类里面、子类、类外面都可以访问
private：	    在类里面可以访问，子类、类外部都没法访问
protected：	    在类里面、子类里面可以访问，类外部无法访问
```
注意：如果没有修饰符，则默认都是public，但是在构造函数中，有所区别。
如果构造函数的参数没有修饰符，那么在构造函数外部是无法使用这个参数的；
如果构造函数的参数使用了public修饰符，那么在构造函数外部可以使用该参数
### 4.3 多态
### 4.4 抽象类
抽象类和抽象方法用来定义标准。抽象方法不包含具体的实现。
注意：抽象类不能实例化，只有其子类才能实例化。
```ts
abstract class Animal {
    abstract eat() :any;    //抽象方法只能出现在抽象类中
}
```
## 五 接口
### 5.1 接口简介
抽象类和抽象方法虽然提供了标准，但是只针对类中的某个函数设立了规范。
接口对类本身设立了规范。
### 5.2 属性接口
属性接口更多是对json对象提供了约束。
```ts
interface Person {
    name:string;
    age:number;
}

class Student {
    //构造函数支持接口
    constructor(public config:Person){
    }
}

//在创建对象时，必须满足接口定义的属性
let s1 = new Student({
    name:'lisi',
    age:18
});
```
### 5.3 类实现接口
```ts
interface Person {
    name:string;
    age:number;
    run() :void;
}
class Student implements Person{
    name:string = '';
    age:number = 0;
    constructor(){
        
    }
    //实现接口方法
    run(){
        console.log(this.name + ' run....');
    }
}
let s1 = new Student();
s1.name = 'lisi';
s1.age = 15;
s1.run();
```
### 5.4 函数接口
```ts
interface encrypt {
    (key:string, val:string) :string;
}

let md5:encrypt = function(key:string, val:string) :string {
    return key+val;
}

console.log("name","zs");
```
### 5.5 可索引接口
可索引接口可以用来对数组的索引进行限制，如果数组中的索引传入了其他类型，那么报错：
```ts
interface UserArr {
    [index:number] :string
}
let arr:UserArr = ['aaa','bbb'];
console.log(arr[0])
```
可索引接口也可以对对象进行限制，如果json中多写了字段，则报错：
```ts
interface UserObj {
    [index:string] :string;
}
let u:UserObj = {
    name:"lisi"
}
```
### 5.6 接口可以被继承
```ts
interface Animal {
    eat() :void;
}

interface Person extends Animal {
    work() :void;
}
```
## 六 泛型
### 6.1 泛型简介
在软件工程中，我们不仅要创建一致的API，也要考虑可重用性，组件不仅能够支持当前的数据类型，同事也支持未来的数据类型。
通俗的理解：泛型就是提升 类 接口 方法的复用性，以支持不特定的数据类型
### 6.2 应用场景
```ts
function f1(value: string): string {
    return value;
}

function f2(value: number): number {
    return value;
}

//上述的明显函数的创建显得累赘，通过泛型解决
function fn<T>(value: T): T {
    return value;
}
console.log(fn<number>(123));
console.log(fn<string>("test"));
```
### 6.3 泛型应用于类
```ts
class Hello<T> {
    public list:T[] = [];
    add(val:T) :void {
        this.list.push(val);
    }
}
var h = new Hello<number>();    //让类型为number
```
同理，泛型可以应用于类，函数
### 七 注解
注解为程序的元素（类、方法、变量）加上更直观更明了的说明，这些说明信息与程序的业务逻辑无关，而是供指定的工具或框架使用。

### 八 类型定义文件
类型定义文件后缀为 .d.ts 。
类型定义文件用来帮助开发者在TS中使用已有的JS工具包，如：jQuery。
这个文件其实是一个typescript文件模块，提供给开发者直接使用，比如引入jQuery的类型定义文件后，就可以使用$与jQuery的方法。












