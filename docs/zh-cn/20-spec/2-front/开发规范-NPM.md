# npm 介绍

## npm 是什么？

> npm 为你和你的团队打开了连接整个 JavaScript 天才世界的一扇大门。它是世界上最大的软件注册表，每星期大约有 30 亿次的下载量，包含超过 600000 个 包（package） （即，代码模块）。来自各大洲的开源软件开发者使用 npm 互相分享和借鉴。包的结构使您能够轻松跟踪依赖项和版本。

下面是关于 npm 的快速介绍：

npm 由三个独立的部分组成：
- 网站 
- 注册表（registry）
- 命令行工具 (CLI)

> 网站 是开发者查找包（package）、设置参数以及管理 npm 使用体验的主要途径。
>
> 注册表 是一个巨大的数据库，保存了每个包（package）的信息。
>
> CLI 通过命令行或终端运行。开发者通过 CLI 与 npm 打交道。

### 为什么使用 npm？

npm(node package manager) 是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题,是Node.js能够如此成功的主要原因之一。npm团队做了很多的工作，以确保npm保持向后兼容，并在不同的环境中保持一致。

总的来说，npm就是一个工具，方便你搜索、重用JS代码片段的工具。开始阶段，很多人认为npm只能用做后端开发，后来前端开发也大量的用到npm，npm改名为JSPM可能更为恰当。

## 使用 npm 可能存在的困扰

### npm install 安装依赖慢

有些依赖包源属于国外，在国内下载第三方包的速度极慢。

解决方案：

1.使用淘宝 NPM 镜像定制的 cnpm (gzip 压缩支持) 命令行工具代替默认的 npm

```
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

使用方法：
```
$ cnpm install [name]
```

2.npm源管理工具

```
$ npm  install   -g   nrm
```

使用方法：
```
$ nrm ls

 * npm ---- https://registry.npmjs.org/
 cnpm --- http://r.cnpmjs.org/
 taobao - https://registry.npm.taobao.org/
 nj ----- https://registry.nodejitsu.com/
 rednpm - http://registry.mirror.cqupt.edu.cn/
 npmMirror  https://skimdb.npmjs.com/registry/
 edunpm - http://registry.enpmjs.org/
 
 // 切换
 $ nrm use taobao
                         verb config Skipping project config: C:\Users\user/.npmrc. (matches userconfig)
 
   Registry has been set to: https://registry.npm.taobao.org
   
 // 增加源
 $ nrm add <registry> <url> [home]

 // 删除源
 $ nrm del <registry>
  
 // 测试速度
 $ nrm test
 
 npm ---- 1177ms
 cnpm --- 236ms
 * taobao - 173ms
 nj ----- Fetch Error
 rednpm - Fetch Error
 npmMirror  3013ms
 edunpm - Fetch Error 
```
一般情况，不使用VPN，淘宝源速度最快，但是可能会出现依赖安装不正确的情况。当然需要更新公司私库，还是需要指定registry的。

### npm 更新版本不稳定

同一个项目，安装的时候无法保持一致性。由于package.json文件中版本号的特点，下面三个版本号在安装的时候代表不同的含义。
```
"5.0.3",
"~5.0.3",
"^5.0.3"
```

- “5.0.3”表示安装指定的5.0.3版本
- “～5.0.3”表示安装5.0.X中最新的版本
- “^5.0.3”表示安装5.X.X中最新的版本。

所以常常会出现同一个项目，有的同事是OK的，有的同事会由于安装的版本不一致出现bug。

同时，大多数npm库都严重依赖于其他npm库，这会导致嵌套依赖关系，并增加无法匹配相应版本的几率。
   
虽然可以通过npm config set save-exact true命令关闭在版本号前面使用^的默认行为，但这个只会影响顶级依赖关系。由于每个依赖的库都有自己的package.json文件，而在它们自己的依赖关系前面可能会有^符号，所以无法通过package.json文件为嵌套依赖的内容提供保证。
   
为了解决这个问题，npm提供了shrinkwrap命令。此命令将生成一个npm-shrinkwrap.json文件，为所有库和所有嵌套依赖的库记录确切的版本。
   
然而，即使存在npm-shrinkwrap.json这个文件，npm也只会锁定库的版本，而不是库的内容。即便npm现在也能阻止用户多次重复发布库的同一版本，但是npm管理员仍然具有强制更新某些库的权力。

所以实际开发中，第三方依赖包版本根据需求写死或限制版本范围更新，预防换包导致的兼容性bug。测试开发时 ，如何确保更新的包是最新版本？

这里得先介绍一下 **package-lock.json**
        
package-lock.json诞生的目的是为了防止出现我们上述的情况。 同一个package.json却产生了不同的运行结果。 package-lock.json在npm 5时添加进来, 所以如果你使用5以上的版本, 你就会看到这个文件, 除非你手动禁用掉它。

所以从此以后npm会根据package-lock.json里的内容来处理和安装依赖而不是根据package.json。因为pacakge-lock.json给每个依赖标明了版本, 获取地址和哈希值, 使得每次安装都会出现相同的结果。不管你在什么机器上面或什么时候安装。
        
官方定义
        
> package-lock.json is automatically generated for any operations where npm modifies either the node_modules tree, or package.json. It describes the exact tree that was generated, such that subsequent installs are able to generate identical trees, regardless of intermediate dependency updates. This file is intended to be committed into source repositories, and serves various purposes.
>
> 大概意思是, 当我们对node_modules或者package.json进行了更改后, package-lock.json文件会自动生成. 里面会描述上一次更改后的确切的依赖管理树. 里面包含了唯一的版本号和相关的包信息. 之后的npm install会根据package-lock.json文件进行安装, 保证不同环境, 不同时间下的依赖是一样的.
>
> 还有一个好处是, 由于package-lock.json文件中记录了下载源地址, 可以加快我们的npm install速度
        
因此，解决方案：

```
// 最简单直接方案，删除重装
 $ rm -rf node_modules

 $ rm package-lock.json

 $ npm i

