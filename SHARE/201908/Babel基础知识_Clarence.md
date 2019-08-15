# Babel 基础知识

### 背景知识

### JavaScript 是怎样添加新的属性与特性的比如 ES6?

是由ECMA组织下一个名为TC39标准委员会提议及制定的。

关于 TC39 提案流程
TC39 提案分为以下几个阶段:

Stage 0 - 设想（Strawman）：任何尚未提交为正式提案的讨论，想法，改变或对已有规范的补充建议都被认为是一个稻草人草案（“strawman” proposal），但只有TC39成员可以提出此阶段的草案。

Stage 1 - 提案（Proposal）：此阶段，稻草人草案升级为正式化的提案，并将逐步解决多部门关切的问题，如与其他提案的相互之间会有什么影响，这一草案具体该如何实施等问题。人们需要对这些问题提供具体的解决方案。stage1的提案通常还需要包括API描述，拥有说明性使用示例，并对语义和算法进行讨论，一般来说草案在这一阶段会经历巨大的变化。

Stage 2 - 草案（Draft）：此阶段，草案就有了初始的规范。通过polyfill，开发者可以开始使用这一阶段的草案了，一些浏览器引擎也会逐步对这一阶段的规范的提供原生支持，此外通过使用构建工具也可以编译源代码为现有引擎可以执行的代码，这些方法都使得这一阶段的草案可以开始被使用了。

Stage 3 - 候选（Candidate）：此阶段的规范就属于候选推荐规范了，这一阶段之后变化就不会那么大了，要达到这一阶段需要满足以下条件：
规范的编辑和指定的审阅者必须在最终规范上签字；
用户也应该对该提议感兴趣；
提案必须至少被一个浏览器原生支持；
拥有高效的Polyfill，或者被Babel支持；

Stage 4 - 完成（Finished）：此阶段的提案必须有两个独立的通过验收测试的实现，进入第4阶段的提案将包含在 ECMAScript 的下一个修订版中。

ES6 是 2015 年制定的一个草案。

### Babel是什么?用来做什么的？

简单来说 Babel 就是一个把 JavaScript 脚本转译器，把新的规范比如 es2015/es2016/es2017 等新语法和特性转化为 es5，或者更低版本让低端运行环境(浏览器)能够认识运行。这样的作用是无论新特性出得多快，浏览器兼容情况多差，都能使用一些新特性去运行在低浏览器环境。甚至可以把 JavaScript 转译为其它不同语言。

### Babel 的运行方式和插件?

简单来说 Babel 运行分为三个阶段: 解析， 转换， 生成.
Babel 自己本身是不具有任何转换功能的，需要把 转化的功能插入到 Plugin 里面才会起作用，所以如果开启 Babel 不装任何插件，输入代码和输出代码是相同没转译的。所以 Babel 需要按需装插件。

### Babel 的插件 Plugins 和 Preset

- Plugins(插件)

```js
/* .babelrc */
{
  "plugins": [
    "@babel/plugin-transform-arrow-functions", // 箭头函数转换插件
    "@babel/plugin-transform-destructuring", // 解构语法插件
  ]
}

```
或者可以自己写转换插件

```js
{
  "plugins": ["babel-plugin-myPlugin"]
}
```

- Preset(预设) Preset是 插件的组合

1. @babel/preset-env

2. @babel/preset-stage-0

3. @babel/preset-stage-1

4. @babel/preset-stage-2

5. @babel/preset-stage-3

6. @babel/preset-react


#### **preset-env**

一般只要装截 preset-env 就会获得最新的，语法转换插件组合, preset-env 等价于 latest，也等价于 es2015 + es2016 + es2017 + es2018 相加(不包含 stage-x 中的插件)。env 包含的插件列表维护在 [这里](https://github.com/babel/babel-preset-env/blob/master/data/plugin-features.js)

```js
/* .babelrc */
{
  "presets": ["@babel/preset-env"]    
}

```

```js
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "entry"
      }
    ]
  ]
}
```


```js
{
  "presets": ["es2015", "react", "stage-2"]
}
```

在 Babel 7 后已经放弃支持 es201x 删除 stage-x, 淘汰 es201x 的目的是把选择环境的工作交给 env 自动进行，而不需要开发者投入精力。凡是使用 es201x 的开发者，都应当使用 env 进行替换。


#### **preset-polyfill**
插件分为语法插件和Api 插件两部分，这个要分清楚。
babel 默认只转换 js 语法，而不转换新的 API，比如 Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，以及一些定义在全局对象上的方法(比如 Object.assign)都不会转码。这个时候就需要 preset-polyfill。一般 preset-polyfill 都很大，因为兼容了全部的 polyfill 。所以的时候会按需加载。或者单独装配 polyfill 插件。这就不深入了。


<!-- ### Babel运行模块

1. babel-core 看名字就知道，babel-core是作为babel的核心存在，babel的核心api都在这个模块里面.

2. babel-cli 顾名思义，cli 就是命令行工具。安装了 babel-cli 就能够在命令行中使用 babel 命令来编译文件。

 -->





Clarence 2019.08.15

