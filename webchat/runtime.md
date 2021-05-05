### 运行时性能
#### setData
- 小程序的视图层目前使用 WebView 作为渲染载体，而逻辑层是由独立的 JavascriptCore 作为运行环境
- 在架构上，WebView 和 JavascriptCore 都是独立的模块，并不具备数据直接共享的通道
- 当前，视图层和逻辑层的数据传输，实际上通过两边提供的 evaluateJavascript 所实现
- 即用户传输的数据，需要将其转换为字符串形式传递，同时把转换后的数据内容拼接成一份 JS 脚本，再通过执行 JS 脚本的形式传递到两边独立环境
- 而 evaluateJavascript 的执行会受很多方面的影响，数据到达视图层并不是实时的

#### 常见的 setData 操作错误
- 频繁的去 setData
- 每次 setData 都传递大量新数据
  - 由setData的底层实现可知，我们的数据传输实际是一次 evaluateJavascript 脚本过程，当数据量过大时会增加脚本的编译执行时间，占用 WebView JS 线程
- 后台态页面进行 setData
  - 当页面进入后台态（用户不可见），不应该继续去进行setData，后台态页面的渲染用户是无法感受的，另外后台态页面去setData也会抢占前台页面的执行
