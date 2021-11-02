# JavaScript 异步编程

> **单线程 JavaScript 异步方案**
>
> JS 执行环境中负责执行代码的线程只有一个
>
> JavaScript 将任务的执行模式分成了两种：
>
> - 同步模式（Synchronous）
> - 异步模式（Asynchronous）

### 同步模式

> 同一时间只能执行一个任务

### 异步模式

> 不会去等待这个任务的结束 才开始下一个任务，开启过后就立即往后执行下一个任务，后续逻辑一般会通过回调函数的方式定义。

### 回调函数

> 所有异步编程方案的根基，回调函数可以理解为一件你想要做的事情。
>
> 由调用者定义，交给执行者执行的函数

```javascript
function foo (callback) {
  setTimeout(function () {
    callback()
  }, 3000)
}

foo(function () {
  console.log('这就是一个回调函数')
  console.log('调用者定义这个函数，执行者执行这个函数')
  console.log('其实就是调用者告诉执行者一步任务结束后应该做什么')
})
```

### Promise 概述

> 一种更优的异步编程统一方案。
>
> CommonJS社区提出了Promise的规范，在ES2015中被标准化，成为语言规范

Promise的三种状态：等待态（Pending）、执行态（Fulfilled）、拒绝态（Rejected）。

####Promise 基本用法

```javascript
// Promise 基本用法

const promise = new Promise(function(resolve, reject) {
  // 这里用于“兑现”承诺
  resolve(100)
  // reject(new Error('promise rejected')) // 承诺失败
})

promise.then(function (value) {
  console.log('resolved', value) // resolved 100
}, function (error) {
  console.log('rejected', error)
})

console.log('end')
```

####Promise 使用案例

```javascript
// Promise 方式的 Ajax

function ajax(url) {
  return new Promise(function (resolve, reject) {
    var xhr = new XMLHttpRequest()
    xhr.open('GET', url)
    xhr.responseType = 'json'
    xhr.onload = function () {
      if (this.status === 200) {
        resolve(this.response)
      } else {
        reject(new Error(this.statusText))
      }
    }
    xhr.send()
  })
}

ajax('/api/users.js').then(function (res) {
  console.log(res)
}, function(error) {
  console.log(error)
})
```

####Promise 常见误区

> Promise本质上也是使用回调函数，定义的异步任务结束后所需要执行的任务.
>
> 避免回调地狱：借助于 Promise.then 方法链式调用的特点，尽可能保证异步任务扁平化

####Promise 链式调用

- Promise 对象的then方法会返回一个全新的Promise对象
- 后面的then方法就是在为上一个then方法返回的Promise注册回调
- 前面then方法中回调函数的返回值会作为后面的then方法回调的参数
- 如果回调中返回的是Promise，那后面的then方法的回调回等待它的结束

####Promise 异常处理

> 正确的处理：在代码中明确的捕获每一个可能的异常

```javascript
// Promise 异常处理

const promise = new Promise((resolve, reject) => {
  resolve('123')
})

promise.then((res) => {
  console.log(res)
  foo()
}, (error) => {
  console.log(error) // 没有捕获到异常
})

promise.then((res) => {
  console.log(res)
  foo()
}).catch((error) => {
  console.log(error) // 可以捕获异常 foo is not defined
})
```

####Promise 静态方法

 ```javascript
// 常见的 Promise 静态方法

// 1. Promise.resolve
Promise.resolve('foo').then((value) => {
  console.log(value) // foo
})

const promise = new Promise((resolve, reject) => {})
const promise2 = Promise.resolve(promise)
console.log(promise === promise2) // true

Promise.resolve({
  then: function(onFulfilled, onRejected) {
    onFulfilled('foo')
  }
}).then(function(value) {
  console.log(value) // foo
})

// 2. Promise.reject
Promise.reject('anything').catch((error) => {
  console.log(error) // anything
})
 ```

####Promise 并行执行

```javascript
// Promise 并行执行

function ajax(url) {
  return new Promise(function (resolve, reject) {
    var xhr = new XMLHttpRequest()
    xhr.open('GET', url)
    xhr.responseType = 'json'
    xhr.onload = function () {
      if (this.status === 200) {
        resolve(this.response)
      } else {
        reject(new Error(this.statusText))
      }
    }
    xhr.send()
  })
}

// Promise.all() : 等待所组合所有promise都结束
const promise = Promise.all([
  ajax('/api/users.json'), 
  ajax('/api/users.json')
])

promise.then(function(values) {
  console.log(values)
}).catch(function(error) {
  console.log(error)
})

// Promise.rece() : 只会等待第一个结束的任务
const request = ajax('/api/posts.json')
const timeout = new Promise((resolve, reject) => {
  setTimeout(() => reject(new Error('timeout')), 500)
})

Promise.race([
  request,
  timeout
]).then(value => {
  console.log(value)
}).catch(error => {
  console.log(error)
})
```

####Promise 执行时序

> 微任务：Promise回调、MutationObserver、process.nextTick (node)
>
> 宏任务：I/O、setTimeout、setInterval、setImmediate (node)、requestAnimationFrame (浏览器)

```javascript
// 微任务

console.log('global start')

setTimeout(() => {
  console.log('setTimeout') // 宏任务
}, 0)

Promise.resolve().then(() => {
  console.log('promise')
})
console.log('global end')
// global start
// global end
// promise
// setTimeout
```

### Generator 异步方案

```javascript
// 生成器函数
function * foo() {
  console.log('start') // start
  try {
    const res = yield 'foo'
    console.log(res) // bar
  } catch (e) {
    console.log(e)
  }
}

const generator = foo()

const result = generator.next()
console.log(result) // {value: 'foo', done: false}

generator.next('bar')

generator.throw(new Error('Generator error'))
```

```javascript
// Generator 配合 Promise 的同步方案

function ajax(url) {
  return new Promise(function (resolve, reject) {
    var xhr = new XMLHttpRequest()
    xhr.open('GET', url)
    xhr.responseType = 'json'
    xhr.onload = function () {
      if (this.status === 200) {
        resolve(this.response)
      } else {
        reject(new Error(this.statusText))
      }
    }
    xhr.send()
  })
}

function * main() {
  const users = yield ajax('/api/users.json')
  console.log(users)
  
  const posts = yield ajax('/api/posts.json')
  console.log(posts)
}

const g = main()

const result = g.next()

result.value.then(data => {
  const result2 = g.next(data)
  if(result2.done) return
  
  result2.value.then(data => {
    const result3 = g.next(data)
    if(result2.done) return
    
    result3.value.then(data => {
      g.next(data)
    })
  })
})

// 递归执行 Generator
function co(generator) {
  function handleResult(result) {
  	if (result.done) return
  	return.value.then(data => {
    	handleResult(g.next(data))
  	}, error => {
    	g.next(error)
  	})
	}
	handleResult(g.next())
}
co(main)
```

### Async / Await 语法糖

> 语言层面的异步编程标准

``` javascript
// Async / Await 语法糖

async function main() {
  try {
    const users = await ajax('/api/users.json')
  	console.log(users)
  
  	const posts = await ajax('/api/posts.json')
  	console.log(posts)
  } catch(e) {
    console.log(e)
  }
}
const promise = main()
promise.then(() => {
  console.log('all completed')
})
```

