## 一 单元测试

### 1.0 单元测试概念

单元测试是软件质量保证最重要的一环。在编写可测试代码时，可以遵循以下原则：
- 单一职责：代码段的职业越多，单元测试就需要构造更多的输入，同时也会影响代码的后期维护
- 接口抽象：通过对程序代码进行接口抽象后，可以针对接口进行测试，而具体代码的变化则不会影响为接口编写的单元测试
- 层次分离：层次分离其实是单一职责原则在项目上的一个整体实现，单元测试应该保证可以逐层测试，逐层保证。

单元测试主要包含：断言、测试框架、测试用例、测试覆盖率、mock模拟异常、持续集成等几个方面。

### 1.1 断言

断言用来检测程序在运行时是否符合期望。Node原生模块 assert 已经实现了断言相关机制：
```js
var assert = require('assert');
assert.equal(Math.max(1, 100), 100);
```
一旦 `assert.equal()` 方法不满足期望，将会抛出 AssertionError 异常，整个程序将会停止执行。assert模块提供了大量判断相关的断言方法，目前市面上的第三方断言库、单元测试库也大多基于该模块构建。 

### 1.2 测试框架

断言遇到检查失败，会退出整个程序，对大型项目的测试是不友好的。更好的办法是：出现异常时，记录下该异常并生成测试报告。这里要提出的是，不同的测试风格有不同的组织方式，当下流行的单元测试风格有：
- TDD：测试驱动开发。关注所有功能是否被正确实现，每一个功能具备对应的测试用例。TDD的表达方式偏向于功能说明书。
- BDD：行为驱动开发。关注整体行为是否符合预期，适合自顶向下的设计方式。BDD的表达方式接近于自然语言。

流行的单元测试框架mocha等都具备该功能，安装方式：
```
npm i mocha -g
```

mocha对TDD方式的支持：采用suite和test完成，suite实现了多层级描述，测试用例使用test。提供的钩子函数有setup、teardown，分别表示suite前、suit后执行。
```js
suite('Array', function(){

    setup(function(){
    // ...
    });

    suite('#indexOf()', function(){
        test('should return -1 when not present', function(){
            assert.equal(-1, [1,2,3].indexOf(4));
        });
    });
});
```

mocha对BDD方式的支持：主要采用describe和it进行组织，describe可以描述多层级结构，具体到测试用例时，可以使用it。另外内部提供了before、after、beforeEache、afterEach这4个钩子函数，用于协助describe中测试用例的准备、安装、卸载、回收工作。before和after分别在进入和退出describe时触发，beforeEach和afterEach分别在describe中每一个测试用例（it）执行前、执行后触发。
```js
describe('Array', function(){

    before(function(){
    // ...
    });

    describe('#indexOf()', function(){
        it('should return -1 when not present', function(){
            [1,2,3].indexOf(4).should.equal(-1);
        });
    });
});
```

mocha框架与断言之间是解耦的，既可以使用Node原生的assert模块，也可以使用should.js、chai等第三方断言库。  

mocha导出测试报告也支持多种格式，一般使用json格式。  

mocha对异步的支持：
```js
it('fs.readFile should be ok', function (done) {
    fs.readFile('file_path', 'utf-8', function (err, data) {
        should.not.exist(err);
        done();             
    });
});
```

## 二 性能测试

### 2.1 基准测试

基准测试用来统计多少时间内执行了多少次某个方法。假设要测试ES5中的Array.prototype.map和循环提取值两种方式，他们都是迭代一个数组，根据回调函数执行的返回值得到的一个新的数组，相关代码如下：
```js
var nativeMap = function (arr, callback) {
    return arr.map(callback);
};

var customMap = function (arr, callback) {
    var ret = [];
    for (var i = 0; i < arr.length; i++) {
        ret.push(callback(arr[i], i, arr));
    }
    return ret;
};
```

比较简单直接的方式就是构造相同的输入数据，然后执行相同的次数，最后比较时间，所以可以写一个方法来执行：
```js
var run = function (name, times, fn, arr, callback) {
    var start = (new Date()).getTime();
    for (var i = 0; i < times; i++) {
        fn(arr, callback);
    }
    var end = (new Date()).getTime();
    console.log('Running s d times cost d ms', name, times, end % % % - start);
};
```

最后分别调用1000000次：
```js
var callback = function (item) {
    return item;
};
run('nativeMap', 1000000, nativeMap, [0, 1, 2, 3, 5, 6], callback);
run('customMap', 1000000, customMap, [0, 1, 2, 3, 5, 6], callback);
```

由上看出，执行相同的任务，map耗费的时间大约是直接for循环的5-7倍以上。  

比较好的基准测试框架式benchmark第三方模块。

### 2.2 压力测试

基准测试一般是对基本的方法记性测试，压测一般是对网络接口进行压力测试。压测需要考察的指标有：吞吐率、响应时间、并发数。  

常用的压测工具有：ab、http_load，使用ab工具如下：
```
# 10个并发用户持续3秒向服务端发送请求
ab -c 10 -t 3 http://localhost:8001/
```