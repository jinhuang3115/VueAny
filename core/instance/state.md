## state.js

state.js 对外返回5个方法:
1. proxy
2. initState
3. getData
4. defineComputed
5. stateMixin

### proxy
```javascript
function proxy (target: Object, sourceKey: string, key: string) {}
```
proxy方法接受3个参数：
target: 对象类型 <br/>
sourceKey: 字符串类型 <br/>
key: 字符串类型 <br/>
```javascript
function proxy (target: Object, sourceKey: string, key: string) {
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```
此方法会为一个对象设置代理拦截方法。

完整proxy
```javascript
function proxy (target: Object, sourceKey: string, key: string) {
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```

### initState
```javascript
function initState (vm: Component) {}
```
initState接受一个参数vm, 类型是vue组件作为参数。
```javascript
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
```
首先会为vue组件的_watchers属性初始化为一个空数组。然后初始化一个常量opts并把vue组件的$options赋值给opts。<br/>
如果opts中有props属性，则执行initProps方法并把vue组件和opts.props作为参数传入进去。<br/>
如果opts中有methods则执行initMethods并把vue组件和opts.methods传进去。<br/>
如果opts中有data则执行initData方法并把vue组件传入进去，否则执行observe方法并把vm._data（默认值{}）和true传进去。<br/>
如果opts中有computed则执行initComputed方法并把vue组件和opts.computed作为参数传进去。<br/>
如果opts中有watch并且watch不等于nativeWatch则执行initWatch方法并把vue组件和opts.watch传进去。

完整initState
```javascript
function initState (vm: Component) {
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

### initProps
```javascript
function initProps (vm: Component, propsOptions: Object) {}
```
initProps接受两个参数vm(vue组件)和propsOptions(对象)
```javascript
  const propsData = vm.$options.propsData || {}
  const props = vm._props = {}
  // cache prop keys so that future props updates can iterate using Array
  // instead of dynamic object key enumeration.
  const keys = vm.$options._propKeys = []
  const isRoot = !vm.$parent
  // root instance props should be converted
  if (!isRoot) {
    toggleObserving(false)
  }
```
首先会去定义一些常量：
propsData: vm.$options.propsData 或者 {} <br/>
props: vm._props 并且等于{} <br/>
keys: vm.$options._propKeys 并且等于 [] <br/>
isRoot: 父级是否存在 <br/>
如果有父级，则执行toggleObserving并传入false

```javascript
for (const key in propsOptions) {
    keys.push(key)
    const value = validateProp(key, propsOptions, propsData, vm)
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      const hyphenatedKey = hyphenate(key)
      if (isReservedAttribute(hyphenatedKey) ||
          config.isReservedAttr(hyphenatedKey)) {
        warn(
          `"${hyphenatedKey}" is a reserved attribute and cannot be used as component prop.`,
          vm
        )
      }
      defineReactive(props, key, value, () => {
        if (vm.$parent && !isUpdatingChildComponent) {
          warn(
            `Avoid mutating a prop directly since the value will be ` +
            `overwritten whenever the parent component re-renders. ` +
            `Instead, use a data or computed property based on the prop's ` +
            `value. Prop being mutated: "${key}"`,
            vm
          )
        }
      })
    } else {
      defineReactive(props, key, value)
    }
    // static props are already proxied on the component's prototype
    // during Vue.extend(). We only need to proxy props defined at
    // instantiation here.
    if (!(key in vm)) {
      proxy(vm, `_props`, key)
    }
  }
