### 高效的 ComputedStyle
浏览器还有一个非常棒的策略，在特定情况下，浏览器会共享 Computed Style ，网页中能共享的标签非常多，所以能极大的提升执行效率
如果能共享，那就不需要执行匹配算法了，执行效率自然非常高
如果两个或多个 Element 的 ComputedStyle 不通过计算可以确认他们相等，那么这些 ComputedStyle 相等的 Elements 只会计算一次样式，其余的仅仅共享该 ComputedStyle
- TagName 和 Class 属性必须一样
- 不能有 Style 属性。哪怕 Style 属性相等，他们也不共享
- 不能使用 Sibling selector，譬如: first-child , :last-selector , + selector 
- mappedAttribute 必须相等 
eg（align）：
``` code
<p align="middle">paragraph</p>
```

### CSS 书写顺序
浏览器并不是一获取到 CSS 样式就立马开始解析，
而是根据 CSS 样式的书写顺序将之按照 DOM 树的结构分布渲染样式，
然后开始遍历每个树结点的 CSS 样式进行解析，
此时的 CSS 样式的遍历顺序完全是按照之前的书写顺序，
在解析过程中，一旦浏览器发现某个元素的定位变化影响布局，则需要倒回去重新渲染
<br />
当浏览器解析到 position 的时候突然发现该元素是绝对定位元素需要脱离文档流，而之前却是按照普通元素进行解析的，所以不得不重新渲染。
渲染引擎首先解除该元素在文档中所占位置，这就导致了该元素的占位情况发生了变化，其他元素可能会受到它回流的影响而重新排位。
```css
.box {
    width: 150px;
    height: 150px;
    font-size: 24px;
    position: absolute;
}
```
优化后
```css
.box {
    position: absolute;
    width: 150px;
    height: 150px;
    font-size: 24px;
}
```


规范（CSSLint），建议顺序：
- 定位属性
```
position  display  float  left  top  right  bottom   overflow  clear   z-index
```
- 自身属性
```
width  height  padding  border  margin   background
```
- 文字样式
```
font-family   font-size   font-style   font-weight   font-varient   color
```
- 文本属性
```
text-align   vertical-align   text-wrap   text-transform   text-indent    text-decoration   letter-spacing    word-spacing    white-space   text-overflow
```
- CSS3中新增属性
```
content   box-shadow   border-radius  transform
```

#### 优化策略
- 使用 id selector 非常的高效
- 避免深层次的 node
- 不要使用 attribute selector ，如：p[att1='val1']。这样的匹配非常慢
- 通常将浏览器前缀置于前面，将标准样式属性置于最后
```
.foo {
  -moz-border-radius: 5px;
  border-radius: 5px;
}
```
- 遵守 CSSLint 规则
  - font-faces        　　　　   不能使用超过5个web字体
  - import        　　　　　　　 禁止使用@import
  - regex-selectors        　　 禁止使用属性选择器中的正则表达式选择器
  - universal-selector    　　  禁止使用通用选择器*
  - unqualified-attributes    　禁止使用不规范的属性选择器
  - zero-units            　　  0后面不要加单位
  - overqualified-elements    　使用相邻选择器时，不要使用不必要的选择器
  - shorthand        　　　　　　简写样式属性
  - duplicate-background-images 相同的url在样式表中不超过一次
- 减少 CSS 文档体积
  - 移除空的 CSS 规则（Remove empty rules）
  - 值为 0 不需要单位
  - 使用缩写
  - 属性值为浮动小数 0.**，可以省略小数点之前的0；
  - 不给 h1-h6 元素定义过多的样式
- CSS Will Change
  - WillChange 属性，允许作者提前告知浏览器的默认样式，使用一个专用的属性来通知浏览器留意接下来的变化，从而优化和分配内存
```
.sidebar {
  will-change: transform;
}
```
- 不要使用 @import：使用 @import 引入 CSS 会影响浏览器的并行下载
- 避免过分重排
  - 常见的重排元素
```
width 
height 
padding 
margin 
display 
border-width 
border 
top 
position 
font-size 
float 
text-align 
overflow-y 
font-weight 
overflow 
left 
font-family 
line-height 
vertical-align 
right 
clear 
white-space 
bottom 
min-height
```
- 高效利用 computedStyle
  - 公共类
  - 慎用 ChildSelector
  - 尽可能共享
- 减少昂贵属性，当页面发生重绘时，它们会降低浏览器的渲染性能
  - box-shadow
  - border-radius
  - filter
  - :nth-child

#### 浏览器渲染过程
- 解析HTML，生成DOM树，解析CSS，生成 CSSOM 树
- 将DOM树和CSSOM树结合，生成渲染树（Render Tree）
- Layout: 根据生成的渲染树，进行 Layout，得到节点的几何信息（位置，大小）
- Painting: 根据渲染树以及回流得到的几何信息，得到节点的绝对像素
- Display: 将像素发送给GPU，展示在页面上

#### 构建渲染树
- 从DOM树的根节点开始遍历每个可见节点
- 对于每个可见的节点，找到CSSOM树中对应的规则，并应用它们
- 根据每个可见节点以及其对应的样式，组合生成渲染树

#### 浏览器的优化机制
  - 由于每次重排都会造成额外的计算消耗，因此大多数浏览器都会通过队列化修改并批量执行来优化重排过程
  - 浏览器会将修改操作放入到队列里，直到过了一段时间或者操作达到了一个阈值，才清空队列
  - 当获取布局信息的操作的时候，会强制队列刷新

#### 减少回流和重绘
- 最小化回流和重排
- 批量修改DOM
- 避免触发同步布局事件
  - 例子 
```
function initParagraphs () {
    for (let i = 0; i < paragraphs.length; i++) {
        paragraphs[i].style.width = box.offsetWidth + 'px';
    }
}
```
  - 优化
```
const width = box.offsetWidth;
function initParagraphs () {
    for (let i = 0; i < paragraphs.length; i++) {
        paragraphs[i].style.width = width + 'px';
    }
}
```
- 对于复杂动画效果,使用绝对定位让其脱离文档流
- css3硬件加速（GPU加速）
  - 使用css3硬件加速，可以让transform、opacity、filters这些动画不会引起回流重绘
  - 但是对于动画的其它属性，还是会引起回流重绘的，不过它还是可以提升这些动画的性能
