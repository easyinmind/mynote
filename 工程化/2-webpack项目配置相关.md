# webpack
基于webpack5 + react hooks + ts + less 项目搭建流程

搭建过程中会描述各项配置的作用，以及一些常用点的讲解

## 如何理解 webpack 
webpack是一个模块化打包工具，解决复杂前端项目的一些，主要实现了
+ 提升本地开发效率
  + 自动刷新、热更新（HMR）、接口代理
+ 可以通过loader解决兼容及框架相关等问题
  + postcss 解决样式兼容
  + babel 解决JS兼容及支持react和ts
  + 支持css预编译器
  + 通过其他loader或自定义loader更多问题
+ 解决生产环境下的一些问题
  + 代码压缩
  + 代码分割
  + ……
+ 模块化开发
  + 默认支持js模块（ESM、CommonJs、AMD、CMD）
  + webpack把一切视为模块，统一模块化开发方案（样式、图片、字体等所有资源文件都作为模块）
+ 提供插件系统，可以扩展更多自定义功能
+ 待补充

webpack 把一切都看作模块，通过 loader 和 plugin 对不同的文件类型进行转换、压缩、合并、分割、优化等操作输出适于生产环境运行的代码


Webpack和另外两个并没有太多的可比性，Gulp/Grunt是一种能够优化前端的开发流程的工具，而WebPack是一种模块化的解决方案，不过Webpack的优点使得Webpack在很多场景下可以替代Gulp/Grunt类的工具
## 项目搭建

### 一、安装 `webpack` 及 `webpack-cli`
```
npm i webpack webpack-cli -D
```

可以全局安装-G，建议安装在项目内（不同项目版本可能有区别，减少对项目的影响）

默认配置：
+ 入口：`src/index.js`
+ 出口：`dist/main.js`
+ mode： `production` (开启压缩和优化)

运行：`npx webpack` node v8.2版本以后都会有一个npx，npx会执行bin里的文件（webpack）

关于 `mode`：

默认值： `production`

设置mode可以改变 process.env.NODE_ENV 的值

配置文件
```
module.exports = {
  mode: 'production'
};
```
命令行
```
webpack --mode=production
```
+ development： 会将 DefinePlugin 中 process.env.NODE_ENV 的值设置为 development. 为模块和 chunk 启用有效的名（会开启一些插件）
+ production：会将 DefinePlugin 中 process.env.NODE_ENV 的值设置为 production 。为模块和 chunk 启用确定性的混淆名称，并开启其它的一些生产环境的优化（会开启一些插件）
+ none：不使用任何默认优化选项
+ 不同 mode 对应的配置、插件可查看文档

关于 `context` 上下文:

默认值是 `process.cwd()` （当前node执行的文件夹地址），推荐将 context 修改为项目的根目录，这样可以使得项目配置独立于CWD

context 是 webpack 编译时的基础目录，是一个绝对路径的字符串，context 用于解析配置中的entry、loader、plugins中的相对路径（除了output），配置方式同 `mode` 参数

### 二、增加配置文件

默认运行 `webpack` 会去查找 webpack.congif.js 配置文件，如果没有这个文件会使用默认值，可以通过如下方式指定配置文件：

命令行（推荐这种方式）
```
webpack --config xxx.js
```
或者在 webpack.congif.js 通过环境变量指定配置

### 三、配置入口（`enry`）及出口（`output`）

命令行
```
webpack 入口 出口
```

配置文件
```js
module.exports = {
  // 单入口 => 单输出（一个chunk，SPA）
  entry: "", 
  // 多入口 => 单输出（一个chunk， SPA）
  entry: [], 
  // 多入口 => 多输出（多个chunk， MPA）
  entry: {}, 
  

  output: {
    // entry 里的输出文件名
    filename: "[name].min.js",
    // 非entry 里的输出文件名
    chunkFilename: 'bundle.js',
    // 输出的文件目录
    path: path.join(__dirname, "/dist"),
    // 个人理解，文件引用的前缀（如： 设置为CDN地址等）
    publicPath: './', 
  },
};
```
这里涉及到一些 chunk 及 占位符的概念，后面会讲到

### 四、配置样式相关的 loader

webpack 默认识别js文件，但不认识css、less等文件，所以需要使用一些loader进行处理，这里不写loader的基础配置（具体配置可查看配置文件或文档），只关心一些常用配置点和注意

```
npm i style-loader css-loader less less-loader postcss postcss-loader -D
```

+ `less-loader sass-loader stylus-loader postcss-loader` 等css预处理器loader需要安装对应的预处理器 `less sass stylus postcss`

+ `less-loader` 支持less文件
+ `postcss-loader`
  + 放在 预编译器之后，css-loader之前执行
  + 可以使用配置文件
  + 通过插件增加浏览器前缀，postcss-preset-env 包含 autoprefixer
+ `css-loader` 用来处理@import、url()语法（把 css 代码打包进 js 文件而已，并不做任何操作）
  + modules css模块化配置
  + importLoaders css-loader 之前有多少 loader 应用于@import
+ `style-loader` 将css插入dom中

### 四、配置图片及字体文件相关 loader
支持图片、字体等文件

```
npm i url-loader -D
```

url-loader 功能类似于 file-loader，增加了 `limit` 属性，可以设置返回 DataURL，用来减少http请求。这两个使用一个即可

### 五、配置 babel 支持 ES6, JSX, TS

主要是babel相关的用法，配置可参考项目内，babel更多详细内容 会另写一篇文章
+ babel 一些概念，核心包、预设、插件等
+ babel 和 webpack 的区别
+ babel-runtime、polyfil
+ 自定义插件
```
npm i babel-loader @babel/core @babel/preset-env @babel/preset-react @babel/preset-typescript -D
```
配置 `tsconfig.json`


### 六、配置本地开发环境 webpack-dev-serve

如果不进行文件监听配置，在本地开发环境下，每次修改代码都需要手动打包刷新浏览器，常用本地开发配置主要通过下面几种方式：
+ watch监听 `webpack --watch`  
  开启这个配置会监听文件配置，自动打包（需要手动刷新浏览器、通过file协议访问）
+ 通过 webpack-dev-server + devServer

  ```
  npm i webpack-dev-server
  ```

  ```js
  devServer: {
    // ...
  }
  ```  
+ 通过 webpack-dev-middleware + 自定义node服务配置  
  自己写一个webpack-dev-server

通常使用 webpack-dev-server，使用本地服务方式，打包文件放在了内存中，提升速度，并且可以使用 ajax请求数据，代理接口

关于 `HMR`  
配置插件 `webpack.HotModuleReplacementPlugin()`及 devServer.hot 开启，需要增加自定义代码，开启热更新有代价，依据情况选择

```js
if (module.hot) {
   module.hot.accept('./print.js', function() {
    //  执行需要更新的代码
   })
 }
```

更多关于本地开发环境的配置，单独进行总结

### 七、安装相关插件

+ html-webpack-plugin 生成HTML，并插入资源
+ clean-webpack-plugin 删除output的输出目录内容
+ mini-css-extract-plugin 提取css
+ optimize-css-assets-webpack-plugin 压缩css
+ terser-webpack-plugin 使用 terser 来压缩 JavaScript，（production 默认开启）

### 八、区分环境

+ 配置npm script
+ webpack-merge 拆分