[TOC]

## Webpack教程

### 目前状况
网站正在向 Web Apps 演变：
* 页面上的 Javascript 越来越多
* 在现代浏览器上用户可以做更多的事情
* 整个页面重新加载的情况更少了，与此同时，页面上的代码量更大了

结果就是：客户端的代码量变得越来越大，庞大的代码量意味着我们需要适当地组织代码，而 **模块系统** 则提供了把代码分割成不同模块的功能。

#### 模块系统实现的演变
关于定义依赖和导出接口有多种标准：
* Script 标签形式，多文件引入
* CommonJS
* AMD和一些变种实现
* ES6模块
* 其他

### 什么是Webpack
Webpack 是一个模块打包器。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。
![webapck](https://github.com/yiyunShm/NoteBook/tree/master/js/images/webpack.jpg)

### Webpack的特点
#### 代码拆分
Webpack 有两种组织模块依赖的方式，同步和异步。异步依赖作为分割点，形成一个新的块。在优化了依赖树后，每一个异步区块都作为一个文件被打包。

#### Loader
Webpack 本身只能处理原生的 Javascript 模块，但是 loader 转换器可以将各种类型的资源转换成 Javascript 模块。这样，任何资源都可以成为 Webpack 可以处理的模块。

#### 智能解析
Webpack 有一个智能解析器，几乎可以处理任何第三方库，无论它们的模块形式是 ComonJS、AMD 还是普通的 JS 文件。甚至在加载依赖的时候，允许使用动态表达式 `require('./tempaltes/' + name + '.jade')`

#### 插件系统
Webpack还有一个功能丰富的插件系统。大多数内容功能都是基于这个插件系统运行的，还可以开发和使用开源的 Webpack 插件，来满足各式各样的需求。

#### 快速运行
Webpack 使用异步 I/O 和多级缓存提高运行效率，这使得 Webpack 能够以令人难以置信的速度快速增量编译。

-----
### 安装
首先要安装 Node.js, Node.js 自带了软件包管理器 npm。用 npm 安装 Webpack: 
```
npm install webpack -g
```
此时 Webpack 已经安装到了全局环境，可以通过命令行 `webpack -h` 试试
通常我们会将 Webpack 安装到项目的依赖中，这样就可以使用项目本地版的 Webpack。
```
npm install webpack --save-dev
```
如果需要使用 Webpack 开发工具，要单独安装: 
```
npm install webpack-dev-server --save-dev
```

### Demo01: 单入口文件
首先创建一个静态页面 index.html 和一个 JS 入口文件 entry.js: 
```
// index.html
<html>
  <head>
    <meta charset="utf-8">
  </head>
  <body>
    <script src="bundle.js"></script>
  </body>
</html>

// entry.js
document.write('hello world');
```
添加 Webpack 配置文件 `webpack.config.js` 来打包生成 bundle.js
```
module.exports = {
  entry: './entry.js',
  output: {
    filename: 'bundle.js'
  }
};
```
登录服务器，访问 `http://127.0.0.1:8000`
```
webpack-dev-server
```

### Demo02: 多入口文件
允许页面有多 JS 文件作为入口执行:
```
// index.html
<html>
  <head>
    <meta charset="utf-8">
  </head>
  <body>
    <script src="bundle1.js"></script>
    <script src="bundle2.js"></script>
  </body>
</html>

// entry1.js
document.write('hello world');

// entry2.js
document.write('hello webpack');

// webpack.config.js
module.exports = {
  entry: {
    bundle1: './entry1.js',
    bundle2: './entry2.js'
  },
  output: {
    filename: '[name].js'
  }
};
```
`entry point` 是文件打包的入口点，如果传递的是一个字符串，那么这个字符串就会在初始化时被当作一个模块处理。如果传递的是一个字符串数组，所有的模块都立即加载，最后一个会被导出:
```
entry: ['./entry1', './entry2']
```
如果你传递的是一个对象，那就会创建多个入口，也就是打了多个包。键对应包的 `[name]` 值，值可以是一个字符串或者一个数组。

`output` 选项影响输出。如果你用任何哈希 `[hash]` 或者 `[chunkhash]`, 注意模块要有统一的顺序，使用 OccurenceOrderPlugin 或者 recordsPath。

### Demo03: Loader(加载器)
Loader 是对你的应用源文件进行转换的工具。比如说你可以用加载器来加载 coffeescript、css 或者 JSX。

Loader 的特征:
* 可以对源文件进行链式操作，最后一个加载器将返回 JS 文件，中间的加载器则返回一些特定格式的文件。
* 可以是异步操作，也可以是同步操作。
* 在 Nodejs 下跑。
* 可以接收查询参数，这一点可以用来向加载器传递配置参数。
* 可以在配置文件里限定文件后缀(正则表达)。
* 可以通过npm发布和下载。
* 可以access到配置文件。
* 插件可以给加载器更多更能

安装 Loader:
```
npm install style-loader css-loader babel-loader url-loader --save-dev
```
对之前的 Webpack 配置文件做修改:
```
module.exports = {
  entry: './entry.js',
  output: {
    filename: 'bundle.js'
  },
  module: {
    loaders:[
      { test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192' },
      { test: /\.css$/, loader: 'style-loader!css-loader' },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        query: {
          presets: ['es2015', 'react']
        }
      }
    ]
  }
}
```
Loader选型配置说明:
* test: 正则，用于匹配要处理的文件
* loader: 所使用的加载器
* exclude: 字符串或者数组,指屏蔽(不需处理)的文件/文件夹
* include: 字符串或者数组,指包含(必须处理)的文件/文件夹, 同exclude配置类似
* query: 为loaders提供额外的设置选项(可选)

### Demo04: Plugin(插件)
Webpack 包含一些内置插件如 DefinePlugin、UglifyJsPlugin、CommonsChukPlugin等，也可以通过npm下载引入:
```
npm install webpack html-webpack-plugin open-browser-webpack-plugin --save-dev
```
为 webpack.config.js 添加相应的插件处理: 
```
var webpack = require('webpack');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var OpenBrowserPlugin = require('open-browser-webpack-plugin');

module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    }),
    new HtmlWebpackPlugin({
      title: 'Webpack-demos'
    }),
    new OpenBrowserPlugin({
      url: 'http://localhost:8080'
    })
  ]
};
```

### Demo05: 代码分割
对于大型的 web 应用，将所有代码打包成一个文件并不是明智的做法。webpack 允许你将代码分割成多个小块，分别打包，然后按需加载。

代码分割是一个可选的功能，由你来决定分割点，webpack 会解决依赖、输出、运行时的问题，而且按需加载能使初始加载时间更短。

CommonJS:
```
require.ensure(dependencies, callback);
```
require.ensure 方法可以确保在调用callback的时候能同步加载，callback以require为参数。例:
```
// 注意：require.ensure只会加载模块，不会计算模块
require.ensure(['module-a', 'module-b'], function(require) {
  var a = require('module-a'),
      b = require('module-b');
      
  // ...
});
```

AMD:
```
require(dependenices, callback);
```
调用的时候所有依赖都被加载。例:
```
require(['module-a', 'module-b'], function(a ,b) {
  // ...
})
```

### Demo06: 对外暴露全局变量
如果你想在项目中使用一些全局变量，例如 jQuery 库的引入，有几种方法可以尝试，下面一个一个来看。

#### ProvidePlugin + expose-loader
ProvidePlugin 的机制是：当 webpack 加载到某个js模块里，出现了未定义且名称符合（字符串完全匹配）配置中 key 的变量时，会自动 require 配置中 value 所指定的js模块(npm管理)。

expose-loader 的作用是将指定 js 模块 export 的变量声明为全局变量。
```
var webpack = require('webpack');

module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  },
  module: {
    rules: [{
      test: require.resolve('jquery'),
      loader: 'expose-loader?jQuery'
    }]
  },
  plugins: [
    new webpack.ProvidePlugin({
      $: 'jquery',
      jQuery: 'jquery',
      'window.jQuery': 'jquery',
      'window.$': 'jquery'
    })
  ]
};
```
当然，如果所有 jQuery 插件都是通过 Webpack 来加载的话，只要 ProvidePlugin 就足够了；然后在某些情况下，有些需求是只能用 `<script>` 来加载的，所以也就有了 expose-loader.

#### webpack.config属性-externals
externals 是 Webpack 配置的属性之一，用来将某个全局变量"伪装"成某个 js 模块然后对外暴露:
```
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  },
  module: {
    rules: [{
      test: /\.js$/,
      loader: 'babel-loader'
    }]
  },
  externals: {
    'jquery': 'window.jQuery'
  }
};

// entry.js
var $ = require('jquery');

// jquery.js 和 entry.js在同级目录
```

### 相关链接
* [Webpack docs](http://webpack.github.io/docs/)
* [webpack-howto](https://github.com/petehunt/webpack-howto), by Pete Hunt
* [Diving into Webpack](https://web-design-weekly.com/2014/09/24/diving-webpack/), by Web Design Weekly
* [Browserify vs Webpack](https://medium.com/@housecor/browserify-vs-webpack-b3d7ca08a0a9), by Cory House