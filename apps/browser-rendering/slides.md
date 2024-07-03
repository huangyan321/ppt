---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides, markdown enabled
title: 前端优化原理
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply any unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
colorSchema: dark
---

# 前端优化原理

<div class="text-6" style="mix-blend-mode: difference">chrome浏览器运行机制</div>
<div class="text-5 mt-lg" style="mix-blend-mode: difference">@huangyan321</div>

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
     <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
transition: fade-out
layout: two-cols
layoutClass: gap-16
---

# 大纲

<div class="mt-10"></div>

- 浏览器的发展历史

- 浏览器的多进程架构

- 浏览器的渲染原理

- JS引擎的运行原理

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

<!--
Here is another comment.
-->

---
transition: slide-up
level: 2
---

# 一、浏览器的发展简史

<img src="/user-agent.png" class="rounded shadow" />

---
layoutClass: gap-16
---

- 1990年：Tim Berners-Lee Nexus 不支持图片
- 1993年：NCSA Mosaic 支持图片，user-agent诞生 NCSA_Mosaic/2.0 (win 3.1)
- 1994年：Netscape Mozilla -> Netscape Navigator 1.0 (win 3.1)
- 1995年： Microsoft 推出IE 1.0 （Internet Explorer Mozilla/1.22），此年 **JavaScript 诞生**
- 1997年：IE 4.0 vs Netscape 4.0 Internet Explorer 与 Windows 操作系统捆绑发行，此后四年内，IE 获得了 75% 的市场份额
- 1998年：Netscape Navigator 与 Mozilla 项目分家
- 1999年：Internet Explorer 市场份额达到99%
- 2002年：Netscape 为反垄断 开放源代码，非营利性质的 Mozilla 诞生
- 2003年： Safari 诞生 Webkit内核
- 2004年：Mozilla Firefox 1.0 诞生
- 2008年：Google Chrome 诞生

世界历史从不缺少史诗般的权力斗争，有征服世界的暴君，也有落败的勇士。Web 浏览器的历史也大抵如此。学术先驱们编写出引发信息革命的简易软件，并为浏览器的优势和互联网用户而战。
---
layout: two-cols
layoutClass: gap-16
---
# 二、浏览器的多进程架构

<div v-click flex="~" gap-10 >
<img src="/process1.jpg" class="w-90%  rounded shadow" />
<img src="/process2.jpg" class="w-90% rounded shadow" />
</div>

---

# 1. 区分进程和线程？

- 系统给进程分配独立的内存空间

- 进程之间相互独立

- <div text-red>一个进程由一个或多个线程组成</div>

- 多个线程在进程中协助完成任务

- 同一进程下的各个线程之间共享程序的内存空间（包括代码段、数据集、堆等）

> Tips
> 进程是CPU资源**分配**的最小单位(能拥有资源和独立运行的最小单位)
> 线程是CPU**调度**的最小单位（线程是建立在进程的基础上的运行单位，一个进程可以有多个线程）
> 一般通用叫法的单线程和多线程，都是指的进程内的线程数量是单个还是多个

---

# 2. 浏览器包含哪些类型的进程？

- Browser 进程
  - 负责浏览器的界面显示，与用户交互，如前进后退按钮等
  - 负责各个页面的管理，创建和销毁其他进程
  - 网络资源的管理，下载等

- Renderer进程：浏览器内核（webkit、blink）渲染进程
  - 负责页面的渲染，JS执行，事件处理等
  - 每个tab页一个进程，互不影响

- GPU进程：负责图形绘制

- 插件进程：仅当使用插件时才创建

---

# 2. 浏览器多进程的特点？

## 优点

- 避免单个page crash（tab、插件等）影响整个浏览器
- 充分利用多核优势
- 更为安全，在系统层面界定了不同进程的权限

## 缺点

- 内存消耗比较大，不同进程之间的内存不共享，不同进程的内存需要包含浏览器内核的多份副本

---

# Chrome浏览器的多进程架构

<img src="/percess_manage.png" class="m-auto w-60%  rounded shadow" />

为了节省内存，Chrome限制了最多的进程数，最大进程数量由设备内存和CPU能力决定，当达到这一限制时，Chrome会将共用之前同一个站点的渲染进程。


---

# 渲染进程（浏览器内核）

- GUI渲染线程（主线程、工作线程、排版线程、光栅线程、合成器线程等）
  负责渲染页面、布局和绘制
  页面需要回流或重绘时，该线程就会执行

- JS引擎线程

  负责解析和执行JavaScript脚本程序
  只有一个JavaScript引擎线程，也就是说JavaScript是单线程执行的

- 事件触发线程
  用来控制事件循环（鼠标点击等）
  当事件满足触发条件时，将事件放入到JS引擎的执行队列中

- 定时器触发线程
  用来控制定时器，如setTimeout、setInterval所在的线程
  定时器任务不是由JS引擎计时的，而是由定时器触发线程来计时的
  当定时器计时完毕后，通知事件触发线程

- 异步http请求线程
  浏览器有一个单独的线程来处理Ajax请求
  当请求完成时，若有回调函数，通知事件触发线程

---

