### 理解 Tapable
Tapable 就是支持下列纬度的 EventEmitter，提供了注册事件回调，并能让事件产生方来灵活选择如何执行这些回调
- 执行方式：同步、异步串行、异步并行
- 过程控制：基本（依次执行）、流式（依次执行但传入上次的结果，类似 reduce）、提前结束（依次执行，但是允许提前结束）、循环（loop，循环执行直到返回的是 undefined）

### Compiler 编译器实例化
- 根据是否 watch 选择 watchRun 还是 run
- 执行 compile，新建一个 Compilation
- 使用 Compilation 执行编译过程
- emit 产物

### 生命周期
![image](https://user-images.githubusercontent.com/5197188/121382235-f9415900-c978-11eb-9c6a-e29589905175.png)

#### compiler.hooks.compilation
- JavascriptModulePlugin 会注册 js 文件的 parser 和 generator
- EntryPlugin 会设置 EntryDependency 的 factory，告诉编译器如何处理 entry 模块

#### compiler.hooks.make
当 Compilation 被创建完成后(下文讲解 Compilation)，就会调用 make 生命周期，webpack 内置了 EntryPlugin 注册了 make 的处理逻辑，用来找到入口模块

### Compilation
Compiler 是经由配置生成的编译器，而一个 Compilation 实例代表一次完整的编译过程；包括加载入口模块，解析依赖，解析 AST，创建 Chunk，生成产物等一系列工作
![image](https://user-images.githubusercontent.com/5197188/121383306-ee3af880-c979-11eb-8774-f37002a69fa5.png)

### NormalModuleFactory 模块工厂
- create 创建模块
- resolve 解析模块依赖
- parse 将模块代码生成 AST

### NormalResolver
### RuleSetCompiler
RuleSetCompiler 会把用户设置的 module.rules 和 webpack 内置的默认 rules 结合起来，形成一个方法，在后续处理中直接调用来确定模块处理用哪些 loaders 还有 parse 代码时需要的参数
### CompileRule 过程

