## 一 Array的基础使用

### 1.1 Array对象创建

Object对象是JavaScript所有对象的祖先对象，通过该对象也派生了一部分子数据类型，如Array、Function、Date等。  

Array即JavaScript的数组，和其他静态编程语言不同的是，JavaScript的Array对象可以保存任意类型数据，且长度不固定（内部自动扩容缩容）。  

创建Array有多种方式：
```js
var arr1 = new Array();
var arr2 = new Array(1, 2, 3)
var arr3 = [];
var arr4 = [1, 2, 3];

// 贴士：new操作符可以省略，不过不推荐
```

### 1.2 数组元素获取/设置

使用索引即可直接获取数组的元素：
```js
var arr = [1,3,6];
console.log(arr[1]);            // 3
```
注意：数组的首位元素是从0开始的！  

上述方法也可以用来设置数组的元素：
```js
var arr = [1,3,6];
arr[1] = 9;
console.log(arr[1]);            // 9
```

### 1.3 length属性

数组对象的length属性用于获取数组数目：
```js
var arr = [2,3,5];
console.log(arr.length);        // 3
```

当超出数组length位置被设置了数据，数组会重新计算length的值：
```js
var arr = [2,3,5];
console.log(arr.length);        // 3

arr[10] = 10;
console.log(arr.length);        // 11，此时3-10位置数据是不存在的，值都是undefined
```

### 1.4 push、pop、shift、unshift

数组对象的实例都具备push和pop方法，用来模仿数据结构栈、队列：
```js
var arr = [2,3,5];

// push：末尾追加元素
arr.push(7)              
console.log(arr);        // [ 2, 3, 5, 7 ]

// pop：末尾删除元素
arr.pop();
console.log(arr);        // [ 2, 3, 5 ]

// unshift: 首位添加元素
arr.unshift(1);
console.log(arr);       // [ 1, 2, 3, 5 ]

// shift：首位删除元素
arr.shift();
console.log(arr);        // [ 2, 3, 5 ]
```

### 1.5 数组排序

```js
var arr = [3, 7, 2, 5];

// 排序：
arr.sort();             // 默认由小到大排序
console.log(arr);       // [ 2, 3, 5, 7 ]

// 反转：
arr.reverse();
console.log(arr);       // [ 7, 5, 3, 2 ]
```

sort方法默认是由小到大排序，其内部会调用每个元素的toString()方法，然后得到可比较字符串，最后确定如何排序。但是会遇到下列问题：
```js
var arr = [0, 1, 5, 10, 15];

arr.sort();
console.log(arr);       // [ 0, 1, 10, 15, 5 ]
```
这里并未得到用户期望的数据。sort方法可以接收一个比较函数作为参数，以确保哪个值位于哪个值的前面：
```js
function compare(v1, v2){
    if(v1 < v2){
        return -1;
    } else if(v1 > v2){
        return 1;
    } else {
        return 0;
    }
}

var arr = [0, 1, 5, 10, 15];

arr.sort(compare);
console.log(arr);       // [ 0, 1, 5, 10, 15 ]
```

### 1.6 获取索引位置

ECMAScript 5 为数组实例添加了两个位置方法： indexOf()和 lastIndexOf()。这两个方法都接收两个参数：要查找的项和（可选的）表示查找起点位置的索引。其中， indexOf()方法从数组的开头（位置 0）开始向后查找， lastIndexOf()方法则从数组的末尾开始向前查找。

这两个方法都返回要查找的项在数组中的位置，或者在没找到的情况下返回1。在比较第一个参数与数组中的每一项时，会使用全等操作符；也就是说，要求查找的项必须严格相等：
```js
var numbers = [1,2,3,4,5,4,3,2,1];
alert(numbers.indexOf(4)); //3
alert(numbers.lastIndexOf(4)); //5
alert(numbers.indexOf(4, 4)); //5
alert(numbers.lastIndexOf(4, 4)); //3
var person = { name: "Nicholas" };
var people = [{ name: "Nicholas" }];
var morePeople = [person];
alert(people.indexOf(person)); //-1
alert(morePeople.indexOf(person)); //0
```

