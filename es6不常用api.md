### Object
- Object.freeze() 
  - 方法可以冻结一个对象
  - 一个被冻结的对象再也不能被修改
  - 冻结了一个对象则不能向这个对象添加新的属性，不能删除已有属性，不能修改该对象已有属性的可枚举性、可配置性、可写性，以及不能修改已有属性的值
  - 冻结一个对象后该对象的原型也不能被修改
  - freeze() 返回和传入的参数相同的对象
- Object.seal() 方法封闭一个对象，阻止添加新属性并将所有现有属性标记为不可配置；当前属性的值只要原来是可写的就可以改变
- Object.preventExtensions()方法让一个对象变的不可扩展，也就是永远不能再添加新的属性
- Proxy.revocable() 方法可以用来创建一个可撤销的代理对象
- configurable 能否使用delete、能否需改属性特性、或能否修改访问器属性、false为不可重新定义、默认值为true
```javascript
// 不可重新定义
Object.defineProperty(obj, 'prop', {
    configurable: false,
    get: function() {},
    set: function() {}
});
```
