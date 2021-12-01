## 一、概念介绍

**1、POST请求：** HTTP/1.1` 协议规定的` HTTP` 请求方法有 `OPTIONS、GET、HEAD、POST、PUT、DELETE、TRACE、CONNECT `这几种。其中` POST` 一般用来向服务端提交数据。

**2、Content-Type：**是指`http/https`发送信息至服务器时的内容编码类型，`Content-Type`用于表明发送数据流的类型，服务器根据编码类型使用特定的解析方式，获取数据流中的数据。四种常见的 `POST `请求的 `Content-Type `数据类型：

- `application/x-www-form-urlencoded`
- `multipart/form-data`
- `application/json`
- `text/xml`

**3、Express.js：**`Express` 是一个保持最小规模的灵活的 `Node.js Web` 应用程序开发框架，为 `Web` 和移动应用程序提供一组强大的功能。

​       本文我们主要介绍 `Post` 请求的 4 种 `Content-Type` 数据类型，以及如何使用 `Express` 来对每种` Content-Type` 类型进行解析。已经将完整的代码实例上传到 `github，github`地址为：[github.com/fengshi123/…](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Ffengshi123%2Frequest_example) ，欢迎 star 。

##  二、四种POST请求的Content-Type数据类型解析

### 1、application/x-www-form-unlencoded

最常见的 `POST` 提交数据的方式，浏览器的原生 `form` 表单，如果不设置 `enctype` 属性，那么最终就会默认以` application/x-www-form-urlencoded` 方式提交数据。

**1.1、前端请求代码**

```
var reqParam = "name=jack";
xhr.setRequestHeader('Content-type', 'application/x-www-form-urlencoded');
xhr.send(reqParam);
复制代码
```

**1.2、服务端解析代码**

```
app.post('/urlencoded', bodyParser.urlencoded({extend:true}), function (req, res) {   
  var result = {
     name: req.body.name,       
     sex: '男',        
     age: 15    
  };   
  res.send(result);
});复制代码
```

**1.3、浏览器请求 / 响应截图**

请求：

![img]()![img](https://user-gold-cdn.xitu.io/2019/5/29/16b036827787b451?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

响应：

![img](https://user-gold-cdn.xitu.io/2019/5/29/16b03687a6985329?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 2、multipart/form-data

使用表单上传文件时，必须指定表单的 `enctype`属性值为 `multipart/form-data`. 请求体被分割成多部分，每部分使用 `--boundary`分割开始，紧接着内容描述信息，最后是字段具体内容（文本或二进制）；如果传输的是文件，还要包含文件名和文件类型信息；

**2.1、前端请求代码**

```
var reqParam = new FormData(document.form2);
xhr.send(reqParam);复制代码
```

**2.2、服务端解析代码**

`express` 提供了两种插件 `formidable` 和  `multiparty` 来处理数据类型为` multipart/form-data `的情况，以下我们分别用两个插件进行处理；

**2.2.1、formidable 插件**

（1）安装插件

```
npm install formidable --save复制代码
```

（2）服务端解析处理

```
app.post('/formData1', function (req, res) {   
    var form = new formidable.IncomingForm();    
    form.uploadDir = "upload/";    
    form.parse(req, function (err, fields, files) {        
      var obj = {};        
      Object.keys(fields).forEach(function (name) {  
          obj[name] = fields[name];       
      });        
      Object.keys(files).forEach(function (name) {            
          if (files[name] && files[name].name) {                
             obj[name] = files[name];                
             fs.renameSync(files[name].path, form.uploadDir + files[name].name);          
        }        
     });      
     res.send(obj);    
   });
});复制代码
```

**2.2.2、multiparty 插件**

（1）安装插件

```
npm install multiparty--save复制代码
```

（2）服务端解析处理

```
app.post('/formData2', function (req, res) {   
 // 解析一个文件上传    
var form = new multiparty.Form();    
//设置编辑    
form.encoding = 'utf-8';    
//设置文件存储路径    
form.uploadDir = "upload/";   
 //设置单文件大小限制    
form.maxFilesSize = 2000 * 1024 * 1024;    
form.parse(req, function (err, fields, files) {        
     var obj = {};        
     Object.keys(fields).forEach(function (name) {            
          obj[name] = fields[name];       
     });       
     Object.keys(files).forEach(function (name) {            
         if (files[name] && files[name][0] && files[name][0].originalFilename) {               
             obj[name] = files[name];               
             fs.renameSync(files[name][0].path, form.uploadDir + files[name][0].originalFilename); 
         }       
      });        
      res.send(obj);    
    });
});复制代码
```

**2.3、浏览器请求 / 响应截图**

请求：

![img](https://user-gold-cdn.xitu.io/2019/5/29/16b036d853ad3525?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

响应：

![img](https://user-gold-cdn.xitu.io/2019/5/29/16b036dd758aca16?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

### 3、application/json

`application/json` 这个 `Content-Type` 作为响应头，用来告诉服务端消息主体是序列化后的 `JSON` 字符串。由于 `JSON` 规范的流行，除了低版本 `IE` 之外的各大浏览器都原生支持 `JSON.stringify`，服务端语言也都有处理 `JSON` 的函数，使用 `JSON` 不会遇上什么麻烦。

**3.1、前端请求代码**

```
var reqParam = {   
     name: 'jack'
};
xhr.setRequestHeader('Content-type', 'application/json');
xhr.send(JSON.stringify(reqParam));复制代码
```

**3.2、服务端解析代码**

```
app.post('/applicationJson', bodyParser.json(), function (req, res) {    
var result = {        
    name: req.body.name,       
    sex: '男',        
    age: 15    
  };    
   res.send(result);
});复制代码
```

**3.3、浏览器请求 / 响应截图**

请求：

![img](https://user-gold-cdn.xitu.io/2019/5/29/16b036f97c57d5a6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

响应：

![img](https://user-gold-cdn.xitu.io/2019/5/29/16b036fc230643cf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

### **4、text/xml**

它是一种使用 `HTTP` 作为传输协议，`XML` 作为编码方式的远程调用规范,它的使用也很广泛，能很好的支持已有的` XML-RPC` 服务。不过，`XML` 结构还是过于臃肿，一般场景用 `JSON` 会更灵活方便。

**4.1、前端请求代码**

```
var text = '<?xml version="1.0"?><methodCall><methodName>examples.getStateName</methodName>' +    '<params><param><value><i4>41</i4></value></param></params></methodCall>';
xhr.setRequestHeader('Content-type', 'text/xml');
xhr.send(text);复制代码
```

**4.2、服务端解析代码**

```
app.post('/textXml',  bodyParser.urlencoded({extend:true}), function (req, res) {    
   var result = ''; 
   req.on('data', function (chunk) {       
    result += chunk;   
   });    
   req.on('end', function () {        
   res.send(result);   
   });
});复制代码
```

**4.3、浏览器请求 / 响应截图**

请求：

![img](https://user-gold-cdn.xitu.io/2019/5/29/16b037176db32103?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

响应：

![img](https://user-gold-cdn.xitu.io/2019/5/29/16b0371ba93cc9e5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##  三、踩坑汇总

1、对于跨域请求，当`contentType`改为`application/json`，将触发浏览器发送一个预检`OPTIONS`请求到服务器，再发送正常的 `post` 请求；

2、使用 `new FormData()`，然后设置 `Content-type `为 `application/x-www-form-urlencoded `或者 `multipart/form-data` 会导致后端无法正常解析，解决方法：就是不进行头部设置，` Content-type `会默认 为 `multipart/form-data`，服务端正常解析；

3、`contentType` 设置为 `application/x-www-form-urlencoded` 时，传给后端的请求参数为`JSON`字符串，`chrome` 调试框查看发送的请求参数多了冒号，如下所示：

![img](https://user-gold-cdn.xitu.io/2019/5/29/16b0378298c449e0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

这是因为`application/x-www-form-urlencoded` 它将被解析成键值对展示，但是字符串进去是没有改变的，但是展示的时候能看见。解决方法：如果为 `JSON`字符串，则设置数据类型为 `application/json`；

## 四、总结

​     本文我们主要介绍 `Post` 请求的 4 种` Content-Type` 数据类型，以及如何使用 `Express` 来对每种` Content-Type` 类型进行解析。已经将完整的代码实例上传到 `github`，`github`地址为：[github.com/fengshi123/…](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Ffengshi123%2Frequest_example) ，欢迎 star 。`demo` 截图如下所示：

![img](https://user-gold-cdn.xitu.io/2019/5/29/16b037869d1707ee?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 

 

 

 

 

 

 

 

 

 