## 二 基于数组创建新数组

### 2.1 concat()

concat()方法可以基于当前数组中的所有项创建一个新数组。这个方法会先创建当前数组一个副本，然后将接收到的参数添加到这个副本的末尾，最后返回新构建的数组。在没有给 concat()方法传递参数的情况下，它只是复制当前数组并返回副本。如果传递给 concat()方法的是一或多个数组，则该方法会将这些数组中的每一项都添加到结果数组中。如果传递的值不是数组，这些值就会被简单地添加到结果数组的末尾：
```js
var colors = ["red", "green", "blue"];
var colors2 = colors.concat("yellow", ["black", "brown"]);
alert(colors); //red,green,blue
alert(colors2); //red,green,blue,yellow,black,brown
```

### 2.2 slice()

slice()能够基于当前数组中的一或多个项创建一个新数组。 slice()方法可以接受一或两个参数，即要返回项的起始和结束位置。在只有一个参数的情况下， slice()方法返回从该参数指定位置开始到当前数组末尾的所有项。如果有两个参数，该方法返回起始和结束位置之间的项——但不包括结束位置的项。注意， slice()方法不会影响原始数组。
```js
var colors = ["red", "green", "blue", "yellow", "purple"];
var colors2 = colors.slice(1);
var colors3 = colors.slice(1,4);
alert(colors2); //green,blue,yellow,purple
alert(colors3); //green,blue,yellow
```

splice()的主要用途是向数组的中部插入项，但使用这种方法的方式则有如下 3 种。
- 删除：可以删除任意数量的项，只需指定 2 个参数：要删除的第一项的位置和要删除的项数。例如， splice(0,2)会删除数组中的前两项。
- 插入：可以向指定位置插入任意数量的项，只需提供 3 个参数：起始位置、 0（要删除的项数）和要插入的项。如果要插入多个项，可以再传入第四、第五，以至任意多个项。例如，splice(2,0,"red","green")会从当前数组的位置 2 开始插入字符串"red"和"green"。
- 替换：可以向指定位置插入任意数量的项，且同时删除任意数量的项，只需指定 3 个参数：起始位置、要删除的项数和要插入的任意数量的项。插入的项数不必与删除的项数相等。例如，splice (2,1,"red","green")会删除当前数组位置 2 的项，然后再从位置 2 开始插入字符串"red"和"green"。

splice()方法始终都会返回一个数组，该数组中包含从原始数组中删除的项（如果没有删除任何项，则返回一个空数组）。
```js
var colors = ["red", "green", "blue"];
var removed = colors.splice(0,1); // 删除第一项
alert(colors); // green,blue
alert(removed); // red，返回的数组中只包含一项
removed = colors.splice(1, 0, "yellow", "orange"); // 从位置 1 开始插入两项
alert(colors); // green,yellow,orange,blue
alert(removed); // 返回的是一个空数组
removed = colors.splice(1, 1, "red", "purple"); // 插入两项，删除一项
alert(colors); // green,red,purple,orange,blue
alert(removed); // yellow，返回的数组中只包含一项
```

## 三 数组迭代

### 3.0 数组的简单迭代

使用for循环即可直接迭代数组：
```js
var arr = [1, 2, 3];
for(var i = 0; i < arr.length; i++){
    console.log(arr[i]);
}
```

ECMAScript 5 为数组定义了 5 个迭代方法。每个方法都接收两个参数：要在每一项上运行的函数和（可选的）运行该函数的作用域对象——影响 this 的值。传入这些方法中的函数会接收三个参数：数组项的值、该项在数组中的位置和数组对象本身:
- every()：对数组中的每一项运行给定函数，如果该函数对每一项都返回 true，则返回 true。
- filter()：对数组中的每一项运行给定函数，返回该函数会返回 true 的项组成的数组。
- forEach()：对数组中的每一项运行给定函数。这个方法没有返回值。
- map()：对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。
- some()：对数组中的每一项运行给定函数，如果该函数对任一项返回 true，则返回 true。

