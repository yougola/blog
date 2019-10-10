<!--
 * @Description: In User Settings Edit
 * @Author: your name
 * @Date: 2019-10-10 16:31:18
 * @LastEditTime: 2019-10-10 16:31:18
 * @LastEditors: your name
 -->
## 定义
ES6 原生提供了 Promise 对象。

所谓 Promise，就是一个对象，用来传递异步操作的消息。它代表了某个未来才会知道结果的事件（通常是一个异步操作），并且这个事件提供统一的 API，可供进一步处理。
Promise是异步编程的一种解决方案

## 基本API
基本的 api

Promise.resolve()

Promise.reject()

Promise.prototype.then()

Promise.prototype.catch()

Promise.all() // 所有的完成

var p = Promise.all([p1,p2,p3]);

Promise.race() // 竞速，完成一个即可

## 三种状态
Promise的内部实现是一个状态机。
Promise有三种状态：pending，resolved，rejected。
当Promise刚创建完成时，处于pending状态；
当Promise中的函数参数执行了resolve后，Promise由pending状态变成resolved状态；
如果在Promise的函数参数中执行的不是resolve方法，而是reject方法，那么Promise会由pending状态变成rejected状态。

## 用法与特性
### 1、代码立即执行
```js
var p = new Promise(function(resolve, reject){
  console.log("create a promise");
  resolve("success");
});

console.log("after new Promise");

p.then(function(value){
  console.log(value);
});
```
输出结果

1、"create a promise"

2、"after new Promise"

3、"success"

### 2、状态的不可逆性
```js
var p1 = new Promise(function(resolve, reject){
  resolve("success1");
  resolve("success2");
});

var p2 = new Promise(function(resolve, reject){
  resolve("success");
  reject("reject");
});

p1.then(function(value){
  console.log(value);
});

p2.then(function(value){
  console.log(value);
});
```
Promise状态的一旦变成resolved或rejected时，Promise的状态和值就固定下来了，不论你后续再怎么调用resolve或reject方法，都不能改变它的状态和值。
### 3、回调异步性
```js
var p = new Promise(function(resolve, reject){
  resolve("success");
});

p.then(function(value){
  console.log(value);
});

console.log("which one is called first ?");
```
### 4、链式调用
```js
var p = new Promise(function(resolve, reject){
  resolve(1);
});
p.then(function(value){               //第一个then
  console.log(value);
  return value*2;
}).then(function(value){              //第二个then
  console.log(value);
}).then(function(value){              //第三个then
  console.log(value);
  return Promise.resolve('resolve'); 
}).then(function(value){              //第四个then
  console.log(value);
  return Promise.reject('reject');
}).then(function(value){              //第五个then
  console.log('resolve: '+ value);
}, function(err){
  console.log('reject: ' + err);
})
```
结果输出

1

2

undefined

"resolve"

"reject: reject"

Promise对象的then方法返回一个新的Promise对象，因此可以通过链式调用then方法。then方法接收两个函数作为参数，第一个参数是Promise执行成功时的回调，第二个参数是Promise执行失败时的回调。两个函数只会有一个被调用，函数的返回值将被用作创建then返回的Promise对象。这两个参数的返回值可以是以下三种情况中的一种：

return 一个同步的值 ，或者 undefined（当没有返回一个有效值时，默认返回undefined），then方法将返回一个resolved状态的Promise对象，Promise对象的值就是这个返回值。
return 另一个 Promise，then方法将根据这个Promise的状态和值创建一个新的Promise对象返回。
throw 一个同步异常，then方法将返回一个rejected状态的Promise,  值是该异常。

## 异常
```js
var p1 = new Promise( function(resolve,reject){
  foo.bar();
  resolve( 1 );	  
});

p1.then(
  function(value){
    console.log('p1 then value: ' + value);
  },
  function(err){
    console.log('p1 then err: ' + err);
  }
).then(
  function(value){
    console.log('p1 then then value: '+value);
  },
  function(err){
    console.log('p1 then then err: ' + err);
  }
);
```

```js
var p2 = new Promise(function(resolve,reject){
  resolve( 2 );	
});

p2.then(
  function(value){
    console.log('p2 then value: ' + value);
    foo.bar();
  }, 
  function(err){
    console.log('p2 then err: ' + err);
  }
).then(
  function(value){
    console.log('p2 then then value: ' + value);
  },
  function(err){
    console.log('p2 then then err: ' + err);
    return 1;
  }
).then(
  function(value){
    console.log('p2 then then then value: ' + value);
  },
  function(err){
    console.log('p2 then then then err: ' + err);
  }
);
```
p1 then err: ReferenceError: foo is not defined

p2 then value: 2

p1 then then value: undefined

p2 then then err: ReferenceError: foo is not defined

p2 then then then value: 1
Promise中的异常由then参数中第二个回调函数（Promise执行失败的回调）处理，异常信息将作为Promise的值。异常一旦得到处理，then返回的后续Promise对象将恢复正常，并会被Promise执行成功的回调函数处理。另外，需要注意p1、p2 多级then的回调函数是交替执行的 ，这正是由Promise then回调的异步性决定的。

then()方法使Promise原型链上的方法，它包含两个参数方法，分别是已成功resolved的回调和已失败rejected的回调

.catch()的作用是捕获Promise的错误，与then()的rejected回调作用几乎一致。但是由于Promise的抛错具有冒泡性质，能够不断传递，这样就能够在下一个catch()中统一处理这些错误。同时catch()也能够捕获then()中抛出的错误，所以建议不要使用then()的rejected回调，而是统一使用catch()来处理错误

同样，catch()中也可以抛出错误，由于抛出的错误会在下一个catch中被捕获处理，因此可以再添加catch()

使用rejects()方法改变状态和抛出错误 throw new Error() 的作用是相同的

当状态已经改变为resolved后，即使抛出错误，也不会触发then()的错误回调或者catch()方法

then() 和 catch() 都会返回一个新的Promise对象，可以链式调用

## Promise.resolve() / Promise.reject()

用来包装一个现有对象，将其转变为Promise对象，但Promise.resolve()会根据参数情况返回不同的Promise：

参数是Promise：原样返回
参数带有then方法：转换为Promise后立即执行then方法
参数不带then方法、不是对象或没有参数：返回resolved状态的Promise

Promise.reject()会直接返回rejected状态的Promise

## Promise.all()
参数为Promise对象数组，如果有不是Promise的对象，将会先通过上面的Promise.resolve()方法转换
```js
var promise = Promise.all( [p1, p2, p3] )
promise.then(
    ...
).catch(
    ...
)
```
当p1、p2、p3的状态都变成resolved时，promise才会变成resolved，并调用then()的已完成回调，但只要有一个变成rejected状态，promise就会立刻变成rejected状态

## Promise.race()
```js
var promise = Promise.race( [p1, p2, p3] )
promise.then(
    ...
).catch(
    ...
)
```
“竞速”方法，参数与Promise.all()相同，不同的是，参数中的p1、p2、p3只要有一个改变状态，promise就会立刻变成相同的状态并执行对于的回调

## Promise.done() / Promise. finally()
Promise.done() 的用法类似 .then() ，可以提供resolved和rejected方法，也可以不提供任何参数，它的主要作用是在回调链的尾端捕捉前面没有被 .catch() 捕捉到的错误

Promise. finally() 接受一个方法作为参数，这个方法不管promise最终的状态是怎样，都一定会被执行
