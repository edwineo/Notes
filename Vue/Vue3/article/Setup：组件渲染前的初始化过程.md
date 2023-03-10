我们先来回想一下 Vue.js 2.x 编写组件的时候，会在 props、data、methods、computed 等 options 中定义一些变量。在组件初始化阶段，Vue.js 内部会处理这些 options，即把定义的变量添加到了组件实例上。等模板编译成 render 函数的时候，内部通过 with(this){} 的语法去访问在组件实例中的变量。

那么到了 Vue.js 3.0，既支持组件定义 setup 函数，而且在模板 render 的时候，又可以访问到 setup 函数返回的值，这是如何实现的？

## 创建和设置组件实例

首先，我们来回顾一下组件的渲染流程：创建 vnode 、渲染 vnode 和生成 DOM。

其中渲染 vnode 的过程主要就是在挂载组件：

```ts
const mountComponent = (
  initialVNode,
  container,
  anchor,
  parentComponent,
  parentSuspense,
  isSVG,
  optimized
) => {
  // 创建组件实例
  const instance = (initialVNode.component = createComponentInstance(initialVNode, parentComponent, parentSuspense))
  // 设置组件实例
  setupComponent(instance)
  // 设置并运行带副作用的渲染函数
  setupRenderEffect(instance, initialVNode, container, anchor, parentSuspense, isSVG, optimized)
}
```

可以看到，这段挂载组件的代码主要做了三件事情：

1. 创建组件实例
2. 设置组件实例
3. 设置并运行带副作用的渲染函数

前两个流程就跟我们今天提到的问题息息相关，我们将重点分析它们。



先看创建组件实例的流程，我们要关注 createComponentInstance 方法的实现：

```ts
// packages/runtime-core/src/component.ts

function createComponentInstance (vnode, parent, suspense) {
  // 继承父组件实例上的 appContext，如果是根组件，则直接从根 vnode 中取。
  const appContext = (parent ? parent.appContext : vnode.appContext) || emptyAppContext;
  const instance = {
    // 组件唯一 id
    uid: uid++,
    // 组件 vnode
    vnode,
    // 父组件实例
    parent,
    // app 上下文
    appContext,
    // vnode 节点类型
    type: vnode.type,
    // 根组件实例
    root: null,
    // 新的组件 vnode
    next: null,
    // 子节点 vnode
    subTree: null,
    // 带副作用更新函数
    update: null,
    // 渲染函数
    render: null,
    // 渲染上下文代理
    proxy: null,
    // 带有 with 区块的渲染上下文代理
    withProxy: null,
    // 响应式相关对象
    effects: null,
    // 依赖注入相关
    provides: parent ? parent.provides : Object.create(appContext.provides),
    // 渲染代理的属性访问缓存
    accessCache: null,
    // 渲染缓存
    renderCache: [],
    // 渲染上下文
    ctx: EMPTY_OBJ,
    // data 数据
    data: EMPTY_OBJ,
    // props 数据
    props: EMPTY_OBJ,
    // 普通属性
    attrs: EMPTY_OBJ,
    // 插槽相关
    slots: EMPTY_OBJ,
    // 组件或者 DOM 的 ref 引用
    refs: EMPTY_OBJ,
    // setup 函数返回的响应式结果
    setupState: EMPTY_OBJ,
    // setup 函数上下文数据
    setupContext: null,
    // 注册的组件
    components: Object.create(appContext.components),
    // 注册的指令
    directives: Object.create(appContext.directives),
    // suspense 相关
    suspense,
    // suspense 异步依赖
    asyncDep: null,
    // suspense 异步依赖是否都已处理
    asyncResolved: false,
    // 是否挂载
    isMounted: false,
    // 是否卸载
    isUnmounted: false,
    // 是否激活
    isDeactivated: false,
    // 生命周期，before create
    bc: null,
    // 生命周期，created
    c: null,
    // 生命周期，before mount
    bm: null,
    // 生命周期，mounted
    m: null,
    // 生命周期，before update
    bu: null,
    // 生命周期，updated
    u: null,
    // 生命周期，unmounted
    um: null,
    // 生命周期，before unmount
    bum: null,
    // 生命周期, deactivated
    da: null,
    // 生命周期 activated
    a: null,
    // 生命周期 render triggered
    rtg: null,
    // 生命周期 render tracked
    rtc: null,
    // 生命周期 error captured
    ec: null,
    // 派发事件方法
    emit: null
  }
  // 初始化渲染上下文
  instance.ctx = { _: instance }
  // 初始化根组件指针
  instance.root = parent ? parent.root : instance
  // 初始化派发事件方法
  instance.emit = emit.bind(null, instance)
  return instance
}
```

从上述代码中可以看到，组件实例 instance 上定义了很多很多的属性

