Lighthouse 是一个开源的自动化工具，用于改进网络应用的质量
您可以将其作为一个 Chrome 扩展程序运行，或从命令行运行
您为 Lighthouse 提供一个您要审查的网址，它将针对此页面运行一连串的测试，然后生成一个有关页面性能的报告

### 安装 Lighthouse Chrome 扩展程序
地址：https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk

### 命令行工具
```nodejs
npm install -g lighthouse
lighthouse https://www.example.com --view
lighthouse --help
```

### Metrics指标
- First Contentful Paint (FCP)
  - 第一次内容丰富的绘画(FCP)指标衡量了从页面开始加载到页面内容的任何部分呈现在屏幕上的时间。对于该指标，"内容"指的是文本、图像（包括背景图像）、svg元素或非白色canvas元素
  - 要在JavaScript中测量FCP，你可以使用Paint Timing API
```
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntriesByName('first-contentful-paint')) {
    console.log('FCP candidate:', entry.startTime, entry);
  }
}).observe({type: 'paint', buffered: true});
```

- Speed Index
  - 内容在页面加载过程中的视觉显示速度
  - Lighthouse首先会在浏览器中捕获一段页面加载的视频，并计算出各帧之间的视觉进度
  - 然后，Lighthouse使用Speedline Node.js模块来生成速度指数得分

- Largest Contentful Paint (LCP)：最大内容画（LCP）指标报告了在视口中可见的最大图像或文本块的渲染时间
- Cumulative Layout Shift (CLS)：Cumulative Layout Shift (CLS)是一种视觉稳定性的测量方法，它量化了页面内容在视觉上的移动程度
- Total Blocking Time (TBT)：总阻塞时间（Total Blocking Time，TBT）量化了负载响应能力，测量了主线程被阻塞的时间长到足以阻止输入响应的总时间
