## 一 path模块

API地址：http://nodejs.cn/api/path.html

该API用户获取路径：
```JavaScript
const path = require('path');
let myurl = path.join(__dirname, 'username', '123');
console.log(myurl);     //输出类似这样的绝对路径 E:\ProjectWeb\Test\username\123
```

__dirname 和 path.dirname 的区别:
```js
console.log(__dirname);  // 指向被执行 js 文件的绝对路径,包含的有文件名称
console.log(path.dirname('/abc/www/abc.txt'));  // 只有路径，没有文件名称
```

获取路径的最后一部分：
```js
path.basename('/foo/bar/aaa.html')              // 得到的结果是： aaa.html

path.basename('/foo/bar/aaa.html', '.html')     // 得到的结果是： aaa
```

获取扩展名:
```js
// 如果后面只是个‘.’，那么得到的就是‘.’； 如果没有扩展名，得到的就是空
path.extname('index.html')                      // 得到的结果： ‘.html’
```

路径的格式化处理：
```js
    let obj = {
        root: 'd:\\',
        base: 'abc.txt',
        ext: '.txt',
        name: 'abc'
    }

    console.log(path.format(obj));       //  输出的结果：d:\abc.txt
```


路径字符串转成对象
```js
    console.log(path.parse(__filename));
   //  输出的结果：
    {
        root: '文件根路径',
        dir: '文件的全路径',
        base: '文件名称',
        ext: '扩展名',
        name: '文件名称'
    }
```

连接路径：
```js
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..');
// 返回: '/foo/bar/baz/asdf'

path.join('foo', {}, 'bar');
// 抛出 'TypeError: Path must be a string. Received {}'
```