Vue.js 2.x 使用 new Vue 来初始化一个组件的实例，到了 Vue.js 3.0，我们直接通过创建对象去创建组件的实例。这两种方式并无本质的区别，都是引用一个对象，在整个组件的生命周期中去维护组件的状态数据和上下文环境。

创建好 instance 实例后，接下来就是设置它的一些属性。目前已完成了组件的上下文、根组件指针以及派发事件方法的设置。我们在后面会继续分析更多 instance 实例属性的设置逻辑。



接下来是组件实例的设置流程，**对 setup 函数的处理就在这里完成**，我们来看一下 setupComponent 方法的实现：

```ts
// packages/runtime-core/src/component.ts

function setupComponent (instance, isSSR = false) {
  const { props, children } = instance.vnode
  // 判断是否是一个有状态的组件
  const isStateful = isStatefulComponent(instance)
  // 初始化 props
  initProps(instance, props, isStateful, isSSR)
  // 初始化 插槽
  initSlots(instance, children)
  // 设置有状态的组件实例
  const setupResult = isStateful
    ? setupStatefulComponent(instance, isSSR)
    : undefined
  return setupResult
}

function isStatefulComponent(instance: ComponentInternalInstance) {
  return instance.vnode.shapeFlag & ShapeFlags.STATEFUL_COMPONENT
}
```

可以看到，我们从组件 vnode 中获取了 props、children、shapeFlag 等属性，然后分别对 props 和插槽进行初始化，这两部分逻辑在后续再详细分析

根据 shapeFlag 的值，我们可以判断这是不是一个有状态组件，如果是则要进一步去设置有状态组件的实例。



接下来我们要关注到 setupStatefulComponent 函数，它主要做了三件事：

1. 创建渲染上下文代理
2. 判断处理 setup 函数
3. 完成组件实例设置

它代码如下所示：

```ts
// packages/runtime-core/src/component.ts

function setupStatefulComponent (instance, isSSR) {
  const Component = instance.type
  // 创建渲染代理的属性访问缓存
  instance.accessCache = {}
  // 创建渲染上下文代理
  instance.proxy = new Proxy(instance.ctx, PublicInstanceProxyHandlers)
  // 判断处理 setup 函数
  const { setup } = Component
  if (setup) {
    // 如果 setup 函数带参数，则创建一个 setupContext
    const setupContext = (instance.setupContext =
      setup.length > 1 ? createSetupContext(instance) : null)
    // 执行 setup 函数，获取结果
    const setupResult = callWithErrorHandling(setup, instance, 0 /* SETUP_FUNCTION */, [instance.props, setupContext])
    // 处理 setup 执行结果
    handleSetupResult(instance, setupResult)
  } else {
    // 完成组件实例设置
    finishComponentSetup(instance)
  }
}
```



### 创建渲染上下文代理

首先是创建渲染上下文代理的流程，它主要对 instance.ctx 做了代理。在分析实现前，我们需要思考一个问题，这里为什么需要代理呢？



其实在 Vue.js 2.x 中，也有类似的数据代理逻辑，比如 props 求值后的数据，实际上存储在 this.\_props 上，而 data 中定义的数据存储在 this.\_data 上。

举个例子：我们在模板渲染的时候访问 this.msg，实际上访问的是 this.\_data.msg，这是因为 Vue.js 2.x 在初始化 data 的时候，做了一层 proxy 代理。



到了 Vue.js 3.0，为了方便维护，我们把组件中不同状态的数据存储到不同的属性中，比如存储到 setupState、ctx、data、props 中。我们在执行组件渲染函数的时候，为了方便用户使用，会直接访问渲染上下文 instance.ctx 中的属性，所以我们也要做一层 proxy，对渲染上下文 instance.ctx 属性的访问和修改，代理到对 setupState、ctx、data、props 中的数据的访问和修改。



明确了代理的需求后，我们接下来就要分析 proxy 的几个方法： get、set 和 has。

当我们访问 instance.ctx 渲染上下文中的属性时，就会进入 get 函数。我们来看一下它的实现：

