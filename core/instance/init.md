## init.js


init.js会返回3个函数：
1. initMixin
2. initInternalComponent
3. resolveConstructorOptions

另外init.js还会初始化一个变量uid为0。
```javascript
let uid = 0
```


### initMixin

```javascript
function initMixin (Vue: Class<Component>) {}
```

initMixin 接受一个参数，这个参数的数据类型是一个component类。<br />
initMixin函数会给这个参数组件的原型上添加一个_init的函数。
```javascript
Vue.prototype._init = function (options?: Object) {
    
}
```
_init函数接受1个options参数，options是一个对象。

_init函数首先会定义一个变量vm，类型是component，把this赋值给vm。
```javascript
const vm: Component = this
```

之后会给vm添加一个属性，值为uid自加。
```javascript
 vm._uid = uid++
```

添加一个属性_isVue来标志当前this
```javascript
 vm._isVue = true
```

接下来会去合并options
```javascript
if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      
      //优化内部组件初始化，因为一个动态的Options合并是非常慢的，并且内部组件的options不需要去特殊对待。
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
```
如果options存在并且是一个组件，就把vm和options传递给初始化内部组件的函数。否则就取合并options。<br />
接下来会把vm自己暴露出去
```javascript
 vm._self = vm
```
初始化生命周期
```javascript
initLifecycle(vm)
```
初始化事件
```javascript
initEvents(vm)
```
渲染初始化
```javascript
initRender(vm)
```
触发创建前的钩子函数
```javascript
callHook(vm, 'beforeCreate')
```
在data和pros注入前初始化injections
```javascript
initInjections(vm) // resolve injections before data/props
```
初始化state
```javascript
initState(vm)
```
在data和pros注入前初始化provide
```javascript
initProvide(vm) // resolve provide after data/props
```
触发创建后的钩子函数
```javascript
callHook(vm, 'created')
```
如果vm的$options属性存在el属性则执行vm的$mount函数并把vm.$options.el作为参数传入进去。
```javascript
if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
```

完整initMixin函数
```javascript
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // a uid
    vm._uid = uid++

    let startTag, endTag
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }

    // a flag to avoid this being observed
    vm._isVue = true
    // merge options
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }

    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}
```

### initInternalComponent
```javascript
function initInternalComponent (vm: Component, options: InternalComponentOptions) {}
```
初始化内部组件接收两个参数：
1. vm，vue组件
2. options，内部组件的options

首先会定义一个常量opts，会把vm.$options赋值给opts, 这个常量会继承vm.constructor.options的属性。
```javascript
 const opts = vm.$options = Object.create(vm.constructor.options)
```
定义一个常量parentVnode，并把options._parentVnode赋值给它。<br />
再定义一个常量vnodeComponentOptions，并把parentVnode.componentOptions赋值给它。
```javascript
const parentVnode = options._parentVnode
const vnodeComponentOptions = parentVnode.componentOptions
```
接下来会给opts赋值新的属性
```javascript
    opts.parent = options.parent
    opts._parentVnode = parentVnode
    opts._parentElm = options._parentElm
    opts._refElm = options._refElm
    opts.propsData = vnodeComponentOptions.propsData
    opts._parentListeners = vnodeComponentOptions.listeners
    opts._renderChildren = vnodeComponentOptions.children
    opts._componentTag = vnodeComponentOptions.tag
```
如果options有render方法，则把options.render赋值给opts.render。并把staticRenderFns方法也赋值给opts.
```javascript
    if (options.render) {
        opts.render = options.render
        opts.staticRenderFns = options.staticRenderFns
    }
```
完整的initInternalComponent
```javascript
export function initInternalComponent (vm: Component, options: InternalComponentOptions) {
  const opts = vm.$options = Object.create(vm.constructor.options)
  // doing this because it's faster than dynamic enumeration.
  const parentVnode = options._parentVnode
  opts.parent = options.parent
  opts._parentVnode = parentVnode
  opts._parentElm = options._parentElm
  opts._refElm = options._refElm

  const vnodeComponentOptions = parentVnode.componentOptions
  opts.propsData = vnodeComponentOptions.propsData
  opts._parentListeners = vnodeComponentOptions.listeners
  opts._renderChildren = vnodeComponentOptions.children
  opts._componentTag = vnodeComponentOptions.tag

  if (options.render) {
    opts.render = options.render
    opts.staticRenderFns = options.staticRenderFns
  }
}
```

