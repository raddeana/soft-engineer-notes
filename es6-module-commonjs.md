### es6 module：浏览器和服务器通用的模块解决方案
- 静态化；编译时就能确定模块的依赖关系；commonjs和AMD只能在运行时确定
  - 不在需要UMD模块格式了
  - 不再需要对象作为命名空间
- es6模块不是对象，而是通过export命令显式指定输出的代码，再通过import命令输入；缺点：没法引用es6模块本身，因为它不是对象
- 自动采用严格模式，所以严格模式的规则，es6模块要遵守
- 导入和导出：import和export
  - 都可以使用as别名；export语句输出的接口，与其对应的值是动态绑定的
  - 可以出现在模块顶层的任何位置，不能处于块级作用域内，否则会报错；因为这样就没法做静态优化了
  - import命令输入的变量都是只读的；不允许在加载模块的脚本里面改写接口

### commonJs：服务端模块化规范
- 加载机制：输入的是被输出值的拷贝；一旦输出，模块内部变化就影响不到这个值
- 所有代码都运行在模块作用域
- 模块可以多次加载，但只在第一次加载时运行一次，其结果会被缓存；后面再加载的时候，就直接读取缓存结果；要想让模块再运行的话，必须清除缓存
- 模块加载的顺序，按照其在代码中出现的顺序
- module对象
  - module.id：模块的标识符，通常是带有绝对路径的模块文件名
  - module.filename： 模块的文件名，带有绝对路径
  - module.loaded：模块是否已经加载完成
  - module.parents：[]，调用该模块的模块；node下执行，会得到null或者undefined，require引入的会返回它的调用模块；可以用此判断该模块是不是入口模块
  - module.exports：表示模块对外输出的值；还有个exports变量，指向的是module.exports；不能直接将 exports 变量指向一个值，因为这样等于切断了 exports 与 module.exports 的联系

### UMD（Universal Module Definition) 希望提供一个前后端跨平台的解决方案
- 以此判断是否支持Nodejs模块格式：（typeof exports === 'object'），是否支持 AMD（typeof define === 'function'），都不支持则将模块公开到全局（window或global）
