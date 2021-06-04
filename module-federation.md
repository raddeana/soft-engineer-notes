#### 动机
- 多个独立的构建可以组成一个应用程序，这些独立的构建之间不应该存在依赖关系，因此可以单独开发和部署它们
- 这通常被称作微前端，但并不仅限于此

#### 底层概念
- 我们区分本地模块和远程模块；本地模块即为普通模块，是当前构建的一部分；远程模块不属于当前构建，并在运行时从所谓的容器加载；
- 加载远程模块被认为是异步操作；当使用远程模块时，这些异步操作将被放置在远程模块和入口之间的下一个 chunk 的加载操作中；如果没有 chunk 加载操作，就不能使用远程模块；
- chunk 的加载操作通常是通过调用 import() 实现的，但也支持像 require.ensure 或 require([...]);
- 容器是由容器入口创建的，该入口暴露了对特定模块的异步访问。暴露的访问分为两个步骤：
  - 加载模块（异步的）
  - 执行模块（同步的）
  - 容器可以嵌套使用，容器可以使用来自其他容器的模块
  - 容器之间也可以循环依赖

#### 重载（Overriding） 
容器的使用者能够提供“重载”，即替换容器中的一个 "可重载" 的模块
"name" 用于标识容器中可重载的模块，重载（Overrides）的提供和容器暴露模块类似
- 加载（异步）
- 执行（同步）

#### 构建块(Building blocks)
- OverridablesPlugin (底层 API) 特定模块 "可重载"；一个本地 API ( __webpack_override__ ) 允许提供重载
```javascript
const OverridablesPlugin = require('webpack/lib/container/OverridablesPlugin');
module.exports = {
  plugins: [
    new OverridablesPlugin([{
      // 通过 OverridablesPlugin 定义一个可重载的模块
      test1: './src/test1.js'
    }]),
  ],
};

__webpack_override__({
  // 这里我们重写 test1 模块
  test1: () => 'I will override test1 module under src'
});
```
- ContainerPlugin (底层 API)该插件将特定的引用添加到作为外部资源（externals）的容器中，并允许从这些容器中导入远程模块;
  - 它还会调用这些容器的 override API 来为它们提供重载;
  - 本地的重载（当构建也是一个容器时，通过 __webpack_override__ 或 override API）和指定的重载被提供给所有引用的容器
- ModuleFederationPlugin （高级 API）该插件组合了 ContainerPlugin 和 ContainerReferencePlugin；重载（overrides）和可重载（overridables）被合并到指定共享模块的单个列表中；
