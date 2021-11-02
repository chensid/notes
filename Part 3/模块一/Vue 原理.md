# Vue 源码（Vue Router）

## Vue Router

### Hash 和 History 模式

原理的区别：

- Hash 模式是基于锚点，以及 onhashchange 事件
- History 模式是基于 HTML5 中的 History API
  - history.pushState() 方法 IE 10以后才支持
  - history.replaceState() 

#### History 模式的使用

- History 需要服务器的支持
- 单页应用中，服务端不存在 http://www.testurl.com/login 这样的地址会返回找不到该页面
- 在服务端应该除了静态资源外都返回单页应用的 index.html

### Vue Router 实现原理

#### Hash 模式

- URL 中 # 后面的内容作为路径地址
- 监听 hashchange 事件
- 根据当前路由地址找到对应组件重新渲染

#### History 模式

- 通过 history.pushState() 方法改变地址栏
- 监听 popstate 事件
- 根据当前路由地址找到对应组件重新渲染

## Vue 响应式原理

### 数据驱动

- 数据响应式、双向绑定、数据驱动
- 数据响应式
  - 数据模型仅仅是普通的 JavaScript 对象，而当我们修改数据时，视图会进行更新，避免了繁琐的 DOM 操作，提高开发效率
- 双向绑定
  - 数据改变，视图改变；视图改变，数据也随之改变
  - 我们可以使用 v-model 在表单元素上创建双向数据绑定
- 数据驱动是 Vue 最独特的特性之一
  - 开发过程中仅需要关注数据本身，不需要关心数据是如何渲染到视图

### 数据响应式的核心原理

#### Vue 2.0

