#### runtime和ssr的区别
```javascript
// runtime
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

//ssr
function createGetterInvoker(fn) {
  return function computedGetter () {
    return fn.call(this, this)
  }
}
```
#### computed 是如何知道内部依赖发生变化的
依赖收集是在get中完成的
```
const vm = {
    dependencies: [],
    obj: {
        get a () {
            this.dependencies.push('a');
 
            if (this.obj.d) {
                return this.obj.b;
            }
        },
        get b () {
            this.dependencies.push('b');
            return 1;
        },
        get d () {
            this.dependencies.push('d');
            return this._d;
        },
        set d (val) {
            this._d = val;
        }
    },
    computed: {
        c: {
            get () {
                return this.obj.a;
            }
        }
    }
};
```

#### 缓存
在得知 computed 属性发生变化之后， Vue内部并不立即去重新计算出新的 computed 属性值，而是仅仅标记为dirty，下次访问的时候，再重新计算，然后将计算结果缓存起来
