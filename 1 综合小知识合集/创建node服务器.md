## 一.创建一个node服务器

#### 1. 新建一个node-server文件夹，npm 初始化
```
npm init -y
```
![](./node服务器/初始化1.png)

#### 2. 安装依赖包
```
npm i express multer cors
```
![](./node服务器/初始化2.png)

#### 3. 新建`index.js`文件
![](./node服务器/初始化3.png)

访问： `http://127.0.0.1:3000/` 成功。
![](./node服务器/初始化4.png)

![](./node服务器/初始化5.png)

#### 4. 让node-server支持文件
![](./node服务器/初始化6.png)

#### 5. 客户端发起file上传请求
这里请求200成功
![](./node服务器/初始化7.png)

虽然请求200但是，`req.file`一直为undefined，服务端无法获取到上传的文件资源。
![](./node服务器/初始化8.png)


#### 6. form请求表单设置
##### 客户端
```js
<form action="http://127.0.0.1:3000/upload" method="post" enctype="multipart/form-data">
  <input type="file" name="xxx">
  <input type="submit" value="提交">
</form>
```

##### 服务端
```js
app.post('/upload', upload.single('xxx'), (req, res)=>{
  console.log(req.file)
  res.send('hello server')
})
```

![](./node服务器/初始化9.png)


### 综上： 如何将file文件上传到服务器？
https://jsbin.com/nesavozani/1/edit?html,output
#### 客户端
![](./node服务器/file上传到服务端1.png)

#### 服务端
![](./node服务器/file上传到服务端2.png)

---
### 二.form表单的方式会重新刷新页面，改为ajax进行文件上传
[FormData 对象的使用](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/Using_FormData_Objects)
```js
f.addEventListener('submit', (e)=>{
  e.preventDefault();  // 阻止默认的表单提交
  var formData = new FormData();
  var fileInput = document.querySelector('input[name="xxx"]')
  formData.append('xxx', fileInput.files[0]);
  
  var xhr = new XMLHttpRequest()
  xhr.open('post', f.action)
  xhr.onload = function(){
    console.log(xhr.response)
  }
  xhr.send(formData)
})
```

此时点击提交会报跨域：
![](./node服务器/file上传到服务端3.png)

使用`express` 给响应头加上 `Access-Control-Allow-Origin`:
![](./node服务器/file上传到服务端4.png)
![](./node服务器/file上传到服务端5.png)

```js
// 所有都可以访问
res.set('Access-Control-Allow-Origin', '*')
// 只对来自某个特定域名允许访问
res.set('Access-Control-Allow-Origin', 'https://null.jsbin.com')
```

## 三. 服务器回传文件对象，并显示在前端
https://jsbin.com/votejenida/1/edit?html,js,output
**客户端：**
![](./node服务器/file上传到服务端6.png)

**服务端：**
```js
app.get('/preview/:key', (req, res)=>{
  // 获取到前端给的参数后，去发送文件
  res.sendFile(`yyy/${req.params.key}`, {
    root: __dirname,
    headers:{
      'Content-Type': 'image/jpeg',
    },
  }, (error)=>{
    if(error){
      res.status(404).send('Not found')
    }
  })
})
```


## 三. 优化服务器
#### 1. 优化cors跨域:
![](./node服务器/file上传到服务端7.png)


#### 2. 使用heroku作为服务器
https://www.heroku.com/
它可以创建一个node服务器。

##### 1. 创建hero账号：
![](./node服务器/1heroku服务器1.png)

##### 2. 创建app
![](./node服务器/1heroku服务器2.png)
命名通过后点击创建app:
![](./node服务器/1heroku服务器3.png)

##### 3. 关联github
![](./node服务器/1heroku服务器4.png)

1.将node-server代码上传至github
![](./node服务器/1heroku服务器5.png)

> 上传至github具体步骤：
> 1. 将本地node-server文件夹 `git init`
> 2. 新建.gitignore, 排除掉 node_modules/ 和 uploads/文件夹
> 3. `git add .`
> 4. `git commit -m 'init'`
> 5. `git remote add origin git@github.com:xxx`
> 6. `git push -u origin master`

2. 在heroko里找到对应仓库连接
![](./node服务器/1heroku服务器6.png)


3. 生成端口号
因为heroku的端口和我们的端口可能不一致，所以需要更改：
![](./node服务器/1heroku服务器7.png)

4. heroku部署node-server
![](./node服务器/1heroku服务器8.png)
部署失败：
![](./node服务器/1heroku服务器9.png)

5. 添加指令，重新推送代码
![](./node服务器/1heroku服务器10.png)

6. heroku重新部署
![](./node服务器/1heroku服务器11.png)
点击view, 运行成功：
![](./node服务器/1heroku服务器12.png)

7. 测试上传成功

https://jsbin.com/votejenida/1/edit?html,js,output
将上面的url地址复制：
![](./node服务器/1heroku服务器13.png)