```
首先去循环传入的props并把props的key推到数组keys中存储。<br/>
定义一个常量value，调用validateProp去验证当前pros的值是否符合要求。<br/>
如果当前环境是生产环境，则先定义一个常量hyphenatedKey，并把key值修改为驼峰形式并复制给hyphenatedKey。<br/>
然后去检查这个key是不是一个保留字段，如果是保留字段会报出warn。<br/>
通过调用defineReactive去获取props中的每一个属性的get和set的控制权。<br/>
如果key不在vm中则直接调用代理的方法去进行拦截。

完整initProps
```javascript
initProps (vm: Component, propsOptions: Object) {
  const propsData = vm.$options.propsData || {}
  const props = vm._props = {}
  // cache prop keys so that future props updates can iterate using Array
  // instead of dynamic object key enumeration.
  const keys = vm.$options._propKeys = []
  const isRoot = !vm.$parent
  // root instance props should be converted
  if (!isRoot) {
    toggleObserving(false)
  }
  for (const key in propsOptions) {
    keys.push(key)
    const value = validateProp(key, propsOptions, propsData, vm)
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      const hyphenatedKey = hyphenate(key)
      if (isReservedAttribute(hyphenatedKey) ||
          config.isReservedAttr(hyphenatedKey)) {
        warn(
          `"${hyphenatedKey}" is a reserved attribute and cannot be used as component prop.`,
          vm
        )
      }
      defineReactive(props, key, value, () => {
        if (vm.$parent && !isUpdatingChildComponent) {
          warn(
            `Avoid mutating a prop directly since the value will be ` +
            `overwritten whenever the parent component re-renders. ` +
            `Instead, use a data or computed property based on the prop's ` +
            `value. Prop being mutated: "${key}"`,
            vm
          )
        }
      })
    } else {
      defineReactive(props, key, value)
    }
    // static props are already proxied on the component's prototype
    // during Vue.extend(). We only need to proxy props defined at
    // instantiation here.
    if (!(key in vm)) {
      proxy(vm, `_props`, key)
    }
  }
  toggleObserving(true)
}
```

### initData
```javascript
function initData (vm: Component) {}
```
initData接受1个vue组件作为参数
```javascript
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
```
首先定义一个变量data并把vue组件的data赋值给它。<br/>
如果data是一个function就执行getData方法传入data和vue组件作为参数，并把结果赋值给data。<br/>
如果data不是一个方法则把data或者空对象赋值给它。<br>
```javascript
  // proxy data on instance
  const keys = Object.keys(data)
  const props = vm.$options.props
  const methods = vm.$options.methods
```
定义一个常量keys并把data的键作为数组赋值给它。<br/>
定义一个常量props并把vue组件的props赋值给它。<br/>
定义一个常量methods并把vue组件的methods赋值给它。<br/>
```javascript
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
```
首先去循环keys，在循环中定义一个常量key并把当前key赋值给它。<br/>
如果methods中也有这个key同名的键，则输出一个warn：methods中的这个键已经作为属性定义了。<br/>
如果props中也有这个key同名的键，则输出一个warn：props中的这个键已经作为属性定义了。使用Props中的默认值来代替。<br/>
如果这个key是$开头的，就用proxy代理监听这个key。<br>
最后使用观察者模式来监听所有的data。

完整的initData
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

### getData
```javascript
function getData (data: Function, vm: Component): any {}
```
getData接受两个参数:<br/>
data: Function类型<br/>
vm: vue组件

```javascript
 // #7573 disable dep collection when invoking data getters
  pushTarget()
  try {
    return data.call(vm, vm)
  } catch (e) {
    handleError(e, vm, `data()`)
    return {}
  } finally {
    popTarget()
  }
```
返回data执行的结果，并把data中this的指向改成vue组件。

完整getData
```javascript
function getData (data: Function, vm: Component): any {
  // #7573 disable dep collection when invoking data getters
  pushTarget()
  try {
    return data.call(vm, vm)
  } catch (e) {
    handleError(e, vm, `data()`)
    return {}
  } finally {
    popTarget()
  }
}
```

### initComputed
```javascript
function initComputed (vm: Component, computed: Object) {}
```
initComputed接受两个参数:<br/>
vm: vue组件<br/>
computed: 对象类型

```javascript
// $flow-disable-line
const watchers = vm._computedWatchers = Object.create(null)
// computed properties are just getters during SSR
const isSSR = isServerRendering()
```
定义一个常量watchers，然后把vm._computedWatchers赋值给它，并把一个空对象赋值给它们。<br/>
定义一个变量isSSR并执行isServerRendering获取当前环境赋值给它(是否是服务器端渲染)。
```javascript
 for (const key in computed) {
    const userDef = computed[key]
    const getter = typeof userDef === 'function' ? userDef : userDef.get
    if (process.env.NODE_ENV !== 'production' && getter == null) {
      warn(
        `Getter is missing for computed property "${key}".`,
        vm
      )
    }

    if (!isSSR) {
      // create internal watcher for the computed property.
      watchers[key] = new Watcher(
        vm,
        getter || noop,
        noop,
        computedWatcherOptions
      )
    }

    // component-defined computed properties are already defined on the
    // component prototype. We only need to define computed properties defined
    // at instantiation here.
    if (!(key in vm)) {
      defineComputed(vm, key, userDef)
    } else if (process.env.NODE_ENV !== 'production') {
      if (key in vm.$data) {
        warn(`The computed property "${key}" is already defined in data.`, vm)
      } else if (vm.$options.props && key in vm.$options.props) {
        warn(`The computed property "${key}" is already defined as a prop.`, vm)
      }
    }
  }
