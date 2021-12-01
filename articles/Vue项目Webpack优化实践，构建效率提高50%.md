# Vue项目Webpack优化实践，构建效率提高50%

## 前言      

 公司的前端项目使用`Vue`框架，`Vue`框架使用`Webpack`进行构建，随着项目不断迭代，项目逐渐变得庞大，然而项目的构建速度随之变得缓慢，于是对`Webpack`构建进行优化变得刻不容缓。经过不断的摸索和实践，通过以下方法优化后，项目的构建速度提高了50%。现将相关优化方法进行总结分享。

**辛苦整理良久，如果喜欢或者有所启发，请帮忙给个 Star ~，对作者也是一种鼓励。** 

##  1、缩小文件的搜索范围

###  1.1、优化`Loader`配置

​       由于`Loader`对文件的转换操作很耗时，所以需要让尽可能少的文件被`Loader`处理。我们可以通过以下3方面优化`Loader`配置：（1）优化正则匹配（2）通过`cacheDirectory`选项开启缓存（3）通过`include`、`exclude`来减少被处理的文件。实践如下：

**项目原配置：**

```
{
  test: /\.js$/,
  loader: 'babel-loader',
  include: [resolve('src'), resolve('test')]
}
```

**优化后配置：**

```
{
  // 1、如果项目源码中只有js文件，就不要写成/\.jsx?$/，以提升正则表达式的性能
  test: /\.js$/,
  // 2、babel-loader支持缓存转换出的结果，通过cacheDirectory选项开启
  loader: 'babel-loader?cacheDirectory',
  // 3、只对项目根目录下的src 目录中的文件采用 babel-loader
  include: [resolve('src')]
}
```

### 1.2、优化`resolve.modules`配置

​       `resolve.modules` 用于配置`Webpack`去哪些目录下寻找第三方模块。`resolve.modules`的默认值是［`node modules`］，含义是先去当前目录的`/node modules`目录下去找我们想找的模块，如果没找到，就去上一级目录`../node modules`中找，再没有就去`../ .. /node modules`中找，以此类推，这和`Node.js`的模块寻找机制很相似。当安装的第三方模块都放在项目根目录的`./node modules`目录下时，就没有必要按照默认的方式去一层层地寻找，可以指明存放第三方模块的绝对路径，以减少寻找。

**优化后配置：**

```
resolve: {
// 使用绝对路径指明第三方模块存放的位置，以减少搜索步骤
modules: [path.resolve(__dirname,'node_modules')]
}
```

### 1.3、优化`resolve.alias`配置

​       `resolve.alias`配置项通过别名来将原导入路径映射成一个新的导入路径。

**如项目中的配置使用：**

```
alias: {
  '@': resolve('src'),
},
// 通过以上的配置，引用src底下的common.js文件，就可以直接这么写
import common from '@/common.js';
```

### 1.4、优化`resolve.extensions`配置 

​       在导入语句没带文件后缀时，`Webpack` 会在自动带上后缀后去尝试询问文件是否存在。默认是：`extensions :[‘. js ‘,’. json ’]` 。也就是说，当遇到`require ( '. /data ’）`这样的导入语句时，`Webpack`会先去寻找`./data .js` 文件，如果该文件不存在，就去寻找`./data.json` 文件，如果还是找不到就报错。如果这个列表越长，或者正确的后缀越往后，就会造成尝试的次数越多，所以 `resolve .extensions` 的配置也会影响到构建的性能。 

###  优化措施：

 • 后缀尝试列表要尽可能小，不要将项目中不可能存在的情况写到后缀尝试列表中。

 • 频率出现最高的文件后缀要优先放在最前面，以做到尽快退出寻找过程。

 • 在源码中写导入语句时，要尽可能带上后缀，从而可以避免寻找过程。例如在确定的情况下将 `require(’. /data ’)`写成`require(’. /data.json ’)`，可以结合`enforceExtension` 和 `enforceModuleExtension`开启使用来强制开发者遵守这条优化

###  1.5、优化`resolve.noParse`配置

​       `noParse`配置项可以让`Webpack`忽略对部分没采用模块化的文件的递归解析和处理，这 样做的好处是能提高构建性能。原因是一些库如`jQuery`、`ChartJS` 庞大又没有采用模块化标准，让`Webpack`去解析这些文件既耗时又没有意义。 `noParse`是可选的配置项，类型需要是`RegExp` 、`[RegExp]`、`function`中的一种。例如，若想要忽略`jQuery` 、`ChartJS` ，**则优化配置如下：**

```
// 使用正则表达式 
noParse: /jquerylchartjs/ 
// 使用函数，从 Webpack3.0.0开始支持 
noParse: (content)=> { 
// 返回true或false 
return /jquery|chartjs/.test(content); 
}
```

##  2、减少冗余代码

