# get、post 请求中常见 content-type 请求头以及 nodeJs 解析请求参数

## form 标签的 enctype 属性的定义和用法
`enctype` 属性规定在发送到服务器之前应该如何对表单数据进行编码。
默认地，表单数据会编码为 `application/x-www-form-urlencoded`。就是说，在发送到服务器之前，所有字符都会进行编码（空格转换为 "+" 加号，特殊符号转换为 ASCII HEX 值）

值 | 描述
---|---
application/x-www-form-urlencoded | 在发送前编码所有字符（默认）
multipart/form-data	 | 不对字符编码，<br/>在使用包含文件上传控件的表单时，必须使用该值
text/plain | 空格转换为 "+" 加号，但不对特殊字符编码


> `HTTP/1.1` 协议规定的 HTTP 请求方法有 OPTIONS、GET、HEAD、POST、PUT、DELETE、TRACE、CONNECT 这几种。  
其中 POST 一般用来向服务端提交数据, GET 一般用来从指定的资源请求数据。  
本文主要讨论 POST 提交数据的几种方式及GET请求

## HTTP 协议数据格式

HTTP 协议是以 ASCII 码传输，建立在 TCP/IP 协议之上的应用层规范。
规范把 HTTP 请求分为三个部分：状态行、请求头、消息主体。结构类似如下：

 
```
<method> <request-URL> <version>
<headers>

<entity-body>
```
 
协议规定 POST 提交的数据必须放在消息主体（entity-body）中，但协议并没有规定数据必须使用什么编码方式。实际上，开发者完全可以自己决定消息主体的格式，只要最后发送的 HTTP 请求满足上面的格式就可以。

 
但是，数据发送出去，还要服务端解析成功才有意义。一般服务端语言如 php、python 等，以及它们的 framework，都内置了自动解析常见数据格式的功能。
服务端通常是根据请求头（headers）中的 Content-Type 字段来获知请求中的消息主体是用何种方式编码，再对主体进行解析。


所以说到 POST 提交数据方案，包含了 Content-Type 和消息主体编码方式两部分。

## 演示代码准备

这里我们基于 `node Express` 库搭建一个简单的服务器，用来解析页面发起的各种类型请求

环境要求：安装 `nodeJs`, 安装 web 服务启动工具 `npm install -g serve`


目录结构  
```
|____form-get
| |____index.html // get 请求 demo
|____index.html // web 服务入口文件
|____.gitignore
|____package-lock.json
|____package.json
|____form-data
| |____index.html // content-type => form-data 请求 demo
|____application-json
| |____index.html // content-type => application-json 请求 demo
|____form-urlencode
| |____index.html // content-type => application/x-www-form-urlencoded 请求 demo
| |____serve.js // node 服务
```

`serve.js` 创建服务 监听本地 `3111` 端口
```
const express = require('express')
const app = express()
const port = 3111
app.listen(port, () => {
  console.log(`Started at port ${port}`)
})
```

切换到当前目录  

启动 web 服务 `serve`, 默认会启用 `5000` 端口，浏览器访问 `localhost:5000`  

安装 npm 包依赖 `npm i`

启动 node 服务 `node serve.js`

## application/x-www-form-urlencoded

这应该是最常见的 POST 提交数据的方式了。浏览器的原生 `<form>` 表单，如果不设置 enctype 属性，那么最终就会以 `application/x-www-form-urlencoded` 方式提交数据

`form-urlencode/index.html`
```
  <!-- 注意必须是 post 请求 -->
  <form action="http://localhost:3111/form-urlencode" method="post" enctype="application/x-www-form-urlencoded">
    <p>First name: <input type="text" name="fname" /></p>
    <p>Last name: <input type="text" name="lname" /></p>
    <input type="submit" value="Submit" />
  </form>
```

浏览器访问 `localhost:5000/form-urlencode/index.html`并填写表单，点击`Submit`按钮

