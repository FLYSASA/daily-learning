## 构建工具：

参考： https://zhuanlan.zhihu.com/p/44438844

### 1. webpack与grunt、gulp的不同？
共同点： 三者都是前端构建工具。
差异点：
1. webpack相对来说比较主流，不过一些轻量化的任务还是会用gulp来处理，比如单独打包css文件等。

2. grunt和gulp是基于任务和流的。找到一个（或一类）文件，对其做一系列链式操作，更新流上的数据，整条链式操作构成一个任务，多个任务就构成了整个web构建流程。

3. webpack是基于入口的。其会自动的递归解析入口所需要加载的所有资源文件，然后用不同的Loader来处理不同的文件，不同plugin来扩展webpack功能。


### 2. webpack

#### 1. Loader和Plugin的不同？

- **Loader**: 加载器。**让webpack拥有加载和解析非JavaScript文件的能力。**
webpack将一切文件视为模块，但是原生webpack只能解析js文件，如果想将其它文件打包的话就要用到loader。

- **Plugin**: 插件。**Plugin可以扩展webpack的功能，让webpack具有更多的灵活性。**
webpack运行的生命周期中会广播许多事件，Plugin可以监听这些事件，在合适的时机通过webpack提供的API改变输出结果。


#### 2. 有哪些常见的Loader? 他们是解决什么问题的？

- **file-loader**: 把文件输出到一个文件夹中，在代码中通过相对URL去引用输出的文件。比如在html引入image的src，打包后图片路径是不一样的，file-loader会修改打包后的图片储存路径。
- **url-loader**: 和file-loader类似，但是能在文件很小的情况下以base64的方式把文件内容注入到代码中去。
- **source-map-loader**: 加载额外的 Sourse Map文件，以方便断点调试
- **image-loader**: 加载并且压缩图片文件
- **babel-loader**: 把ES6 转换成 ES5
- **css-loader**: 加载css，支持模块化、压缩、文件导入等特性
- **style-loader**：把css代码注入到 JavaScript，通过DOM操作去加载css
- **eslint-loader**：通过ESLint 检查 JavaScript 代码

#### 3. 有哪些常见的Plugin? 他们是解决什么问题的？
- **define-plugin**：定义环境变量
- **commons-chunk-plugin**: 提取公共代码
- **uglifyjs-webpack-plugin**: 通过 uglifyES 压缩 ES6 代码

#### 4. 如何在vue项目中实现按需加载？
Vue UI组件库按需加载，为了快速开发前端项目，经常引入现成的UI组件库如ElementUI、iView等，整个引入的体积是很庞大的。通常我们引入常用的几个组件就可以了。

很多组件库以及提供了现成的解决方案，如 Element 的 `babel-plugin-component` 和 AntDesign 的 `babel-plugin-import` 安装插件后，在 `.babelrc` 配置中或`babel-loader`的参数中进行设置，即可实现组件的按需加载。