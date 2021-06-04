### webpack 的打包原理
- module：每一个源码 js 文件其实都可以看成一个 module
- chunk：每一个打包落地的 js 文件其实都是一个 chunk，每个 chunk 都包含很多 module

### 打包之后的代码
```javascript
(function (module, __webpack_exports__, __webpack_require__) {
    "use strict";
    __webpack_require__.r(__webpack_exports__);
    /* harmony import */
    var _moduleA__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__( /*!        ./moduleA */ "./src/moduleA.js");
    Object(_moduleA__WEBPACK_IMPORTED_MODULE_0__["default"])();
    __webpack_require__.e( /*! import() */ 0).then(__webpack_require__.bind(null, /*! ./moduleB             */ "./src/moduleB.js")).then(module => {});
})
```

### webpack_require
```javascript
function __webpack_require__(moduleId) {
    // Check if module is in cache
    // 先检查模块是否已经加载过了，如果加载过了直接返回
    if (installedModules[moduleId]) {
        return installedModules[moduleId].exports;
    }
    // Create a new module (and put it into the cache)
    // 如果一个import的模块是第一次加载，那之前必然没有加载过，就会去执行加载过程
    var module = installedModules[moduleId] = {
        i: moduleId,
        l: false,
        exports: {}
    };
    // Execute the module function
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    // Flag the module as loaded
    module.l = true;
    // Return the exports of the module
    return module.exports;
}
```

```javascript
(function (modules) {})({
    "./src/main.js": (function (module, __webpack_exports__, __webpack_require__) {}),
    "./src/moduleA.js": (function (module, __webpack_exports__, __webpack_require__) {})
});
```

### 动态引入
```javascript
// __webpack_require__.e (require.ensure)
__webpack_require__.e( /*! import() */ 0).then(__webpack_require__.bind(null, /*! ./moduleB             */ "./src/moduleB.js")).then(module => {});
```
```javascript
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([
[0], {
    "./src/moduleB.js": (function (module, __webpack_exports__, __webpack_require__) {})
}]);
```

### bundle.js
```javascript
var jsonpArray = window["webpackJsonp"] = window["webpackJsonp"] || [];
var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
jsonpArray.push = webpackJsonpCallback;
jsonpArray = jsonpArray.slice();
for(var i = 0; i < jsonpArray.length; i++) webpackJsonpCallback(jsonpArray[i]);
var parentJsonpFunction = oldJsonpFunction;
```
### webpack5 的 Module federation
```javascript
// import打包代码 在app1/src_bootstrap.js里面
/* harmony import */
var app2_Button__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__( /*! app2/Button */ "?ad8d");
/* harmony import */
var app2_Button__WEBPACK_IMPORTED_MODULE_1___default = /*#__PURE__*/ __webpack_require__.n(app2_Button__WEBPACK_IMPORTED_MODULE_1__);
```
#### webpack_modules
```javascript
var __webpack_modules__ = ({
    "./src/index.js": ((__unused_webpack_module, __unused_webpack_exports, __webpack_require__) => {
        __webpack_require__.e( /*! import() */ "src_bootstrap_js").then(__webpack_require__.bind(__webpack_require__, /*! ./bootstrap */ "./src/bootstrap.js"));
    }),
    "container-reference/app2": ((module) => {
        "use strict";
        module.exports = app2;
    }),
    "?8bfd": ((module, __unused_webpack_exports, __webpack_require__) => {
        "use strict";
        var external = __webpack_require__("container-reference/app2");
        module.exports = external;
    })
});
```
实际上 webpck_require.e 就是去加载 chunk 文件，等到加载成功 bundle.js 里面的 modules 就一定会有了 chunk 包含的那些 modules
#### webpack_require__.e
```
__webpack_require__.e = (chunkId) => {
    return Promise.all(Object.keys(__webpack_require__.f).reduce((promises, key) => {
        __webpack_require__.f[key](chunkId, promises);
        return promises;
    }, []));
};
```

#### webpack_require.f

##### 加载流程
- 先加载 src_main.js，这个没什么好说的，注入在 html 里面的
- src_main.js 里面执行 webpack_require("./src/index.js")
- src/index.js 这个 module 的逻辑很简单，就是动态加载 src_bootstrap_js 这个 chunk
- 动态加载 src_bootstrap_js 这个 chunk 时，经过 overridables，发现这个 chunk 依赖了 react、react-dom，那就看是否已经加载，没有加载就去加载对应的 js 文件，地址也告诉你了
- 动态加载 src_bootstrap_js 这个 chunk 时，经过 remotes，发现这个 chunk 依赖了?ad8d，那就去加载这个 js
- 动态加载 src_bootstrap_js 这个 chunk 时，经过 jsonp，就正常加载就好了
- 所有依赖以及 chunk 都加载完成了，就去执行 then 逻辑：webpack_require src_bootstrap_js 里面的 module：./src/bootstrap.js
两个独立打包的应用之间，通过全局变量去建立了一座彩虹桥