### resolveConstructorOptions
```javascript
function resolveConstructorOptions (Ctor: Class<Component>) {}
```
resolveConstructorOptions 是用来解析构造函数的options属性。<br />
resolveConstructorOptions接受一个参数Ctor，Ctor的类型是VUE子组件。<br />
首先定义一个变量，把Ctor.options赋值给它。
```javascript
let options = Ctor.options
```
之后判断Ctor是否有super方法。如果有说明它是一个VUE子组件，继承自VUE。否则直接返回基础构造器的options。
```javascript
if (Ctor.super) {}
```
如果是VUE的子组件，首先会去递归，把父类的options赋值给superOptions <br/>
然后会把当前的options赋值给cachedSuperOptions。
```javascript
const superOptions = resolveConstructorOptions(Ctor.super)
const cachedSuperOptions = Ctor.superOptions
```
之后会去对比superOtions是否等于cachedSuperOptions。如果不相等说明父类的options被改变了。这时就需要去自身的options赋新的值。
```javascript
if (superOptions !== cachedSuperOptions) {}
```
把父类的Options赋值给自己的options。
```javascript
Ctor.superOptions = superOptions
```
定义一个变量modifiedOptions，通过调用resolveModifiedOptions来验证自身的options是否改变了。
```javascript
const modifiedOptions = resolveModifiedOptions(Ctor)
```
如果自身的options改变了。则把新增的options属性添加到Ctor.extendOptions上
```javascript
if (modifiedOptions) {
    extend(Ctor.extendOptions, modifiedOptions)
}
```
之后调用mergeOptions来合并自身的options和父类的options。
```javascript
options = Ctor.options = mergeOptions(superOptions, Ctor.extendOptions)
```
如果options有name属性。就把Ctor赋值给options的子组件。
```javascript
if (options.name) {
    options.components[options.name] = Ctor
}
```
完整的resolveConstructorOptions
```javascript
export function resolveConstructorOptions (Ctor: Class<Component>) {
  let options = Ctor.options
  if (Ctor.super) {
    const superOptions = resolveConstructorOptions(Ctor.super)
    const cachedSuperOptions = Ctor.superOptions
    if (superOptions !== cachedSuperOptions) {
      // super option changed,
      // need to resolve new options.
      Ctor.superOptions = superOptions
      // check if there are any late-modified/attached options (#4976)
      const modifiedOptions = resolveModifiedOptions(Ctor)
      // update base extend options
      if (modifiedOptions) {
        extend(Ctor.extendOptions, modifiedOptions)
      }
      options = Ctor.options = mergeOptions(superOptions, Ctor.extendOptions)
      if (options.name) {
        options.components[options.name] = Ctor
      }
    }
  }
  return options
}
```

### resolveModifiedOptions
resolveModifiedOptions是用来检查VUE组件的options是否更新了。<br/>
resolveModifiedOptions接受一个VUE组件作为参数。
```javascript
function resolveModifiedOptions (Ctor: Class<Component>): ?Object {}
```
首先会定义1个变量和三个常量
latest是当前组件的options <br/>
extended是继承自父类的options <br/>
sealed是执行Vue.extend时封装的"自身"options，这个属性就是方便检查"自身"的options有没有变化
```javascript
let modified
const latest = Ctor.options
const extended = Ctor.extendOptions
const sealed = Ctor.sealedOptions
```
之后会去便利自身的options。如果自身options改变了，把改变的值调用dedupe方法添加到modified。
```javascript
  for (const key in latest) {
    if (latest[key] !== sealed[key]) {
      if (!modified) modified = {}
      modified[key] = dedupe(latest[key], extended[key], sealed[key])
    }
  }
```
随后返回改变的值。
```javascript
return modified
```
完整的resolveModifiedOptions
```javascript
function resolveModifiedOptions (Ctor: Class<Component>): ?Object {
  let modified
  const latest = Ctor.options
  const extended = Ctor.extendOptions
  const sealed = Ctor.sealedOptions
  for (const key in latest) {
    if (latest[key] !== sealed[key]) {
      if (!modified) modified = {}
      modified[key] = dedupe(latest[key], extended[key], sealed[key])
    }
  }
  return modified
}
```

### dedupe
dedupe防止生命周期构造函数重复。<br/>
dedupe函数接受3个参数：当前的options，继承的options，封装的options
```javascript
function dedupe (latest, extended, sealed) {
  // compare latest and sealed to ensure lifecycle hooks won't be duplicated
  // between merges
}
```
首先去判断当前的options是否是一个数组，如果不是就返回当前options。 <br/>
如果latest是一个数组。定义一个空数组res。 <br/>
如果sealed不是一个数组就把sealed放到一个数组中赋值给sealed。 <br/>
如果extended不是一个数组就把extended放到一个数组中赋值给extended。 <br/>
```javascript
  if (Array.isArray(latest)) {
    const res = []
    sealed = Array.isArray(sealed) ? sealed : [sealed]
    extended = Array.isArray(extended) ? extended : [extended]
  } else {
    return latest
  }
```
之后循环当前的options。如果继承的options中有当前options中的值或封装的options中有当前options的值。就把当前值推到res中。并返回res。
```javascript
for (let i = 0; i < latest.length; i++) {
  // push original options and not sealed options to exclude duplicated options
  if (extended.indexOf(latest[i]) >= 0 || sealed.indexOf(latest[i]) < 0) {
    res.push(latest[i])
  }
}
return res
```
完整的dedupe
```javascript
function dedupe (latest, extended, sealed) {
  // compare latest and sealed to ensure lifecycle hooks won't be duplicated
  // between merges
  if (Array.isArray(latest)) {
    const res = []
    sealed = Array.isArray(sealed) ? sealed : [sealed]
    extended = Array.isArray(extended) ? extended : [extended]
    for (let i = 0; i < latest.length; i++) {
      // push original options and not sealed options to exclude duplicated options
      if (extended.indexOf(latest[i]) >= 0 || sealed.indexOf(latest[i]) < 0) {
        res.push(latest[i])
      }
    }
    return res
  } else {
    return latest
  }
}
```