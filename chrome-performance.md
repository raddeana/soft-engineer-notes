### Main
Main 代表渲染进程中的主线程，渲染相关的事情基本都是它来做，脚本执行、样式计算、布局计算、绘制等等
### Compositor & Raster
Compositor 代表渲染进程中的合成线程，Raster 代表渲染进程中的栅格线程
- 主线程将页面分成若干图层（后文中会提及 Update Layer Tree）
- 栅格线程分别对每一个层进行栅格化处理
- 合成线程将栅格化的图块合并成一个页面
### beforeunload
因为 Performance 的录制是在已有页面上进行 reload，所以记录的生命周期从页面的卸载开始；beforeunload 事件首先被浏览器触发
### pagehide
  - 为什么事件的触发顺序和上面的生命周期流程图不一致，是 pagehide -> visibilitychange -> unload 呢？
  - 事实上，在浏览器之前的设计中，如果页面在卸载阶段可视，visibilitychange 就会在 pagehide 之后触发
  - 这就使得页面的卸载在不同可视情况下，有着不一致的生命周期与事件顺序

### pageshow/load
因导航而使得浏览器在窗口内呈现文档时，浏览器会在 window 上触发 pageshow 事件

### 任务与性能问题
Performance 还无法清晰的看出 Event Loop