```ts
// packages/runtime-core/src/componentPublicInstance.ts

const PublicInstanceProxyHandlers = {
  get ({ _: instance }, key) {
    const { ctx, setupState, data, props, accessCache, type, appContext } = instance
    if (key[0] !== '$') {
      // setupState / data / props / ctx
      // 渲染代理的属性访问缓存中
      const n = accessCache[key]
      if (n !== undefined) {
        // 从缓存中取
        switch (n) {
          case 0: /* SETUP */
            return setupState[key]
          case 1 :/* DATA */
            return data[key]
          case 3 :/* CONTEXT */
            return ctx[key]
          case 2: /* PROPS */
            return props[key]
        }
      }
      else if (setupState !== EMPTY_OBJ && hasOwn(setupState, key)) {
        accessCache[key] = 0
        // 从 setupState 中取数据
        return setupState[key]
      }
      else if (data !== EMPTY_OBJ && hasOwn(data, key)) {
        accessCache[key] = 1
        // 从 data 中取数据
        return data[key]
      }
      else if (
        type.props &&
        hasOwn(normalizePropsOptions(type.props)[0], key)) {
        accessCache[key] = 2
        // 从 props 中取数据
        return props[key]
      }
      else if (ctx !== EMPTY_OBJ && hasOwn(ctx, key)) {
        accessCache[key] = 3
        // 从 ctx 中取数据
        return ctx[key]
      }
      else {
        // 都取不到
        accessCache[key] = 4
      }
    }
    const publicGetter = publicPropertiesMap[key]
    let cssModule, globalProperties
    // 公开的 $xxx 属性或方法
    if (publicGetter) {
      return publicGetter(instance)
    }
    else if (
      // css 模块，通过 vue-loader 编译的时候注入
      (cssModule = type.__cssModules) &&
      (cssModule = cssModule[key])) {
      return cssModule
    }
    else if (ctx !== EMPTY_OBJ && hasOwn(ctx, key)) {
      // 用户自定义的属性，也用 `$` 开头
      accessCache[key] = 3
      return ctx[key]
    }
    else if (
      // 全局定义的属性
      ((globalProperties = appContext.config.globalProperties),
        hasOwn(globalProperties, key))) {
      return globalProperties[key]
    }
    else if ((process.env.NODE_ENV !== 'production') &&
      currentRenderingInstance && key.indexOf('__v') !== 0) {
      if (data !== EMPTY_OBJ && key[0] === '$' && hasOwn(data, key)) {
        // 如果在 data 中定义的数据以 $ 开头，会报警告，因为 $ 是保留字符，不会做代理
        warn(`Property ${JSON.stringify(key)} must be accessed via $data because it starts with a reserved ` +
          `character and is not proxied on the render context.`)
      }
      else {
        // 在模板中使用的变量如果没有定义，报警告
        warn(`Property ${JSON.stringify(key)} was accessed during render ` +
          `but is not defined on instance.`)
      }
    }
  }
}
```

可以看到，函数首先判断 key 不以 $ 开头的情况，这部分数据可能是 setupState、data、props、ctx 中的一种，其中 data、props 我们已经很熟悉了；setupState 就是 setup 函数返回的数据，稍后我们会详细说；ctx 包括了计算属性、组件方法和用户自定义的一些数据。

如果 key 不以 $ 开头，那么就依次判断 setupState、data、props、ctx 中是否包含这个 key，如果包含就返回对应值。注意这个判断顺序很重要，在 key 相同时它会决定数据获取的优先级，举个例子：

```html
<template>
  <p>{{msg}}</p>
</template>
<script>
  import { ref } from 'vue'
  export default {
    data() {
      return {
        msg: 'msg from data'
      }
    },
    setup() {
      const msg = ref('msg from setup')
      return {
        msg
      }
    }
  }
</script>
```

我们在 data 和 setup 中都定义了 msg 变量，但最终输出到界面上的是"msg from setup"，这是因为 setupState 的判断优先级要高于 data。

再回到 get 函数中，我们可以看到这里定义了 accessCache 作为渲染代理的属性访问缓存，它具体是干什么的呢？组件在渲染时会经常访问数据进而触发 get 函数，这其中最昂贵的部分就是多次调用 hasOwn 去判断 key 在不在某个类型的数据中，但是在普通对象上执行简单的属性访问相对要快得多。所以在第一次获取 key 对应的数据后，我们利用 accessCache[key] 去缓存数据，下一次再次根据 key 查找数据，我们就可以直接通过 accessCache[key] 获取对应的值，就不需要依次调用 hasOwn 去判断了。这也是一个性能优化的小技巧。

如果 key 以 $ 开头，那么接下来又会有一系列的判断，首先判断是不是 Vue.js 内部公开的 $xxx 属性或方法（比如 $parent）；然后判断是不是 vue-loader 编译注入的 css 模块内部的 key；接着判断是不是用户自定义以 $ 开头的 key；最后判断是不是全局属性。如果都不满足，就剩两种情况了，即在非生产环境下就会报两种类型的警告，第一种是在 data 中定义的数据以 $ 开头的警告，因为 $ 是保留字符，不会做代理；第二种是在模板中使用的变量没有定义的警告。



接下来是 set 代理过程，当我们修改 instance.ctx 渲染上下文中的属性的时候，就会进入 set 函数。我们来看一下 set 函数的实现：

