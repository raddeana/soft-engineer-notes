### 综述
- 通过持久化缓存提高性能
- 采用更好的持久化缓存算法和默认行为
- 通过优化 Tree Shaking 和代码生成来减小Bundle体积
- 提高 Web 平台的兼容性
- 清除之前为了实现 Webpack4 没有不兼容性变更导致的不合理 state
- 尝试现在引入重大更改来为将来的功能做准备，以使我们能够尽可能长时间地使用 Webpack 5

### 过时功能移除
- 首先是去掉了在 Webpack4 里面已经 Warming 的功能。
- 同时 IgnorePlugin 和 BannerPlugin 现在必须传入一个参数，参数可以是 Object、String或者Function
- require.include 语法被废弃，使用时会有 Warming。当然这个行为可以通过 Rule.parser.requireInclude 来把这个语法改成 allowed, deprecated 或者 disabled
- 去掉 Node.js Polyfills

### 长期缓存
- 确定性的模块、模块ID和导出名称
  - 首先是模块、ID和导出名称都唯一确定下来，背后对应的配置是 chunkIds: "deterministic", moduleIds: "deterministic", mangleExports: "deterministic"
  - 其中模块和模块ID用 3 ~ 4 位的数字ID，导出名称用 2 位的数字ID
  - 这个设置是默认开启的，但也允许通过上述配置修改
- 真实内容哈希
  - 在 Webpack5 里会使用文件内容的真实哈希 [contenthash]，而不是之前的仅仅使用文件内部结构的哈希
  - 这对于长期缓存有着积极的影响，尤其是代码里面只有注释和变量名修改的时候，Webpack会继续用之前的缓存而不是新的文件内容

### 开发支持
- 新的 Chunk IDs 使用了新的语法生成 Chunk ID，一个 Chunk ID 是有 chunk 的内容来决定的
- Module Federation
- 更好的 Tree Shaking
  - 嵌套 tree-shaking：Webpack现在会去追踪 export 的链路，对于嵌套场景有更好的优化
  - 内部模块：optimization.innerGraph记录依赖关系
- 支持 Top Level Await，从此告别 async，即允许开发者在 async 函数外部使用 await 字段

### 内置静态资源构建能力 —— Asset Modules
```
module.exports = {
    module: {
      rules: [
          {
            test: /\.(png|jpg|svg|gif)$/,
            type: 'asset/resource',
            generator: {
                // [ext]前面自带"."
                filename: 'assets/[hash:8].[name][ext]',
            },
        },
      ],
    },
}
```
type 取值如下几种
- asset/source ——功能相当于 raw-loader。
- asset/inline——功能相当于 url-loader，若想要设置编码规则，可以在 generator 中设置 dataUrl。具体可参见官方文档[10]。
- asset/resource——功能相当于 file-loader。项目中的资源打包统一采用这种方式，得益于团队项目已经完全铺开使用了 HTTP2 多路复用的相关特性，我们可以将资源统一处理成文件的形式，在获取时让它们能够并行传输，避免在通过编码的形式内置到 js 文件中，而造成资源体积的增大进而影响资源的加载。
- asset—— 默认会根据文件大小来选择使用哪种类型，当文件小于 8 KB 的时候会使用 asset/inline，否则会使用 asset/resource

### 内置 FileSystem Cache 能力加速二次构建
生产环境下默认的缓存存放目录在 node_modules/.cache/webpack/default-production 中
```
// webpack.config.js
module.exports = {
    cache: {
        type: 'filesystem',
        // 可选配置
        buildDependencies: {
            config: [__filename],  // 当构建依赖的config文件（通过 require 依赖）内容发生变化时，缓存失效
        },
        name: ''                   // 配置以name为隔离，创建不同的缓存文件，如生成PC或mobile不同的配置缓存
    },
}
```

### 内置 WebAssembly 编译及异步加载能力
WebAssembly[15] 被设计为一种面向 web 的二进制的格式文件，以其更接近于机器码而拥有着更小的文件体积和更快速的执行效率
### 内置 Web Worker 构建能力



