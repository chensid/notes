# ECMAScript新特性

ES标准的意义

- 解决原有语法上的一些问题或者不足（let、const）
- 对原有语法进行增强
- 全新的对象、全新的方法、全新的功能（promise、async/await）
- 全新的数据类型和数据结构（symbol、set）

## ECMAScript 2015

### let和块级作用域

> 块级作用域在外部是不可以访问的，没有变量提升

```javascript
for (let i = 0; i < 3; i ++) {
	let i = 'foo'
	console.log(i) // foo
}
```

### const

> 常量：声明过后不允许再被修改（不允许修改内存地址），需要初始值，也有块级作用域

### 数组的解构

```	javascript
// 数组的解构
const arr = [100, 200, 300]
const [foo, bar, baz] = arr
console.log(baz) // 300
// const [, , baz] = arr
// const [foo, ...rest] = arr
// console.log(rest) // [200, 300]
// const [foo, bar, baz, more = 'default value'] = arr
// console.log(more) // default value
```

### 对象的解构

```javascript
// 对象的解构
const obj = { name: 'cs', age: 18}
// const { name } = obj
// console.log(name) // cs
// 别名
const name = 'tom'
const { name: objName = 'default' } = obj
console.log(objName) // cs

const { log } = console
log('log')	// log
```

### 模版字符串

```javascript
// 模版字符串
const name = 'tom'
const str = `hey, ${name}`
console.log(str)	// hey, tom
```

### 带标签的模版字符串

```javascript
// 带标签的模版字符串
// const str = console.log`hello world`	// ['hello world']
const name = 'tom'
const gender = true
function myTagFunc(strings, name, gender){
  // console.log(strings)	// ['hey, ', ' is a ', '.']
  console.log(name, gender) // tom true
  const sex = gender ? 'man' : 'womon'
  return strings[0] + name + strings[1] + sex + strings[2]
}
const result = myTagFunc`hey, ${name} is a ${gender}.`
```

### 字符串的扩展方法

常用的几个方法

- includes()
- startsWith()
- endsWith()

``` javascript
// 字符串的扩展方法
const message = 'Error: foo is not defined.'
console.log(
  // message.startsWith('Error') //true
  // message.endsWith('.') //true
  message.includes('foo') //true
)
```

### 参数默认值

``` javascript
// 函数参数的默认值
function foo(enable = true) {
  console.log(enable)
}
foo() // true
```

### 剩余参数

```javascript
// 剩余参数
function foo(...args) {
  	console.log(args)
}
foo(1, 2, 3, 4) // [1, 2, 3, 4]
```

### 展开数组

``` javascript
// 展开数组
const arr = ['foo', 'bar', 'baz']
console.log(...arr) // foo bar baz
```

### 箭头函数

```javascript
// 箭头函数
const inc = n => n + 1
console.log(inc(100)) // 101
```

### 箭头函数与this

> 箭头函数不会改变this的指向

```javascript
const person = {
  name: 'tom',
  sayHi: () => {
    console.log(`hi, my name is ${this.name}`) // hi, my name is undefined
  },
  sayHiAsync: function () {
    setTimeout(() => {
      console.log(this.name) // tom
    }, 1000)
  }
}
person.sayHiAsync()
```

### 对象字面量增强

``` javascript
// 对象字面量
const bar = '123'
const obj = {
  foo: 123,
  bar,
  func() {
    console.log(this)
  },
  [1 + 2]: 3 // 计算属性名
}
console.log(obj) // {foo: 123, bar: "123", func: ƒ}
obj.func()
```

### 对象扩展方法

- Object.assign(): 将多个原对象中的属性复制到一个目标对象中

  ```javascript
  // Object.assign()
  
  const obj = {
    a: 123,
    b: 123
  }
  const target = {
    a: 234,
    c: 234
  }
  const result = Object.assign(target, obj)
  console.log(target) // { a: 123, c: 234, b: 123 }
  console.log(result === target) // true
  ```