// npm-check检查更新

 $ npm install -g npm-check

 $ npm-check

 $ npm-check -u
    
? Choose which packages to update. (Press <space> to select)
    
 Update package.json to match version installed.
 ❯◯ chalk     ^1.1.3   ❯  2.4.2   https://github.com/chalk/chalk#readme
  ◯ cheerio   ^0.22.0  ❯  0.22.0  https://github.com/cheeriojs/cheerio#readme
  ◯ debug     ^2.3.3   ❯  4.1.1   https://github.com/visionmedia/debug#readme
  ◯ log4js    ^1.0.1   ❯  4.1.0   https://log4js-node.github.io/log4js-node/
  ◯ mustache  ^2.3.0   ❯  3.0.1   https://github.com/janl/mustache.js
  ◯ request   2.79.0   ❯  2.88.0  https://github.com/request/request#readme
  ◯ unescape  ^0.2.0   ❯  1.0.1   https://github.com/jonschlinkert/unescape
  ◯ yargs     ^6.4.0   ❯  13.2.2  https://yargs.js.org/
    
       Space to select. Enter to start upgrading. Control-C to cancel.
 
// npm-upgrade更新
 $ npm install -g npm-upgrade

 $ npm-upgrade

// 更新全局包：
 $ npm update <name> -g

// 更新生产环境依赖包：
 $ npm update <name> --save

// 更新开发环境依赖包：
 $ npm update <name> --save-dev

// 升级package-lock.json里面的库包
 $ npm install XXX@x.x.x  

```

## yarn 是什么？

> Yarn发布于2016年10月，并在Github上迅速拥有了2.4万个Star(最近3.6万)。而npm只有1.2万个Star(最近1.7万)。这个项目由一些高级开发人员维护，包括了Sebastian McKenzie（Babel.js）和Yehuda Katz（Ember.js、Rust、Bundler等）。由Facebook、Google、Exponent 和 Tilde 联合推出的 JS 包管理工具 

### 为什么使用 yarn？

> Yarn 是为了弥补 npm 的一些缺陷而出现的。

- 离线模式
如果你以前安装过某个包，再次安装时可以在没有任何互联网连接的情况下进行。

- 确定性
不管安装顺序如何，相同的依赖关系将在每台机器上以相同的方式安装。

- 网络性能
Yarn 有效地对请求进行排队处理，避免发起的请求如瀑布般倾泻，以便最大限度地利用网络资源。

- 相同的软件包
从 npm 安装软件包并保持相同的包管理流程。

- 网络弹性
重试机制确保单个请求失败并不会导致整个安装失败。

- 扁平模式
将依赖包的不同版本归结为单个版本，以避免创建多个副本。

最开始的yarn公告是这么介绍yarn的安装的：
> *最简单的入门方法是运行： 
>
> npm install -g yarn 
>
> yarn*

现在的yarn安装页面是这么说的：

> 注意：通常情况下不建议通过npm进行安装。npm安装是非确定性的，程序包没有签名，并且npm除了做了基本的SHA1哈希之外不执行任何完整性检查，这给安装系统程序带来了安全风险。

基于这些原因，强烈建议你通过最适合于你的操作系统的安装方法来安装yarn。

##命令对比

常用命令示例

npm | yarn
---|---
npm install | yarn
npm install react --save | yarn add react
npm uninstall react --save | yarn remove react
npm install react --save-dev | yarn add react --dev
npm update --save | yarn upgrade 

更多参考[从 npm 迁移到 yarn](https://yarn.bootcss.com/docs/migrating-from-npm/)

## 总结

在npm v5 发布之后，两个依赖包管理工具的差别没有多大，甚至可以混合使用，只要心细，注意不要重复安装(使用yarn下载过的包，再使用npm cnpm下载 会重复下载，删除之前的包)，版本分清楚线上依赖还是开发依赖、lock内的版本是否一致等。

## 相关资料

> [Understanding differences between npm, yarn and pnpm](https://www.alexkras.com/understanding-differences-between-npm-yarn-and-pnpm/) 
>
> 作者：Alex Kras 翻译：雁惊寒
>
> npm 官网：https://www.npmjs.cn/
>
> yarn 官网：https://yarn.bootcss.com/


