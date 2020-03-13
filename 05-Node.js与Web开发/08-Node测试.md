## 一 测试的概念

### 1.1 TDD与BDD

- TDD：测试驱动开发，倡导首先书写测试程序，然后编码实现功能
- BDD：行为驱动开发，鼓励开发人员、测试人员等进行协作

### 1.2 单元测试

单元测试也成为模块测试，是针对某个具体的功能（函数）进行的测试，

### 1.3 集成测试

对项目整体进行测试

## 二 常见测试库

### 2.1 Chai断言库

Node内置了断言库Assert，但是使用不太方便。  

单元测试中，推荐使用Chai。Chai包含三个断言库，有BDD风格的Expect/Should，TDD风格的Assert。  

在实际开发中，Chai需要配合测试框架如Mocha一起使用。
```js
const { assert, expect } = require("chai");
const should = require("chai").should();

const foo = 'bar';
const beverage = { 
    tea: ['chai', 'match', 'oolong']
};

// TDD风格
assert.typeOf(foo, 'string')
assert.equal(foo, 'bar', 'foo equal bar...');
assert.lengthOf(foo, 3, `foo's length is 3`);
assert.lengthOf(beverage.tea, 3, `beverage have 3 types`);

// BDD风格
expect(foo).to.be.a('string');
expect(foo).to.equal('bar');
expect(foo).to.have.lengthOf(3);
expect(beverage).to.have.property('tea').with.lengthOf(3);


// BDD风格
foo.should.be.a('string');
foo.should.equal('bar');
foo.should.have.lengthOf(3);
beverage.should.have.property('tea').with.lengthOf(3)
```

上述文件运行后，不会有任何输出，但是当修改foo或者beverage的值与数量后，则会输出上述内容。

### 2.2 Mocha测试框架

```js
const should = require("chai").should();

const foo = 'bar';
describe('String', ()=>{
    it('foo should be string', ()=>{
        foo.should.be.a('string');
    });
});
```


### 2.3 Nyc测试覆盖率