- Object.is()

  ```javascript
  // Object.is
  
  console.log(Object.is(NaN, NaN)) // true
  ```

  ###Proxy

- Proxy 基本用法

  ```javascript
  // Proxy 对象
  
  // 语法: const p = new Proxy(target, handler)
  
  const person = {
    name: 'zce',
    age: 20
  }
  const personProxy = new Proxy(person, {
    get(target, property) {
      //console.log(target, property) // {name: "zce", age: 20} "name"
      return property in target ? target[property] : 'default'
    },
    set(target, property, value) {
      if (property === 'age') {
        if (!Number.isInteger(value)) {
          throw new TypeError(`${value} is not a Int`)
        }
      }
      target[property] = value
    }
  })
  personProxy.age = 100
  console.log(personProxy.name)
  ```

- Proxy 对比 Object.defineProperty()

  1. Proxy能够监视到更多对象操作

  ```javascript
  // Proxy 对比 Object.defineProperty()
  
  const person = {
    name: 'zce',
    age: 20
  }
  
  const personProxy = new Proxy(person, {
    deleteProperty(target, property) {
      console.log('delete', property)
      delete target[property]
    }
  })
  
  delete personProxy.age
  console.log(person) // { name: 'zce' }
  ```

  | handler方法                | 触发方式                                                     |
  | -------------------------- | ------------------------------------------------------------ |
  | get()                      | 读取某个属性                                                 |
  | set()                      | 写入某个属性                                                 |
  | has()                      | in 操作符                                                    |
  | deleteProperty             | delete 操作符                                                |
  | getPrototypeOf()           | Object.getPrototypeOf()                                      |
  | setPrototypeOf()           | Object.setPrototypeOf()                                      |
  | isExtensible()             | Object.isExtensible()                                        |
  | preventExtensions()        | Object.preventExtensions()                                   |
  | getOwnPropertyDescriptor() | Object.getOwnPropertyDescriptor()                            |
  | defineProperty()           | Object.defineProperty()                                      |
  | ownKeys()                  | Object.getOwnPropertyNames() 、 Object.getOwnPropertySymbols() |
  | apply()                    | 调用一个函数                                                 |
  | construct()                | 用new调用一个函数                                            |
  2. Proxy 更好的支持数组对象的监视
  
     > 一般情况，重写数组的操作方法
  
     ```javascript
     // Proxy 监视数组操作
     
     const list = []
     
     const listProxy = new Proxy(list, {
       set(target, property, value) {
         console.log(target, property, value) // [] '0' 100
         target[property] = value
         return true
       }
     })
     
     listProxy.push(100)
     ```
  
  3. Proxy 是以非侵入的方式监管了对象的读写
  
  ###Reflect
  
  > Reflect属于一个静态类
  >
  > Reflect内部封装了一系列对对象的底层操作
  >
  > 统一提供一套用于操作对象的API
  
  ```javascript
  // Reflect 对象
  
  const obj = {
    foo: '123',
    bar: '456'
  }
  
  const proxy = new Proxy(obj, {
    get(target, property) {
      console.log('watch logic~')
      return Reflect.get(target, property)
    }
  })
  
  console.log(proxy.foo) // '123'
  
  // 对象操作
  const obj = {
    name: 'zce',
    age: 18
  }
  // console.log('name' in obj) // true
  // console.log(delete obj['age']) // true
  // console.log(Object.keys(obj)) // ['name']
  
  console.log(Reflect.has(obj, 'name')) // true
  console.log(Reflect.deleteProperty(obj, 'age') // true
  console.log(Reflect.ownkeys(obj)) // ['name']
  ```

###Promise

> 一种更优的异步编程解决方案，解决了传统异步编程中回调函数嵌套过深的问题

###class 关键字

