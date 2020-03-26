## 一 Object

## 1.1 object数据类型

object数据类型可以通过 Object对象来创建，Object对象可以视为所有对象的祖先对象（基本类），创建方式：
```js
var obj1 = new Object();
var obj2 = new Object;          // 有效，但是不推荐该方式
```

### 1.2 Object对象实例

由于Object对象是所有对象的祖先对象，所以其属性和方法，其他对象都会拥有：
- constructor：保存着用于创建当前对象的函数。对于前面的例子而言，构造函数（ constructor）就是 Object()。
- hasOwnProperty(propertyName)：用于检查给定的属性在当前对象实例中（而不是在实例的原型中）是否存在。其中，作为参数的属性名（ propertyName）必须以字符串形式指定（例如： o.hasOwnProperty("name")）。
- isPrototypeOf(object)：用于检查传入的对象是否是传入对象的原型
- propertyIsEnumerable(propertyName)：用于检查给定的属性是否能够使用 for-in 语句来枚举。与 hasOwnProperty()方法一样，作为参数的属性名必须以字符串形式指定。
- toLocaleString()：返回对象的字符串表示，该字符串与执行环境的地区对应。
- toString()：返回对象的字符串表示。
- valueOf()：返回对象的字符串、数值或布尔值表示。通常与 toString()方法的返回值

## 二 全局对象

JS已经提供许多对象，包括一些全局对象（global object）在JS中可以直接使用，每当JS的引擎启动时，或者是网页加载新页面时，就会创建一个新的全局对象，并给与一组定义好的初始属性。  

全局对象（global object）在JS中可以直接使用：
- 全局属性：undefined、Infinity、NaN等
- 全局函数：isNaN()、parseInt()、eval()、encodeURI()和 encodeURIComponent()方法等
- 构造函数：Date()、RegExp()、String()、Array()等
- 全局对象：Math、JSON等

ECMAScript没有给出如何直接访问全局对象的方式，但是Web浏览器中全局对象被包含在了window对象中。

## 三 常见方法

### 3.1 toString()

toString()方法用来返回对象实例的字符串，但是默认情况下返回值信息量极少：
```js
var p = {
    name: "lisi"
}

console.log(p.toString());      // 输出 [object Object]
```

很多类都会带一些自定义的toString()，以强化该方法的能力：
```js
console.log([1,2,3].toString());      // 输出 1,2,3
console.log((new Date).toString());      // 输出 Tue Nov 12 2019 17:18:31 GMT+0800 (China Standard Time)
console.log((function(){}).toString());      // 输出 该函数字符串
```

### 3.2 toLocaleString()

Object中默认的`toLocaleString()`方法仅仅调用 `toString()`方法并返回对应值，不做本地化操作。Date和Number做本地化转换：
```js
console.log((new Date).toLocaleString());      // 输出 11/12/2019, 5:21:07 PM
```

### 3.3 valueOf()

`valueOf()`方法与`toString()`类似，但是在转换类似数字时，会仍然保留其数字类型。
