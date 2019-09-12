# Decorator 装饰器 
### Decorator 规范

装饰器是一种 Js 编译时执行的函数， 可以写 成 `@ + 函数`。 可以放在类和类方法的定义前面。

```js
@frozen class Foo {
  @configurable(false)
  @enumerable(true)
  method() {}

  @throttle(500)
  expensiveMethod() {}
}
```

#### 1.方法的装饰

装饰器可以装饰类函数属性。

装饰器函数一共可以接受三个参数
```js
function decorators(target, name, descriptor) {
  console.log(target) // target 参数指的是类本身
  console.log(name)  // name 参数指的是属性名, 如果装在类中返回为空
  console.log(descriptor) // 参数是该属性的描述对象，如果装在类中返回为空
}
```

下面例子可以打印执行注释。

```js
class Math {
  @log
  add(a, b) {
    return a + b
  }
}

function log(target, name, descriptor) {
  var oldValue = descriptor.value

  descriptor.value = function() {
    console.log(`Calling ${name} with`, arguments)
    return oldValue.apply(this, arguments)
  }

  return descriptor
}

const math = new Math()

console.log(math.add(2, 4))
// Calling add with 
// Arguments(2) [2, 4, callee: ƒ, Symbol(Symbol.iterator): ƒ]
// 0: 2
// 1: 4
// callee: ƒ ()
// length: 2
// Symbol(Symbol.iterator): ƒ values()
// __proto__: Object
```

装饰器也可以携带注释的作用，以下面例子作描述，Person 可测试的， name 函数只读，不可枚举。

```js
@testable
class Person {
  @readonly
  @noneumerable
  name() { return `${this.first} ${this.last}` }
}
```

如果同一个方法有多个装饰器，会像剥洋葱一样，先从外到内进入，然后由内向外执行。

```js
function dec(id) {
  console.log('evaluated', id)
  return (target, property, descriptor) => console.log('executed', id)
}

class Example {
  @dec(1)
  @dec(2)
  method() {}
}

const e = new Example()

e.method()

// evaluated 1
// evaluated 2
// executed 2
// executed 1
```

#### 2.类的装饰

装饰器可以用来装饰整个类

下面是用装饰器为类添加静态属性的例子。
```js
function setTestable(isTestable) {
  return function(target) {
    target.isTestable = isTestable
  }
}

@setTestable(true)
class A() {}

@setTestable(false)
class B() {}

console.log(A.isTestable) // true

console.log(B.isTestable) // false

```

装饰器对类的行为改变，是代码编译时发生的，而不是在运行时，这意味着，装饰器能在编译阶段运行代码。装饰器是编译时执行的函数。

```js
@decorator
class A {}
// equal
class A {}
A = decorator(A) || A
```

基本上装饰器的原理就是编译时自动添加装饰器函数像上面这样。

```js
function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list)
  }
}

const Foo = {
  foo() { console.log('foo') },
  zoo() { console.log('zoo') }
}

@mixins(Foo)
class MyClass {}

let obj = new MyClass()
obj.foo() // 'foo'
obj.zoo() // 'zoo'

```

上面的例子是把属性添加到目标类的 'prototype' 对象上面了，因此可以在实例上面调用，这也是类似 mixins 的使用方法。



#### 3. 装饰器不能用于函数

因为函数存在函数提升
```js
var counter = 0

var add = function () {
  counter++
}

@add
function foo() {}

// 编译出来会等于下面这种写法

@add
function foo() {
}

var counter
var add

counter = 0

add = function() {
  counter++
}
```

#### 5. Decorator 的使用场景

1) 利用 Decorator 实现 `Mixin` 模式。
在装饰器的基础上，可以实现 `Mixin` 模式， `Mixin` 模式其实 就是对象继承的另一种替代方法。可以看下面方法。

```js
const Foo = {
  foo() { console.log('foo') }
}
class MyClass {}
Object.assign(MyClass.prototype, Foo)

let obj = new MyClass()
obj.foo() // 'foo'
```

下面我们可以通过自己写的装饰器实现上面步骤。

```js
// mixins.js
export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list)
  }
}

// index.js
import { mixins } from './mixins'

const Foo = {
  foo() { console.log('foo') }
}

@mixins(Foo)
class MyClass {}

let obj = new MyClass()
obj.foo() // 'foo'
```

2. 通过 Decorators 绑定类的方法


在实际开发中 React 和 Redux 库结合使用时，常常需要写成下面这样，用来绑定 `store` 属性到组件`MyReactComponent` 里面。
```js
class MyReactComponent extends React.Component  {}
export default connect(mapStateToProps, mapDispatchToProps) { MyReactComponent }
```
其实使用装饰器代码会更加方便

```js
@connect(mapStateToProps, mapDispatchToProps)
export defalut class MyReactComponent extends React.Component {}
```

像我们现在 React 组件里面也利用了这个特性去绑定 store
```js
@connect(({timeline, app, flight}) => ({
  timeline,
  flight,
  lang: app.languageObj.lang,
  currency: app.currencyObj.currCode,
  symbol: app.currencyObj.currSymbol,
}))
class Timeline extends Component {
  ...
}
```

3. 使用 Decorators 实现自动发布事件，
我们可以使用装饰器，令对象方法被调用的时候，自动发出一个事件。像之前 aoping 的无侵入埋点方案就利用这个特性去实现便利性。
以下代码从 aoping 方案里面截取出来。

```js
//
/* track by decorator
 * class SomeComponent {
 *     @track(before(() => console.log('hello, trackpoint')))
 *     onClick = () => {
 *         ...
 *     }
 * } */
 
export const track = partical => (target, key, descriptor) => {
  if (!isFunction(partical)) {
    throw new Error('trackFn is not a function ' + partical)
  }
  const value = function (...args) {
    return partical.call(this, descriptor.value, this).apply(this, args)
  }
  const res = Object.assign({}, descriptor, {
    value
  })
  return res
}
```



#### 6. Decorator 还处于[Stage-2]

```js
@connect(({timeline, app, flight}) => ({
  timeline,
  flight,
  lang: app.languageObj.lang,
  currency: app.currencyObj.currCode,
  symbol: app.currencyObj.currSymbol,
}))
class Timeline extends Component { // 对修饰器的实验支持是一项将在将来版本中更改的功能。设置 "experimentalDecorators" 选项以删除此警告。
   ...
}
```

因为 Decorator 现在只是处理 Stage-2 阶段, 其实这个特性被作用 ** Yehuda Katz** 三年前就已经提出来的了，一出来就很受欢迎瞬间被 TypeScript 支持，像 Angular 和 Mobx 一些流行的框架都有使用到，但是由于性能问题和其它作者的大量改动的原因， Decorator 一直没能推出到生产版本中，而且 TC39 委员会对此特性进行大量更改出了新版本的 Decorator，这就很悲催了，旧版本上不了线新版本还没准备好，就连 babel 也是只支持旧版本的 Decorator，新版本功能还没实现，所以请谨慎使用。

在新版本 Decorator 有很多大量重量级的变化包括：

- 1. 语法写法。
- 2. 对象装饰器
- 3. 函数参数装饰器等。

请尽情期待吧。