​        `babel-plugin-transform-runtime` 是`Babel`官方提供的一个插件，作用是减少冗余的代码 。 `Babel`在将`ES6`代码转换成`ES5`代码时，通常需要一些由`ES5`编写的辅助函数来完成新语法的实现，例如在转换 `class` `extent` 语法时会在转换后的 `ES5` 代码里注入 `extent` 辅助函数用于实现继承。`babel-plugin-transform-runtime`会将相关辅助函数进行替换成导入语句，从而减小`babel`编译出来的代码的文件大小。

## 3、使用`HappyPack`多进程解析和处理文件

​       由于有大量文件需要解析和处理，所以构建是文件读写和计算密集型的操作，特别是当文件数量变多后，`Webpack`构建慢的问题会显得更为严重。运行在 `Node`之上的`Webpack`是单线程模型的，也就是说`Webpack`需要一个一个地处理任务，不能同时处理多个任务。`Happy Pack` ( https://github.com/amireh/happypack ）就能让`Webpack`做到这一点，它将任务分解给多个子进程去并发执行，子进程处理完后再将结果发送给主进程。

**项目中`HappyPack`使用配置：**

```
（1）HappyPack插件安装：
    $ npm i -D happypack
（2）webpack.base.conf.js 文件对module.rules进行配置
    module: {
     rules: [
      {
        test: /\.js$/,
        // 将对.js 文件的处理转交给 id 为 babel 的HappyPack实例
          use:['happypack/loader?id=babel'],
          include: [resolve('src'), resolve('test'),   
            resolve('node_modules/webpack-dev-server/client')],
        // 排除第三方插件
          exclude:path.resolve(__dirname,'node_modules'),
        },
        {
          test: /\.vue$/,
          use: ['happypack/loader?id=vue'],
        },
      ]
    },
（3）webpack.prod.conf.js 文件进行配置    const HappyPack = require('happypack');
    // 构造出共享进程池，在进程池中包含5个子进程
    const HappyPackThreadPool = HappyPack.ThreadPool({size:5});
    plugins: [
       new HappyPack({
         // 用唯一的标识符id，来代表当前的HappyPack是用来处理一类特定的文件
         id:'vue',
         loaders:[
           {
             loader:'vue-loader',
             options: vueLoaderConfig
           }
         ],
         threadPool: HappyPackThreadPool,
       }),

       new HappyPack({
         // 用唯一的标识符id，来代表当前的HappyPack是用来处理一类特定的文件
         id:'babel',
         // 如何处理.js文件，用法和Loader配置中一样
         loaders:['babel-loader?cacheDirectory'],
         threadPool: HappyPackThreadPool,
       }),
    ]
```

##  4、使用ParallelUglifyPlugin多进程压缩代码文件

​       由于压缩`JavaScript` 代码时，需要先将代码解析成用 `Object` 抽象表示的 `AST` 语法树，再去应用各种规则分析和处理`AST` ，所以导致这个过程的计算量巨大，耗时非常多。当`Webpack`有多个`JavaScript` 文件需要输出和压缩时，原本会使用`UglifyJS`去一个一个压缩再输出，但是`ParallelUglifyPlugin`会开启多个子进程，将对多个文件的压缩工作分配给多个子进程去完成，每个子进程其实还是通过`UglifyJS`去压缩代码，但是变成了并行执行。所以 `ParallelUglify Plugin`能更快地完成对多个文件的压缩工作。

 **项目中`ParallelUglifyPlugin`使用配置：**

```
（1）ParallelUglifyPlugin插件安装：
     $ npm i -D webpack-parallel-uglify-plugin
（2）webpack.prod.conf.js 文件进行配置
    const ParallelUglifyPlugin =require('webpack-parallel-uglify-plugin');
    plugins: [
    new ParallelUglifyPlugin({
      cacheDir: '.cache/',
      uglifyJs:{
        compress: {
          warnings: false
        },
        sourceMap: true
      }
     }),
    ]
复制代码
```

## 5、使用自动刷新 

​       借助自动化的手段，在监听到本地源码文件发生变化时，自动重新构建出可运行的代码后再控制浏览器刷新。`Webpack`将这些功能都内置了，并且提供了多种方案供我们选择。

 **项目中自动刷新的配置：**

```
devServer: {
  watchOptions: {
    // 不监听的文件或文件夹，支持正则匹配
    ignored: /node_modules/,
    // 监听到变化后等300ms再去执行动作
    aggregateTimeout: 300,
    // 默认每秒询问1000次
    poll: 1000
  }
},
复制代码
```

**相关优化措施：** 

（1）配置忽略一些不监听的一些文件，如：`node_modules`。 

（2）`watchOptions.aggregateTirneout` 的值越大性能越好，因为这能降低重新构建的频率。

（3） `watchOptions.poll` 的值越小越好，因为这能降低检查的频率。

##  6、开启模块热替换 

​       `DevServer` 还支持一种叫做模块热替换（ `Hot Module Replacement` ）的技术可在不刷新整个网页的情况下做到超灵敏实时预览。原理是在一个源码发生变化时，只需重新编译发生变化的模块，再用新输出的模块替换掉浏览器中对应的老模块 。模块热替换技术在很大程度上提升了开发效率和体验 。 

**项目中模块热替换的配置：**

```
devServer: {
  hot: true,
},
plugins: [
  new webpack.HotModuleReplacementPlugin(),
// 显示被替换模块的名称
  new webpack.NamedModulesPlugin(), // HMR shows correct file names
]
复制代码
```

## 7、提取公共代码 

​        如果每个页面的代码都将这些公共的部分包含进去，则会造成以下问题 ： 

 • 相同的资源被重复加载，浪费用户的流量和服务器的成本。

 • 每个页面需要加载的资源太大，导致网页首屏加载缓慢，影响用户体验。 

​       如果将多个页面的公共代码抽离成单独的文件，就能优化以上问题 。`Webpack`内置了专门用于提取多个`Chunk`中的公共部分的插件`CommonsChunkPlugin`。 

**项目中`CommonsChunkPlugin`的配置：**

```
// 所有在 package.json 里面依赖的包，都会被打包进 vendor.js 这个文件中。
new webpack.optimize.CommonsChunkPlugin({
  name: 'vendor',
  minChunks: function(module, count) {
    return (
      module.resource &&
      /\.js$/.test(module.resource) &&
      module.resource.indexOf(
        path.join(__dirname, '../node_modules')
      ) === 0
    );
  }
}),
// 抽取出代码模块的映射关系
new webpack.optimize.CommonsChunkPlugin({
  name: 'manifest',
  chunks: ['vendor']
}),
复制代码
```

## 8、按需加载代码 

​       通过`vue`写的单页应用时，可能会有很多的路由引入。当打包构建的时候，`javascript`包会变得非常大，影响加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应的组件，这样就更加高效了。这样会大大提高首屏显示的速度，但是可能其他的页面的速度就会降下来。 

**项目中路由按需加载（懒加载）的配置：**

```
const Foo = () => import('./Foo.vue')
const router = new VueRouter({
  routes: [
    { path: '/foo', component: Foo }
  ]
})
```

##  9、优化SourceMap 

​       我们在项目进行打包后，会将开发中的多个文件代码打包到一个文件中，并且经过压缩，去掉多余的空格，且`babel`编译化后，最终会用于线上环境，那么这样处理后的代码和源代码会有很大的差别，当有`bug`的时候，我们只能定位到压缩处理后的代码位置，无法定位到开发环境中的代码，对于开发不好调式，因此`sourceMap`出现了，它就是为了解决不好调式代码问题的。

 **`SourceMap`的可选值如下：**

![img](https://user-gold-cdn.xitu.io/2018/12/23/167dba50e9d02b22?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**开发环境推荐： `cheap-module-eval-source-map`** 

**生产环境推荐： `cheap-module-source-map**`

**原因如下：** 

1. 源代码中的列信息是没有任何作用，因此我们打包后的文件不希望包含列相关信息，只有行信息能建立打包前后的依赖关系。因此不管是开发环境或生产环境，我们都希望添加`cheap`的基本类型来忽略打包前后的列信息。 
2. 不管是开发环境还是正式环境，我们都希望能定位到`bug`的源代码具体的位置，比如说某个`vue`文件报错了，我们希望能定位到具体的`vue`文件，因此我们也需要`module`配置。 
3. 我们需要生成`map`文件的形式，因此我们需要增加`source-map`属性。 
4. 我们介绍了`eval`打包代码的时候，知道`eval`打包后的速度非常快，因为它不生成`map`文件，但是可以对`eval`组合使用 `eval-source-map`使用会将`map`文件以`DataURL`的形式存在打包后的`js`文件中。在正式环境中不要使用 `eval-source-map`, 因为它会增加文件的大小，但是在开发环境中，可以试用下，因为他们打包的速度很快。

##  10、构建结果输出分析 

​       `Webpack`输出的代码可读性非常差而且文件非常大，让我们非常头疼。为了更简单、直观地分析输出结果，社区中出现了许多可视化分析工具。这些工具以图形的方式将结果更直观地展示出来，让我们快速了解问题所在。接下来讲解vue项目中用到的分析工具：`webpack-bundle-analyzer` 。

**项目中在`webpack.prod.conf.js`进行配置：**

```
if (config.build.bundleAnalyzerReport) {
  var BundleAnalyzerPlugin =   require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
  webpackConfig.plugins.push(new BundleAnalyzerPlugin());
}
执行 $ npm run build --report 后生成分析报告如下：
```

 ![img](https://user-gold-cdn.xitu.io/2018/12/23/167dba6f9640342d?imageslim) 



 **辛苦整理良久，如果喜欢或者有所启发，请帮忙给个 Star ~，对作者也是一种鼓励。** 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 