```ts
const PublicInstanceProxyHandlers = {
  set ({ _: instance }, key, value) {
    const { data, setupState, ctx } = instance
    if (setupState !== EMPTY_OBJ && hasOwn(setupState, key)) {
      // 给 setupState 赋值
      setupState[key] = value
    }
    else if (data !== EMPTY_OBJ && hasOwn(data, key)) {
      // 给 data 赋值
      data[key] = value
    }
    else if (key in instance.props) {
      // 不能直接给 props 赋值
      (process.env.NODE_ENV !== 'production') &&
      warn(`Attempting to mutate prop "${key}". Props are readonly.`, instance)
      return false
    }
    if (key[0] === '$' && key.slice(1) in instance) {
      // 不能给 Vue 内部以 $ 开头的保留属性赋值
      (process.env.NODE_ENV !== 'production') &&
      warn(`Attempting to mutate public property "${key}". ` +
        `Properties starting with $ are reserved and readonly.`, instance)
      return false
    }
    else {
      // 用户自定义数据赋值
      ctx[key] = value
    }
    return true
  }
}
```

结合代码来看，函数主要做的事情就是对渲染上下文 instance.ctx 中的属性赋值，它实际上是代理到对应的数据类型中去完成赋值操作的。这里仍然要注意顺序问题，和 get 一样，优先判断 setupState，然后是 data，接着是 props。

我们对之前的例子做点修改，添加一个方法：

```html
<template>
  <p>{{ msg }}</p>
  <button @click="random">Random msg</button>
</template>
<script>
  import { ref } from 'vue'
  export default {
    data() {
      return {
        msg: 'msg from data'
      }
    },
    setup() {
      const msg = ref('msg from setup')
      return {
        msg
      }
    },
    methods: {
      random() {
        this.msg = Math.random()
      }
    }
  }
</script>
```

我们点击按钮会执行 random 函数，这里的 this 指向的就是 instance.ctx，我们修改 this.msg 会触发 set 函数，所以最终修改的是 setupState 中的 msg 对应的值。

注意，如果我们直接对 props 中的数据赋值，在非生产环境中会收到一条警告，这是因为直接修改 props 不符合数据单向流动的设计思想；如果对 Vue.js 内部以 $ 开头的保留属性赋值，同样也会收到一条警告。

如果是用户自定义的数据，比如在 created 生命周期内定义的数据，它仅用于组件上下文的共享，如下所示：

```ts
export default {
  created() {
    this.userMsg = 'msg from user'
  }
}
```

当执行 this.userMsg 赋值的时候，会触发 set 函数，最终 userMsg 会被保留到 ctx 中。



最后是 has 代理过程，当我们判断属性是否存在于 instance.ctx 渲染上下文中时，就会进入 has 函数，这个在平时项目中用的比较少，同样来举个例子，当执行 created 钩子函数中的 'msg' in this 时，就会触发 has 函数。

```ts
export default {
  created () {
    console.log('msg' in this)
  }
}
```

下面我们来看一下 has 函数的实现：

```ts
const PublicInstanceProxyHandlers = {
  has
    ({ _: { data, setupState, accessCache, ctx, type, appContext } }, key) {
    // 依次判断
    return (accessCache[key] !== undefined ||
      (data !== EMPTY_OBJ && hasOwn(data, key)) ||
      (setupState !== EMPTY_OBJ && hasOwn(setupState, key)) ||
      (type.props && hasOwn(normalizePropsOptions(type.props)[0], key)) ||
      hasOwn(ctx, key) ||
      hasOwn(publicPropertiesMap, key) ||
      hasOwn(appContext.config.globalProperties, key))
  }
}
```

这个函数的实现很简单，依次判断 key 是否存在于 accessCache、data、setupState、props 、用户数据、公开属性以及全局属性中，然后返回结果。



至此，我们就搞清楚了创建上下文代理的过程，让我们回到 setupStatefulComponent 函数中，接下来分析第二个流程——判断处理 setup 函数。



### 判断处理 setup 函数

整个逻辑涉及的代码：

```ts
// 判断处理 setup 函数
const { setup } = Component
if (setup) {
  // 如果 setup 函数带参数，则创建一个 setupContext
  const setupContext = (instance.setupContext =
    setup.length > 1 ? createSetupContext(instance) : null)
  // 执行 setup 函数获取结果
  const setupResult = callWithErrorHandling(setup, instance, 0 /* SETUP_FUNCTION */, [instance.props, setupContext])
  // 处理 setup 执行结果
  handleSetupResult(instance, setupResult)
}
```

如果我们在组件中定义了 setup 函数，接下来就是处理 setup 函数的流程，主要是三个步骤：创建 setup 函数上下文、执行 setup 函数并获取结果和处理 setup 函数的执行结果。接下来我们就逐个来分析。

首先判断 setup 函数的参数长度，如果大于 1，则创建 setupContext 上下文。

```ts
const setupContext = (instance.setupContext =
    setup.length > 1 ? createSetupContext(instance) : null)
```