思考两个问题：
1. 为什么Javascript执行是单线程的？
2. 为什么GUI线程与JS引擎线程互斥？

创建js这门语言的时候，多进程多线程的架构不流行，硬件的支持不好，而且多线程操作时需要加锁，编码的复杂性会增加。

js能操作DOM，如果JS和GUI两个线程同时操作DOM，那么渲染可能会导致不可预测的结果，所以一开始的设定就是互斥的关系。


---

# 三、浏览器渲染流程

1. 解析和构建DOM Tree (DOM树到屏幕图形的转化原理，其本质就是树结构到层结构的演化)
2. 解析CSS构建CSSOM Tree，合成Render Tree，同一z轴的元素合并为一个渲染对象
   - 根元素document
   - 具有明确定位属性（relative、fixed、sticky、absolute）
   - CSS filter属性
   - CSS transform属性
   - overflow 不为 visible
   - etc...


---

# 三、浏览器渲染流程

3. 布局（Layout）计算每个节点在屏幕中的位置，生成布局树
4. 合成层： 图形层 Graphics Layer（第二个层模型），满足下列条件把渲染层提升为一个合成层
   - 3D transform
   - 对子元素使用了`will-change: transform`属性的元素
   - 使用加速视频解码的`<video>`元素，拥有3D（WebGL）上下文或加速的2D上下文的`<canvas>`元素
   - 对opacity、transform、filter属性使用了animation或transition的元素

---

# 三、浏览器渲染流程

4. 绘制：合成层的绘制，生成每一个合成层的绘制记录
5. 栅格化： 将合成层分为图块，每个图块转化为合成帧（位图），作为纹理通过GPU进程存储在GPU的显存中。
6. 合成与显示： 合成帧随后会通过IPC协议将消息传递给浏览器主进程。浏览器收到消息后，会将页面内容绘制到内存中。最后再将内存中的内容显示在屏幕上。

---

# Q1： DOMContentLoaded 和 Load 事件的区别？
<div></div>
DOMContentLoaded：仅当DOM加载完成，不包括样式表、图片等外部资源加载完成；  

Load：当所有资源加载完成，包括样式表、图片等外部资源加载完成；


<img src="/load.png" class="m-auto w-60%  rounded shadow" />

---

# Q2： CSS加载是否会阻塞页面加载？

<div></div>

结论：

1. 不会阻塞DOM树的解析（因此js中能获取dom元素，虽然页面还未渲染）
2. 会阻塞render树渲染（浏览器渲染到页面时需等待css加载完毕）
3. 会阻塞后面的js执行（js执行时可能会操作dom元素，如果css未加载完毕，dom元素可能未加载完毕）

---

# Q3： JS的defer和async的区别？

<div></div>

绿色线代表HTML解析过程  
蓝色线代表网络读取  
红色线代表执行时间  

1. 网络读取都是异步的，不会阻塞HTML解析。
2. 下载后的执行时机不一样。
3. defer是顺序的。


---

# JS的运行原理

<img src="/js_cover.png" class="m-auto mt-10 w-40%  rounded shadow" />


---

# 1. JS是单线程的

单线程的优点：
1. 简单、安全
2. 无锁问题
3. 节省内存

---
layout: two-cols
layoutClass: gap-16
---

# 2. JS是非阻塞的

实现机制： 事件循环机制（event loop）

（1）所有同步任务都在主线程上执行，形成一个执行栈（execution context stack）。

（2）主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。

（3）一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。

（4）主线程不断重复上面的第三步。

[JavaScript 运行机制详解：再谈Event Loop —— 阮一峰](https://www.ruanyifeng.com/blog/2014/10/event-loop.html)
:: right ::

<img src="/event_loop.png" class="m-auto mt-10  rounded shadow" />

---

# 同步任务和异步任务

<img src="/event_loop.png" class="m-auto mt-10  rounded shadow" />

1. ajax 进入EventTable并注册回调函数success
2. 执行栈执行console.log('start')
3. ajax事件完成，回调函数进入Callback Queue
4. 主线程空闲时从Callback Queue读取回调函数success执行



想下setTimeOut(fn,0)的执行过程？
主线程执行完同步任务后，会立即执行setTimeout的回调函数吗？
答案是不会，setTimeout的回调函数会进入任务队列，等待主线程空闲时才会执行。

---

# 宏任务与微任务

<div></div>

除了同步任务异步任务之分，任务还可以细分为宏任务与微任务：

macro-task（宏任务）：包括整体脚本代码script、setTimeout、setInterval
micro-task（微任务）：Promise、process.nextTick、MutationObserver等

---
layout: two-cols
layoutClass: gap-16
---

# 宏任务与微任务

<img src="/tasks.jpg" class="m-auto mt-10  rounded shadow" />

:: right ::

进入整体代码（宏任务）后，开始第一次循环。接着执行所有的微任务，当微任务中发起了新的微任务，会继续执行微任务，直到微任务队列为空，浏览器端在微任务执行完成之后会进行一次UI的渲染。接着执行宏任务队列中的下一个任务，然后再次执行所有的微任务，如此循环。


---
layout: center
class: text-center
---

# 谢谢


