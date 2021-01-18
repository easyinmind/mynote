# webpack
1. 常用属性
2. 常用配置
3. 高级特性
4. 性能优化
5. babel相关

## 常用属性

1. `mode`  
  `development`: 会将 DefinePlugin 中 process.env.NODE_ENV 的值设置为 development. 为模块和 chunk 启用有效的名。 
  `production`: 会将 DefinePlugin 中 process.env.NODE_ENV 的值设置为 production。开启一些优化（代码压缩等）。`none`: 不使用任何默认优化选项。
  设置 `NODE_ENV` 并不会自动地设置 mode

2. `context`  
   用于从配置中解析入口点(entry point)和 加载器(loader)

3. `entry`  (产出 `chunk` )  
   string：单入口 => 单页应用 （输出一个文件，chunk）  
   array：多入口 => 单页应用 （输出一个文件，chunk）可以通过动态链的方式（Dll）打包第三方库  
   object：多入口 => 多页应用 （输出多个文件，chunk）  

4. `output`  
   `filename`： 输出的 chunk名 （对应 entry 中定义的module）  
   `chunkFilename`： 输出的 chunk名（非entry 中的输出，懒加载产生的 chunk），默认值取 `[id].`  
   `path`：输出文件的目录  
   `publicPath`： 简单理解，增加前缀（可设置为CDN），在输出的文件名前面增加前缀，这些前缀可以是路径。
5. `module`  
   `noParse`： 不去解析某些第三方库（被忽略的库不能有其他依赖）  
   `unsafeCache`：缓存模块，默认 false，如果为true，并且是node_modules里的模块，则开启
   `rules`： 配置 loader 处理非 js 文件
6. `resolve`  设置模块如何被解析  
   `alias`： 配置别名  
   `extensions`：配置忽略后缀规则，会按照数组顺序解析，如果名字相同后缀不同，则使用数组第一项
7. `optimization`  
   `minimize`： 告知 webpack 使用 TerserPlugin 或其它在 optimization.minimizer 定义的插件压缩 bundle  
   `minimizer`： 提供一个或多个定制过的 TerserPlugin 实例， 覆盖默认压缩工具(minimizer)  
   `splitChunks`: 提取公共代码，防止代码被重复打包，拆分过大的js文件
8. `plugins`：配置插件，值为数组
9. `devtools`：控制source map的生成及格式
10. `externals`：打包的时候排除一些模块，通过手动方式使用。请不要将这个模块注入编译后的js文件里，并保留源码内import/require这个模块的语句。（如jq，lodash等）

## 常用配置

### 1、支持样式文件
+ style-loader
+ css-loader
+ postcss-loader（autoprefixer）
+ less-loader

### 2、支持ES+、JSX、TS文件
+ babel-loader
+ babel及相关预设 @babel/core @babel/preset-env @babel/preset-react @babel/preset-typescript

### 3、支持图片、字体等文件
+ file-loader
+ url-loader

### 4、本地开发环境配置
+ 使用插件 webpack-dev-server
+ 配置 devServer

### 5、常用 plugin
+ html-webpack-plugin 生成html
+ mini-css-extract-plugin 拆分css（替代style-loader）
+ optimize-css-assets-webpack-plugin 压缩css
+ terser-webpack-plugin 压缩js （production默认开启）
+ clean-webpack-plugin 清除打包目录（output定义的输出目录）
+ copy-webpack-plugin 复制文件到打包目录（icon文件等）
+ imagemin-webpack-plugin 批量压缩图片


## 高级特性

### 1、多入口配置
entry 配置为对象，使用多个 html-webpack-plugin 实例对应每个entry，需注意设置插件的chunks属性指定每个页面引入的js

### 2、抽取及压缩css
使用插件 mini-css-extract-plugin 及 MiniCssExtractPlugin.loader 替代style-loader 抽取css  
在 optimization.minimizer 中使用 optimize-css-assets-webpack-plugin 压缩css
### 3、code splitting及懒加载
optimization.splitChunks 可以配置拆包规则。Dynamic import（import()） 可以实现懒加载， webpack会把这种语法单独打包成一个 chunk
### 4、Tree Shaking
默认 production 会开启 Tree Shaking，如果需要手动开启，设置 optimization.usedExports: true，
### 5、Library 的打包
使用 externals 避免其他第三方包的打包（如果有），设置 output.library 暴露名称，设置 output.libraryTarget 可以引入的方式

## 性能优化

### 构建速度
+ 缓存 二次构建速度
  + 优化babel-loader（默认支持缓存）
  + 使用cache-loader 优化不支持缓存的loader（写在use最前面）
  + DllPlugin 配置动态链接 减少二次不必要打包
+ 多核打包 提升构建速度
  + thread-loader
  + happypack
  + 多核压缩 
+ 减少不必要的构建过程
  + IgnorePlugin 忽略三方包的指定目录，指定目录不会被打包进去
  + 使用 Loader 时可以通过 test 、 include 、 exclude 三个配置项来命中 Loader 要应用规则的文件
  + module.noParse 忽略对部分没采用模块化的文件的递归解析处理
  + resolve.extensions 
    + 列表要尽可能的小，不要把项目中不可能存在的情况写到后缀尝试列表中
    + 频率出现最高的文件后缀要优先放在最前面，以做到尽快的退出寻找过程
    + 在源码中写导入语句时，要尽可能的带上后缀，从而可以避免寻找过程
  + resolve.alias 通过别名来把原导入路径映射成一个新的导入路径，减少耗时的递归解析操作
+ 避免手动重复操作
  + 自动刷新
  + 热更新

### 产出代码
+ 开启生产环境
  + 默认会开启压缩、tree sharking等优化
+ 配置 CDN
+ 公用代码提取、合理分包
+ 懒加载
+ bundle + hash 利用浏览器缓存
+ scope hosting 将多个模块打包到一个函数 减少体积
+ 小图片偏使用base64 减少http