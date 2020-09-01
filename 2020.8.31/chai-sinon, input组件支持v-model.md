# 关于测试用例语法使用的详细解析

### 为什么可以支持 describe 和 it?

由mocha引入，直接挂载在window上的全局函数，可以直接使用。
![1](1.png)


### 为什么支持 sinon 和 expect ?

`sinon.fake()` 由sinon提供， `expect`语法由chai提供， `calledWith`由 sinon-chai 提供
![2](2.png)