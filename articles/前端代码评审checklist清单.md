# **前端代码评审 Checklist**

## 前言

​       前端团队有评审代码的要求，但由于每个开发人员的水平不同，技术关注点不同，所以对代码评审的关注点不同，为了保证代码质量，团队代码风格统一，特此拟定一份《前端团队代码评审 CheckList 清单》，这样代码评审人员在评审代码时，可以参照这份清单，对代码进行评审。从而辅助整个团队提高代码质量、统一代码规范。如果你的团队还没有这么一份代码评审 CheckList 清单，也许这正是你需要的；如果你的团队已经有了代码评审参照标准，这份清单也许能起到锦上添花的效果。

**辛苦整理良久，还望手动 Star 鼓励~** 

## 一、**代码静态检查工具**

#### 1.1、使用 eslint 工具对 javascript 代码进行检查

​     eslint 检查的规范继承自 eslint-config-standard 检验规则，具体的规则介绍参照链接：<https://cn.eslint.org/docs/rules/> ，这里及以下部分不再重复介绍这些检验规则。

#### 1.2、使用 stylelint 工具对 css 样式代码进行检查

​      stylelint 检查的规范继承自 stylelint-config-standard 检验规则，具体的规则介绍参照链接：<https://www.npmjs.com/package/stylelint-config-standard> ，这里及以下部分不再重复介绍这些检验规则。

## 二、命名

#### 2.1、JS 采用 Camel Case 小驼峰式命名

推荐：

```
   studentInfot
```

#### 2.2、避免名称冗余

推荐：

```
const Car = {
  make: "Honda",
  model: "Accord",
  color: "Blue"
};
```

不推荐：

```
const Car = {
  carMake: "Honda",
  carModel: "Accord",
  carColor: "Blue"
};
```

#### 2.3、CSS 类名采用 BEM 命名规范

推荐：

```
.block__element{} 
.block--modifier{}
```

#### 2.4、命名符合语义化

命名需要符合语义化，如果函数命名，可以采用加上动词前缀：

| 动词 | 含义                   |
| ---- | ---------------------- |
| can  | 判断是否可执行某个动作 |
| has  | 判断是否含有某个值     |
| is   | 判断是否为某个值       |
| get  | 获取某个值             |
| set  | 设置某个值             |

推荐：

```
//是否可阅读 
function canRead(){ 
   return true; 
} 
//获取姓名 
function getName{
   return this.name 
} 
```

## 三、JS 推荐写法

#### 3.1、每个常量都需命名

每个常量应该命名，不然看代码的人不知道这个常量表示什么意思。

推荐：

```
const COL_NUM = 10;
let row = Math.ceil(num/COL_NUM);
```

不推荐：

```
let row = Math.ceil(num/10);
```

#### 3.2、推荐使用字面量

创建对象和数组推荐使用字面量，因为这不仅是性能最优也有助于节省代码量。 

推荐：

```
let obj = {   
     name:'tom',     
     age:15,     
     sex:'男' 
} 
```

不推荐：

```
let obj = {};
obj.name = 'tom';
obj.age = 15;
obj.sex = '男';
```

#### 3.3、对象设置默认属性的推荐写法

推荐：

```
const menuConfig = {
  title: "Order",
  // User did not include 'body' key
  buttonText: "Send",
  cancellable: true
};

function createMenu(config) {
  config = Object.assign(
    {
      title: "Foo",
      body: "Bar",
      buttonText: "Baz",
      cancellable: true
    },
    config
  );

  // config now equals: {title: "Order", body: "Bar", buttonText: "Send", cancellable: true}
  // ...
}

createMenu(menuConfig);
```

不推荐：

```
const menuConfig = {
  title: null,
  body: "Bar",
  buttonText: null,
  cancellable: true
};

function createMenu(config) {
  config.title = config.title || "Foo";
  config.body = config.body || "Bar";
  config.buttonText = config.buttonText || "Baz";
  config.cancellable =
    config.cancellable !== undefined ? config.cancellable : true;
}

createMenu(menuConfig);
```

#### 3.4、将对象的属性值保存为局部变量

对象成员嵌套越深，读取速度也就越慢。所以好的经验法则是：如果在函数中需要多次读取一个对象属性，最佳做法是将该属性值保存在局部变量中，避免多次查找带来的性能开销。 

推荐：