- [Vue 2.x 深入响应式原理](https://cn.vuejs.org/v2/guide/reactivity.html)
- [Object.defineProperty()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
- 浏览器兼容 IE8 以上（不兼容 IE8）

#### Vue 3.0

- [Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
- 直接监听对象，而非属性
- ES 6 中新增，IE 不支持，性能由浏览器优化

### 发布订阅模式和观察者模式

#### 发布 / 订阅模式

- 订阅者
- 发布者
- 信号中心

> 我们假定，存在一个“信号中心”，某个任务执行完成，就向信号中心“发布”（publish）一个信号，其他任务可以向信号中心“订阅”（subscribe）这个信号，从而知道什么时候自己可以开始执行。这就是“发布/订阅模式”。

#### 观察者模式

- 观察者（订阅者）- Watcher
  - update()：当事件发生时，具体要做的事情
- 目标（发布者）- Dep
  - subs数组：存储所有的观察者
  - addSub()：添加观察者
  - notify()：当事件发生，调用所有观察者的update()方法
- 没有事件中心

#### 总结

- 观察者模式是由具体目标调度，比如当事件触发，Dep 就会去调用观察者的方法，所以观察者模式的订阅者与发布者之间是存在依赖的。
- 发布/订阅模式由统一调度中心调用，因此发布者和订阅者不需要知道对方的存在。

### Vue实现

- 功能
  - 负责接受初始化的参数（选项）
  - 负责把 data 中的属性注入到 Vue 实例，转换成 getter / setter
  - 负责调用 observer 监听 data 中所有属性的变化
  - 负责调用 compiler 解析指令 / 插值表达式

#### Observer

- 功能
  - 负责把 data 选项中的属性转化成响应式数据
  - data 中的某个属性也是对象，把该属性转换成响应式数据
  - 数据变化发送通知

#### Compiler

- 功能
  - 负责编译模板，解析指令/ 插值表达式
  - 负责页面的首次渲染
  - 当数据变化后重新渲染视图

## Virtual DOM

### 什么是Virtual DOM

- Virtual DOM ( 虚拟 DOM )，是由普通的 JS 对象来描述 DOM 对象，因为不是真实的 DOM 对象，所以叫 Virtual DOM

- 真实 DOM 成员

- 可以使用 Virtual DOM 来描述真实 DOM，示例

  ```json
  {
    sel: 'div',
    data: {},
    children: undefined,
    text: 'Hello Virtual DOM',
    elm: undefined,
    key: undefined
  }
  ```

### 为什么使用 Virtual DOM

- 手动操作 DOM 比较麻烦，还需要考虑浏览器兼容问题，虽然有 jQurey 等库简化 DOM 操作，但是随着项目的复杂 DOM 操作复杂提升
- 为了简化 DOM 的复杂操作，于是出现了各种 MVVM 框架，MVVM 框架解决了视图和状态的同步问题
- 为了简化视图的操作我们可以使用模板引擎，但是模板引擎没有解决跟踪状态变化的问题，于是 Virtual DOM 出现了
- Virtual DOM 的好处是当状态改变时不需要立即更新 DOM，只需要创建一个虚拟树来描述 DOM，Virtual DOM 内部将弄清楚如何有效（diff）的更新 DOM
- 参考 github 上 [virtual-dom](https://github.com/Matt-Esch/virtual-dom) 的描述
  - 虚拟 DOM 可以维护程序的状态，跟踪上一次的状态
  - 通过比较前后两次状态的差异更新真实 DOM

### 虚拟 DOM 的作用

- 维护视图和状态的关系
- 复杂视图情况下提升渲染性能
- 除了渲染 DOM 以外，还可以实现 SSR (Nuxt.js / Next.js)、原生应用 (Weex / React Native)、小程序(movie / uni-app) 等

### Snabbdom

#### 创建项目

- 打包工具为乐方便使用 parcel

- 创建项目，并安装 parcel

  ```bash
  # 创建项目目录
  mkdir snabbdom-demo
  # 进入项目目录
  cd snabbdom-demo
  # 创建package.json
  yarn init -y
  # 本地安装 parcel
  yarn add parcel-bundler
  ```

- 配置 package.json 的 scripts

  ```json
  "scripts": {
    "dev": "parcel index.html --open",
    "build": "parcel build index.html"
  }
  ```

- 创建目录结构

  ```json
  | index.html
  |	package.json
  |_src
  		| 01-basicusage.js
  ```

#### 安装 Snabbdom

```bash
$ yarn add snabbdom
```

#### 导入 Snabbdom

- Snabbdom 的官网 demo 中导入使用的是 commonjs 模块化瘀法，我们使用更流行的 ES Module 语法

  ```js
  import { init, h, thunk } from 'snabbdom'
  ```

- Snabbdom 的核心仅提供最基本的功能

  - init() 是一个高阶函数，返回 patch()
  - h() 返回虚拟节点 VNode，这个函数我们在使用 Vue.js 的时候见过
  - thunk() 是一种优化策略，可以在处理不可变数据时使用

#### 模块

Snabbdom 的核心库并不能处理元素的属性 / 样式 / 事件等，如果需要处理的话，可以使用模块。

**常用模块（官方提供了6个模块）**

- attributes
  - 设置 DOM 元素的属性，使用`setAtribute()`
  - 处理布尔类型的属性
- props
  - 和`attributes`模块相似，设置 DOM 元素的属性`element[attr] = value`
  - 不处理布尔类型的属性
- class
  - 切换类样式
  - 注意：给元素设置类样式是通过`sel`选择器
- dataset
  - 设置`data-*`的自定义属性
- eventlisteners
  - 注册和移除事件
- style
  - 设置行内样式，支持动画
  - delayed / remove / destroy

#### 模块使用

模块使用步骤：

- 导入需要的模块
- init() 中注册模块
- 使用 h() 函数创建 VNode 的时候，可以把第二个参数设置为对象，其他参数往后移

### Snabbdom源码解析

#### Snabbdom的核心

- 使用 h() 函数创建 JavaScript 对象 (VNode) 描述真实 DOM
- init() 设置模块，创建 patch()
- patch() 比较新旧两个 VNode
- 把变化的内容更新到真实 DOM 树上

#### Snabbdom源码

> 源码地址：https://github.com/snabbdom/snabbdom

##### h函数

- h() 函数
  - h() 函数最早见于 [hyperscript](https://github.com/hyperhype/hyperscript)，使用 JavaScript 创建超文本
  - Snabbdom 中的 h() 函数不是用来创建超文本，而是创建 VNode
- 函数重载
  - 概念
    - 参数**个数**或**类型**不同的函数
    - JavaScript 中没有重载的概念
    - TypeScript 中有重载，不过重载的实现还是通过代码调整参数

##### init函数

- 返回 patch 函数

##### patch函数

- 创建真实 DOM，返回vnode

