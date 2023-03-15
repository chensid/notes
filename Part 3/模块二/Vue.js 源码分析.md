# Vue.js 源码分析

> 目标：
>
> - Vue.js 的静态成员和实例成员初始化过程
> - 首次渲染的过程
> - 数据响应式原理

## Vue 源码

- Vue 2.6 项目地址：https://github.com/vuejs/vue
- Fork 仓库，克隆本地，可以提交自己的注释
- 3.0 项目地址：https://github.com/vuejs/vue-next

### 源码的目录结构

```
src
├── compiler
├── core
├── platforms
├── server
├── sfc
└── shared
```

### 了解Flow

- 官网：
- JavaScript 的**静态类型检查器**
- Flow 的静态类型检查错误是通过静态类型推断实现的
  - 文件开头通过 **// @flow**  或者 **/* @flow */** 声明

```js
/* @flow */
function square(n: number): number {
  return n * n;
}
square("2"); // Error!
```

### 调试设置

#### 打包

- 打包工具Rollup
  - Vue.js 源码的打包工具使用的是 Rollup，比 Webpack 轻量
  - Webpack 把所有文件当作模块，Rollup只处理js文件更适合在 Vue.js 这样的额库中使用
  - Rolluo 打包不会生成冗余的代码
- 安装依赖

```bash
npm i rollup
```

- 设置 sourcemap

  - package.json文件中的dev脚本中添加参数 --sourcemap


```bash
"dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev"
```

- 执行 dev
  - **npm run dev** 执行打包，用的是 rollup，-w 参数是监听文件的变化，文件变化自动重新打包

### Vue 的不同构建版本