```
let person = {
    info:{
        sex:'男'
    }
}
function  getMaleSex(){
    let sex = person.info.sex;
    if(sex === '男'){
        console.log(sex)
    }
} 
```

不推荐：

```
let person = {
    info:{
        sex:'男'
    }
}
function  getMaleSex(){
    if(person.info.sex === '男'){
        console.log(person.info.sex)
    }
} 
```

#### 3.5、字符串转为整型

当需要将浮点数转换成整型时，应该使用`Math.floor()`或者`Math.round()`，而不是使用`parseInt()`将字符串转换成数字。Math`是内部对象，所以`Math.floor()`其实并没有多少查询方法和调用时间，速度是最快的。 

推荐：

```
let num = Math.floor('1.6');
```

不推荐：

```
let num = parseInt('1.6');
```

#### 3.6、函数参数

函数参数越少越好，如果参数超过两个，要使用 ES6的解构语法，不用考虑参数的顺序。

推荐：

```
function createMenu({ title, body, buttonText, cancellable }) {
  // ...
}

createMenu({
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true
});
```

不推荐：

```
function createMenu(title, body, buttonText, cancellable) {
  // ...
}
```

#### 3.7、使用参数默认值

使用参数默认值 替代 使用条件语句进行赋值。

推荐：

```
function createMicrobrewery(name = "Hipster Brew Co.") {
  // ...
}
```

不推荐：

```
function createMicrobrewery(name) {
  const breweryName = name || "Hipster Brew Co.";
  // ...
}
```

#### 3.8、最小函数准则

这是一条在软件工程领域流传久远的规则。严格遵守这条规则会让你的代码可读性更好，也更容易重构。如果违反这个规则，那么代码会很难被测试或者重用 。



#### 3.9、不要写全局方法

在 JavaScript 中，永远不要污染全局，会在生产环境中产生难以预料的 bug。举个例子，比如你在 `Array.prototype` 上新增一个 `diff` 方法来判断两个数组的不同。而你同事也打算做类似的事情，不过他的 `diff` 方法是用来判断两个数组首位元素的不同。很明显你们方法会产生冲突，遇到这类问题我们可以用 ES2015/ES6 的语法来对 `Array` 进行扩展。

 推荐:

```
class SuperArray extends Array {
  diff(comparisonArray) {
    const hash = new Set(comparisonArray);
    return this.filter(elem => !hash.has(elem));        
  }
}
```

不推荐：

```
Array.prototype.diff = function diff(comparisonArray) {
  const hash = new Set(comparisonArray);
  return this.filter(elem => !hash.has(elem));
};
```



####  3.10、推荐函数式编程

函数式变编程可以让代码的逻辑更清晰更优雅，方便测试。 

推荐：

```
const programmerOutput = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500
  }, {
    name: 'Suzie Q',
    linesOfCode: 1500
  }, {
    name: 'Jimmy Gosling',
    linesOfCode: 150
  }, {
    name: 'Gracie Hopper',
    linesOfCode: 1000
  }
];
let totalOutput = programmerOutput
  .map(output => output.linesOfCode)
  .reduce((totalLines, lines) => totalLines + lines, 0)
```

 不推荐：

```
 const programmerOutput = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500
  }, {
    name: 'Suzie Q',
    linesOfCode: 1500
  }, {
    name: 'Jimmy Gosling',
    linesOfCode: 150
  }, {
    name: 'Gracie Hopper',
    linesOfCode: 1000
  }
];

let totalOutput = 0;

for (let i = 0; i < programmerOutput.length; i++) {
  totalOutput += programmerOutput[i].linesOfCode;
}


```

#### 3.11、使用多态替换条件语句

为了让代码更简洁易读，如果你的函数中出现了条件判断，那么说明你的函数不止干了一件事情，违反了函数单一原则 ；并且绝大数场景可以使用多态替代

推荐：

```
class Airplane {
  // ...
}
// 波音777
class Boeing777 extends Airplane {
  // ...
  getCruisingAltitude() {
    return this.getMaxAltitude() - this.getPassengerCount();
  }
}
// 空军一号
class AirForceOne extends Airplane {
  // ...
  getCruisingAltitude() {
    return this.getMaxAltitude();
  }
}
// 赛纳斯飞机
class Cessna extends Airplane {
  // ...
  getCruisingAltitude() {
    return this.getMaxAltitude() - this.getFuelExpenditure();
  }
}
```

不推荐：

```
class Airplane {
  // ...

