## 前言

 本文并不是`Vue SSR`的入门指南，没有一步步介绍`Vue SSR`入门，如果你想要`Vue SSR`入门教程，建议阅读`Vue`官网的《`Vue SSR`指南》，那应该是最详细的`Vue SSR`入门教程了。这篇文章的意义是，主要介绍如何在`SSR`服务端渲染中使用最受欢迎的`vue ui` 库`element-ui`组件库和`echarts`插件，以及本文中介绍的实例克服尤大大给的  `HackerNews Demo` 需要翻墙才能运行起来的问题，新手在阅读`SSR`官方文档时，如果遇到疑惑点，可以直接在本文实例的基础上进行相关实验验证，从而解决疑惑。本文实例的 `github`地址为：[github.com/fengshi123/…](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Ffengshi123%2Fvue-ssr) （欢迎 `star`）

 一、什么是服务端渲染（SSR）？

官网给出的解释：

​     Vue.js 是构建客户端应用程序的框架。默认情况下，可以在浏览器中输出 Vue 组件，进行生成 DOM 和操作 DOM。然而，也可以将同一个组件渲染为服务端的 HTML 字符串，将它们直接发送到浏览器，最后将这些静态标记"激活"为客户端上完全可交互的应用程序。

即：SSR大致的意思就是vue在客户端将标签渲染成的整个html片段的工作在服务端完成，服务端形成的html片段直接返回给客户端这个过程就叫做服务端渲染。

## 二、服务端渲染的优缺点

### **2.1、服务端渲染的优点：**

（1）更好的`SEO`： 因为`SPA`页面的内容是通过`Ajax`获取，而搜索引擎爬取工具并不会等待`Ajax`异步完成后再抓取页面内容，所以在`SPA`中是抓取不到页面通过`Ajax`获取到的内容；而`SSR`是直接由服务端返回已经渲染好的页面（数据已经包含在页面中），所以搜索引擎爬取工具可以抓取渲染好的页面；

（2）更快的内容到达时间（首屏加载更快）： `SPA`会等待所有vue编译后的`js`文件都下载完成后，才开始进行页面的渲染，文件下载等需要一定的时间等，所以首屏渲染需要一定的时间；`SSR`直接由服务端渲染好页面直接返回显示，无需等待下载`js`文件及再去渲染等，所以`SSR`有更快的内容到达时间；

### **2.2、服务端渲染的缺点：**

（1）更多的开发条件限制： 例如服务端渲染只支持`beforCreate`和`created`两个钩子函数，这会导致一些外部扩展库需要特殊处理，才能在服务端渲染应用程序中运行；并且与可以部署在任何静态文件服务器上的完全静态单页面应用程序SPA不同，服务端渲染应用程序，需要处于`Node.js server`运行环境；

（2）更多的服务器负载：在 `Node.js` 中渲染完整的应用程序，显然会比仅仅提供静态文件的 `server` 更加大量占用` CPU` 资源 (`CPU-intensive - CPU` 密集)，因此如果你预料在高流量环境 (`high traffic`) 下使用，请准备相应的服务器负载，并明智地采用缓存策略。

##  三、vue-ssr demo 介绍

本文示例基于尤大大给的  `HackerNews Demo` 进行改造，去除需要翻墙访问

`https://hacker-news.firebaseio.com` 的相关`api` ， 然后项目中使用了最受欢迎的`vue ui `库`element-ui` ，并且调研了`echarts.js `插件在服务端渲染的可行性；实例的目录结构以及实例的效果图分别如下所示：

