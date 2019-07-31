# 0 到 1 掌握：Vue 核心之数据双向绑定

## 前言

​	当被问到 Vue 数据双向绑定原理的时候，大家可能都会脱口而出：Vue 内部通过    `Object.defineProperty`方法属性拦截的方式，把 `data` 对象里每个数据的读写转化成 `getter`/`setter`，当数据变化时通知视图更新。虽然一句话把大概原理概括了，但是其内部的实现方式还是值得深究的，本文就以通俗易懂的方式剖析 `Vue` 内部双向绑定原理的实现过程。然后再根据 `Vue` 源码的数据双向绑定实现，来进一步巩固加深对数据双向绑定的理解认识。以下为我们实现的数据双向绑定的效果图：

![1.gif](https://github.com/fengshi123/blog/blob/master/assets/mvvm/1.gif?raw=true) 

**github地址为：github.com/fengshi123/…，上面汇总了作者所有的博客文章，如果喜欢或者有所启发，请帮忙给个 star ~，对作者也是一种鼓励。** 



## 一、什么是 MVVM 数据双向绑定

`MVVM` 数据双向绑定主要是指：数据变化更新视图，视图变化更新数据，如下图所示：

![2.png](https://github.com/fengshi123/blog/blob/master/assets/mvvm/2.png?raw=true) 

即：

- 输入框内容变化时，`Data` 中的数据同步变化。即 `View` => `Data` 的变化。
- `Data` 中的数据变化时，文本节点的内容同步变化。即 `Data` => `View` 的变化。

其中，`View` 变化更新 `Data` ，可以通过事件监听的方式来实现，所以我们本文主要讨论如何根据 `Data` 变化更新 `View`。

我们会通过实现以下 4 个步骤，来实现数据的双向绑定：

1、实现一个监听器 `Observer` ，用来劫持并监听所有属性，如果属性发生变化，就通知订阅者；

2、实现一个订阅器 `Dep`，用来收集订阅者，对监听器 `Observer` 和 订阅者 `Watcher` 进行统一管理；

3、实现一个订阅者 `Watcher`，可以收到属性的变化通知并执行相应的方法，从而更新视图；

4、实现一个解析器 `Compile`，可以解析每个节点的相关指令，对模板数据和订阅器进行初始化。

以上四个步骤的流程图表示如下：

![3.png](https://github.com/fengshi123/blog/blob/master/assets/mvvm/3.png?raw=true) 

## 二、监听器 Observer 实现

监听器 `Observer` 的实现，主要是指让数据对象变得“可观测”，即每次数据读或写时，我们能感知到数据被读取了或数据被改写了。要使数据变得“可观测”，`Vue 2.0` 源码中用到 `Object.defineProperty()`  来劫持各个数据属性的 `setter / getter`，`Object.defineProperty` 方法，在 MDN 上是这么定义的：

>  Object.defineProperty() 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。 

### 2.1、Object.defineProperty() 语法

`Object.defineProperty` 语法，在 MDN 上是这么定义的：

> Object.defineProperty(obj, prop, descriptor)

**（1）参数**

- `obj`

  要在其上定义属性的对象。

- `prop`

  要定义或修改的属性的名称。

- `descriptor`

  将被定义或修改的属性描述符。

**（2）返回值**

​      被传递给函数的对象。 

**（3）属性描述符**

`Object.defineProperty()` 为对象定义属性，分 数据描述符 和 存取描述符 ，两种形式不能混用。

**数据描述符和存取描述符均具有以下可选键值：**

- `configurable`

当且仅当该属性的 `configurable` 为 `true` 时，该属性描述符才能够被改变，同时该属性也能从对应的对象上被删除。**默认为 false**。

- `enumerable`

当且仅当该属性的 `enumerable` 为 `true` 时，该属性才能够出现在对象的枚举属性中。**默认为 false**。

**数据描述符具有以下可选键值**： 

- `value`

该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。**默认为 undefined**。

- `writable`

当且仅当该属性的 `writable` 为 `true` 时，`value `才能被赋值运算符改变。**默认为 false**。

**存取描述符具有以下可选键值**： 

- `get`

一个给属性提供 `getter` 的方法，如果没有 `getter` 则为 `undefined`。当访问该属性时，该方法会被执行，方法执行时没有参数传入，但是会传入`this`对象（由于继承关系，这里的`this`并不一定是定义该属性的对象）。默认为 `undefined`。

- `set`

一个给属性提供 `setter` 的方法，如果没有 `setter` 则为 `undefined`。当属性值修改时，触发执行该方法。该方法将接受唯一参数，即该属性新的参数值。默认为 `undefined`。

### 2.2、监听器 Observer 实现

**（1）字面量定义对象**

首先，我们先看一下假设我们通过以下字面量的方式定义一个对象：

```javascript
let person = {
    name:'tom',
    age:15
}
```

我们可以通过 `person.name` 和 `person.age` 直接读写这个 `person` 对应的属性值，但是，当这个 `person` 的属性被读取或修改时，我们并不知情。那么，应该如何定义一个对象，它的属性被读写时，我们能感知到呢？

**（2）Object.defineProperty() 定义对象**

假设我们通过 `Object.defineProperty()` 来定义一个对象：

```javascript
let val = 'tom'
let person = {}
Object.defineProperty(person,'name',{
    get(){
        console.log('name属性被读取了...');
        return val;
    },
    set(newVal){
        console.log('name属性被修改了...');
        val = newVal;
    }
})
```

我们通过 `object.defineProperty()` 方法给  `person` 的 `name` 属性定义了 `get()` 和`set()`进行拦截，每当该属性进行读或写操作的时候就会触发`get()`和`set()` ，这样，当对象的属性被读写时，我们就能感知到了。测试结果图如下所示：

![1564302855198](C:\Users\fengshi\AppData\Local\Temp\1564302855198.png) 

**（3）改进方法**

通过第（2）步的方法，`person` 数据对象已经是“可观测”的了，能满足我们的需求了。但是如果数据对象的属性比较多的情况下，我们一个一个为属性去设置，代码会非常冗余，所以我们进行以下封装，从而让数据对象的所有属性都变得可观测：

```javascript
/**
  * 循环遍历数据对象的每个属性
  */
function observable(obj) {
    if (!obj || typeof obj !== 'object') {
        return;
    }
    let keys = Object.keys(obj);
    keys.forEach((key) => {
        defineReactive(obj, key, obj[key])
    })
    return obj;
}
/**
 * 将对象的属性用 Object.defineProperty() 进行设置
 */
function defineReactive(obj, key, val) {
    Object.defineProperty(obj, key, {
        get() {
            console.log(`${key}属性被读取了...`);
            return val;
        },
        set(newVal) {
            console.log(`${key}属性被修改了...`);
            val = newVal;
        }
    })
}
```

通过以上方法封装，我们可以直接定义 `person`：

```javascript
let person = observable({
    name: 'tom',
    age: 15
});
```

这样定义的 `person` 的 两个属性都是“可观测”的。

## 三、订阅器 Dep 实现

### 3.1、发布 —订阅设计模式

​	发布-订阅模式又叫观察者模式，它定义对象间的一种一对多的依赖关系，当一个对象的状态改变时，所有依赖于它的对象都将得到通知。

**（1）发布—订阅模式的优点：**

- 发布-订阅模式广泛应用于异步编程中，这是一种替代传递回调函数的方案，比如，我们可以订阅 ajax 请求的 error 、succ 等事件。在异步编程中使用发布-订阅模式， 我们就无需过多关注对象在异步运行期间的内部状态，而只需要订阅感兴趣的事件发生点。
- 发布-订阅模式可以取代对象之间硬编码的通知机制，一个对象不用再显式地调用另外一个对象的某个接口。发布-订阅模式让两个对象松耦合地联系在一起，虽然不太清楚彼此的细节，但这不影响它们之间相互通信。当有新的订阅者出现时，发布者的代码不需要任何修改；同样发布者需要改变时，也不会影响到之前的订阅者。只要之前约定的事件名没有变化，就 可以自由地改变它们。

**（2）发布—订阅模式的生活实例**

​	我们以售楼处的例子来举例说明发布-订阅模式：

​	小明最近看上了一套房子，到了售楼处之后才被告知，该楼盘的房子早已售罄。好在售楼 MM 告诉小明，不久后还有一些尾盘推出，开发商正在办理相关手续，手续办好后便可以购买。

​	但到底是什么时候，目前还没有人能够知道。 于是小明记下了售楼处的电话，以后每天都会打电话过去询问是不是已经到了购买时间。除 了小明，还有小红、小强、小龙也会每天向售楼处咨询这个问题。一个星期过后，售楼 MM 决定辞职，因为厌倦了每天回答 1000个相同内容的电话。

​	当然现实中没有这么笨的销售公司，实际上故事是这样的：小明离开之前，把电话号码留在 了售楼处。售楼 MM 答应他，新楼盘一推出就马上发信息通知小明。小红、小强和小龙也是一样，他们的电话号码都被记在售楼处的花名册上，新楼盘推出的时候，售楼 MM会翻开花名册，遍历上面的电话号码，依次发送一条短信来通知他们。这就是发布-订阅模式在现实中的例子。

### 3.2、订阅器 Dep 实现

​	完成了数据的'可观测'，即我们知道了数据在什么时候被读或写了，那么，我们就可以在数据被读或写的时候通知那些依赖该数据的视图更新了，为了方便，我们需要先将所有依赖收集起来，一旦数据发生变化，就统一通知更新。其实，这就是前一节所说的“发布订阅者”模式，数据变化为“发布者”，依赖对象为“订阅者”。 

​	现在，我们需要创建一个依赖收集容器，也就是消息订阅器 `Dep`，用来容纳所有的“订阅者”。订阅器 `Dep` 主要负责收集订阅者，然后当数据变化的时候后执行对应订阅者的更新函数。

创建消息订阅器 `Dep`:

```javascript
function Dep () {
    this.subs = [];
}
Dep.prototype = {
    addSub: function(sub) {
        this.subs.push(sub);
    },
    notify: function() {
        this.subs.forEach(function(sub) {
            sub.update();
        });
    }
};
Dep.target = null;
```

有了订阅器，我们再将 `defineReactive` 函数进行改造一下，向其植入订阅器：

```javascript
defineReactive: function(data, key, val) {
	var dep = new Dep();
	Object.defineProperty(data, key, {
		enumerable: true,
		configurable: true,
		get: function getter () {
			if (Dep.globalWatcher) {
				dep.addWatcher(Dep.globalWatcher);
			}
			return val;
		},
		set: function setter (newVal) {
			if (newVal === val) {
				return;
			}
			val = newVal;
			dep.notify();
		}
	});
}
```

从代码上看，我们设计了一个订阅器 `Dep` 类，该类里面定义了一些属性和方法，这里需要特别注意的是它有一个静态属性 `Dep.target`，这是一个全局唯一 的`Watcher`，因为在同一时间只能有一个全局的 `Watcher` 被计算，另外它的自身属性 `subs` 也是 `Watcher` 的数组。 

## 四、订阅者 Watcher 实现

订阅者 `Watcher` 在初始化的时候需要将自己添加进订阅器 `Dep` 中，那该如何添加呢？我们已经知道监听器`Observer` 是在 get 函数执行了添加订阅者 Wather 的操作的，所以我们只要在订阅者 `Watcher` 初始化的时候触发对应的 `get` 函数去执行添加订阅者操作即可，那要如何触发 `get` 的函数，再简单不过了，只要获取对应的属性值就可以触发了，核心原因就是因为我们使用了 `Object.defineProperty( )` 进行数据监听。这里还有一个细节点需要处理，我们只要在订阅者 `Watcher` 初始化的时候才需要添加订阅者，所以需要做一个判断操作，因此可以在订阅器上做一下手脚：在 `Dep.target` 上缓存下订阅者，添加成功后再将其去掉就可以了。订阅者 `Watcher` 的实现如下： 

```javascript
function Watcher(vm, exp, cb) {
    this.vm = vm;
    this.exp = exp;
    this.cb = cb;
    this.value = this.get();  // 将自己添加到订阅器的操作
}

Watcher.prototype = {
    update: function() {
        this.run();
    },
    run: function() {
        var value = this.vm.data[this.exp];
        var oldVal = this.value;
        if (value !== oldVal) {
            this.value = value;
            this.cb.call(this.vm, value, oldVal);
        }
    },
    get: function() {
        Dep.target = this; // 全局变量 订阅者 赋值
        var value = this.vm.data[this.exp]  // 强制执行监听器里的get函数
        Dep.target = null; // 全局变量 订阅者 释放
        return value;
    }
};
```

订阅者 `Watcher` 分析如下：

订阅者 `Watcher` 是一个 类，在它的构造函数中，定义了一些属性：

- **vm：**一个 Vue 的实例对象；
- **exp：**是 `node` 节点的 `v-model` 等指令的属性值 或者插值符号中的属性。如 `v-model="name"`，`exp` 就是`name`;
- **cb：**是 `Watcher` 绑定的更新函数;

当我们去实例化一个渲染 `watcher` 的时候，首先进入 `watcher` 的构造函数逻辑，就会执行它的 `this.get()` 方法，进入 `get` 函数，首先会执行：

```javascript
Dep.target = this;  // 将自己赋值为全局的订阅者
```

实际上就是把 `Dep.target` 赋值为当前的渲染 `watcher` ,接着又执行了：

```
let value = this.vm.data[this.exp]  // 强制执行监听器里的get函数
```

在这个过程中会对 `vm` 上的数据访问，其实就是为了触发数据对象的 `getter`。

每个对象值的 `getter` 都持有一个 `dep`，在触发 `getter` 的时候会调用 `dep.depend()` 方法，也就会执行`this.addSub(Dep.target)`，即把当前的 `watcher` 订阅到这个数据持有的 `dep` 的 `watchers` 中，这个目的是为后续数据变化时候能通知到哪些 `watchers` 做准备。

这样实际上已经完成了一个依赖收集的过程。那么到这里就结束了吗？其实并没有，完成依赖收集后，还需要把 `Dep.target` 恢复成上一个状态，即：

```javascript
Dep.target = null;  // 释放自己
```

而 `update()` 函数是用来当数据发生变化时调用 `Watcher` 自身的更新函数进行更新的操作。先通过 `let value = this.vm.data[this.exp];` 获取到最新的数据,然后将其与之前 `get()` 获得的旧数据进行比较，如果不一样，则调用更新函数 `cb` 进行更新。

至此，简单的订阅者 `Watcher` 设计完毕。



## 五、解析器 Compile 实现

### 5.1、解析器 Compile 关键逻辑代码分析

​	通过监听器 `       Observer` 订阅器 `Dep` 和订阅者 `Watcher` 的实现，其实就已经实现了一个双向数据绑定的例子，但是整个过程都没有去解析 `dom` 节点，而是直接固定某个节点进行替换数据的，所以接下来需要实现一个解析器 `Compile` 来做解析和绑定工作。解析器 `Compile` 实现步骤： 

- 解析模板指令，并替换模板数据，初始化视图；
- 将模板指令对应的节点绑定对应的更新函数，初始化相应的订阅器；

我们下面对 '{{变量}}' 这种形式的指令处理的关键代码进行分析，感受解析器 `Compile` 的处理逻辑，关键代码如下：

```javascript
compileText: function(node, exp) {
	var self = this;
	var initText = this.vm[exp]; // 获取属性值
	this.updateText(node, initText); // dom 更新节点文本值
    // 将这个指令初始化为一个订阅者，后续 exp 改变时，就会触发这个更新回调，从而更新视图
	new Watcher(this.vm, exp, function (value) { 
		self.updateText(node, value);
	});
}
```

### 5.2、简单实现一个 Vue 实例

完成监听器 `Observer` 、订阅器 `Dep` 、订阅者 `Watcher` 和解析器 `Compile` 的实现，我们就可以模拟初始化一个`Vue` 实例，来检验以上的理论的可行性了。我们通过以下代码初始化一个 `Vue` 实例：

```javascript
<body>
    <div id="mvvm-app">
        <input v-model="title">
        <h2>{{title}}</h2>
        <button v-on:click="clickBtn">数据初始化</button>
    </div>
</body>
<script src="../dist/bundle.js"></script>
<script type="text/javascript">
    var vm = new MVVM({
        el: '#mvvm-app',
        data: {
            title: 'hello world'
        },

        methods: {
            clickBtn: function (e) {
                this.title = 'hello world';
            }
        },
    });
</script>
```

运行以上实例，效果图如下所示，跟实际的 Vue 数据绑定效果是不是一样！

![1.gif](https://github.com/fengshi123/blog/blob/master/assets/mvvm/1.gif?raw=true)  

## 六、Vue 源码 — 数据双向绑定

以上第二章节到第六章节，从监听器 `Observer` 、订阅器 `Dep` 、订阅者 `Watcher` 和解析器 `Compile` 的实现，完成了一个简单的 `Vue` 数据绑定实例的实现。本章节，我们从 `Vue` 源码层面分析监听器 `Observer` 、订阅器 `Dep` 、订阅者 `Watcher` 的实现，帮助大家了解 `Vue` 源码如何实现数据双向绑定。

### 6.1、监听器 Observer 实现

我们在本小节主要介绍 监听器 Observer 实现，核心就是利用 `Object.defineProperty` 给数据添加了 `getter` 和 setter，目的就是为了在我们访问数据以及写数据的时候能自动执行一些逻辑 。

**（1）initState**

在 `Vue` 的初始化阶段，`_init` 方法执行的时候，会执行 `initState(vm)` 方法，它的定义在 `src/core/instance/state.js` 中。 

```javascript
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```

`initState` 方法主要是对 `props`、`methods`、`data`、`computed` 和 `wathcer` 等属性做了初始化操作。这里我们重点分析 `data`，对于其它属性的初始化我们在以后的文章中再做介绍。

**（2）initData**

```javascript
function initData (vm: Component) {
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  if (!isPlainObject(data)) {
    data = {}
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object:\n' +
      'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
      vm
    )
  }
  // proxy data on instance
  const keys = Object.keys(data)
  const props = vm.$options.props
  const methods = vm.$options.methods
  let i = keys.length
  while (i--) {
    const key = keys[i]
    if (process.env.NODE_ENV !== 'production') {
      if (methods && hasOwn(methods, key)) {
        warn(
          `Method "${key}" has already been defined as a data property.`,
          vm
        )
      }
    }
    if (props && hasOwn(props, key)) {
      process.env.NODE_ENV !== 'production' && warn(
        `The data property "${key}" is already declared as a prop. ` +
        `Use prop default value instead.`,
        vm
      )
    } else if (!isReserved(key)) {
      proxy(vm, `_data`, key)
    }
  }
  // observe data
  observe(data, true /* asRootData */)
}
```

`data` 的初始化主要过程也是做两件事，一个是对定义 `data` 函数返回对象的遍历，通过 `proxy` 把每一个值 `vm._data.xxx` 都代理到 `vm.xxx` 上；另一个是调用 `observe` 方法观测整个 `data` 的变化，把 `data` 也变成响应式，我们接下去主要介绍 observe 。 

**（3）observe**

`observe` 的功能就是用来监测数据的变化，它的定义在 `src/core/observer/index.js` 中： 

```javascript
export function observe (value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  let ob: Observer | void
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else if (
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    ob = new Observer(value)
  }
  if (asRootData && ob) {
    ob.vmCount++
  }
  return ob
}
```

`observe` 方法的作用就是给非 VNode 的对象类型数据添加一个 `Observer`，如果已经添加过则直接返回，否则在满足一定条件下去实例化一个 `Observer` 对象实例。接下来我们来看一下 `Observer` 的作用。 

**（4）Observer**

`Observer` 是一个类，它的作用是给对象的属性添加 getter 和 setter，用于依赖收集和派发更新： 

```javascript
xport class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that have this object as root $data

  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      if (hasProto) {
        protoAugment(value, arrayMethods)
      } else {
        copyAugment(value, arrayMethods, arrayKeys)
      }
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }

  /**
   * Walk through all properties and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }

  /**
   * Observe a list of Array items.
   */
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
```

`Observer` 的构造函数逻辑很简单，首先实例化 `Dep` 对象， `Dep` 对象，我们第2小节会介绍。接下来会对 `value` 做判断，对于数组会调用 `observeArray` 方法，否则对纯对象调用 `walk` 方法。可以看到 `observeArray` 是遍历数组再次调用 `observe` 方法，而 `walk` 方法是遍历对象的 key 调用 `defineReactive` 方法，那么我们来看一下这个方法是做什么的。 

**（5）defineReactive**

`defineReactive` 的功能就是定义一个响应式对象，给对象动态添加 `getter` 和 `setter`，它的定义在 `src/core/observer/index.js` 中： 

```javascript
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }

  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      // #7981: for accessor properties without setter
      if (getter && !setter) return
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
```

`defineReactive` 函数最开始初始化 `Dep` 对象的实例，接着拿到 `obj` 的属性描述符，然后对子对象递归调用 `observe` 方法，这样就保证了无论 `obj` 的结构多复杂，它的所有子属性也能变成响应式的对象，这样我们访问或修改 `obj` 中一个嵌套较深的属性，也能触发 getter 和 setter。 

### 6.2、订阅器 Dep 实现

订阅器`Dep` 是整个 `getter` 依赖收集的核心，它的定义在 `src/core/observer/dep.js` 中： 

```javascript
export default class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;

  constructor () {
    this.id = uid++
    this.subs = []
  }

  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    if (process.env.NODE_ENV !== 'production' && !config.async) {
      // subs aren't sorted in scheduler if not running async
      // we need to sort them now to make sure they fire in correct
      // order
      subs.sort((a, b) => a.id - b.id)
    }
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

// The current target watcher being evaluated.
// This is globally unique because only one watcher
// can be evaluated at a time.
Dep.target = null
```

`Dep` 是一个 `Class`，它定义了一些属性和方法，这里需要特别注意的是它有一个静态属性 `target`，这是一个全局唯一 `Watcher`，这是一个非常巧妙的设计，因为在同一时间只能有一个全局的 `Watcher` 被计算，另外它的自身属性 `subs` 也是 `Watcher` 的数组。`Dep` 实际上就是对 `Watcher` 的一种管理，`Dep` 脱离 `Watcher` 单独存在是没有意义的。

### 6.3、订阅者 Watcher 实现

订阅者`Watcher` 的一些相关实现，它的定义在 `src/core/observer/watcher.js` 中 

```javascript
export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: SimpleSet;
  newDepIds: SimpleSet;
  before: ?Function;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    this.vm = vm
    if (isRenderWatcher) {
      vm._watcher = this
    }
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
      this.before = options.before
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.lazy // for lazy watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = noop
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    this.value = this.lazy
      ? undefined
      : this.get()
  }
   。。。。。。
}
```

`Watcher` 是一个 `Class`，在它的构造函数中，定义了一些和 `Dep` 相关的属性 ，其中，`this.deps` 和 `this.newDeps` 表示 `Watcher` 实例持有的 `Dep` 实例的数组；而 `this.depIds` 和 `this.newDepIds` 分别代表 `this.deps` 和 `this.newDeps` 的 `id` Set 。

**（1）过程分析**

当我们去实例化一个渲染 `watcher` 的时候，首先进入 `watcher` 的构造函数逻辑，然后会执行它的 `this.get()` 方法，进入 `get` 函数，首先会执行： 

```javascript
pushTarget(this)
```

实际上就是把 `Dep.target` 赋值为当前的渲染 `watcher` 并压栈（为了恢复用）。接着又执行了：

```javascript
value = this.getter.call(vm, vm)
```

这个时候就触发了数据对象的 `getter` 。

么每个对象值的 `getter` 都持有一个 `dep`，在触发 `getter` 的时候会调用 `dep.depend()` 方法，也就会执行 `Dep.target.addDep(this)`。

刚才我们提到这个时候 `Dep.target` 已经被赋值为渲染 `watcher`，那么就执行到 `addDep` 方法：

```
addDep (dep: Dep) {
  const id = dep.id
  if (!this.newDepIds.has(id)) {
    this.newDepIds.add(id)
    this.newDeps.push(dep)
    if (!this.depIds.has(id)) {
      dep.addSub(this)
    }
  }
}
```

这时候会做一些逻辑判断（保证同一数据不会被添加多次）后执行 `dep.addSub(this)`，那么就会执行 `this.subs.push(sub)`，也就是说把当前的 `watcher` 订阅到这个数据持有的 `dep` 的 `subs` 中，这个目的是为后续数据变化时候能通知到哪些 `subs` 做准备。所以在 `vm._render()` 过程中，会触发所有数据的 `getter`，这样实际上已经完成了一个依赖收集的过程。

当我们在组件中对响应的数据做了修改，就会触发 `setter` 的逻辑，最后调用 `watcher` 中的 `update` 方法： 

```javascript
  update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      this.run()
    } else {
      queueWatcher(this)
    }
  }
```

这里会对于 `Watcher` 的不同状态，会执行不同的更新逻辑。

 

## 七、总结

本文通过监听器 `Observer` 、订阅器 `Dep` 、订阅者 `Watcher` 和解析器 ·的实现，模拟初始化一个 `Vue` 实例，帮助大家了解数据双向绑定的基本原理。接着，从 `Vue` 源码层面介绍了 `Vue` 数据双向绑定的实现过程，了解 `Vue` 源码的实现逻辑，从而巩固加深对数据双向绑定的理解认识。希望本文对您有帮助。

**github地址为：github.com/fengshi123/…，上面汇总了作者所有的博客文章，如果喜欢或者有所启发，请帮忙给个 star ~，对作者也是一种鼓励。**



## 参考文献

1、Vue 的双向绑定原理及实现：<https://www.cnblogs.com/canfoo/p/6891868.html> 

2、Vue 技术揭秘：<https://ustbhuangyi.github.io/vue-analysis/> 

 

 

 