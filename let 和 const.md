### 基本特性
#### let命令
- 代码块内有效
- 不能重复声明
- 不存在变量提升
- 只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个上下文，不再受外部的影响
```code
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```
```code
if (true) {
   tmp = 'abc'; // Cannot access 'tmp' before initialization
 
   let tmp;
   console.log(tmp); // undefined
 
   tmp = 123;
   console.log(tmp); // 123
}
```

#### const命令
- const 声明一个只读变量，声明之后不允许改变
- const声明的变量不得改变值，这意味着，const一旦声明变量，就必须立即初始化，不能留到以后赋值
- const的作用域与let命令相同：只在声明所在的块级作用域内有效
- const命令声明的常量也是不提升，同样存在绑定上下文，只能在声明的位置后面使用

### 原理
- const 其实保证的不是变量的值不变，而是保证变量指向的内存地址所保存的数据不允许改动
- 简单类型和复合类型保存值的方式是不同的，对于简单类型（数值 number、字符串 string 、布尔值 boolean），值就保存在变量指向的那个内存地址，因此 const 声明的简单类型变量等同于常量
- 而复杂类型（对象 object，数组 array，函数 function），变量指向的内存地址其实是保存了一个指向实际数据的指针，所以 const 只能保证指针是固定的，至于指针指向的数据结构变不变就无法控制了，所以使用 const 声明复杂类型对象时要慎重
- const 运行效率高于let
#### babel
babel对const和let的处理不同，同时兼顾了性能考虑。const声明的变量是不允许重新赋值的，所以转换逻辑很简单。
在compilation time检查完重新赋值的情况，检查通过就将const替换为var。
let的话做的就比较巧妙了，babel会做variable replacement，所以块级作用域内的变量实际上已经被改过名字了，因此上下文变量查找不存在严重的闭包性能问题。

#### Node.js
关于let，则是V8层面scope parser做的事情，相比closure形式的作用域，let会极大的降低上下文变量查找时间和闭包带来的性能问题

