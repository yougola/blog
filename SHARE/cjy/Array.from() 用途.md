# Array.from() 用途

- JavaScript 中有一个这样的函数: Array.from：允许在 JavaScript 集合(如: 数组、类数组对象、或者是字符串、map 、set 等可迭代对象) 上进行有用的转换。
- 在本文中，我将描述 5 个有用且有趣的 Array.from() 用例。

## 1. 介绍

在开始之前，我们先回想一下 Array.from() 的作用。语法:

```
Array.from(arrayLike[, mapFunction[, thisArg]])

```

- arrayLike：必传参数，想要转换成数组的伪数组对象或可迭代对象。
- mapFunction：可选参数，mapFunction(item，index){...} 是在集合中的每个项目上调用的函数。返回的值将插入到新集合中。
- thisArg：可选参数，执行回调函数 mapFunction 时 this 对象。这个参数很少使用。

例如，让我们将类数组的每一项乘以 2：

```
const someNumbers = { '0': 10, '1': 15, length: 2 };

Array.from(someNumbers, value => value * 2); // => [20, 30]

```

## 2.将类数组转换成数组

Array.from() 第一个用途：将类数组对象转换成数组。

通常，你会碰到的类数组对象有：函数中的 arguments 关键字，或者是一个 DOM 集合。

在下面的示例中，让我们对函数的参数求和：

```
function sumArguments() {
    return Array.from(arguments).reduce((sum, num) => sum + num);
}

sumArguments(1, 2, 3); // => 6
```

Array.from(arguments) 将类数组对象 arguments 转换成一个数组，然后使用数组的 reduce 方法求和。

此外，Array.from() 的第一个参数可以是任意一个可迭代对象，我们继续看一些例子:

```
Array.from('Hey');                   // => ['H', 'e', 'y']
Array.from(new Set(['one', 'two'])); // => ['one', 'two']

const map = new Map();
map.set('one', 1)
map.set('two', 2);
Array.from(map); // => [['one', 1], ['two', 2]]

```

## 3.克隆一个数组

在 JavaScript 中有很多克隆数组的方法。正如你所想，Array.from() 可以很容易的实现数组的浅拷贝。

```
function recursiveClone(val) {
    return Array.isArray(val) ? Array.from(val, recursiveClone) : val;
}

const numbers = [[0, 1, 2], ['one', 'two', 'three']];
const numbersClone = recursiveClone(numbers);

numbersClone; // => [[0, 1, 2], ['one', 'two', 'three']]
numbers[0] === numbersClone[0] // => false

```

recursiveClone() 能够对数组的深拷贝，通过判断 数组的 item 是否是一个数组，如果是数组，就继续调用 recursiveClone() 来实现了对数组的深拷贝。

## 4. 使用值填充数组

当初始化数组的每个项都应该是一个新对象时，Array.from() 是一个更好的解决方案：

```
const length = 3;
const resultA = Array.from({ length }, () => ({}));
const resultB = Array(length).fill({});

resultA; // => [{}, {}, {}]
resultB; // => [{}, {}, {}]

resultA[0] === resultA[1]; // => false
resultB[0] === resultB[1]; // => true


```

由 Array.from 返回的 resultA 使用不同空对象实例进行初始化。之所以发生这种情况是因为每次调用时，mapFunction，即此处的 () => ({}) 都会返回一个新的对象。
然后，fill() 方法创建的 resultB 使用相同的空对象实例进行初始化。不会跳过空项

## 5. 生成数字范围

你可以使用 Array.from() 生成值范围。例如，下面的 range 函数生成一个数组，从 0 开始到 end - 1。

```
function range(end) {
    return Array.from({ length: end }, (_, index) => index);
}

range(4); // => [0, 1, 2, 3]

```

在 range() 函数中，Array.from() 提供了类似数组的 {length：end} ，以及一个简单地返回当前索引的 map 函数 。这样你就可以生成值范围。

## 6.数组去重

由于 Array.from() 的入参是可迭代对象，因而我们可以利用其与 Set 结合来实现快速从数组中删除重复项

```
function unique(array) {
  return Array.from(new Set(array));
}

unique([1, 1, 2, 3, 3]); // => [1, 2, 3]

```

首先，new Set(array) 创建了一个包含数组的集合，Set 集合会删除重复项。

因为 Set 集合是可迭代的，所以可以使用 Array.from() 将其转换为一个新的数组。

这样，我们就实现了数组去重。

## 7.结论

Array.from() 方法接受类数组对象以及可迭代对象，它可以接受一个 map 函数，并且，这个 map 函数不会跳过值为 undefined 的数值项。这些特性给 Array.from() 提供了很多可能。
如上所述，你可以轻松的将类数组对象转换为数组，克隆一个数组，使用初始化填充数组，生成一个范围，实现数组去重。
