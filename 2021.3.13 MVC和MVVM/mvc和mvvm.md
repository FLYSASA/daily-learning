
### MVC
http://www.ruanyifeng.com/blog/2007/11/mvc.html

MVC: Model(模型)-View(视图)-Controller(控制)

MVC 是桌面程序，由微软开发。后来后端引入这个概念。

从结构上看，都可以分成三层。
1）最上面的一层，是直接面向最终用户的"视图层"（View）。它是提供给用户的操作界面，是程序的外壳。

2）最底下的一层，是核心的"数据层"（Model），也就是程序需要操作的数据或信息。

3）中间的一层，就是"控制层"（Controller），它负责根据用户从"视图层"输入的指令，选取"数据层"中的数据，然后对其进行相应的操作，产生最终结果。

### MVVM

MVVM: Model-View-ViewModel 
MVVM最早由微软提出来，它借鉴了桌面应用程序的MVC思想, 把Model用纯JavaScript对象表示，View负责显示，两者做到了最大限度的分离。
把Model和View关联起来的就是ViewModel。ViewModel负责把Model的数据同步到View显示出来，还负责把View的修改同步回Model。

MVVM的设计思想：关注Model的变化，让MVVM框架去自动更新DOM的状态，从而把开发者从操作DOM的繁琐步骤中解脱出来！

缺点： 
1. model修改后，操作dom会有渲染性能问题。
2. 无法检测属性和数组的变化。



mvc与mvvm的区别：
参考： vue.js mvvm 1.33分
1. mvc将操作dom的事件封装，只需要告知容器事件等，监听事件需要自己进行
2. mvc有个需要主动渲染render的操作，而mvvm会帮你渲染