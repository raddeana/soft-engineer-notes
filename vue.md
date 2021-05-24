### router
- $route先发生变化（在watch中被触发），在下一个周期（nextTick）才完成路由替换；
- 如果使用了transition，路由替换将经过三个过程，$route发生变化，transition name发生变化，transition新旧路由动画过程，完成路由替换

### watch
- watch在update之前被触发
