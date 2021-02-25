### Http详解

参考：https://zhuanlan.zhihu.com/p/23299600?refer=study-fe
http://book.jirengu.com/fe/%E4%B8%93%E9%A2%98%E6%89%A9%E5%B1%95/HTTP%E4%B8%93%E9%A2%98/url.html

HTTP(HyperText Transfer Protocal): **超文本传输协议**，是一种用于分布式、协作式和超媒体信息系统的应用层协议。

设计HTTP最初的目的是为了**提供一种发布和接收HTML页面的方法**。通过HTTP或者HTTPS协议请求的资源由统一资源标识符（Uniform Resource Identifiers，URI）来标识。


### web本质：
- 用户请求远程资源
- 浏览器查找远程资源，打包用户请求并发送
- 服务器根据用户请求的资源路径及附带参数，配合自身逻辑生成相关内容，发送给浏览器
- 浏览器解析结果，翻译为直观方式呈现

### URI和URL:
- URI(Uniform Resource Identifier): 统一资源标识符
- URL: 统一资源定位符
与URI相比我们更熟悉URL，URL是使用浏览器等访问web页面的时候需要输入的网页地址：
`http://www.baidu.com`

URI是更通用的资源标识符，URL是它的一个子集

URI由两个主要的子集构成:

1. URL：通过描述资源的位置来描述资源

2. URN：通过名字来识别资源，和位置无关

url里的参数：
例如： http://sj.gtcloud.cn/sso.html?/operation/index.html#/tasksTotalBrowse/3b515acf-24bd-406d-9331-4a167e66af7c?type=personal&isFromBPM=1&return_type=pageUrl
```js

host: 主机名 + 端口                  	// "sj.gtcloud.cn:80"
hostname: 主机名			// "sj.gtcloud.cn"
port: 端口号				// "80"
hash: #开始的锚		 // "#/tasksTotalBrowse/3b515acf-24bd-406d-9331-4a167e66af7c?type=personal&isFromBPM=1&return_type=pageUrl"
search: 以问号开头的参数，到了#号会截掉,没有#号会一直到末尾 // "?/operation/index.html"
origin: 返回URL的协议,主机名和端口号    // "http://sj.gtcloud.cn"
pathname: URL 的相对路径	 // "/sso.html"
```