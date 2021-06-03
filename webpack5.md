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