以上方法都不会修改数组中的包含的值。

### 3.1 every()和some()

在这些方法中，最相似的是 every()和 some()，它们都用于查询数组中的项是否满足某个条件。对 every()来说，传入的函数必须对每一项都返回 true，这个方法才返回 true；否则，它就返回false。而 some()方法则是只要传入的函数对数组中的某一项返回 true，就会返回 true。  

```js
var numbers = [1,2,3,4,5,4,3,2,1];

var everyResult = numbers.every(function(item, index, array){
    return (item > 2);
});

alert(everyResult); //false

var someResult = numbers.some(function(item, index, array){
    return (item > 2);
});

alert(someResult);
```

### 3.2 filter()

filter()函数，它利用指定的函数确定是否在返回的数组中包含某一项。例如，要返回一个所有数值都大于 2 的数组，可以使用以下代码:
```js
var numbers = [1,2,3,4,5,4,3,2,1];
var filterResult = numbers.filter(function(item, index, array){
    return (item > 2);
});
alert(filterResult); //[3,4,5,4,3]
```

### 3.3 map()

map()也返回一个数组，而这个数组的每一项都是在原始数组中的对应项上运行传入函数的结果。例如，可以给数组中的每一项乘以 2，然后返回这些乘积组成的数组:
```js
var numbers = [1,2,3,4,5,4,3,2,1];
var mapResult = numbers.map(function(item, index, array){
    return item * 2;
});
alert(mapResult); //[2,4,6,8,10,8,6,4,2]    
```

### 3.4 forEach()
forEach()只是对数组中的每一项运行传入的函数。这个方法没有返回值，本质上与使用 for 循环迭代数组一样。
```js
var numbers = [1,2,3,4,5,4,3,2,1];
numbers.forEach(function(item, index, array){
    //执行某些操作
});
```

## 四 Array对象的一些静态方法

### 4.1 检测数组

instanceof可以用来检测数据是否是数组类型：
```js
var arr = [1, 2, 3];
console.log(arr instanceof Array);          // true
```

instanceof 用于在一个执行环境中判断数组，如果在网页中包含多个框架，即多个全局执行环境，也即会存在多个不同版本的Array构造函数，那么从一个框架向另一个框架传入一个数组，那么传入的数组与在第二个框架中原生创建的数组分别具有各自不同的构造函数。  

ES5为了解决上述问题，增加了Array.isArray()方法，用于最终确定某个值到底是不是数组，而不管它是在哪个全局执行环境中创建的：
```js
var arr = [1, 2, 3];
console.log(Array.isArray(arr));          // true
```

## 五 归并

ECMAScript 5 还新增了两个归并数组的方法： reduce()和 reduceRight()。这两个方法都会迭代数组的所有项，然后构建一个最终返回的值。其中， reduce()方法从数组的第一项开始，逐个遍历到最后。而 reduceRight()则从数组的最后一项开始，向前遍历到第一项。  

这两个方法都接收两个参数：一个在每一项上调用的函数和（可选的）作为归并基础的初始值。传给 reduce()和 reduceRight()的函数接收 4 个参数：前一个值、当前值、项的索引和数组对象。这个函数返回的任何值都会作为第一个参数自动传给下一项。第一次迭代发生在数组的第二项上，因此第一个参数是数组的第一项，第二个参数就是数组的第二项。  

使用 reduce()方法可以执行求数组中所有值之和的操作，比如：
```js
var values = [1,2,3,4,5];

var sum = values.reduce(function(prev, cur, index, array){
    return prev + cur;
});

alert(sum); //15
```

reduceRight()的作用类似，只不过方向相反：
```js
var values = [1,2,3,4,5];

var sum = values.reduceRight(function(prev, cur, index, array){
    return prev + cur;
});

alert(sum); //15
```







