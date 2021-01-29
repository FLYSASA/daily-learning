### 如何将设置less样式文件引入的根目录？

1. 修改 `vue.config.js` 文件
将src目录作为less的根目录
![](1.png)

2. 其它文件中less引入
![](2.png)

3. docs里的问题
![](3.png)

这里改为相对路径引入就没问题了：
![](4.png)



附上：
[github有色图标emoji](https://gist.github.com/parmentf/035de27d6ed1dce0b36a)