```javascript
// class 关键字

class Person {
  constructor(name) {
    this.name = name
  }
  
  say() {
    console.log(`hi, my name is ${this.name}.`)
  }
}
const person = new Person('tom')
person.say() // hi, my name is tom.
```

###静态方法

> ES2015 中新增添加静态成员的static关键词

```javascript
// static 关键字

class Person {
  constructor(name) {
    this.name = name
  }
  
  say() {
    console.log(`hi, my name is ${this.name}.`)
  }
  
  static create(name) {
    return new Person(name)
  }
}
const person = Person.create('tom')
person.say() // hi, my name is tom.
```

### extends 类的继承

```javascript
// extends 继承

class Person {
  constructor(name) {
    this.name = name
  }
  
  say() {
    console.log(`hi, my name is ${this.name}.`)
  }
}

class Student extends Person {
  constructor(name, number) {
    super(name)
    this.number = number
  }
  hello() {
    super.say()
    console.log(`my school number is ${this.number}.`)
  }
}
const student = new Student('jack', '100')
student.hello() // my school number is 100.
```

###Set 数据结构

```javascript
// Set 数据结构

const arr = [1, 2, 1, 3, 4, 5, 2]

const result = Array.from(new Set(arr))
const result = [...new Set(arr)]
console.log(result) // [1, 2, 3, 4, 5]
```

###Map 数据结构

```javascript
// Map 数据结构 : 可以使用任意类型的值作为键

const map = new Map()

const tom = { name: 'tom'}

map.set(tom, 90)

console.log(map) // {{ name: 'tom'} => 90}
```

###Symbol 一种全新的原始数据类型

``` javascript
// Symbol 数据类型
// 最主要的作用就是为对象添加独一无二的属性名
const s = Symbol()
console.log(s) // symbol

console.log(Symbol() === Symbol()) // false
```

> Symbol 补充

```javascript
// Symbol 补充

console.log(Symbol('foo') === Symbol('foo')) // false

console.log(Symbol.for('foo') === Symbol.for('foo')) // true

const obj = {
  [Symbol.toStringTag]: 'XObject'  // 内置Symbol常量
}
console.log(obj.toString()) // [object XObject]

const object = {
  [Symbol()]: 'symbol value',
  foo: 'normal value'
}
console.log(Object.getOwnPropertySymbols(object)) // [Symbol()]
```

###ES2015  for...of 循环

> 作为遍历所有数据结构的统一方式

```javascript
// for ... of 循环

const arr = [100, 200, 300, 400]

for (const item of arr) {
  console.log(item)
  if (item > 100) {
    break
  }
}
// arr.forEach() // 不能跳出循环

const m = new Map()
m.set('foo', 123)
m.set('bar', 345)
for (const [key, value] of m) {
  console.log(key, value) // foo 123   bar 345
}
```

###可迭代接口

> ES2015 提供了Iterable接口，`实现Iterable接口就是 for ... of 的前提`

```javascript
// [Iterator] 迭代器

const set = new Set(['foo', 'bar', 'baz'])

const iterator = set[Symbol.iterator]()

console.log(iterator.next()) // { value: 'foo', done: false }
console.log(iterator.next()) // { value: 'bar', done: false }
console.log(iterator.next()) // { value: 'baz', done: false }
console.log(iterator.next()) // { value: undefined, done: true }
```

####实现可迭代接口

```javascript
// 实现可迭代接口 [iterable]

const obj = {
  store: ['foo', 'bar', 'baz'],
  [Symbol.iterator]: function() {
    let index = 0
    const self = this
    return {
      next: function() {
        const result = {
          value: self.store[index],
          done: index >= self.store.length
        }
        index++
        return result
      }
    }
  }
}
for (const item of obj) {
  console.log(item) // foo  bar  baz
}
```

####迭代器模式

