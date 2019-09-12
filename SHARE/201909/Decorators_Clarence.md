# Decorator 装饰器 

### 基础知识
### 关于 TC39 提案流程

TC39 提案分为以下几个阶段:

- Stage 0 - 设想（Strawman）：任何尚未提交为正式提案的讨论，想法，改变或对已有规范的补充建议都被认为是一个稻草人草案（“strawman” proposal），但只有TC39成员可以提出此阶段的草案。

- Stage 1 - 提案（Proposal）：此阶段，稻草人草案升级为正式化的提案，并将逐步解决多部门关切的问题，如与其他提案的相互之间会有什么影响，这一草案具体该如何实施等问题。人们需要对这些问题提供具体的解决方案。stage1的提案通常还需要包括API描述，拥有说明性使用示例，并对语义和算法进行讨论，一般来说草案在这一阶段会经历巨大的变化。并初步尝试实现出来。

- Stage 2 - 草案（Draft）：此阶段，草案就有了初始的规范。通过polyfill，开发者可以开始使用这一阶段的草案了，一些浏览器引擎也会逐步对这一阶段的规范的提供原生支持，此外通过使用构建工具也可以编译源代码为现有引擎可以执行的代码，这些方法都使得这一阶段的草案可以开始被使用了。完成初步规范。

- Stage 3 - 候选（Candidate）：此阶段的规范就属于候选推荐规范了，这一阶段之后变化就不会那么大了，基本已经完成规范并在浏览器上初步实现。要达到这一阶段还需要满足以下条件：
规范的编辑和指定的审阅者必须在最终规范上签字；
用户也应该对该提议感兴趣；
提案必须至少被一个浏览器原生支持；
拥有高效的Polyfill，或者被Babel支持；

- Stage 4 - 完成（Finished）：此阶段的提案必须有两个独立的通过验收测试的实现，进入第4阶段的提案将包含在 ECMAScript 的下一个修订版中。将添加到下一个年度版本发布中。

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

在实际开发中 React 和 Redux 库结合使用时，常常需要写成下面这样。
```js
class MyReactComponent extends React.Component  {}
export default connect(mapStateToProps, mapDispatchToProps) { MyReactComponent }
```
其实使用装饰器代码会更加方便

```js
@connect(mapStateToProps, mapDispatchToProps)
export defalut class MyReactComponent extends React.Component {}
```

3. 使用 Decorators 实现自动发布事件，
我们可以使用装饰器，令对象方法被调用的时候，自动发出一个事件。像之前 aoping 的无侵入埋点方案就可以利用这个特性去更方便实现。





#### 6. Decorator 还处于[Stage-2]

Decorator 现在只是处理 Stage-2 阶段，TC39委员会可能还会对此特性进行大量更改，请谨慎使用。