![img](https://user-gold-cdn.xitu.io/2019/4/17/16a29f40405bc58f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![img](https://user-gold-cdn.xitu.io/2019/4/17/16a29f42b6c14393?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)![img](https://user-gold-cdn.xitu.io/2019/4/17/16a29f4438fc2f03?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![img](https://user-gold-cdn.xitu.io/2019/4/17/16a29f45385e137e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

具体每个文件的相关代码的逻辑在代码中都有进行详细的注释，所以这里就不详细再介绍一遍，可以在`github`上面` clone `进行查看，这里主要看下` Vue`官网上的服务端渲染的示意图

![img](https://user-gold-cdn.xitu.io/2019/4/17/16a29f4ba5bf20f2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

从图上可以看出，`ssr` 有两个入口文件，`client.js `和` server.js`， 都包含了应用代码，`webpack` 通过两个入口文件分别打包成给服务端用的 `server bundle` 和给客户端用的` client bundle`. 当服务器接收到了来自客户端的请求之后，会创建一个渲染器 `bundleRenderer`，这个 `bundleRenderer `会读取上面生成的` server bundle `文件，并且执行它的代码， 然后发送一个生成好的 `html `到浏览器，等到客户端加载了 `client bundle` 之后，会和服务端生成的`DOM `进行 `Hydration`(判断这个`DOM` 和自己即将生成的`DOM` 是否相同，如果相同就将客户端的`vue`实例挂载到这个`DOM`上， 否则会提示警告)。

##  四、本实例踩坑汇总

**4.1、引用 vue 文件会报文件找不到**

**问题：**如果引用` vue`文件没有加 `.vue `后缀，会报文件找不到，即：

```
import adminContent from './views/adminContent';复制代码
```

会报以下错误：

![img](https://user-gold-cdn.xitu.io/2019/4/17/16a29f69f7140bf2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**解决：**引用 `vue`文件时添加` .vue`后缀名，即：

```
import adminContent from './views/adminContent.vue';复制代码
```

**4.2、引入 element-ui 的样式时，报错**

**问题：** 在项目中引入 `element-ui` 的样式时，即：

```
import 'element-ui/lib/theme-chalk/index.css';复制代码
```

会报以下的错误，不引入样式文件则不会报错：

```
ReferenceError: window is not defined复制代码
```

**解决：** 需要进行样式文件的解析配置：

（1）安装样式解析插件：

```
npm install css-loader --save复制代码
```

（2）在`webpack.base.config.js` 中进行配置`css-loader`：

```
{ 
 test: /\.css$/,  
 loader: ["css-loader"]
}复制代码
```

**4.3、引入 element-ui 的样式时，报错** 

**问题：** 在项目中引入` element-ui `的样式时，即：

```
import 'element-ui/lib/theme-chalk/index.css';复制代码
```

会报以下的错误，不引入样式文件则不会报错：

```
Module parse failed: Unexpected character '@' (1:0) You may need an 
appropriate loader to handle this file type.复制代码
```

**解决：** 需要进行样式文件的解析配置：

（1）安装样式解析插件：

```
npm install style-loader --save复制代码
```

（2）在`webpack.base.config.js` 中进行相关配置：

```
{  
 test: /\.css$/,  
 loader: ["vue-style-loader", "css-loader"]
},
{  
 test: /\.(eot|svg|ttf|woff|woff2)(\?\S*)?$/,  
 loader: 'file-loader'
},复制代码
```

**4.4、element-ui的组件 el-table 不支持服务端渲染**

**问题：** 如果服务端渲染中的页面包含 `el-table`组件，从服务端返回的页面中的 `el-table` 组件中数据为空的，原因是` el-table` 组件在`mount`钩子函数中初始化`table`数据，而服务端渲染时，不支持`mount`钩子函数。

**解决：**`github` 上面的 `elment-ui `有分支修复了这个问题，[github.com/ElemeFE/ele…](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FElemeFE%2Felement%2Fpull%2F13018) ；将该分支的源码进行编译，然后替换在`node_modules`中替换` element-ui `的 `lib`编译包即可。

**4.5、el-table 服务端渲染后，表格宽度不是 100%**

**问题：**`el-table` 服务端渲染后，表格的宽度不是代码中设置的 100%，表格宽度比较小。

**解决：**进行样式额外设置：

```
.el-table__header{ 
 width: 100%;
}
.el-table__body{ 
 width: 100%;
}
.el-table-column--selection{
 width: 48px;
}复制代码
```

**4.6、echarts 插件怎么支持服务端渲染？**

**解决：**使用 `node-canvas` 插件，具体使用可以查看本实例的写法，也可以查看 `node-canvas `在`github`上面的介绍：[github.com/Automattic/…](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FAutomattic%2Fnode-canvas)

**存在问题：**实例中是利用`node-canvas` 生成对应的图片，然后页面中引用该图片，存在问题：生成的图片没有动效的效果。（这个问题没有继续研究，因为：图片没有文字内容，`seo `是不需要的；然后图片在服务端生成，在下载图片在页面中渲染，会直接在客户端渲染更节省资源吗？）

### 五、总结

 `SSR`有更好的`SEO`和更快的内容到达时间的优点，但也存在开发条件限制、服务器资源消耗多、新手上手难等缺点，所以你的项目是否需要服务端渲染，需要你结合你的项目具体进行相关指标的评估，切勿跟风，为 `SSR`而 `SSR`。

本文实例主要基于尤大大给的 `HackerNews Demo` 进行改造，去除需要翻墙访问`https://hacker-news.firebaseio.com` 的相关`api `， 然后项目中使用了最受欢迎的`vue ui` 库`element-ui `，并且调研了`echarts.js` 插件在服务端渲染的可行性，帮助新手更好更快地入门 `ssr`，如果在阅读官方`SSR`文档的过程中，有些疑问点，可以自己在本文实例中进行相关的试验验证，然后帮助解决疑惑。如果觉得本文以及`github`的实例帮助到你，请帮忙给个 star ，本文实例的 `github`地址为：[github.com/fengshi123/…](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Ffengshi123%2Fvue-ssr)   

 

 

 

 

 

 

 

 

 

 

 

 

 