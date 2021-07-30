### 主线程和多线程
用户使用浏览器一般会打开多个页面 (多 Tab), 现代浏览器使用单独的进程 (Render Process) 渲染每个页面, 以提升页面性能和稳定性, 并进行操作系统级别的内存隔离

### 主线程 (Main Thread)
主线程渲染页面每一帧 (Frame), 如下图所示, 会包含 5 个步骤: JavaScript → Style → Layout → Paint → Composite, 如果 JS 的执行修改了 DOM, 可能还会暂停 JS, 插入并执行 Style 和 Layout

### 多线程
JS 多线程, 是有独立于主线程的 JS 运行环境； 如下图所示: Worker 线程有独立的内存空间, Message Queue, Event Loop, Call Stack 等, 线程间通过 postMessage 通信
JS 单线程中的" 并发", 准确来说是 Concurrent. 如下图所示, 运行时只有一个函数调用栈, 通过 Event Loop 实现不同 Task 的上下文切换 (Context Switch). 这些 Task 通过 BOM API 调起其他线程为主线程工作, 但回调函数代码逻辑依然由 JS 串行运行

### 应用场景:
- 可以减少主线程卡顿
- 可能会带来性能提升（Worker 多线程并不会直接带来计算性能的提升, 能否提升与设备 CPU 核数和线程策略有关）（任务在 Worker 线程上运行并不会比原本主线程更快, 而线程新建消耗和通信开销使得渲染间隔可能变得更久）

### 多线程与 CPU 核数
- CPU 的单核 (Single Core) 和多核 (Multi Core) 离前端似乎有点远了. 但在页面上运用多线程技术时, 核数会影响线程创建策略
- 进程是操作系统资源分配的基本单位，线程是操作系统调度 CPU 的基本单位（协程（nodejs）） 操作系统对线程能占用的 CPU 计算资源有复杂的分配策略. 如下图所示:
  - 单核多线程通过时间切片交替执行
  - 多核多线程可在不同核中真正并行

### DedicatedWorker 和 SharedWorker
Web Worker 规范中包括: DedicatedWorker 和 SharedWorker; 规范并不包括 Service Worker
- DedicatedWorker 简称 Worker, 其线程只能与一个页面渲染进程 (Render Process) 进行绑定和通信
- 而 SharedWorker 可以在多个浏览器 Tab 中访问到同一个 Worker 实例, 实现多 Tab 共享数据, 共享 webSocket 连接等

### Worker 环境和主线程环境的异同
Worker 是无 UI 的线程, 无法调用 UI 相关的 DOM/BOM API
Worker 运行环境与主线程的共同点主要包括：
- 含完整的 JS 运行时, 支持 ECMAScript 规范定义的语言语法和内置对象
- 持 XmlHttpRequest, 能独立发送网络请求与后台交互
- 含只读的 Location, 指向 Worker 线程执行的 script url, 可通过 url 传递参数给 Worker 环境
- 含只读的 Navigator, 用于获取浏览器信息, 如通过 Navigator.userAgent 识别浏览器
- 持 setTimeout / setInterval 计时器, 可用于实现异步逻辑
- 持 WebSocket 进行网络 I/O; 支持 IndexedDB 进行文件 I/O

### 通信速度
Worker 多线程虽然实现了 JS 任务的并行运行, 也带来额外的通信开销；从线程 A 调用 postMessage 发送数据到线程 B onmessage 接收到数据有时间差, 这段时间差称为通信消耗；
而且主线程 postMessage 会占用主线程同步执行, 占用时间与数据传输方式和数据规模相关；要避免多线程通信导致的主线程卡顿, 需选择合适的传输方式, 并控制每个渲染周期内的数据传输规模

### 数据传输方式
根据计算机进程模型, 主线程和 Worker 线程属于同一进程, 可以访问和操作进程的内存空间. 但为了降低多线程并发的逻辑复杂度, 部分传输方式直接隔离了线程间的内存, 相当于默认加了锁
- Structured Clone：Structured Clone 是 postMessage 默认的通信方式. 复制一份线程 A 的 JS Object 内存给到线程 B, 线程 B 能获取和操作新复制的内存
  -  线程 A 要同步执行 Object Serialization, 线程 B 要同步执行 Object Deserialization;
  -  如果 Object 规模过大, 会占用大量的线程时间
- Transfer Memory：Transfer Memory 意为转移内存, 它不需要 Serialization/Deserialization, 能大大减少传输过程占用的线程时间；线程 A 将指定内存的所有权和操作权转给线程 B, 但转让后线程 A 无法再访问这块内存
  - Transfer Memory 以失去控制权来换取高效传输, 通过内存独占给多线程并发加锁
  - 但只能转让 ArrayBuffer 等大小规整的二进制 (Raw Binary) 数据; 对矩阵数据 (如 RGB 图片) 比较适用
- Shared Array Buffers：是共享内存, 线程 A 和线程 B 可以同时访问和操作同一块内存空间
  - 但多个并行的线程共享内存, 会产生竞争问题 (Race Conditions)
  - 不像前 2 种传输方式默认加锁, Shared Array Buffers 把难题抛给开发者, 开发者可以用 Atomics 来维护这块共享的内存

### 数据传输规模
- JS 代码里面不包括动画渲染 (100ms), 数据传输规模应该保持在 100KB 以下;
- JS 代码里面包括动画渲染 (16ms), 数据传输规模应该保持在 10KB 以下.

### 实际场景
- 接口轮询 websocket
- 图像处理，大计算场景

