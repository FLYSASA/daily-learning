### npm发布包

#### 1. 新增`.npmignore`
```
/node_modules/     // 前面加 / 代表根目录下， 后面加 / 代表目录
/.cache/
/docs/
/src/
/tests/
```

#### 2. 执行 npm publish
1. 报错：私人库的问题
![](2npm发布1.png)
**解决：去掉`package.json`里的 `private: true`**
![](2npm发布2.png)

2. 报错：报淘宝源无权限的问题
![](2npm发布3.png)
**解决：设置回淘宝源**
`npm config set registry https://registry.npmjs.org`

3. 报错：Not-found
![](2npm发布4.png)
![](2npm发布5.png)
**解决：登录账号**
![](2npm发布6.png)

4. 报错：403 Forbidden
![](2npm发布8.png)
**解决：更新版本**
![](2npm发布9.png)

5. 发布成功
`npm ublish`
![](2npm发布7.png)


## 测试发布的包：
### 1. 创建临时文件夹
![](3测试发布的包1.png)

### 2. 使用vue初始化项目
1. 其实可以直接 `vue create vue-demo`
![](3测试发布的包2.png)
2. `npm i koma-ui` 安装包
![](3测试发布的包3.png)
![](3测试发布的包4.png)

3. demo使用包报错
这是因为没有在koma-ui里定义入口文件
![](3测试发布的包8.png)


3. 在koma-ui里定义入口文件
![](3测试发布的包6.png)
记得重新打包，重新发布

4. 在demo里重新更新koma-ui包
![](3测试发布的包7.png)

5. 重新引入，引入成功~
![](3测试发布的包9.png)

6. 如何使用koma-ui的组件
**koma-ui里的组件导出的时候命名是Button, 所以demo里引入的时候要先引入Button,然后重命名自己的命名。**
![](3测试发布的包11.png)
```js
import {  Button as GButton } from 'koma-ui'  // 因为koma-ui里的重命名为GButton
import 'koma-ui/dist/koma.css'
```
![](3测试发布的包10.png)


