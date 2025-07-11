webview本质就是在移动端系统中，内嵌的可以用来展示 Web 应用的组件。这让移动端可以像打开浏览器一样打开页面，被称为 Hybrid （混合）模式。

webview两个重要的落地点是：嵌入H5的混合式APP、小程序（微信、支付宝...）。
在H5内嵌方面，渲染流程类似于传统浏览器，首先是Native打开Webview容器，该容器打开对应的URL地址，最后请求数据并加载页面。

下面将对小程序的webView应用做介绍
## 架构设计
小程序自身采用双线程架构，分为<font color="#d83931">渲染层和逻辑层</font>
![[Pasted image 20240724165114.png]]
### 逻辑层是什么
在微信小程序中，开发者在app.json文件注册页面，在/pages目录下创建对应页面。代码如下：
![[Pasted image 20240724170713.png]]
在index.wxml中： `<view bind:tap="handleClick" >hello,world</view>`
页面就会呈现出`hello, world`，这一过程暴露出几个问题：
- Page函数从何而来？
- index.js中的代码是怎么执行的？message属性如何通过JS传递到WXML中？

首先，来看Page函数的来源。在onLoad中直接打印window对象，输出为undefined，证明小程序的runtime运行时并不在浏览器环境下。

小程序主包的打包产物（service.js文件）如下：
```js
/* 存放对应的页面 */
self.source_code.pages = [
   {
       name:'pages/index/index', //对应的页面路径
       source:{ // js 逻辑资源
           jsCode:function(exports, require, module){
               module.exports = function(wx,App,Page,Component,getApp,global){
                   // 编译后小程序业务代码，这样就可以获取 wx,Page,Component 属性。
                   // 业务代码
               }
           },
           jsJson:{...}
       }
   },
   {
       name:'/app' // 小程序 app 文件
       source:{
           jsCode:function(exports, require, module){
                module.exports = function(wx,App,Page,Component,getApp,global){
                   // 业务代码 
                   App({})
                }
           },
       }
   },
];
/* 存放对应的组件 */
self.source_code.components = []
/* 存放正常的 js 文件 */
self.source_code.modules = []
```
可以看到，上述代码中，页面结构被存放在self.source_code.pages这个数组中；组件和其他的JS代码被分别存放在components和modules数组中。

### 小程序为什么是双引擎架构？
浏览器因为一些历史问题，从设计之初就支持`JS操作DOM`，所以渲染和逻辑两个线程是互斥的，后期通过web Worker等技术弥补在复杂计算方面的不足，是一个渐进式更新的情况。

而小程序的从0-1设计，摒弃了浏览器的一些包袱，双引擎的架构有以下几点考虑：

#### 安全性
隔离恶意代码
```js
// 逻辑层：只能处理数据和业务逻辑
Page({
  data: { message: 'Hello' },
  onLoad() {
    // ❌ 无法直接操作 DOM
    // document.getElementById() // 不存在
    
    // ✅ 只能通过 setData 更新视图
    this.setData({ message: 'Updated' });
  }
});

// 渲染层：只负责展示，无法执行复杂逻辑
<!-- WXML 模板，无法执行 JavaScript -->
<view>{{message}}</view>
```

#### 性能优化
```js
// 逻辑层：处理计算（独立线程）
Page({
  onLoad() {
    // 大量计算在逻辑层进行，不阻塞渲染
    this.heavyComputation().then(result => {
      this.setData({ result }); // 只在必要时更新视图
    });
  }
});

// 渲染层：专注渲染（独立线程）
// 用户交互始终流畅响应
```
#### 内存分离式管理
逻辑层内存：
- JavaScript 执行环境
- 业务数据存储
- API 调用缓存

渲染层内存：
- DOM 树结构
- CSS 样式计算
- 渲染缓存
## 通信设计
WebView的本质就是由移动端提供可以内嵌Web应用的组件。目前有两种应用方式：APP中打开H5页面（Hybird）和小程序。无论哪一种方式，都涉及到WebView和Native之间的通信。

### WebView 通信之 JSBridge
### 小程序通信方式