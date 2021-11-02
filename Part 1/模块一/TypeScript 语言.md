# TypeScript 语言

> 解决 JavaScript 类型系统的问题，大大提高代码的可靠程度。

### 强类型与弱类型

> **`类型安全`**：
>
> - 强类型：语言层面限制函数的实参必须与形参类型相同
> - 弱类型：语言层面不会限制实参的类型
>
> 强类型有更强的类型的约束，而弱类型中几乎没有什么约束
>
> 强类型语言中不允许任意的隐式类型转换，而在弱类型语言中则允许任意的数据隐式类型转换

### 静态类型与动态类型

> **`类型检查：`**
>
> - 静态类型：一个变量声明时它的类型就是明确的，声明过后，它的类型就不允许再修改
> - 动态类型：运行阶段才能够明确变量类型而且变量的类型随时可以改变
>
> 动态类型语言中的变量没有类型，变量中存放的值是有类型的。

### javaScript 类型系统的特征

> 弱类型 / 动态类型
>
> 灵活多变，但缺失了类型系统的可靠性
>
> JavaScript没有编译环节

### 弱类型的问题

```javascript
// JavaScript 弱类型的问题

// 1. 程序中的一些异常需要等到运行时才能够发现
const obj = {}
obj.foo() // Uncaught TypeError: obj.foo is not a function

// 2. 类型不明确可能导致函数功能发生改变
function sum(a , b) {
  return a + b
}
console.log(sum(100, 100)) // 200
console.log(sum(100, '100')) // 100100

// 3. 因为弱类型的原因，可能导致索引器的错误使用
const obj = {}
obj[true] = 100
console.log(obj['true']) // 100
```

### 强类型的优势

- 错误更早暴露
- 代码更智能，编码更准确
- 重构更牢靠
- 减少不必要的类型判断

## Flow

> JavaScript 的类型检查器

```bash
# 快速上手

# 初始化安装
yarn init
yarn add flow-bin --dev
# 初始化flow
yarn flow init
# 代码文件中添加 // @flow
# 代码中添加类型注解
# 运行flow
yarn flow
```

### Flow 编译移除注解

```bash
# 1. 官方库
$ yarn add flow-remove-types --dev
# 删除类型注解
$ yarn flow-remove-types src -d dist

# 2. babel
$ yarn add @babel/core @babel/cli @babel/preset-flow --dev
# 添加 .babelrc 文件
# 执行
$ yarn babel src -d dist
```

### Flow 开发工具插件

> Flow Language Support 插件

### Flow 类型推断

```javascript
// @flow
function square(n) {
  return n * n;
}

square("100") // Cannot perform arithmetic operation because string [1] is not a number.
```

### Flow 类型注解

```javascript
// @flow
function square(n: number) {
  return n * n;
}
let num: number = 100;

function foo(): number {
  return 100;
}

function foo(): void {
}
```

### Flow 原始类型

