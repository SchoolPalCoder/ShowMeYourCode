# Vue源码学习1——Vue构造函数#

这是我第一次正式阅读大型框架源码，刚开始的时候完全不知道该如何入手。Vue源码clone下来之后这么多文件夹，Vue的这么多方法和概念都在哪，完全没有头绪。现在也只是很粗略的了解一下，个人认为这篇只是能做到大家阅读Vue的参考导航，可以较快的找到需要看的文件或方法。很多细节依然没有理解到位，但是可以慢慢来，先分享一波~

## 源码文件目录结构 ##

![](https://i.imgur.com/6VHLK18.png)

**- benchmarks** 暂时不知道是什么

**- dist**  存放打包后的文件夹

**- examples** 示例，这个地方可以自己写一些简单例子，然后通过调试看整个代码运行的过程来了解源码是怎么写的 

**- flow** 静态类型检查，比如 （n:number）即n需要是number类型

**- packages** 查资料说是vue还可以分别生成其他的npm包

**- scripts** 打包相关的配置文件夹

**- src 我们研究的主要文件夹，下面会详细再说明**

**- test** 测试文件夹

**- types** 暂时不知道是什么

## /src ##

接下来重点说src这个文件夹，这里面需要重点看core这个文件夹，这里面才是我们真正需要研究的地方如下图：

![](https://i.imgur.com/AgFCJ2C.png)

**- components**  组件，现在里面只有KeepAlive一个

**- global-api**  全局api,可以给Vue添加全局方法，比如里面我们常使用的Vue.use()

**-instance**  核心文件夹，里面是实例相关的一些方法，例如初始化实例、实例事件绑定、渲染、状态、生命周期等

**- observe** 双向数据绑定相关文件（暂时不太清楚）

**- util** 工具方法，看到里面有props、nextTick之类的方法（暂时不太清楚）

**- vdom** 虚拟dom

大体文件结构说了一下，但是很多还不是很清晰。对于我这样的小白来说，我的建议是可以从 `npm run dev` 开始一步步开始看，采取的方法是“倒序”

## package.json ##

打开这个文件找到'dev'命令，如下

     "dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev",
    
'rollup' 是Vue使用的打包工具，从上面可以看出执行这个命令是到 'scripts/config.js' 那就打开这个文件

## scripts/config.js ##

```js
if (process.env.TARGET) {
  module.exports = genConfig(process.env.TARGET)
} else {
  exports.getBuild = genConfig
  exports.getAllBuilds = () => Object.keys(builds).map(genConfig)
}
```

genConfig 方法是设置一些配置，和webpack里的设置差不多，然后找到 web-full-dev
```js
  'web-full-dev': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.js'),
    format: 'umd',
    env: 'development',
    alias: { he: './entity-decoder' },
    banner
  }
```
可以看到入口文件是'web/entry-runtime-with-compiler.js' 

## src/platforms/web/entry-runtime-with-compiler.js ##
在这个文件里可以看到
    
    import Vue from './runtime/index'

该方法中有一个$mount需要注意，这个就是渲染的入口，接下来会说这个方法

## src/platforms/web/runtime/index.js ##
	`import Vue from 'core/index'`

## src/core/index.js ##
    import Vue from './instance/index'
## src/core/instance/index.js ##
终于进入到最重要的方法，Vue的构造方法如下

```js
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue(options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```
- initMixin ：对于各种vue实例各种属性进行初始化
- stateMixin ：Vue原型上绑定state相关的方法和属性，data、props等
- eventsMixin ：Vue原型上绑定事件相关方法
- lifecycleMixin ：Vue原型上绑定生命周期相关方法，比如_update、$forceUpdate、$destroy
- renderMixin : Vue原型上绑定和渲染相关的方法

## src/core/instance/init.js ##
- **第一点：**initMixin 是Vue的一些初始化实例的方法，在还没有构造一个对象前是不会进入到这个方法内部，当通过new出一个对象后才会进入，原因如下：
```js
 Vue.prototype._init = function (options?: Object) {
```
这里有一个`options?:object`的校验，刚开始即只是引入`<script src="../../dist/vue.js"></script>`这个文件，当` var vm = new Vue()`之后才进入_init方法内部。

- **第二点：**_init方法中对于$options设置：
```js
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
```
mergeOptions这个方法是合并option,第一个参数是往$options塞入下面的参数

![](https://i.imgur.com/FxYtBge.png)

第二个参数就是我们自己设置的option，比如data、el；第三个参数如下：

![](https://i.imgur.com/xopx8NG.png)

- **第三点：**_init方法中其他初始化方法
```js
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
```


接下来将会一个个初始化方法说明，初次之外_init方法还有一些变量的初始化，比如_uid、_isVue、_name、_renderProxy的初始化

- **第四点：**最后在_init方法中需要注意
```js
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
```
调用$mount挂载根元素，这个方法就是之前提到的

## src/core/instance/state.js ##
- **第一点：** stateMixin是对于Vue原型对象(Vue.prototype)加上$data、$props、$delete、$watch、$set属性。并且通过Object.defineProperty对$data、$props属性进行set和get

- **第二点：**initState方法是在init.js中调用，即实例化之后才调用的，是个实例对象添加属性。
```js
export function initState(vm: Component) {
// 首先在vm上初始化一个_watchers数组，缓存这个vm上的所有watcher
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
对于实例对象进行相关属性的初始化，另外data、props因为需要双向绑定，在initData、initProps中都有一个proxy方法对这两个属性进行set和get的设置

## src/core/instance/events.js ##
- **第一点：** eventsMixin是对于Vue原型对象(Vue.prototype)绑定一些事件方法，比如$on、$once、$off、$emit

- **第二点：** initEvents是对于实例对象初始化事件
```js
export function initEvents(vm: Component) {
  vm._events = Object.create(null)
  vm._hasHookEvent = false
  // init parent attached events 初始化父级相关事件
  const listeners = vm.$options._parentListeners
  if (listeners) {
    updateComponentListeners(vm, listeners)
  }
}
```
创建_events一个空对象之后用来存放事件，_hasHookEvent是一个优化标记（可以暂时不理会），然后初始化父级事件。根据是否有父级监听事件，如果有则更新父级事件

## src/core/instance/lifecycle.js ##
- **第一点：** lifecycleMixin是对Vue原型对象(Vue.prototype)绑定_update、$forceUpdate、$destroy三个生命周期方法。_update方法中通过调用__patch__方法更新虚拟dom；
```js
    if (!prevVnode) {
      // initial render
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
    } else {
      // updates
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
```
$forceUpdate强制重新渲染实例本身和插入插槽内容的子组件；$destroy销毁一个实例，清理它与其它实例的连接，解绑它的全部指令及事件监听器，触发 beforeDestroy 和 destroyed 的钩子

- **第二点：** initLifecycle是在_init方法中调用，是实例生命周期的初始化，其中会包括很多变量
```js
export function initLifecycle(vm: Component) {
  const options = vm.$options

  // locate first non-abstract parent 创建第一个非抽象父组件，抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中，例如<keep-alive>
  let parent = options.parent
  if (parent && !options.abstract) {
    while (parent.$options.abstract && parent.$parent) {
      parent = parent.$parent
    }
    parent.$children.push(vm)
  }

  vm.$parent = parent
  vm.$root = parent ? parent.$root : vm

  vm.$children = []
  vm.$refs = {}

  vm._watcher = null // watcher对象
  vm._inactive = null // 和keep-alive中组件状态有关系
  vm._directInactive = false // 和keep-alive中组件状态有关系
  vm._isMounted = false //当前实例是否被挂载
  vm._isDestroyed = false // 当前实例是否被销毁
  vm._isBeingDestroyed = false // 当前实例是否正在被销毁或者没销毁完全
}
```

- **第三点：** callHook是在_init方法中调用，这个方法是直接调用钩子，调用形式如下
    `callHook(vm, 'beforeCreate')`
	`callHook(vm, 'created')`

## src/core/instance/render.js ##
- **第一点：** renderMixin方法主要是给Vue原型对象绑定$nextTick、_render两个方法,其中_render方法代码如下：
```js
  Vue.prototype._render = function (): VNode {
    ……
    // set parent vnode. this allows render functions to have access
    // to the data on the placeholder node.
    vm.$vnode = _parentVnode
    // render self
    let vnode
    try {
      vnode = render.call(vm._renderProxy, vm.$createElement)
    } catch (e) {
      handleError(e, vm, `render`)
      if (process.env.NODE_ENV !== 'production') {
        if (vm.$options.renderError) {
          try {
            vnode = vm.$options.renderError.call(vm._renderProxy, vm.$createElement, e)
          } catch (e) {
            handleError(e, vm, `renderError`)
            vnode = vm._vnode
          }
        } else {
          vnode = vm._vnode
        }
      } else {
        vnode = vm._vnode
      }
    }
    ……
    return vnode
  }
```
在这个方法中主要是try……catch这里创建了vnode。 `vnode = render.call(vm._renderProxy, vm.$createElement)` 创建一个vnode并且返回，如果失败则返回一个空的vnode `vnode = createEmptyVNode()`

- **第二点：** initRender是在_init方法中调用，进行实例渲染属性的绑定并且对一些属性的监听
```js
export function initRender(vm: Component) {
  ……
  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
  // normalization is always applied for the public version, used in
  // user-written render functions.
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)
  ……
}
```
这里着重关注一下createElement方法，传入vnode以及dom的属性创建真正dom节点。



## src/core/instance/inject.js ##
在_init方法中调用了initProvide、initInjections两个方法，这两个方法在实际应用中不是很多，查看Vue API说provide 和 inject 主要为高阶插件/组件库提供用例。并不推荐直接用于应用程序代码中。所以这里不做说明，有需要的可以到这个文件查看相关方法

## vue的渲染过程 ##

- **第一步：** _init方法中
```js
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
```
渲染入口，调用$mount方法开始

- **第二步：**entry-runtime-with-compiler.js中的$mount方法，代码如下
```js
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && query(el)
  ……
  if (!options.render) {
    let template = options.template
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') { // 图一
          template = idToTemplate(template)
			……
        }
      } else if (template.nodeType) {  // 图一 
        template = template.innerHTML
      } else { // 图二
		 ……
         return this
      }

    } else if (el) { // 图三
      template = getOuterHTML(el)
    }

    if (template) {
	  ……

      const { render, staticRenderFns } = compileToFunctions(template, {
        shouldDecodeNewlines,
        shouldDecodeNewlinesForHref,
        delimiters: options.delimiters,
        comments: options.comments
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns

	   ……
    }
  }
  return mount.call(this, el, hydrating)
}
```
可以从上面大致看出结构，template是可以从el传入，也可以是options中的template以及render方法三种方式传入，对应Vue官网如下：
![](https://i.imgur.com/4y1JmCB.png)
![](https://i.imgur.com/Rbhg4gl.png)

![](https://i.imgur.com/37PZMOk.png)

其中可以看到，通过el或者template的方式都需要调用compileToFunctions将字符串转换成方法，而render是不需要，这里可以看出render的性能应该会好一些，但是el和template我们使用较易理解。但是不管是哪一种最后都是生成render方法，然后再绑定到实例对象上。另外方法中的mount是从runtime/index.js中创建的。

- **第三步：** 接下来就进入runtime/index.js看到mount方法调用mountComponent，然后找到这个方法是在lifecycle.js
```js
export function mountComponent(
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  ……
  callHook(vm, 'beforeMount')

  let updateComponent
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = () => {
		……
      const vnode = vm._render()
		……
      vm._update(vnode, hydrating)
		……
    }
  } else {
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  }

  new Watcher(vm, updateComponent, noop, {
 		……
     callHook(vm, 'beforeUpdate')
 		……
  }, true /* isRenderWatcher */)
		……
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```

然后调用前一步调用的_render方法是在render.js中的_render方法中try……catch地方调用了第二步中生成的render方法。通过_render方法生成vnode，传入_update方法

- **第四步：**最后在前面也提到在_update方法中有一个patch对比更新真实dom,这里是涉及到diff算法进行对比新旧VNode对象进行更新，暂时不太了解,下图是我网上找到可以比较形象解释diff、patch的作用
![](https://i.imgur.com/BL5NnUw.png)

以上就是一个大体渲染的过程。

## 总结 ##
本文只是做了Vue构造函数整体的一个流程展示，哪些参数是在哪个文件中挂载上去的以及vue渲染的一个简单流程。但其实每个环节都可以拓展出很多知识，比如响应式的数据绑定、虚拟DOM、diff算法、patch、生命周期等等，这些可以在之后再一个个点进行了解。下图是没有参数的vue实例的参数
![](https://i.imgur.com/Ar7fG54.png)

