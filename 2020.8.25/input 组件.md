1. Vue组件标签的html语法
自建组件的标签不能直接简化为自闭合标签，如下面的`<g-input></g-input>`不能简化成`<g-input/>`, 因为vue只支持html语法，而不是xml语法
![5](./5.png)

2. 组件`name`的定义
配合chrome插件 `vuetool` 可以看到，组件的名称变成了我们定义的name
![5](./6.png)