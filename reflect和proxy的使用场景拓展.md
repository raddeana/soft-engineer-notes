```es6
// 定义 Set 集合
const queuedObservers = new Set();

function set (target, key, value, receiver) {
  // 获取对象的赋值操作
  const result = Reflect.set(target, key, value, receiver);
  // 执行所有观察者
  queuedObservers.forEach(observer => observer());
  // 执行赋值操作
  return result;
}

// 把观察者函数都放入 Set 集合中
const observe = fn => queuedObservers.add(fn);
// observable 返回原始对象的代理，拦截赋值操作
const observable = obj => new Proxy(obj, { set });
```