> string 、number (100 // NaN // Infinity) 、boolean (true // false) 、null 、undefined 、symbol

### Flow 数组类型

```javascript
// @flow
const arr: Array<number> = [1, 2, 3]

const arr1: number[] = [1, 2, 3]
// 元组
const foo: [String, number] = ['foo', 100]
```

### Flow 对象类型

```javascript
// @flow
const obj: { foo: tring, bar: number } = { foo: 'string', bar: 100 }

const obj1: { foo?: tring, bar: number } = { bar: 100 }

const obj2: { [string]: string } = {}
obj2.key1 = 'value'
obj2.key2 = 'value2'
```

### Flow 函数类型

```javascript
// @flow
function foo(callback: (string, number) => void) {
  callback('string', 100)
}
foo(function (str, num){})
```

### Flow 特殊类型

```javascript
// @flow
const a: 'foo' = 'foo'

const type: 'success' | 'warning' | 'danger'  = 'success'

const b: string | number = 'string'
type StringOrNumber = string | number
const c: StringOrNumber = 'string' // 100

const gender: ?number = null
const gender: number | null | undefined = null
```

### Flow Mixed 与 Any

```javascript
// @flow
// mixed: 强类型
function passMixed(value: mixed) {
  if (typeof value === 'string') {
    // ...
  }
  // ...
}
passMixed('string')
passMixed(100)

// any: 仍然是弱类型
function passAny(value: any) {
}
passAny('string')
passAny('100')
```

## TypeScript

> JavaScript 超集（superset）
>
> 任何一种 JavaScript 运行环境都支持
>
> 功能更为强大，生态也更健全、更完善
>
> TypeScript属于 「渐进式」
>
> 缺点一：语言本身多了很多概念
>
> 缺点二：项目初期，TypeScript 会增加一些成本

### 快速上手

```bash
# 初始化
$ yarn init --yes
# 安装 TypeScript
$ yarn add typescript --dev
# 编译
$ yarn tsc demo.ts
```

### 配置文件

```bash
$ yarn tsc --init
# tsconfig.json 配置文件
# 编译
$ yarn tsc
```

### 原始类型

```typescript
const a: string = 'foo'
const b: number =  100 // NaN Infinity
const c: boolean = true // false
const d: void = undefined
const e: null = null
const f: undefined = undefined
const g: symbol = Symbol() // ES2015 标准库
```

### 标准库声明

> 标准库就是内置对象所对应的声明

```typescript
// tsconfig.json
"lib": ["ES2015", "DOM"],
```

### 中文错误消息

```bash
# 编译
$ yarn tsc --locale zh-CN
```

### 作用域问题

```typescript
// 模块化
export {}
```

### Object 类型

```typescript
// object 类型不单指对象，除了原始类型之外的其他类型
const foo: object = [] // {} function (){}
// 对象的类型限制可以使用类似对象字面量的形式限制
const obj: { foo: number, bar: string} = {foo: 123, bar: 'bar'}
```

### 数组类型

```typescript
// 数组类型
const arr1: Array<number> = [1, 2, 3]

const arr2: number[] = [1, 2, 3]

function sum(...args: number[]) {
  return args.reduce((prev, current) => prev + current, 0)
}
sum(1, 2, 3)
```

### 元组类型

> 一个明确元素数量以及每个元素类型的数组

```typescript
const tuple: [number, string] = [12, 'zxc']
const [age, name] = tuple
```

### 枚举类型

```typescript
// 枚举 Enum
enum postStatus {
    Draft = 0,
    Unpubilshed = 1,
    Published = 2
}
const post ={
    title: 'Hello TypeScript',
    content: 'TypeScript is a typed superset of JavaScript.',
    status: postStatus.Draft
}
// 常量枚举 const enum
```

### 函数类型

```typescript
// 函数类型
function func1(a: number, b?: number, ...rest: number[]): string {
    return 'func1'
}
func1(100, 200)
func1(100)

const func2: (a: number, b?: number) => string = 
      function (a: number, b?: number): string{
        return 'func2'
      }
```

### 任意类型

```typescript
// 任意类型 any类型是不安全的
function stringify(value: any) {
    return JSON.stringify(value)
}
stringify('string')
stringify(100)

let foo: any = 'string'
foo = 100
```

### 隐式类型推断

```typescript
// 隐式类型推断
let age = 10 // number

let foo // any
foo = 100
foo = 'string'
```

### 类型断言

```typescript
// 类型断言
const nums = [110, 120, 119, 113]
const res = nums.find((i) => i > 0)

const num1 = res as number
const num2 = <number>res // JSX 下不能使用
```

### 接口

```typescript
// 接口
interface Post {
  title: string
  content: string
  subTitle?: string // 可选成员
  readonly summary: string // 只读成员
}
function printPost(post: Post) {
  console.log(post.title);
  console.log(post.content);
}
printPost({
  title: 'Hello TypeScript',
  content: 'A javascript superset',
  readonly summary: string
})

// 动态成员
interface Caches {
  [prop: string]: string // 动态成员
}

const cache: Caches = {}
cache.foo = "value"
```

### 类

> 描述一类具体事物的抽象特征
>
> ES6 开始 JavaScript 中有了专门的 class ，TypeScript 增强了 class 的相关语法

```typescript
// 类
class Person {
  name: string // = 'init name'
  age: number
  constructor(name: string, age: number) {
    this.name = name
    this.age = age
  }
  sayHi(msg: string): void {
    console.log(`I am ${this.name}, ${msg}`)
  }
}
```

#### 类的访问修饰符

```typescript
// 类的访问修饰符
class Person {
  public name: string // = 'init name'
  private age: number
  protected gender: boolean
  constructor(name: string, age: number) {
    this.name = name
    this.age = age
    this.gender = true
  }
}
const tom = new Person('tom', 18)

class Student extends Person {
  private constructor(name: string, age: number) {
    super(name, age)
    console.log(this.gender)
  }
  static create(name: string, age: number) {
    return new Student(name, age)
  }
}
const jack = Student.create('jack', 18)
```

#### 类的只读属性

```typescript
// 类的只读属性
class Person {
  public name: string // = 'init name'
  private age: number
  protected readonly gender: boolean
  constructor(name: string, age: number) {
    this.name = name
    this.age = age
    this.gender = true
  }
}
const tom = new Person('tom', 18)
```

#### 类与接口

```typescript
// 类与接口
interface Eat {
  eat(food: string): void;
}
interface Run {
  run(distance: number): void;
}

class Person implements Eat, Run {
  eat(food: string): void {
    // ...
  }
  run(distance: number): void {
    // ...
  }
}
class Animal implements Eat, Run {
  eat(food: string): void {
    // ...
  }
  run(distance: number): void {
    // ...
  }
}
```

#### 抽象类

```typescript
// 抽象类
abstract class Animal {
  eat(food: string): void {
    // ...
  }
  abstract run(distance: number): void;
}
class Dog extends Animal {
  run(distance: number): void {
    // ...
  }
}
const dog = new Dog();
dog.eat("肉");
dog.run(100);

```

### 泛型

```typescript
// 泛型

function createArr<T>(length: number, value: T): T[] {
  const arr = Array<T>(length).fill(value);
  return arr;
}

const res = createArr(3, 100); // res => [100, 100, 100]
```

### 类型声明

```typescript
// 类型声明  （*.d.ts 文件）
import { camelCase } from "lodash"
// declare function camelCase(input: string): string
```