```javascript
// 迭代器模式

// 场景：你我共同开发一个任务清单应用
// 我的代码 ====
const todos = {
  life: ['吃饭', '睡觉', '打豆豆'],
  learn: ['语文', '数学', '外语'],
  work: ['办公'],
  each: function(callback) {
    const all = [].concat(this.life, this.learn, this.work)
    for (const item of all) {
      callback(item)
    }
  },
  [Symbol.iterator]: function() {
    const all = [...this.life, ...this.learn, ...this.work]
    let index = 0
    return {
      next: function() {
        return {
          value: all[index],
          done: index++ >= all.length
        }
      }
    }
  }
}
// 你的代码 ====
todos.each(function (item) {
  console.log(item)
})
for (const item of todos) {
  console.log(item)
}
```

###生成器

> 避免异步编程中回调嵌套过深，提供更好的异步编程解决方案

``` javascript
// Generator 函数

function * foo () {
  console.log('zce') // zce
  return 100
}

const result = foo()
console.log(result.next()) // { value: 100, done: true }

function * func () {
  console.log('111') // 111
  yield 100
  console.log('222') // 222
  yield 200
  console.log('333') // 333
  yield 300
}

const generator = func()

console.log(generator.next()) // {value: 100, done: false}
console.log(generator.next()) // {value: 200, done: false}
console.log(generator.next()) // {value: 300, done: false}
console.log(generator.next()) // {value: undefined, done: true}
```

####生成器应用

```javascript
// Generator 应用

// 案例1:发号器
function * createIdMaker() {
  let id = 1
  while (true) {
    yield id ++
  }
}
const idMaker = createIdMaker()

console.log(idMaker.next().value) // 1
console.log(idMaker.next().value) // 2
console.log(idMaker.next().value) // 3
console.log(idMaker.next().value) // 4

// 案例2:使用 Generator 函数实现 iterator 方法

const todos = {
  life: ['吃饭', '睡觉', '打豆豆'],
  learn: ['语文', '数学', '外语'],
  work: ['办公'],
  [Symbol.iterator]: function * () {
    const all = [...this.life, ...this.learn, ...this.work]
    for (const item of all) {
      yield item
    }
  }
}

for (const item of todos) {
  console.log(item)
}
```

###ES Modules

> 语言层面的模块化标准（后面模块化开发中详细介绍）

##ECMAScript 2016

```javascript
// ECMAScript 2016

// Array.prototype.includes
const arr = ['foo', 1, NaN, false]
console.log(arr.includes(NaN)) // true

// 指数运算符
console.log(2 ** 10) // 1024
```

## ECMAScript 2017



```javascript
// ECMAScript 2017

const obj = {
  foo: 'value',
  bar: 'value2'
}

// Object.values
console.log(Object.values(obj)) // ['value', 'value2']

// Object.entries
console.log(Object.entries(obj)) // [["foo", "value"], ["bar", "value2"]]
for (const [key, value] of Object.entries(obj)) {
  console.log(key, value) // foo value   bar value2
}

// Object.getOwnPropertyDescriptors
const p1 = {
  firstName: 'Lei',
  lastName: 'Wang',
  get fullName() {
    return this.firstName + ' ' + this.lastName
  }
}
console.log(p1.fullName) // Lei Wang
const p2 = Object.assign({}, p1)
p2.firstName = 'zce'
console.log(p2.fullName) // Lei Wang
const descriptors = Object.getOwnPropertyDescriptors(p1)
const p3 = Object.defineProperties({}, descriptors)
p3.firstName = 'zxc'
console.log(p3.fullName) // zxc Wang

// String.prototype.padStart / String.prototype.padEnd
const books = {
  html: 5,
  css: 16,
  javascript: 128
}
for (const [name, count] of Object.entries(books)) {
  console.log(`${name.padEnd(16, '-')}|${count.toString().padStart(3, '0')}`)
}

// 在函数中添加尾逗号
const arr = [100, 200, 300,]

// Async / Await 异步编程中详细说明
```