- **npm run build** 重新打包所有文件
- [官方文档-对不同构建版本的解释](https://cn.vuejs.org/v2/guide/installation.html#对不同构建版本的解释)
- dist\README.md

|                               | UMD                | CommonJS              | ES Module (基于构建工具使用) | ES Module (直接用于浏览器) |
| :---------------------------- | :----------------- | :-------------------- | :--------------------------- | :------------------------- |
| **Full**                      | vue.js             | vue.common.js         | vue.esm.js                   | vue.esm.browser.js         |
| **Runtime-only**              | vue.runtime.js     | vue.runtime.common.js | vue.runtime.esm.js           | -                          |
| **Full(production)**          | vue.min.js         | -                     | -                            | vue.esm.browser.min.js     |
| **Runtime-only (production)** | vue.runtime.min.js | -                     | -                            | -                          |

#### 术语

- **完整版**：同时包含编译器和运行时的版本。
- **编译器**：用来将模板字符串编译成为 JavaScript 渲染函数的代码。
- **运行时**：用来创建 Vue 实例、渲染并处理虚拟 DOM 等的代码。基本上就是除去编译器的其它一切。
- **[UMD](https://github.com/umdjs/umd)**：UMD 版本可以通过 `<script>` 标签直接用在浏览器中。jsDelivr CDN 的 https://cdn.jsdelivr.net/npm/vue 默认文件就是运行时 + 编译器的 UMD 版本 (`vue.js`)。
- **[CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1)**：CommonJS 版本用来配合老的打包工具比如 [Browserify](http://browserify.org/) 或 [webpack 1](https://webpack.github.io/)。这些打包工具的默认文件 (`pkg.main`) 是只包含运行时的 CommonJS 版本 (`vue.runtime.common.js`)。
- **[ES Module](http://exploringjs.com/es6/ch_modules.html)**：从 2.6 开始 Vue 会提供两个 ES Modules (ESM) 构建文件：
  - 为打包工具提供的 ESM：为诸如 [webpack 2](https://webpack.js.org/) 或 [Rollup](https://rollupjs.org/) 提供的现代打包工具。ESM 格式被设计为可以被静态分析，所以打包工具可以利用这一点来进行“tree-shaking”并将用不到的代码排除出最终的包。为这些打包工具提供的默认文件 (`pkg.module`) 是只有运行时的 ES Module 构建 (`vue.runtime.esm.js`)。
  - 为浏览器提供的 ESM (2.6+)：用于在现代浏览器中通过 `<script type="module">` 直接导入。

### 查找入口文件

- 查看 dist/vue.js 的构建过程

#### 执行构建

```bash
npm run dev
# "dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev"
# --environment TARGET:web-full-dev 设置环境变量 TARGET
```

- script/config.js 的执行过程
  - 作用：生成 rollup 构建的配置文件
  - 使用环境变量 TARGET = web-full-dev

```js
// 判断环境变量是否有 TARGET
// 如果有的话 使用 genConfig() 生成 rollup 配置文件
if (process.env.TARGET) {
  module.exports = genConfig(process.env.TARGET)
} else {
  // 否则获取全部配置
  exports.getBuild = genConfig
  exports.getAllBuilds = () => Object.keys(builds).map(genConfig)
}
```

- genConﬁg(name)
  - 根据环境变量 TARGET 获取配置信息
  - builds[name] 获取生成配置的信息

```js
// Runtime+compiler development build (Browser)
'web-full-dev': { 
  entry: resolve('web/entry-runtime-with-compiler.js'),
  dest: resolve('dist/vue.js'),
  format: 'umd',
  env: 'development',
  alias: { he: './entity-decoder' },
  banner
},
```

- resolve()
  - 获取入口和出口文件的绝对路径

```js
const aliases = require('./alias')
const resolve = p => {
	// 根据路径中的前半部分去alias中找别名
  const base = p.split('/')[0]
  if (aliases[base]) {
    return path.resolve(aliases[base], p.slice(base.length + 1)) 
  } else {
    return path.resolve(__dirname, '../', p) 
  }
}
```

#### 结果

- 把 src/platforms/web/entry-runtime-with-compiler.js 构建成 dist/vue.js，如果设置 -sourcemap 会生成 vue.js.map
- src/platform 文件夹下是 Vue 可以构建成不同平台下使用的库，目前有 weex 和 web，还有服务器端渲染的库

### 从入口开始

- src/platform/web/entry-runtime-with-compiler.js

#### 通过查看源代码解决下面问题

- 观察以下代码，通过阅读源码，判断在页面上输出的结果

```js
const vm = new Vue({
  el: '#app',
  template: '<h3>Hello template</h3>',
  render(h) {
    return h('h4', 'Hello render')
  }
})
```

- 阅读源码
  - el 不能是 body 或者 html 标签
  - 如果没有 render，把 template 转换成 render 函数
  - 如果有 render 方法，直接调用 mount 挂载 DOM

```js
// el 不能是 body 或者 html 标签
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }

  const options = this.$options
  
  // 把 template/el 转换成 render 函数
  if (!options.render) {
    // ...
  }
  // 调用 mount 方法，挂载 DOM
  return mount.call(this, el, hydrating)
```

### Vue 的构造函数

- src/platform/web/entry-runtime-with-compiler.js 中引用了 './runtime/index'
- src/platform/web/runtime/index.js
  - 设置 Vue.conﬁg
  - 设置平台相关的指令和组件
    - 指令 v-model、v-show
    - 组件 transition、transition-group
  - 设置平台相关的 __patch__ 方法（打补丁方法，对比新旧的 VNode）
  - 设置 $mount 方法，挂载 DOM

```js
// install platform runtime directives & components
extend(Vue.options.directives, platformDirectives)
extend(Vue.options.components, platformComponents)

// install platform patch function
Vue.prototype.__patch__ = inBrowser ? patch : noop

// public mount method
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```

- src/platform/web/runtime/index.js 中引用了 'core/index'
- src/core/index.js
  - 定义了 Vue 的静态方法
  - initGlobalAPI(Vue)
- src/core/index.js 中引用了 './instance/index'
- src/core/instance/index.js
  - 定义了 Vue 的构造函数

```js
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)
```