```
首先循环computed。<br/>
定义一个变量userDef并把当前循环中的key所对应的值赋值给它。<br/>
定义一个变量getter，如果userDef是一个方法就把userDef赋值给getter，否则就把userDef.get赋值给getter。<br/>
如果getter的值是null则返回一个warn:getter在computed的属性中找不到。<br/>
如果当前环境不是服务器，则为每一个key创建一个watcher。<br/>
如果key在vue组件中，则执行defineComputed去定义每个computed。

完整的initComputed
```javascript
function defineComputed (
  target: any,
  key: string,
  userDef: Object | Function
) {
  const shouldCache = !isServerRendering()
  if (typeof userDef === 'function') {
    sharedPropertyDefinition.get = shouldCache
      ? createComputedGetter(key)
      : userDef
    sharedPropertyDefinition.set = noop
  } else {
    sharedPropertyDefinition.get = userDef.get
      ? shouldCache && userDef.cache !== false
        ? createComputedGetter(key)
        : userDef.get
      : noop
    sharedPropertyDefinition.set = userDef.set
      ? userDef.set
      : noop
  }
  if (process.env.NODE_ENV !== 'production' &&
      sharedPropertyDefinition.set === noop) {
    sharedPropertyDefinition.set = function () {
      warn(
        `Computed property "${key}" was assigned to but it has no setter.`,
        this
      )
    }
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```

### defineComputed
```javascript
function defineComputed (
  target: any,
  key: string,
  userDef: Object | Function
) {}

```
defineComputed接受三个参数:<br/>
target: 可以是任何值。<br>
key: 字符串。<br/>
userDef: 对象或者函数。

```javascript
const shouldCache = !isServerRendering()
```
定义一个常量shouldCache存储当前环境变量。

```javascript
  if (typeof userDef === 'function') {
    sharedPropertyDefinition.get = shouldCache
      ? createComputedGetter(key)
      : userDef
    sharedPropertyDefinition.set = noop
  } else {
    sharedPropertyDefinition.get = userDef.get
      ? shouldCache && userDef.cache !== false
        ? createComputedGetter(key)
        : userDef.get
      : noop
    sharedPropertyDefinition.set = userDef.set
      ? userDef.set
      : noop
  }
  if (process.env.NODE_ENV !== 'production' &&
      sharedPropertyDefinition.set === noop) {
    sharedPropertyDefinition.set = function () {
      warn(
        `Computed property "${key}" was assigned to but it has no setter.`,
        this
      )
    }
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
```
如果userDef是一个function, 如果当前环境是服务器环境则执行createComputedGetter(key)返回值赋值给sharedPropertyDefinition.get，
否则把userDef赋值给sharedPropertyDefinition.get。把noop赋值给 sharedPropertyDefinition.set。<br/>
如果userDef不是一个function, 如果当前环境是服务器环境并且userDef.cache是
true则执行createComputedGetter(key)返回值赋值给sharedPropertyDefinition.get, 
否则把userDef.get赋值给sharedPropertyDefinition.get。如果userDef.set存在把userDef.set赋值给 sharedPropertyDefinition.set。
否则把noop赋值给 sharedPropertyDefinition.set。<br/>
最后通过defineProperty来监听computed的变化。


完整defineComputed
```javascript
function defineComputed (
  target: any,
  key: string,
  userDef: Object | Function
) {
  const shouldCache = !isServerRendering()
  if (typeof userDef === 'function') {
    sharedPropertyDefinition.get = shouldCache
      ? createComputedGetter(key)
      : userDef
    sharedPropertyDefinition.set = noop
  } else {
    sharedPropertyDefinition.get = userDef.get
      ? shouldCache && userDef.cache !== false
        ? createComputedGetter(key)
        : userDef.get
      : noop
    sharedPropertyDefinition.set = userDef.set
      ? userDef.set
      : noop
  }
  if (process.env.NODE_ENV !== 'production' &&
      sharedPropertyDefinition.set === noop) {
    sharedPropertyDefinition.set = function () {
      warn(
        `Computed property "${key}" was assigned to but it has no setter.`,
        this
      )
    }
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```

### createComputedGetter
```javascript
function createComputedGetter (key) {}
```
createComputedGetter 接受一个参数key。

```javascript
 return function computedGetter () {
    const watcher = this._computedWatchers && this._computedWatchers[key]
    if (watcher) {
      if (watcher.dirty) {
        watcher.evaluate()
      }
      if (Dep.target) {
        watcher.depend()
      }
      return watcher.value
    }
  }
```
返回一个叫做computedGetter的函数<br/>
computedGetter中首先定义了一个变量watcher。并把this._computedWatchers && this._computedWatchers[key]的结果赋值给它。<br/>
如果watcher存在，返回监听的值。

完整createComputedGetter
```javascript
function createComputedGetter (key) {
  return function computedGetter () {
    const watcher = this._computedWatchers && this._computedWatchers[key]
    if (watcher) {
      if (watcher.dirty) {
        watcher.evaluate()
      }
      if (Dep.target) {
        watcher.depend()
      }
      return watcher.value
    }
  }
}
```

### initMethods
```javascript
function initMethods (vm: Component, methods: Object) {}
```
initMethods接受两个参数: <br/>
vm: vue组件<br/>
methods: 对象

```javascript
const props = vm.$options.props
  for (const key in methods) {
    if (process.env.NODE_ENV !== 'production') {
      if (methods[key] == null) {
        warn(
          `Method "${key}" has an undefined value in the component definition. ` +
          `Did you reference the function correctly?`,
          vm
        )
      }
      if (props && hasOwn(props, key)) {
        warn(
          `Method "${key}" has already been defined as a prop.`,
          vm
        )
      }
      if ((key in vm) && isReserved(key)) {
        warn(
          `Method "${key}" conflicts with an existing Vue instance method. ` +
          `Avoid defining component methods that start with _ or $.`
        )
      }
    }
    vm[key] = methods[key] == null ? noop : bind(methods[key], vm)
  }
```
定义一个常量props，把组件的props赋值给它。<br/>
循环methods, <br/>
如果当前methods的属性为null，则返回warn<br/>
如果props中有当前methods中的属性，则返回warn<br/>
如果当前key存在于vue组件中并且是服务器环境，则返回warn<br/>
如果methods中的key属性等于null，则把noop赋值给vm[key]，否则把methods[key]赋值给vm[key]并把this指向改成vm。

完整initMethods
```javascript
function initMethods (vm: Component, methods: Object) {
  const props = vm.$options.props
  for (const key in methods) {
    if (process.env.NODE_ENV !== 'production') {
      if (methods[key] == null) {
        warn(
          `Method "${key}" has an undefined value in the component definition. ` +
          `Did you reference the function correctly?`,
          vm
        )
      }
      if (props && hasOwn(props, key)) {
        warn(
          `Method "${key}" has already been defined as a prop.`,
          vm
        )
      }
      if ((key in vm) && isReserved(key)) {
        warn(
          `Method "${key}" conflicts with an existing Vue instance method. ` +
          `Avoid defining component methods that start with _ or $.`
        )
      }
    }
    vm[key] = methods[key] == null ? noop : bind(methods[key], vm)
  }
}
```


### initWatch
```javascript
function initWatch (vm: Component, watch: Object) {}
```
initWatch接受两个参数: <br/>
vm: vue组件<br/>
watch: 对象
```javascript
for (const key in watch) {
    const handler = watch[key]
    if (Array.isArray(handler)) {
      for (let i = 0; i < handler.length; i++) {
        createWatcher(vm, key, handler[i])
      }
    } else {
      createWatcher(vm, key, handler)
    }
  }
```
循环watch<br/>
在循环中，定义一个常量handler并把watch当前key属性的值赋值给handler<br/>
如果handler是一个数组，就循环handler，然后对数组中的每一项执行createWatcher。<br/>
如果不是数组，就对watch当前key属性的值执行createWatcher。

完整initWatch
```javascript
function createWatcher (
  vm: Component,
  expOrFn: string | Function,
  handler: any,
  options?: Object
) {
  if (isPlainObject(handler)) {
    options = handler
    handler = handler.handler
  }
  if (typeof handler === 'string') {
    handler = vm[handler]
  }
  return vm.$watch(expOrFn, handler, options)
}

```

### stateMixin
```javascript
function stateMixin (Vue: Class<Component>) {}
```
stateMixin接受一个参数vue组件
```javascript
 // flow somehow has problems with directly declared definition object
  // when using Object.defineProperty, so we have to procedurally build up
  // the object here.
  const dataDef = {}
  dataDef.get = function () { return this._data }
  const propsDef = {}
  propsDef.get = function () { return this._props }
  if (process.env.NODE_ENV !== 'production') {
    dataDef.set = function (newData: Object) {
      warn(
        'Avoid replacing instance root $data. ' +
        'Use nested data properties instead.',
        this
      )
    }
    propsDef.set = function () {
      warn(`$props is readonly.`, this)
    }
  }
  Object.defineProperty(Vue.prototype, '$data', dataDef)
  Object.defineProperty(Vue.prototype, '$props', propsDef)
  Vue.prototype.$set = set
  Vue.prototype.$delete = del
```
定义一个常量dataDef并把{}赋值给它<br>
给dataDef.get赋值一个返回_data属性的函数。<br/>
定义一个常量propsDef并把{}赋值给它<br>
给propsDef.get赋值一个返回_props属性的函数。<br/>
通过dataDef来监听Vue原型上的$data属性<br/>
通过propsDef来监听Vue原型上的$props属性<br/>
把ovserver/index.js中的set方法和del方法分别赋值给Vue.prototype.$set和Vue.prototype.$delete

```javascript
Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
  ): Function {
    const vm: Component = this
    if (isPlainObject(cb)) {
      return createWatcher(vm, expOrFn, cb, options)
    }
    options = options || {}
    options.user = true
    const watcher = new Watcher(vm, expOrFn, cb, options)
    if (options.immediate) {
      cb.call(vm, watcher.value)
    }
    return function unwatchFn () {
      watcher.teardown()
    }
  }
```
给Vue原型上的$watch赋值一个匿名函数<br/>
这个函数接受三个参数: <br/>
expOrFn: 字符串或函数<br/>
cb: 任何<br>
options: 对象<br>
在函数中先定义一个常量vm 把this赋值给它。<br>
如果cb是一个简单对象, 则返回createWatcher(vm, expOrFn, cb, options) <br/>
如果options为空，就给options赋值{}<br/>
options.user赋值true<br/>
定义常量watcher并初始化Watcher赋值给它。<br>
如果options.immediate的布尔值为true，则调用cb，并把this指向改成vm传入watch.value作为参数<br/>
返回一个函数unwatchFn，这个函数会执行watcher.teardown方法<br/>

完整stateMixin
```javascript
function stateMixin (Vue: Class<Component>) {
  // flow somehow has problems with directly declared definition object
  // when using Object.defineProperty, so we have to procedurally build up
  // the object here.
  const dataDef = {}
  dataDef.get = function () { return this._data }
  const propsDef = {}
  propsDef.get = function () { return this._props }
  if (process.env.NODE_ENV !== 'production') {
    dataDef.set = function (newData: Object) {
      warn(
        'Avoid replacing instance root $data. ' +
        'Use nested data properties instead.',
        this
      )
    }
    propsDef.set = function () {
      warn(`$props is readonly.`, this)
    }
  }
  Object.defineProperty(Vue.prototype, '$data', dataDef)
  Object.defineProperty(Vue.prototype, '$props', propsDef)

  Vue.prototype.$set = set
  Vue.prototype.$delete = del

  Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
  ): Function {
    const vm: Component = this
    if (isPlainObject(cb)) {
      return createWatcher(vm, expOrFn, cb, options)
    }
    options = options || {}
    options.user = true
    const watcher = new Watcher(vm, expOrFn, cb, options)
    if (options.immediate) {
      cb.call(vm, watcher.value)
    }
    return function unwatchFn () {
      watcher.teardown()
    }
  }
}

```