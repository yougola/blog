<!--
 * @Description: In User Settings Edit
 * @Author: your name
 * @Date: 2019-10-10 13:18:14
 * @LastEditTime: 2019-10-10 14:09:22
 * @LastEditors: Please set LastEditors
 -->
## npm版本号管理策略
___

#### 1. npm version的含义

每个npm包都有一个package.json，如果要发布包的话，package.json里面的version字段就是决定发包的版本号。
version字段是这样一个结构：0.0.1，是有三位的版本号。分别是对应的version里面的：major, minor, patch。
也就是说当发布大版本的时候会升级为 1.0.0，小版本是0.1.0，一些小修复是0.0.2。

如果执行了prepatch，版本号会从1.1.1变成 1.1.2-0

antd-design 也是用预发布的方式管理npm版本号
![输入图片说明](https://images.gitee.com/uploads/images/2019/1010/140000_4a48abfb_1663282.png "屏幕截图.png")
我们称版本号1.1.2-0的4位分别是 大号.中号.小号-预发布号

|  version   |  功能   |
| --- | --- |
|  major   |  如果没有预发布号，则直接升级一位大号，其他位都置为0。**即2.0.1变成3.0.0**<br> 如果有预发布号：<br>中号和小号都为0，则不升级大号，而将预发布号删掉。 **即2.0.0-1变成2.0.0** ，这就是预发布的作用<br>如果中号和小号有任意一个不是0，那边会升级一位大号，其他位都置为0，清空预发布号。**即2.0.1-0变成3.0.0**  |
|  minor   |   如果没有预发布号，则升级一位中号，大号不动，小号置为0。**即2.0.1变成2.1.0**<br>如果有预发布号:<br>如果小号为0，则不升级中号，将预发布号去掉。**即2.0.0-1变成2.1.0**<br>如果小号不为0，同理没有预发布号。**即2.0.1-1变成2.0.5**  |
|  patch   |   如果没有预发布号：直接升级小号，去掉预发布号。**即2.0.1变成2.0.2**<br>如果有预发布号：去掉预发布号，其他不动。**即2.0.1-0变成2.0.1**  |
|  premajor   |  直接升级大号，中号和小号置为0，增加预发布号为0。**即2.0.1变成3.0.0-0**   |
|  preminor   |   直接升级中号，小号置为0，增加预发布号为0。**即2.0.1-0变成2.1.0-0**  |
|  prepatch  |   直接升级小号，增加预发布号为0。**即2.0.1变成2.0.2-0**  |
|  prerelease   |  如果没有预发布号：增加小号，增加预发布号为0。**即2.0.1变成2.0.1-0** <br>如果有预发布号，则升级预发布号。**即2.0.1-0变成2.0.1-1**  |

#### 2. 一次完整的npm package发布流程是怎样的？

##### 1. 使用Git Bash(命令行工具)切换到组件库项目目录，如下图。
``` 命令行
cd igola-weex-ui的项目目录
```
![输入图片说明](https://images.gitee.com/uploads/images/2019/1008/115418_b1472335_1663282.png "屏幕截图.png")

##### 2. 登录npm账号，到官网注册用户 https://www.npmjs.com 进行注册。
``` 命令行
$ npm adduser // or $ npm login
Username: npm-user-name
Password:
Email: your-email
```
![输入图片说明](https://images.gitee.com/uploads/images/2019/1008/134009_59f8a80b_1663282.png "屏幕截图.png")

##### 3. 发布（由于之前已经发布到npm，所以这里提示我们去更新 package.json 中的 version）
``` 命令行
$ npm publish
```
![输入图片说明](https://images.gitee.com/uploads/images/2019/1008/141425_2c72faaa_1663282.png "屏幕截图.png")

##### 4. 更新

- 发布一个新的稳定版本

当模块有更新的时候，需要发布一个新版本，当所有需要更新的文件都 commit 完了后，就可以更新到 npm 了。
``` 命令行
# 更新版本号（major | minor | patch | premajor | preminor | prepatch | prerelease）
# 大版本并且不向下兼容时，使用 major
# 有新功能且向下兼容时，使用 minor
# 修复一些问题、优化等，使用 patch
# 下面比如更新一个 patch 小版本号
$ npm version patch
$ npm publish
```

- 预发布版本

很多时候一些新改动，并不能直接发布到稳定版本上（稳定版本的意思就是使用 npm install weex-ui-demo 即可下载的最新版本），这时可以发布一个 “预发布版本“，不会影响到稳定版本。
``` 命令行
# 发布一个 prelease 版本，tag=beta
$ npm version prerelease
$ npm publish --tag beta
```
比如原来的版本号是 1.0.1，那么以上发布后的版本是 1.0.1-0，用户可以通过 npm install weex-ui-demo@beta 或者 npm install weex-ui-demo@1.0.1-0 来安装。

- 当 prerelease 版本稳定之后，可以把它设置为稳定版本

``` 命令行
# 首先可以查看当前所有的最新版本，包括 prerelease 与稳定版本。
$ npm dist-tag ls

# 设置 1.0.1-1 版本为稳定版本
$ npm dist-tag add weex-ui-demo@1.0.1-1 latest

# 或者通过 tag 来设置
$ npm dist-tag add weex-ui-demo@beta latest
```
这时候，latest 稳定版本已经是 1.0.1-1 了，用户可以直接通过 npm install weex-ui-demo 即可安装该版本。

- 当需要删除多余的tag时，可以用下面的指令。
``` 命令行
# 删除 tag beta
$ npm dist-tag rm weex-ui-demo beta
```

___


#### 3.一个完整的例子关于的npm package发布&更新流程

##### 1. 发布第一个稳定版本
``` 命令行
npm publish //1.0.0
```
##### 2. 修改文件继续发布第二个版本
``` 命令行
npm version patch
npm publish //1.0.1
```
##### 3. 继续修改文件发布一个prerelease版本
``` 命令行
 npm version prerelease
 npm publish --tag beta//版本<pkg>@1.0.2-0
```
##### 4. 继续修改发布第二个prerelease版本
``` 命令行
npm version prerelease
npm publish --tag beta//版本<pkg>@1.0.2-1
```
##### 5. npm info查看我们的版本信息
``` 命令行
{ 
   name: <pkg>,
  'dist-tags': { latest: '1.0.1', 'beta': '1.0.2-1' },
  versions: [ '1.0.0', '1.0.1', '1.0.2-0', '1.0.2-1' ],
  maintainers: [ 'huangsibo <3570411064@qq.com>' ],
  time: { 
    modified: '2019-10-08T12:17:56.755Z',
    created: '2019-10-08T12:15:23.605Z',
    '1.0.0': '2019-10-08T12:15:23.605Z',
    '1.0.1': '2019-10-08T12:16:24.916Z',
    '1.0.2-0': '2019-10-08T12:17:23.354Z',
    '1.0.2-1': '2019-10-08T12:17:56.755Z'
  },
  homepage: 'https://github.com/liangklfang/n#readme',
  repository: { type: 'git', url: 'git+https://github.com/liangklfang/n.git' },
  bugs: { url: 'https://github.com/liangklfang/n/issues' },
  license: 'ISC',
  readmeFilename: 'README.md',
  version: '1.0.1',
  description: '',
  main: 'index.js',
  scripts: { test: 'echo "Error: no test specified" && exit 1' },
  author: '',
  gitHead: '8123b8addf6fed83c4c5edead1dc2614241a4479',
  dist: { 
    shasum: 'a60d8b02222e4cae74e91b69b316a5b173d2ac9d',
    tarball: 'https://registry.npmjs.org/<pkg>/-/<pkg>-1.0.1.tgz' 
  },
  directories: {}
}
```
我们只要注意下面者两个部分：
``` 命令行
 'dist-tags': { latest: '1.0.1', 'beta': '1.0.2-1' },
  versions: [ '1.0.0', '1.0.1', '1.0.2-0', '1.0.2-1' ],
```
其中最新的稳定版本和最新的beta版本可以在dist-tags中看到，而versions数组中存储的是所有的版本。

##### 6. npm dist-tag命令
``` 命令行
npm dist-tag ls <pkg>
```
即npm dist-tag获取到所有的最新的版本，包括prerelease与稳定版本，得到下面结果：
```
beta: 1.0.2-1
latest: 1.0.1
```
##### 7. 当我们的prerelease版本已经稳定了，重新设置为稳定版本
``` 命令行
npm dist-tag add <pkg>@1.0.2-1 latest
```
此时你通过npm info查看可以知道：
```
{ name: <pkg>,
  'dist-tags': { latest: '1.0.2-1', 'beta': '1.0.2-1' },
  versions: [ '1.0.0', '1.0.1', '1.0.2-0', '1.0.2-1' ],
  maintainers: [ 'huangsibo <3570411064@qq.com>' ],
  time:{ 
    modified: '2019-10-08T12:24:55.800Z',
    created: '2019-10-08T12:15:23.605Z',
    '1.0.0': '2019-10-08T12:15:23.605Z',
    '1.0.1': '2019-10-08T12:16:24.916Z',
    '1.0.2-0': '2019-10-08T12:17:23.354Z',
    '1.0.2-1': '2019-10-08T12:17:56.755Z' 
  },
  homepage: 'https://github.com/liangklfang/n#readme',
  repository: { type: 'git', url: 'git+https://github.com/liangklfang/n.git' },
  bugs: { url: 'https://github.com/liangklfang/n/issues' },
  license: 'ISC',
  readmeFilename: 'README.md',
  version: '1.0.2-1',
  description: '',
  main: 'index.js',
  scripts: { test: 'echo "Error: no test specified" && exit 1' },
  author: '',
  gitHead: '03189d2cc61604aa05f4b93e429d3caa3b637f8c',
  dist: { 
    hasum: '41ea170a6b155c8d61658cd4c309f0d5d1b12ced',
    tarball: 'https://registry.npmjs.org/<pkg>/-/<pkg>-1.0.2-1.tgz' 
  },
  directories: {} 
}
```
主要关注如下:
```
 'dist-tags': { latest: '1.0.2-1', 'beta': '1.0.2-1' },
  versions: [ '1.0.0', '1.0.1', '1.0.2-0', '1.0.2-1' ]
```
此时latest版本已经是prerelease版本"1.0.2-1"了！此时用户如果直接运行npm install就会安装我们的prerelease版本了，因为版本已经更新了！

当然，我们的npm publish可以有很多tag的，比如上面是beta，也可以是dev, sit, uat等，比如下面你继续运行：
```
 npm version prerelease
 npm publish --tag dev
```
此时你运行npm info就会得到下面的信息：
```
{ name: <pkg>,
  'dist-tags': { latest: '1.0.2-1', 'beta': '1.0.2-1', 'dev': '1.0.2-2' },
  versions: [ '1.0.0', '1.0.1', '1.0.2-0', '1.0.2-1', '1.0.2-2' ],
  maintainers: [ 'huangsibo <3570411064@qq.com>' ],
  time: { 
    modified: '2019-10-08T13:01:17.106Z',
    created: '2019-10-08T12:15:23.605Z',
    '1.0.0': '2019-10-08T12:15:23.605Z',
    '1.0.1': '2019-10-08T12:16:24.916Z',
    '1.0.2-0': '2019-10-08T12:17:23.354Z',
    '1.0.2-1': '2019-10-08T12:17:56.755Z',
    '1.0.2-2': '2019-10-08T13:01:17.106Z' 
  },
  homepage: 'https://github.com/liangklfang/n#readme',
  repository: { type: 'git', url: 'git+https://github.com/liangklfang/n.git' },
  bugs: { url: 'https://github.com/liangklfang/n/issues' },
  license: 'ISC',
  readmeFilename: 'README.md',
  version: '1.0.2-1',
  description: '',
  main: 'index.js',
  scripts: { test: 'echo "Error: no test specified" && exit 1' },
  author: '',
  gitHead: '03189d2cc61604aa05f4b93e429d3caa3b637f8c',
  dist: { 
    shasum: '41ea170a6b155c8d61658cd4c309f0d5d1b12ced',
    tarball: 'https://registry.npmjs.org/<pkg>/-/<pkg>-1.0.2-1.tgz' 
  },
  directories: {} 
}
```
重点关注如下内容
```
 'dist-tags': { latest: '1.0.2-1', 'beta': '1.0.2-1', 'dev': '1.0.2-2' },
  versions: [ '1.0.0', '1.0.1', '1.0.2-0', '1.0.2-1', '1.0.2-2' ],
```
此时你会看到beta版本最新是1.0.2-1，而dev版本最新是1.0.2-2


参考资料：

[npm的tag管理](http://cnodejs.org/topic/537b47d1cbcc39634983b739)

[npm-dist-tag](https://docs.npmjs.com/cli/dist-tag)

[npm-version](https://docs.npmjs.com/cli/version)
