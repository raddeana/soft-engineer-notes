#### $destroy
```javascript
    const vm: Component = this
    if (vm._isBeingDestroyed) {
      return
    }
    callHook(vm, 'beforeDestroy')
    vm._isBeingDestroyed = true
    // remove self from parent
    const parent = vm.$parent
    if (parent && !parent._isBeingDestroyed && !vm.$options.abstract) {
      remove(parent.$children, vm)
    }
    // teardown watchers
    if (vm._watcher) {
      vm._watcher.teardown()
    }
    let i = vm._watchers.length
    while (i--) {
      vm._watchers[i].teardown()
    }
    // remove reference from data ob
    // frozen object may not have observer.
    if (vm._data.__ob__) {
      vm._data.__ob__.vmCount--
    }
    // call the last hook...
    vm._isDestroyed = true
    // invoke destroy hooks on current rendered tree
    vm.__patch__(vm._vnode, null)
    // fire destroyed hook
    callHook(vm, 'destroyed')
    // turn off all instance listeners.
    vm.$off()
    // remove __vue__ reference
    if (vm.$el) {
      vm.$el.__vue__ = null
    }
    // release circular reference (#6759)
    if (vm.$vnode) {
      vm.$vnode.parent = null
    }
```
#### remove
```javascript 
if (arr.length) {
  const index = arr.indexOf(item)
  if (index > -1) {
    return arr.splice(index, 1)
  }
}
```
#### mount component
```javascript
callHook(vm, 'beforeMount');
updateComponent = () => {
    vm._update(vm._render(), hydrating)
};
new Watcher(vm, updateComponent, noop, {
    before () {
        if (vm._isMounted && !vm._isDestroyed) {
            callHook(vm, 'beforeUpdate')
        }
    }
}, true)
```
#### render
```javascript
// render.js
Vue.prototype._render = function (): VNode {
    vnode = render.call(vm._renderProxy, vm.$createElement)
    return vnode
}
```

#### _update
```javascript
// lifecycle.js
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    if (!prevVnode) {
        // initial render
        vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
    } else {
        // updates
        vm.$el = vm.__patch__(prevVnode, vnode)
    }
}
```

####  __patch__
##### diff between srr & browser，render directly
```javascript
Vue.prototype.__patch__ = inBrowser ? patch : noop
```

##### invokeDestroyHook
```javascript
let i, j
const data = vnode.data

if (isDef(data)) {
  if (isDef(i = data.hook) && isDef(i = i.destroy)) i(vnode)
  for (i = 0; i < cbs.destroy.length; ++i) cbs.destroy[i](vnode)
}

if (isDef(i = vnode.children)) {
  for (j = 0; j < vnode.children.length; ++j) {
    invokeDestroyHook(vnode.children[j])
  }
}
```

##### cbs.destroy
```javascript
const { modules, nodeOps } = backend

for (i = 0; i < hooks.length; ++i) {
  cbs[hooks[i]] = []
  for (j = 0; j < modules.length; ++j) {
    if (isDef(modules[j][hooks[i]])) {
      cbs[hooks[i]].push(modules[j][hooks[i]])
    }
  }
}
```

##### refs destroy，registerRef
```javascript
  const key = vnode.data.ref
  if (!isDef(key)) return

  const vm = vnode.context
  const ref = vnode.componentInstance || vnode.elm
  const refs = vm.$refs
  if (isRemoval) {
    if (Array.isArray(refs[key])) {
      remove(refs[key], ref)
    } else if (refs[key] === ref) {
      refs[key] = undefined
    }
  }
```

##### directives destroy
```javascript
if (oldVnode.data.directives || vnode.data.directives) {
  _update(oldVnode, vnode)
}

callHook(oldDirs[key], 'unbind', oldVnode, oldVnode, isDestroy)
```

##### vnode.data destroy
call componentInstance $destroy()
```javascript
var componentInstance = vnode.componentInstance;
if (!componentInstance._isDestroyed) {
  if (!vnode.data.keepAlive) {
    componentInstance.$destroy();
  } else {
    deactivateChildComponent(componentInstance, true /* direct */);
  }
}
```
##### vnode unbind
define by directives，v-show destroy hook
```
el.style.display = el.__vOriginalDisplay
```
