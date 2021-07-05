```javascript
function getDynamicComponent (filePath) {
  // 异步动态路径加载，每个模块编译成独立的文件
  // return import(`../utils/${filePath}`);

  // 1、同步动态路径，将被打包至common-chunk中，生成一个sync recursive ^\\.\\/.*$文件
  // 2、如果和异步模块加载同时使用，被视为加载两次
  return new Promise(function (resolve) {
      require([`../utils/${filePath}`], resolve);
  });

  // return new Promise(function (resolve, reject) {
  //   require.ensure([], function(require){
  //     const module = require(`../utils/${filePath}`);
  //     resolve(module);
  //   });
  // });
}

getDynamicComponent('chunk').then((module) => {
  console.log(module);
})

// 错误的实现
// function getAsyncComponent (path) {
//   return new Promise((resolve, reject) => {
//     require([path], resolve);
//   });
// }

// getAsyncComponent('../utils/tr').then((module) => {
//   console.log(module);
// });

// 异步模块
require.ensure([], function(require){
  const module = require('../utils/tr');
  console.log(module);
});

require.ensure([], function(require){
  const module = require('../utils/kk');
  console.log(module);
});

// 异步模块
// import('../utils/tr').then((module) => {
//   console.log(module);
// });

// import('../utils/kk').then((module) => {
//   console.log(module);
// });
```
