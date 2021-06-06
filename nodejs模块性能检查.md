引入模块时建议做如下检查
- 包含的依赖项的大小
- 需要加载的(require()) 资源
- 你所加载的资源能够执行你关心的操作
```javascript
// 检查加载request需要的时间和内存
node --cpu-prof --heap-prof -e "require('request')"
```
执行此命令将在您执行的目录下生成一个.cpuprofile和一个.heapprofile 文件；这两个文件都可以使用 Chrome 开发者工具进行分析，分别使用 Performance 和 Memory 标签 进行分析。