  // 获取巡航高度
  getCruisingAltitude() {
    switch (this.type) {
      case '777':
        return this.getMaxAltitude() - this.getPassengerCount();
      case 'Air Force One':
        return this.getMaxAltitude();
      case 'Cessna':
        return this.getMaxAltitude() - this.getFuelExpenditure();
    }
  }
}
```

#### 3.12、定时器是否清除

代码中使用了定时器 setTimeout 和 setInterval，需要在不使用时进行清除。

## 四、SCSS 推荐写法

#### 4.1、变量 $ 使用

利用scss中的变量配置，可以进行项目的颜色、字体大小统一更改（换肤），有利于后期项目的维护。

推荐：

```
$--color-success: #67C23A;
$--color-warning: #E6A23C;
$--color-danger: #F56C6C;
$--color-info: #909399;
```

#### 4.2、@import 导入样式文件

scss中的@import规则在生成css文件时就把相关文件导入进来。这意味着所有相关的样式被归纳到了同一个css文件中，而无需发起额外的下载请求，在构建我们自己的组件库时推荐使用。

```
@import "./base.scss";
@import "./pagination.scss";
@import "./dialog.scss";
@import "./autocomplete.scss";
@import "./dropdown.scss";
@import "./dropdown-menu.scss";
```

#### 4.3、局部文件命名的使用

scss局部文件的文件名以下划线开头。这样，scss就不会在编译时单独编译这个文件输出css，而只把这个文件用作导入。

推荐：

![1.png](https://github.com/fengshi123/blog/blob/master/assets/checklist/1.png?raw=true) 

#### 4.4、父选择器标识符 & 实现BEM 命令规范

scss的嵌套和父选择器标识符&能解决BEM命名的冗长，且使样式可读性更高。

推荐：

```
.el-input {
  display: block;
  &__inner {
     text-align: center;
  }
 }
```

#### 4.5、@mixin 混合器的使用

mixin混合器用来实现大段样式的重用，减少代码的冗余，且支持传参。

```
@mixin button-size($padding-vertical, $padding-horizontal, $font-size, $border-radius) {
  padding: $padding-vertical $padding-horizontal;
  font-size: $font-size;
  border-radius: $border-radius;
  &.is-round {
    padding: $padding-vertical $padding-horizontal;
  }
}

  @include m(medium) {
    @include button-size($--button-medium-padding-vertical, $--button-medium-padding-horizontal, $--button-medium-font-size, $--button-medium-border-radius);   
  }

  @include m(small) {
    @include button-size($--button-small-padding-vertical, $--button-small-padding-horizontal, $--button-small-font-size, $--button-small-border-radius);
  }

```

#### 4.6、@extend 指令的使用

（1）使用@extend产生 [DRY CSS](http://vanseodesign.com/css/dry-principles/)风格的代码（Don't repeat yourself）

（2）@mixin主要的优势就是它能够接受参数。如果想传递参数，你会很自然地选择@mixin而不是@extend

推荐：

```
.common-mod {
  height: 250px;
  width: 50%;
  background-color: #fff;
  text-align: center;
}

 .show-mod--right {
   @extend .common-mod;
   float: right;
 }

.show-mod--left {
   @extend .common-mod;
}
```

#### 4.7、**#{} 插值的使用**

插值能动态定义类名的名称，当有两个页面的样式类似时，我们会将类似的样式抽取成页面混合器，但两个不同的页面样式的命名名称根据BEM命名规范不能一样，这时我们可使用插值进行动态命名。

推荐：

```
@mixin home-content($class) {
  .#{$class} {
    position: relative;
    background-color: #fff;
    overflow-x: hidden;
    overflow-y: hidden;

    &--left {
      margin-left: 160px;
    }

    &--noleft {
      margin-left: 0;
    }
  }
}
```

####  4.8、each遍历、map数据类型、@mixin/@include混合器、#{}插值 结合使用

可通过each遍历、map数据类型、@mixin/@include混合器、#{}插值 结合使用，从而减少冗余代码，使代码更精简。

推荐：

```
$img-list: (
   (xlsimg, $papers-excel),
   (xlsximg, $papers-excel),
   (gifimg, $papers-gif),
   (jpgimg, $papers-jpg),
   (mp3img, $papers-mp3),
   (mp4img, $papers-mp3),
   (docimg, $papers-word),
   (docximg, $papers-word),
   (rarimg, $papers-zip),
   (zipimg, $papers-zip),
   (unknownimg, $papers-unknown)
);

