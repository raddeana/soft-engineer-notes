### Local variables
访问响应式数据，会触发它的 getter，避免重复访问，改为本地变量的方式

### Deferred features
```javascript
export default function (count = 10) {
  return {
    data () {
      return {
        displayPriority: 0
      }
    },
    mounted () {
      this.runDisplayPriority()
    },
    methods: {
      runDisplayPriority () {
        const step = () => {
          requestAnimationFrame(() => {
            this.displayPriority++
            if (this.displayPriority < count) {
              step()
            }
          })
        }
        step()
      },
      defer (priority) {
        return this.displayPriority >= priority
      }
    }
  }
}
```
```javascript
<template>
  <div class="deferred-on">
    <VueIcon icon="fitness_center" class="gigantic"/>
    <h2>I'm an heavy page</h2>
    <template v-if="defer(2)">
      <Heavy v-for="n in 8" :key="n"/>
    </template>
    <Heavy v-if="defer(3)" class="super-heavy" :n="9999999"/>
  </div>
</template>
```
Defer 的主要思想就是把一个组件的一次渲染拆成多次，然后在通过 requestAnimationFrame 在每一帧渲染的时候自增
### Time slicing
将运行任务切片，在每一帧中提交数据
```
fetchItems ({ commit }, { items, splitCount }) {
  commit('clearItems')
  const queue = new JobQueue()
  splitArray(items, splitCount).forEach(
    chunk => queue.addJob(done => {
      requestAnimationFrame(() => {
        commit('addItems', chunk)
        done()
      })
    })
  )
  await queue.start()
}
```
### Non-reactive data
### 只渲染可视区域的dom
