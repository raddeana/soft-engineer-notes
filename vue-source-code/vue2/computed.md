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