@each $label, $value in $img-list {
  .com-hwicon__#{$label} {
    @include commonImg($value);
  }
}
```

#### 4.9、scss 自带函数的应用

scss自带函数的应用，从而进行相关的计算，例如 mix函数的使用如下。

```
 @include m(text) {
    &:hover,
    &:focus {
      color: mix($--color-white, $--color-primary, $--button-hover-tint-percent);
      border-color: transparent;
      background-color: transparent;
    }

    &:active {
      color: mix($--color-black, $--color-primary, $--button-active-shade-percent);
      border-color: transparent;
      background-color: transparent;
    }
}
```

#### 4.10、gulp-sass的使用

gulp-sass插件能实时监测scss代码检查其语法错误并将其编译成css代码，帮助开发人员检查scss语法的准确性，且其是否符合我们的预期，相关配置如下：

```
gulp.task('gulpsass', function() {
  return gulp.src('src/style/components/hwIcon.scss')
    .pipe(gulpsass().on('error', gulpsass.logError))
    .pipe(gulp.dest('src/style/dest'));
});

gulp.task('watch', function() {
  gulp.watch('src/style/components/hwIcon.scss', ['gulpsass']);
});
```



## 五、Vue 推荐写法

#### 5.1、组件名为多个单词

我们开发过程中自定义的组件的名称需要为多个单词，这样做可以避免跟现有的以及未来的HTML元素相冲突，因为所有的 HTML 元素名称都是单个单词的。 

推荐：

```
Vue.component('todo-item', {
  // ...
})

export default {
  name: 'TodoItem',
  // ...
}
```

不推荐：

```
Vue.component('todo', {
  // ...
})

export default {
  name: 'Todo',
  // ...
}
```

#### 5.2、组件的 data 必须是一个函数

当在组件中使用 `data` 属性的时候 (除了 `new Vue` 外的任何地方)，它的值必须是返回一个对象的函数。 因为如果直接是一个对象的话，子组件之间的属性值会互相影响。

推荐：

```
export default {
  data () {
    return {
      foo: 'bar'
    }
  }
}
```

不推荐：

```
export default {
  data: {
    foo: 'bar'
  }
}
```

#### 5.3、Prop定义应该尽量详细

prop 的定义应该尽量详细，至少需要指定其类型。 

推荐：

```
props: {
  status: String
}

// 更好的做法！
props: {
  status: {
    type: String,
    required: true,
    validator: function (value) {
      return [
        'syncing',
        'synced',
        'version-conflict',
        'error'
      ].indexOf(value) !== -1
    }
  }
}
```

不推荐：

```
props: ['status'] 
```

#### 5.4、为 v-for 设置键值

v-for 中总是有设置 key 值。在组件上*总是*必须用 `key` 配合 `v-for`，以便维护内部组件及其子树的状态。 

推荐：

```
<ul>
  <li
    v-for="todo in todos"
    :key="todo.id">
    {{ todo.text }}
  </li>
</ul>
```

不推荐：

```
<ul>
  <li v-for="todo in todos">
    {{ todo.text }}
  </li>
</ul>
```

#### 5.5、完整单词的组件名

组件名应该倾向于完整单词而不是缩写，编辑器中的自动补全已经让书写长命名的代价非常之低了，而其带来的明确性却是非常宝贵的。不常用的缩写尤其应该避免。 

推荐：

```
components/ 
|- StudentDashboardSettings.vue 
|- UserProfileOptions.vue 
```

不推荐：

```
components/ 
|- SdSettings.vue 
|- UProfOpts.vue 
```

#### 5.6、多个特性元素的每个特性分行

在 JavaScript 中，用多行分隔对象的多个属性是很常见的最佳实践，因为这样更易读。 

推荐：

```
<MyComponent
  foo="a"
  bar="b"
  baz="c"
/>
```

不推荐：

```
<MyComponent foo="a" bar="b" baz="c"/> 
```

#### 5.7、模板中简单的表达式

组件模板应该只包含简单的表达式，复杂的表达式则应该重构为计算属性或方法。复杂表达式会让你的模板变得不那么声明式。我们应该尽量描述应该出现的*是什么*，而非*如何*计算那个值。而且计算属性和方法使得代码可以重用。 

推荐：

```
<!-- 在模板中 -->
{{ normalizedFullName }}

