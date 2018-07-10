# Mocha, js&node测试框架

> Mocha is a feature-rich JavaScript test framework running on Node.js and in the browser

## 文档在前头
- [Mocha的主页](https://mochajs.org/)
- [Mocha的Github](https://github.com/mochajs/mocha)

## 安装

可以作为全局工具，也可以仅项目dev工具

```
$ npm install --global mocha

$ npm install --save-dev mocha

```

## 使用

在项目目录下新建test文件夹

```
mkdir test
```

在里面放置自己的测试脚本

```
//repos\test\test.js

var assert = require('assert');
describe('Array', function() {
  describe('#indexOf()', function() {
    it('should return -1 when the value is not present', function() {
      assert.equal([1,2,3].indexOf(4), -1);
    });
  });
});

```

## 断言库

测试用例需要断言库来判断测试结果，mocha本身不带断言库，用户可以根据自己的需求使用如下：

- assert
- should
- chai
- better-assert
- expect

