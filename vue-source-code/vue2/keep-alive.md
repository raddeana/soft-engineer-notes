#### watch exclude & include
```javascript
  this.$watch('include', val => {
    pruneCache(this, name => matches(val, name))
  })
  this.$watch('exclude', val => {
    pruneCache(this, name => !matches(val, name))
  })
```

#### pruneCache
```javascript
const { cache, keys, _vnode } = keepAliveInstance
for (const key in cache) {
  const cachedNode: ?VNode = cache[key]
  if (cachedNode) {
    const name: ?string = getComponentName(cachedNode.componentOptions)
    if (name && !filter(name)) {
      pruneCacheEntry(cache, key, keys, _vnode)
    }
  }
}
```

#### pruneCacheEntry
```javascript
const cached = cache[key]
if (cached && (!current || cached.tag !== current.tag)) {
  cached.componentInstance.$destroy()
}
cache[key] = null
remove(keys, key)
```