![图示](https://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/2020-4-9/url-encode-1.png)

`serve.js` 处理`/form-urlencode`请求的代码【示例给到的回包均为 JSON 格式】
```
const bodyParser = require('body-parser')

// 用来解析 form-urlencode body
const urlencodedParser = bodyParser.urlencoded({ extended: false })

app.post('/form-urlencode', urlencodedParser, function (req, res) {
  console.log('get application/x-www-form-urlencoded Params: ', req.body)
  res.json({ result: 'success', data: req.body })
})
```

此时Form提交的请求数据，抓包时看到的请求的 `Content-Type` 请求头值是`application/x-www-form-urlencoded`

提交的数据按照 `key1=val1&key2=val2` 的方式进行编码，key 和 val 都进行了 URL 转码, 点击下图 `Form data` 旁边的 `view source` 即可看到效果 `fname=Jack&lname=Chan`


表单提交的响应结果

![图示](https://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/2020-4-9/url-encode-2.png)

如果用 Ajax 代替 Form 提交数据，也是使用这种方式。
`Content-Type` 默认值都是`application/x-www-form-urlencoded;charset=utf-8`

## multipart/form-data

这又是一个常见的 POST 数据提交的方式。我们使用表单上传文件时，必须让 <form> 表单的enctype 等于 `multipart/form-data`。


`form-data/index.html`
```
  <!-- js中通过 new FormData() 发送数据 表单中通过以下demo -->
  <form action="http://localhost:3111/form-data" method="post" enctype="multipart/form-data">
    <p>name: <input type="text" name="name" /></p>
    <p>age : <input type="number" name="age" /></p>
    <input type="submit" value="Submit" />
  </form>
```

浏览器访问 `localhost:5000/form-data/index.html`并填写表单，点击`Submit`按钮

![图示](https://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/2020-4-9/form-data-1.png)

`serve.js` 处理`/form-data`请求的代码【示例给到的回包均为 JSON 格式】
```
const multipart = require('connect-multiparty')

// 用来解析 form-data body
const multipartMiddleware = multipart()

app.post('/form-data', multipartMiddleware, function (req, res) {
  console.log('get application/form-data Params: ', req.body)
  res.json({ result: 'success', data: req.body })
})
```


表单提交的响应结果

![图示](https://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/2020-4-9/form-data-2.png)

此时Form提交的请求数据，抓包时看到的请求的 `Content-Type` 请求头值是`multipart/form-data`

而且可以看到上图请求的`form data`：

```
------WebKitFormBoundaryTPENO2DooTSePmIO
Content-Disposition: form-data; name="name"

Jack
------WebKitFormBoundaryTPENO2DooTSePmIO
Content-Disposition: form-data; name="age"

30
------WebKitFormBoundaryTPENO2DooTSePmIO--
```
格式说明：

首先生成了一个 `boundary` 用于分割不同的字段，为了避免与正文内容重复，`boundary` 很长很复杂。

然后 `Content-Type` 里指明了数据是以 `multipart/form-data` 来编码，本次请求的 `boundary` 是什么内容。消息主体里按照字段个数又分为多个结构类似的部分，每部分都是以 `--boundary` 开始，紧接着是内容描述信息

然后是回车，最后是字段具体内容（文本或二进制）。如果传输的是文件，还要包含文件名和文件类型信息。消息主体最后以 `--boundary--` 标示结束。

关于 `multipart/form-data` 的详细定义，请前往 [rfc1867](https://www.ietf.org/rfc/rfc1867.txt) 查看

> **上面提到的这两种 POST 数据的方式，都是浏览器原生支持的，而且现阶段标准中原生 <form> 表单也只支持这两种方式（通过 <form> 元素的 enctype 属性指定，默认为 application/x-www-form-urlencoded。 enctype 还支持 text/plain，不过用得非常少）**

## application/json

`application/json` 这个 `Content-Type` 现在越来越多的人把它作为请求头，用来告诉服务端消息主体是序列化后的 JSON 字符串。

由于 JSON 规范的流行，除了低版本 IE 之外的各大浏览器都原生支持`JSON.stringify`，服务端语言也都有处理 JSON 的函数，使用 JSON 不会遇上什么麻烦。


`form-data/application-json.html`
```
  <!-- html -->
  <p>First name: <input type="text" name="fname" /></p>
  <p>Last name: <input type="text" name="lname" /></p>
  <button onclick="submit()">Submit</button>
  <br>
  <p>
    返回结果：
    <p style="color:red" id="result"></p>
  </p>
  
  <!-- script -->
  <script>
    submit = function () {
      var fname = document.getElementsByName('fname')[0].value  //用户输入的 fname
      var lname = document.getElementsByName('lname')[0].value  //用户输入的 lname
      var xhr = new XMLHttpRequest()
      // 使用HTTP POST请求与服务器交互数据
      xhr.open("POST", "http://localhost:3111/application-json", true)
      // 设置发送数据的请求格式 application/json
      xhr.setRequestHeader('content-type', 'application/json')
      xhr.onreadystatechange = function () {
        if (xhr.readyState == 4) {
          document.getElementById('result').innerHTML=JSON.stringify(JSON.parse(xhr.responseText))
          console.log()
        }
      }
      var sendData = { fname, lname }
      //将用户输入值序列化成字符串
      xhr.send(JSON.stringify(sendData))
    }
  </script>
```

浏览器访问 `localhost:5000/application-json/index.html`并填写表单，点击`Submit`按钮，得到的回包内容在如图红色箭头处渲染出来

![图示](https://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/2020-4-9/json-application-1.png)



`serve.js` 处理`/application-json`请求的代码【示例给到的回包均为 JSON 格式】
```
const bodyParser = require('body-parser')

// 用来解析 application-json body
const jsonParser = bodyParser.json({extended: false})

/** application-json 数据类型接口演示 */
app.post('/application-json', jsonParser, function (req, res) {
  console.log('get application-json Params: ', req.body)
  res.json({ result: '您发送的数据是：', data: req.body })
})
```

此时接口提交的请求数据，抓包时看到的请求的 `Content-Type` 请求头值是`application/json`，并且我们把回包内容以红色文本形式渲染在页面上


表单提交的响应结果

![图示](https://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/2020-4-9/json-application-2.png)

`application-json` 类型可以方便的提交复杂的结构化数据，很适合 `RESTful` 的接口。各大抓包工具如 Chrome 自带的开发者工具、Fiddler、Charles，都会以树形结构展示 JSON 数据，非常友好。

## GET 接口

最后附上 get 接口，这样基本上覆盖日常开发的90%以上的接口类型了

请求发起方式我们采用表单形式

> Get 方法是不含“body”的，它的请求参数都会被编码到url后面，所以在 Get 方法中加 Content-type 是无用的

`form-get/index.html`
```
  <form action="http://localhost:3111/get" method="get">
    <p>name: <input type="text" name="name" /></p>
    <p>age : <input type="number" name="age" /></p>
    <input type="submit" value="Submit - get" />
  </form>
```
填写表单，提交

![图示](https://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/2020-4-9/get-1.png)

`serve.js` 接口响应也很简单，直接取`req.query`即可，无需任何中间件
```
app.get('/get', function (req, res) {
  console.log('get Params: ', req.query)
  res.json({ result: '您发送的数据是：', data: req.query })
})
```

表单提交的响应结果

![图示](https://image-bt-1.obs.cn-east-3.myhuaweicloud.com/blog-image/2020-4-9/get-2.png)

可以看到，get 参数通过 url 传递，直接拼接在 query 参数中，请求头中也没有`Content-type`



## 福利 GET - POST 的 区别
```
get 参数通过 url 传递，post 放在 request body 中。 
get 请求在 url 中传递的参数是有长度限制的，而 post 没有。 
get 比 post 更不安全，因为参数直接暴露在 url 中，所以不能用来传递敏感信息。 
get 请求只能进行 url 编码，而 post 支持多种编码方式。
get 请求会浏览器主动 cache，而 post 支持多种编码方式。 
get 请求参数会被完整保留在浏览历史记录里，而 post 中的参数不会被保留。 

GET 和 POST 本质上就是 TCP 链接，并无差别。但是由于 HTTP 的规定和浏览器/服务器 的限制，导致他们在应用过程中体现出一些不同。 

GET 产生一个 TCP 数据包；POST 产生两个 TCP 数据包。
```


---

看到这里，是不是觉得日常业务接口类型也就这些，无非就是在做一些数据库的CRUD操作，so，前后端一起搞起来吧~~

---
[demo源码: https://github.com/melunar](https://github.com/melunar/proj02/tree/master/nodejs/server/content-type)

参考：  
[post请求头中常见content-type（非常重要）](https://www.cnblogs.com/mmzuo-798/p/11634055.html)
------

**更多关于我**
- 💻[博客](http://blog.lalapkp.cn)
- 🐱[Github](https://github.com/melunar)
- 🔨[掘金](https://juejin.cn/user/2612095355979405)
- 👱[关于我](http://www.lalapkp.cn/about)
- 🐒[CSDN](https://blog.csdn.net/Haoyong110?spm=1000.2115.3001.5343&type=1)

**微信公众号 [ 代表moon ]**

![代表moon](http://image-bt-1.obs.cn-east-3.myhuaweicloud.com/qrcode_for_gh_64a22fb6b2a0_344.jpg)
