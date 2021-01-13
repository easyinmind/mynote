# webpack原理、loader、plugin
## webpack及模块化

技术发展的过程中，前端项目变得越来越复杂，通过模块化的思想组织维护复杂的项目是现在的趋势。
把复杂的代码通过功能划分为不同的模块进行维护，来提升开发效率和降低维护成本。  
webpack的本质还是一个模块化打包工具，webpack 把项目中的资源都看作是模块，所有的模块经过处理，再打包到一起。  

技术发展业务需求=》项目复杂=》模块化方案解决=》带来的问题=》工具解决=》webpack=》模块代码合并/不同模块资源处理/适合生产环境（兼容宿主、合理分包输出）

### 模块化的发展及问题

模块化思想最早通过组织文件规范、命名空间、闭包到AMD、CMD、commonjs、ESM模块化规范，现在依旧存在一些问题。
+ 如何将散落的模块组合到一起
+ 如何编译代码中的新特性兼容不同宿主环境
+ 支持不同类型的前端资源  

现在前端有Webpack、Parcel 和 Rollup都能解决上述的问题
#### webpack是如何解决的  
+ webpack作为模块打包工具，本身就可以将分散的js打包到一起
+ 通过loader机制可以对代码编译兼容不同宿主环境
+ webpack支持在js中以模块化的方式加载任意资源（import 'xx.css'...）
+ 同时webpack支持拆分的能力，可以根据生产环境的需要自定义拆分代码及资源，如将除此加载的模块单独打包到一起，其它模块等需要的时候再异步加载实现增量加载。这样在开发环境下可以通过模块化的方式开发，在生产环境下可以拆分代码，不至于所有模块打包成一个文件造成加载慢，也不至于因模块化的划分加载n多的模块。

### webpack的开发流程简述
webpack 可以通过配置文件定义打包过程。 通过vscode编写配置时，在配置文件增加 `/** @type {import('webpack').Configuration} */`注释可以实现智能提示。  
`mode` 参数简化了很多配置，预设了一些配置项，可以通过 CLI --mode 参数传入或者配置文件mode属性
+ production 自动优化了打包结果 （默认值）
+ development 自动优化打包速度
+ none 不做任何处理

webpack 是通过loader处理每个模块的资源，默认只能处理js模块，其它资需要配置不同的loader处理，再交由webpack。loader是串行处理文件的 loader1->loader2->loader3->webpack。通过在js里引入加载其它类型的资源模块，webpack会递归这些依赖，交由对应的loader处理。负责完成项目中各种各样资源模块的加载，从而实现整体项目的模块化。  
为何是在js里引入其它资源模块？  
Webpack 建议我们在 JavaScript 中引入 当前业务所需要的任意资源文件。因为真正需要这个资源的并不是整个应用，而是你此时正在编写的代码。--Webpack 的设计哲学

### 如何开发一个loader
webpack loader具有一些特性，很简单
+ loader需要返回一个函数（这个函数就是最资源的处理过程）
+ 函数 输入的（形参）就是加载到的资源文件内容，输出的（return）就是加工过的内容
+ 函数返回的内容可以是任意类型的，但是在loader串行运行(工作管道)完成交于webpack的时候必须是一段标准的可运行的js代码字符串如：`'console.log("hello loader ~")'`
+ 如果js代码字符串需要导出 支持 `module.exports` 和 `export default`

一个简单的注释console的 loader 
```js
module.exports = (source) => {
  const code = source.replace('console.log', '// console.log')
  return code
}
```

### plugin 机制
plugin机制 增强 webpack 在项目中自动构建方面的能力。借助plugin是webpack的能力范围更广，能实现前端工程化方面的大多数功能。
通常需要某方面功能的时候可以寻找相关插件，如清除dist构建目录，生成html，压缩打包出的文件等。相比较loader只在加载环节工作，plugin的工作范围更广，涉及到webpack运行的整个周期。  
如果需要自定义功能，可以自己开发一个 plugin。webpack为了扩展插件，在其运行的每个环节都埋下了一个钩子，在开发插件的时候可以根据功能的不同，在相应的钩子上挂载不同的功能来拓展webpack的能力。  
webpack plugin也有一些特性
+ 必须是一个构造函数或者具有`apply方法`的类（一般会定义一个类）
+ `apply函数`的形参 `compiler` 包含了本次构建的所有 配置信息
+ 通过 `compiler.hooks.钩子名.tap` 挂载功能
+ `tap` 方法 接受两个参数，第一个是 `插件的名称` ，第二个是要挂载到这个钩子上的`功能函数`
+ 在这个`功能函数`上可以接收到形参 `compilation 对象`
+ `compilation对象` 是本次构建的上下文，通过操作这个上下文对象实现功能

一个插件模板  
```js
class XxxPlugin {
  concurstor(options) {
    this.options = options
  }
  apply(compiler) {
    compiler.hooks.emit.tap('XxxPlugin', (compilation) => {
      
    })
  }
}
```

## webpack 原理简述

### 打包过程
1. webpack启动，找到入口文件（一般为js文件）
2. 根据入口代码里出现的 import、require 之类的语句解析推断出入口文件依赖的资源模块
3. 解析每个资源模块的依赖，生成 依赖关系树
4. 递归依赖关系树，找到每个节点对应的资源文件，根据loader的配置交由对应的 loader加载这个模块
5. 将loader处理后的结果放入 打包结果中（bundle.js）
6. 插件不会影响整个构建过程的运行，只是在不同的运行环节增加了功能，扩展 Webpack打包功能以外 的能力


### 原理解析
待更新...

