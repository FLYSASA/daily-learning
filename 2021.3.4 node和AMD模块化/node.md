# node 

### 1. node api
[Api链接](https://nodejs.org/dist/latest-v14.x/docs/api/fs.html#fs_fs_read_fd_options_callback)

### 2. 区分dependencies 和 devDependencies
dependencies 依赖的库。
devDependencies 安装测试库与功能无关不给用户直接使用。


### 3. 导出和引入
https://nodejs.org/dist/latest-v14.x/docs/api/modules.html#modules_module_exports
```js
// a.js 
// 导出
function add(i) {
  console.log(i + 1)
}
module.exports = add

// b.js
// 引入
const add = require('./a')
add(1)
```

### 4. 模块化
痛点： 为了防止各个文件之间互相影响，所以有了模块化的概念。

#### ① CommonJS规范
CommonJS规范是由NodeJS发扬光大，它有一些特性如下：
1. 定义模块 根据CommonJS规范，一个单独的文件就是一个模块。每一个模块都是单独的作用域，也就是说在该模块内部定义的变量，无法被其他模块读取，除非定义为global对象的属性。
2. 模块输出： 模块只有一个出口， `module.export` 对象, 我们需要把模块需要输出的内容放入该对象。
3. 加载模块： 加载模块使用 `require` 方法，该方法读取一个文件并执行，返回文件内容的 `module.exports`对象。

举例：
```js
// 模块定义 model.js
var name = 'sasa'
function printName () {
  console.log(name)
}

module.exports = {
  printName: printName
}

// 加载模块 a.js
var nameModule = require('./model.js');

nameModule.printName()

```

一般可以省略js扩展名，可以使用相对路径也可以使用绝对路径。

**CommonJS在浏览器中的弊端：**
> require是同步的。但在浏览器端加载js最佳、最容易的方式是document里插入script标签。但脚本标签天生异步，不能保证require的模块已经加载，所以传统CommonJS模块在浏览器环境中无法正常加载。

### ② AMD规范
AMD: Asynchronous Module Definition, 异步模块定义。
由于不是JavaScript原生支持，使用AMD规范进行页面开发需要对应的库函数，也就是大名鼎鼎的 `RequireJS`。

requireJS主要解决两个问题：
> 1. 多个js文件可能有依赖关系，被依赖的文件需要早于依赖它的文件加载到浏览器
> 2. js加载的时候浏览器会停止页面渲染，加载文件越多，页面失去响应时间越长

使用requireJS的例子：
```js
// 导出模块 myModule.js
define(['dependency'], function() {
  var name = 'sasa'
  function printName() {
    console.log(name)
  }
  return {
    printName: printName
  }
})

// 加载模块 a.js
require(['myModule'], function(my){
  my.printName()
})
```

**语法**：
```js
// 定义函数：
define(id?, dependencies?, factory)
require([dependencies], function)
```

### ③ CMD规范
CMD 即 Common Module Definition 通用模块定义。
CMD基于 **SeaJS** 实现的，只不过在模块定义方式和模块加载时机上有所不同。

**语法**：
```js
// 定义函数：
define(id?, deps?, factory)
```

特点：
> 1. 一个文件一个模块，所以经常用文件名作为模块id
> 2. CMD推崇依赖就近，所以一般不在define的参数中写依赖，在factory中写。
factory有三个参数：
`function (require, exports, module)`

例子：
```js
// 定义模块  myModule.js
define(function(require, exports, module) {
  var $ = require('jquery.js')
  $('div').addClass('active');
});

// 加载模块
seajs.use(['myModule.js'], function(my){

})
```

### ③ AMD 与 CMD 区别
1. AMD推崇依赖前置，在定义模块的时候就要声明其依赖的模块
2. CMD推崇就近依赖，只有在用到模块的时候在去require


参考：https://juejin.cn/post/6844903541853650951