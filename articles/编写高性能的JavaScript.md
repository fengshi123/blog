## 前言

随着计算机硬件的不断升级，开发者越发觉得`Javascript`性能优化的好不好对网页的执行效率影响不明显，所以一些性能方面的知识被很多开发者忽视。但在某些情况下，不优化的`Javascript`代码必然会影响用户的体验。因此，即使在当前硬件性能已经大大提升的时代，在编写`Javascript`代码时，若能遵循`Javascript`规范和注意一些性能方面的知识，对于提升代码的可维护性和优化性能将大有好处。那么，接下来我们讨论几种能够提高`JavaScript性能`的方法。

**如果喜欢或者有所启发，欢迎 Star~，对作者也是一种鼓励。** 

## 1、js文件加载和执行

（1）将`<script>`标签放到`<body>`标签的底部

（2）可以合并多个`js`文件，减少页面中`<script>`标签改善性能

（3）使用 `defer` 属性，加载后续文档元素的过程将和` script.js `的加载并行进行，但是 `script.js` 的执行要在所有元素解析完成之后，`DOMContentLoaded `事件触发之前完成。

（4）使用 `async` 属性，加载和渲染后续文档元素的过程将和` script.js `的加载与执行并行进行

（5）动态加载脚本元素，无论在何时启动瞎子，文件的下载和执行过程都不会阻塞页面其它进程

```
var script = document.createElement('script');
script.type = 'text/javascript';
script.src = 'file.js';
document.getElementsByTagName('head')[0].appendChild(script);
```

## 2、标识符所在的作用域链的位置越深

标识符所在的作用域链的位置越深，那么它的标识符解析的性能就越慢。所以一个好的性能提升的经验法则是：如果某个跨作用域的值在函数中被引用一次以上，那么就把它存储到局部变量里。

```
function fun1() {  
// 将全局变量的引用先存储在一个局部变量中，然后使用这个局部变量代替全局变量，从而提高         
// 性能；不然每次(3次)都要遍历整个作用域链找到
document  var doc = document;   
 var bd = doc.body;  
 var links = doc.getElementsByTagName('a');  
 doc.getElementById('btn').onclick = function(){   
 console.log('btn');  
 }
}
```

## 3、避免过长原型链继承

方法或属性在原型链中存在的位置越深，搜索它的性能也就越慢，所以要避免N多层原型链的写法。

## 4、对象成员嵌套过深

对象的嵌套成员，对象成员嵌套越深，读取速度也就越慢。所以好的经验法则是：如果在函数中需要多次读取一个对象属性，最佳做法是将该属性值保存在局部变量中，避免多次查找带来的性能开销。

```
function f() { 
 // 因为在以下函数中需要3次用到DOM对象属性，所以先将它存储在一个局部变量        
 // 中，然后使用这个局部变量代替它进行后续操作，从而提高性能  
var dom = YaHOO.util.Dom;  
if(Dom.hasClass(element,'selected')){   
  Dom.removeClass(elemet,'selected');  
}else{   
  Dom.addClass(elemet,'selected');  
 }
}
```

## 5、DOM操作

用`js`访问和操作`DOM`都会带来性能损失，可通过以下几点来减少性能损失：

（1）尽可能减少`DOM`访问次数；

（2）如果需要多次访问某个`DOM`节点，请使用局部变量存储它的引用；

（3）小心处理`HTML`集合，因为它实时连系着底层文档；我们可以把集合的长度缓存到一个变量中，并在迭代中使用它；

（4）下述情况会发生重排：

- 添加或删除可见的`DOM`元素；
- 元素位置改变；
- 元素尺寸改变（包括：外边距、内边距、边框厚度、宽度、高度等属性）；
- 内容改变（例如：文本改变或图片被另一个不同尺寸的图片改变）；
- 页面渲染器初始化；
- 浏览器窗口尺寸改变

可通过以下方式减少重排：

- 留意上面会导致重排的操作，尽量避免；
- 获取布局信息的操作会导致强制渲染队列重排，应该尽量避免使用以下获取布局信息的操作方法或属性或者缓存布局信息，例如：`offsetTop,offsetLeft,offsetWidthoffsetHeight,``scrollTop,scrollLeft,scrollWidth,scrollHeight,clientTop,clientLeft,clientWidth,clientHeight,getComputedStyle()`等;
- 批量修改样式，例如使用：

```
function f() {  
  // 推荐使用以下操作  
  var el1 = document.getElementById('mydiv');  
  el1.style.cssText = 'border:1px;padding:2px;margin:3px';  
  // 不推荐使用以下操作  
  var el2 = document.getElementById('mydiv');  
  el2.style.border = '1px';  
  el2.style.padding = '2px';  
  el2.style.margin = '3px';
}
```

- 当需要批量修改`DOM`时，可以通过以下步骤减少重绘和重排的次数：

- - 使元素脱离文档流（隐藏元素、拷贝元素）
  - 对其应用多重改变；
  - 把元素带回文档中

- 使用事件委托（事件逐层冒泡并能被父级元素捕获，使用事件代理，只需给外层元素绑定一个处理器，就可以处理其子元素上触发的所用事件），因为给`DOM`元素绑定事件以及浏览器需要跟踪每个事件处理器都需要消耗性能。

## 6、字符串连接

```
str += 'one'+'two';
str= str+'one'+'two';
```

后者方式会比前者少在内存中创建一个临时字符串，所以性能有相应的提升，所以，所以推荐后者的写法。

## 7、直接使用字面量

创建对象和数组推荐使用字面量，因为这不仅是性能最优也有助于节省代码量。

```
var obj = {   
 name:'tom',    
 age:15,    
 sex:'男'
}
```

​    

## 8、数组长度缓存

如果需要遍历数组，应该先缓存数组长度，将数组长度放入局部变量中，避免多次查询数组长度。

## 9、循环比较

`JS`提供了三种循环：`for(;;)、while()、for(in)`。在这三种循环中 `for(in)`的效率最差，因为它需要查询Hash键，因此应尽量少用`for(in)`循环，`for(;;)、while()`循环的性能基本持平。

## 10、少用eval

尽量少使用`eval`，每次使用`eval`需要消耗大量时间，这时候使用`JS`所支持的闭包可以实现函数模板。

## 11、字符串转换

当需要将数字转换成字符时，采用如下方式：`"" + 1`。从性能上来看，将数字转换成字符时，有如下公式：`("" +) > String() > .toString() > new String()`。`String()`属于内部函数，所以速度很快。而`.toString()`要查询原型中的函数，所以速度逊色一些，`new String()`需要重新创建一个字符串对象，速度最慢。

## 12、浮点数转换整形

当需要将浮点数转换成整型时，应该使用`Math.floor()`或者`Math.round()`。而不是使用`parseInt()`,该方法用于将字符串转换成数字。而且`Math`是内部对象，所以`Math.floor()`其实并没有多少查询方法和调用时间，速度是最快的。

**如果喜欢或者有所启发，欢迎 Star~，对作者也是一种鼓励。** 