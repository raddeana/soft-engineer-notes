### 生产环境默认
- chunkIds: "deterministic" moduleIds: "deterministic" mangleExports: "deterministic"
- The algorithms assign short (3 or 5 digits) numeric IDs to modules and chunks and short (2 characters) names to exports in a deterministic way
- This is a trade-off between bundle size and long term caching
- moduleIds/chunkIds/mangleExports: false disables the default behavior and one can provide a custom algorithm via plugin
- Note：that in webpack 4 moduleIds/chunkIds: false without custom plugin resulted in a working build, while in webpack 5 you must provide a custom plugin

### Real Content Hash
Webpack 5 will use a real hash of the file content when using [contenthash] now
