#### __patch__
- 如果没有新的 vnode 只有旧的 vnode，就直接摧毁掉旧的 vnode
- 如果没有旧的 vnode，就直接用新的 vnode 创建 DOM 元素
- 对比新旧 vnode，如果是 sameVnode，就开始进行 patchVnode
- 如果旧的 vnode 是 DOM，并且有 ssr 标记，则清除 ssr 标记并进行客户端激活。（如果激活失败则创建一个空的节点）
- 如果旧的 vnode 是 DOM，并且没有 ssr 标记，就用新的 vnode 创建 DOM 元素，并且挂载到新的 vnode 的父节点上面

#### createElm
- 如果这个 vnode 之前用到过，则从缓存里面克隆（注意，这里不是直接用以前的 vnode，因为所有的 vnode 都不能相同）
- 如果是组件vnode（占位vnode），则使用 createComponent 创建这个组件
- 如果不是组件 vnode，那证明是 div、span 这种 vnode，那么直接用相应平台的方法进行创建
- modules 主要负责操作 dom 上的属性（class、style、events等），nodeOps创建node提供工具函数

#### createComponent
```javascript
if (isUndef(Ctor)) {
  return
}

const baseCtor = context.$options._base

if (isObject(Ctor)) {
  Ctor = baseCtor.extend(Ctor)
}

if (typeof Ctor !== 'function') {
  if (process.env.NODE_ENV !== 'production') {
    warn(`Invalid Component definition: ${String(Ctor)}`, context)
  }
  return
}

let asyncFactory
if (isUndef(Ctor.cid)) {
  asyncFactory = Ctor
  Ctor = resolveAsyncComponent(asyncFactory, baseCtor)
  if (Ctor === undefined) {
    return createAsyncPlaceholder(
      asyncFactory,
      data,
      context,
      children,
      tag
    )
  }
}

data = data || {}
resolveConstructorOptions(Ctor)

if (isDef(data.model)) {
  transformModel(Ctor.options, data)
}

const propsData = extractPropsFromVNodeData(data, Ctor, tag)

if (isTrue(Ctor.options.functional)) {
  return createFunctionalComponent(Ctor, propsData, data, context, children)
}
const listeners = data.on
data.on = data.nativeOn

if (isTrue(Ctor.options.abstract)) {
  const slot = data.slot
  data = {}
  if (slot) {
    data.slot = slot
  }
}

installComponentHooks(data)
const name = Ctor.options.name || tag
const vnode = new VNode(
  `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
  data, undefined, undefined, undefined, context,
  { Ctor, propsData, listeners, tag, children },
  asyncFactory
)

return vnode
```

#### patchVnode
patchVnode 首先会执行 prepatch 和 update 钩子（用来更新 props、listeners 等），然后更新两个节点的文本，然后对各自的子节点进行更新，如果新旧节点的子节点都存在，则使用diff 算法进行更新，最后执行 postpatch 钩子

#### updateChildren（更新子节点）