// 复杂表达式已经移入一个计算属性
computed: {
  normalizedFullName: function () {
    return this.fullName.split(' ').map(function (word) {
      return word[0].toUpperCase() + word.slice(1)
    }).join(' ')
  }
}
```

不推荐：

```
{{
  fullName.split(' ').map(function (word) {
    return word[0].toUpperCase() + word.slice(1)
  }).join(' ')
}}
```

#### 5.8、简单的计算属性

应该把复杂计算属性分割为尽可能多的更简单的属性。

推荐：

```
computed: {
  basePrice: function () {
    return this.manufactureCost / (1 - this.profitMargin)
  },
  discount: function () {
    return this.basePrice * (this.discountPercent || 0)
  },
  finalPrice: function () {
    return this.basePrice - this.discount
  }
}
```

不推荐：

```
computed: {
  price: function () {
    var basePrice = this.manufactureCost / (1 - this.profitMargin)
    return (
      basePrice -
      basePrice * (this.discountPercent || 0)
    )
  }
}
```

#### 5.9、指令缩写

指令推荐都使用缩写形式，(用 : 表示 v-bind: 、用 @ 表示 v-on: 和用 # 表示 v-slot:)。

推荐：

```
<input
  @input="onInput"
  @focus="onFocus"
>
```

不推荐：

```
<input
  v-on:input="onInput"
  @focus="onFocus"
>
```

#### 5.10、标签顺序保持一致

单文件组件应该总是让标签顺序保持为 <template> 、<script>、 <style>  。

推荐：

```
<!-- ComponentA.vue -->

<template>...</template>
<script>/* ... */</script>
<style>/* ... */</style>
```

不推荐：

```
<!-- ComponentA.vue -->

<template>...</template>
<style>/* ... */</style>
<script>/* ... */</script>
```

#### 5.11、组件之间通信

父子组件的通信推荐使用 prop和 emit ，而不是this.$parent或改变 prop；

兄弟组件之间的通信推荐使用 EventBus（$emit　/ $on），而不是滥用 vuex；

祖孙组件之间的通信推荐使用 $attrs / $listeners  或 provide / inject（依赖注入） ，而不是滥用 vuex；

#### 5.12、页面跳转数据传递

页面跳转，例如 A 页面跳转到 B 页面，需要将 A 页面的数据传递到 B 页面，推荐使用 路由参数进行传参，而不是将需要传递的数据保存 vuex，然后在 B 页面取出 vuex的数据，因为如果在 B 页面刷新会导致 vuex 数据丢失，导致 B 页面无法正常显示数据。

推荐：

```
let id = ' 123';
this.$router.push({name: 'homeworkinfo', query: {id:id}}); 
```

####  5.13、script 标签内部声明顺序

script 标签内部的声明顺序如下：

data > prop > components > filter > computed >  watch > 钩子函数（钩子函数按其执行顺序） > methods 

#### 5.14、计算属性 VS 方法 VS 侦听器

- （1）推荐使用计算属性：计算属性基于响应式依赖进行缓存，只在相关响应式依赖发生改变时它们才会重新求值；相比之下，每次调用方法都会再次执行方法；
- （2）推荐使用计算属性：而不是根据 Watch 侦听属性，进行回调； 但是有计算属性做不到的：当需要在数据变化时执行异步或开销较大的操作时，侦听器是最有用的。

#### 5.15、v-if  VS  v-show

- v-if 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。 v-if 也是惰性的：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。
- 相比之下，v-show 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 的属性 display 进行切换。

推荐：

**如果运行时，需要非常频繁地切换，推荐使用 v-show 比较好；如果在运行时，条件很少改变，则推荐使用 v-if 比较好。**

## 六、团队其它规范

#### 6.1、尽量不手动操作 DOM

因为团队现在使用 vue 框架，所以在项目开发中尽量使用 vue 的特性去满足我们的需求，尽量（不到万不得已）不要手动操作DOM，包括：增删改dom元素、以及更改样式、添加事件等。


#### 6.2、删除弃用代码

很多时候有些代码已经没有用了，但是没有及时去删除，这样导致代码里面包括很多注释的代码块，好的习惯是提交代码前记得删除已经确认弃用的代码，例如：一些调试的console语句、无用的弃用代码。

#### 6.3、保持必要的注释

代码注释不是越多越好，保持必要的业务逻辑注释，至于函数的用途、代码逻辑等，要通过语义化的命令、简单明了的代码逻辑，来让阅读代码的人快速看懂。