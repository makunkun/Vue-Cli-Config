# vue-cli生成的项目主要配置文件
## 执行npm run dev 和npm run build时做了什么
*当我们执行npm run dev时，首选执行的是dev-server.js*
*当我们执行npm run build时，首选执行的是build.js*
## dev-server.js

* 引入配置文件
* 引入相关插件
* 创建EXPRESS实例
* 配置WEBPACK-DEV-MIDDLEWARE和WEBPACK-HOT-MIDDLEWARE
* 配置静态资源路径，并挂到EXPRESS服务上
* 启动服务器，并判断是否自动打开默认浏览器
* 监听端口

```
require('./check-versions')()
//引入相关配置
var config = require('../config')
// 检查Node的环境变量，如果没有则使用配置文件中设置的环境
if (!process.env.NODE_ENV) {
process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}

//opn -- A better node-open. Opens stuff like websites, files, executables. Cross-platform.
//这里用来打开默认浏览器，打开dev-server监听的端口
var opn = require('opn')
var path = require('path')
var express = require('express')
var webpack = require('webpack')
//express中间件，用于http请求代理到其他服务器
var proxyMiddleware = require('http-proxy-middleware')
//判断当前环境，选择导入的webpack配置
var webpackConfig = process.env.NODE_ENV === 'testing'
? require('./webpack.prod.conf')
: require('./webpack.dev.conf')

// default port where dev server listens for incoming traffic
//默认的dev-server的监听端口
var port = process.env.PORT || config.dev.port
// automatically open browser, if not set will be false
//是否自动打开浏览器，默认是false
var autoOpenBrowser = !!config.dev.autoOpenBrowser
// Define HTTP proxies to your custom API backend
// https://github.com/chimurai/http-proxy-middleware
//定义http代理到你的自定义的API后端
var proxyTable = config.dev.proxyTable
//创建express实例
var app = express()
// 根据webpack的config创建Compiler对象
var compiler = webpack(webpackConfig)

//使用compiler相应的文件进行编译和绑定，编译后的内容将存放在内存中
var devMiddleware = require('webpack-dev-middleware')(compiler, {
publicPath: webpackConfig.output.publicPath,
quiet: true
})
//用于实现热重载
var hotMiddleware = require('webpack-hot-middleware')(compiler, {
log: false,
heartbeat: 2000
})
// force page reload when html-webpack-plugin template changes
//当html-webpack-plugin提交页面之后，使用热重载强制页面重载
compiler.plugin('compilation', function (compilation) {
compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
hotMiddleware.publish({ action: 'reload' })
cb()
})
})

// proxy api requests
//在express上使用代理表中的配置
Object.keys(proxyTable).forEach(function (context) {
var options = proxyTable[context]
if (typeof options === 'string') {
options = { target: options }
}
app.use(proxyMiddleware(options.filter || context, options))
})

// handle fallback for HTML5 history API
//一个解决单页的重定向错误的插件
app.use(require('connect-history-api-fallback')())

// serve webpack bundle output
// 使用devMiddleware，webpack编译后的文件将挂到express服务器上
app.use(devMiddleware)

// enable hot-reload and state-preserving
// compilation error display
// 使用热重载中间件
app.use(hotMiddleware)

// serve pure static assets
//配置静态资源路径
var staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)
//将静态文件挂到express服务器上
app.use(staticPath, express.static('./static'))
//设置应用的url
var uri = 'http://localhost:' + port

var _resolve
var readyPromise = new Promise(resolve => {
_resolve = resolve
})

console.log('> Starting dev server...')
//devMiddleware valid之后，启动服务
devMiddleware.waitUntilValid(() => {
console.log('> Listening at ' + uri + '\n')
// when env is testing, don't need open it
//如果设置为自动打开浏览器，通过opn打开uri
if (autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
opn(uri)
}
_resolve()
})
//监听配置的端口
var server = app.listen(port)

module.exports = {
ready: readyPromise,
close: () => {
server.close()
}
}
```
## webpack.base.conf.js

* 配置编译入口和输出路径
* 模块resolve的规则
* 配置不同类型模块的处理规则

```
var path = require('path')
var utils = require('./utils')
var config = require('../config')
var vueLoaderConfig = require('./vue-loader.conf')
//绝对路径
function resolve (dir) {
return path.join(__dirname, '..', dir)
}

module.exports = {
//webpack的入口文件
entry: {
app: './src/main.js'
},
output: {
//webpack输出文件的路径
path: config.build.assetsRoot,
//输出的文件命名格式
filename: '[name].js',
// webpack编译输出的发布路径
publicPath: process.env.NODE_ENV === 'production'
? config.build.assetsPublicPath
: config.dev.assetsPublicPath
},
//模块resolve的规则
resolve: {
//resolve的后缀名
extensions: ['.js', '.vue', '.json'],
//配置路径别名，比如import Vue from 'vue/dist/vue.common.js'--> import Vue from 'vue'
alias: {
'vue$': 'vue/dist/vue.esm.js',
'@': resolve('src')
}
},
//配置不同类型模块的处理规则
module: {
rules: [
// src和test文件夹下的.js和.vue文件使用eslint-loader
{
test: /\.(js|vue)$/,
loader: 'eslint-loader',
enforce: 'pre',
include: [resolve('src'), resolve('test')],
options: {
formatter: require('eslint-friendly-formatter')
}
},
//所有的.vue文件使用vue-loader
{
test: /\.vue$/,
loader: 'vue-loader',
options: vueLoaderConfig
},
//src和test下的.js文件使用babel-loader
{
test: /\.js$/,
loader: 'babel-loader',
include: [resolve('src'), resolve('test')]
},
//所有的图片文件使用url-loader
{
test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
loader: 'url-loader',
options: {
limit: 10000,
name: utils.assetsPath('img/[name].[hash:7].[ext]')
}
},
//所有的音频文件使用url-loader
{
test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
loader: 'url-loader',
options: {
limit: 10000,
name: utils.assetsPath('media/[name].[hash:7].[ext]')
}
},
//所有的字体文件使用url-loader
{
test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
loader: 'url-loader',
options: {
limit: 10000,
name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
}
}
]
}
}
```

## webpack.dev.conf.js

* 合并基础的webpack配置
* 使用styleLoaders
* 配置Source Maps
* 配置webpack插件

```
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
//生成html文件并自动注入依赖文件的插件， script & link
var HtmlWebpackPlugin = require('html-webpack-plugin')
//一个输出webpack警告，错误的插件
var FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')

// add hot-reload related code to entry chunks
//添加热重载相关的代码到entry chunks
Object.keys(baseWebpackConfig.entry).forEach(function (name) {
baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])
})
//合并webpack.base.conf
module.exports = merge(baseWebpackConfig, {
module: {
//使用styleLoaders处理样式文件
rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap })
},
// cheap-module-eval-source-map is faster for development
//配置Source Maps

devtool: '#cheap-module-eval-source-map',
//配置webpack插件
plugins: [
new webpack.DefinePlugin({
'process.env': config.dev.env
}),
// https://github.com/glenjamin/webpack-hot-middleware#installation--usage
new webpack.HotModuleReplacementPlugin(),
//在编译出现错误时，使用 NoEmitOnErrorsPlugin 来跳过输出阶段。这样可以确保输出资源不会包含错误。
new webpack.NoEmitOnErrorsPlugin(),
// https://github.com/ampedandwired/html-webpack-plugin
new HtmlWebpackPlugin({
filename: 'index.html',
template: 'index.html',
inject: true
}),
new FriendlyErrorsPlugin()
]
})
```
** 配置Source Maps (devtool)**
* source-map
>在一个单独的文件中产生一个完整且功能完全的文件。这个文件具有最好的source-map，但是它会减慢打包文件的构建速度；
* cheap-module-source-map
>在一个单独的文件中生成一个不带列映射的map，不带列映射提高项目构建速度，但是也使得浏览器开发者工具只能对应到具体的行，不能对应到具体的列（符号），会对调试造成不便；
* eval-source-map
>使用eval打包源文件模块，在同一个文件中生成干净的完整的source
map。这个选项可以在不影响构建速度的前提下生成完整的sourcemap，但是对打包后输出的JS文件的执行具有性能和安全的隐患。不过在开发阶段这是一个非常好的选项，但是在生产阶段一定不要用这个选项；
* cheap-module-eval-source-map 
> 这是在打包文件时最快的生成source map的方法，生成的Source
Map 会和打包后的JavaScript文件同行显示，没有列映射，和eval-source-map选项具有相似的缺点；

## build.js
* webpack编译
* 输出信息

```
require('./check-versions')()
process.env.NODE_ENV = 'production'
//控制台loading动画
var ora = require('ora')
var rm = require('rimraf')
var path = require('path')
// 高亮控制台输出的插件
var chalk = require('chalk')
var webpack = require('webpack')
var config = require('../config')
var webpackConfig = require('./webpack.prod.conf')
//在控制台输出building for production...
var spinner = ora('building for production...')
//开始loading动画
spinner.start()
//获取输出文件路径
rm(path.join(config.build.assetsRoot, config.build.assetsSubDirectory), err => {
  if (err) throw err
  //webpack编译  
  webpack(webpackConfig, function (err, stats) {
    //停止loading动画
    spinner.stop()
    //如果错误抛出错误
    if (err) throw err
    //完成输出相关信息 
    process.stdout.write(stats.toString({
      colors: true,
      modules: false,
      children: false,
      chunks: false,
      chunkModules: false
    }) + '\n\n')

    console.log(chalk.cyan('  Build complete.\n'))
    console.log(chalk.yellow(
      '  Tip: built files are meant to be served over an HTTP server.\n' +
      '  Opening index.html over file:// won\'t work.\n'
    ))
  })
})
```
## webpack.prod.conf.js
- 合并基础的webpack配置
- 配置webpack的输出
- 配置webpack插件
- 配置gzip模式
- 配置webpack-bundle-analyzer，分析打包后生成的文件结构

```
var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var CopyWebpackPlugin = require('copy-webpack-plugin')
var HtmlWebpackPlugin = require('html-webpack-plugin')
// 抽取css，js文件,与webpack输出的bundle分离
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')

var env = config.build.env
// 合并webpack.base.conf
var webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  devtool: config.build.productionSourceMap ? '#source-map' : false,
  output: {
    publicPath:'https://inhdres.huisou.cn/webapp1/',
   // 配置输出路径
    path: config.build.assetsRoot,
    // 输出的文件命名格式
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    // 未指定文件名的文件的文件名格式
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
    // 相关插件
  plugins: [
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
    new webpack.DefinePlugin({
      'process.env': env
    }),
    // 压缩js插件
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      sourceMap: true
    }),
    // extract css into its own file
     // 从bundle中抽取css文件
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    // Compress extracted CSS. We are using this plugin so that possible
    // duplicated CSS from different components can be deduped.
    // 压缩抽取的css文件
    new OptimizeCSSPlugin(),
    // generate dist index.html with correct asset hash for caching.
    // you can customize output by editing /index.html
    // see https://github.com/ampedandwired/html-webpack-plugin
    //用于生成dist/index.html，加入hash用于缓存。hash不改变不进行更新
    new HtmlWebpackPlugin({
      filename: config.build.index,
      template: 'index.html',
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
      chunksSortMode: 'dependency'
    }),
    // split vendor js into its own file
    // 分离第三方js到单独的文件中
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
    }),
    // copy custom static assets
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})
// 配置gzip模式
if (config.build.productionGzip) {
  var CompressionWebpackPlugin = require('compression-webpack-plugin')

  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}
//配置webpack-bundle-analyzer，分析打包后生成的文件结构
if (config.build.bundleAnalyzerReport) {
  var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}

module.exports = webpackConfig

```
## config/index.js
```
var path = require('path')

module.exports = {
    //打包时使用的配置
  build: {
    //webpack的环境
    env: require('./prod.env'),
    //输入的index.html的路径
    index: path.resolve(__dirname, '../dist/index.html'),
    //输出的目标文件夹路径
    assetsRoot: path.resolve(__dirname, '../dist'),
    //输出的子文件夹路径
    assetsSubDirectory: 'static',
    //发布路径
    assetsPublicPath: '/',
    //是否使用SourceMap
    productionSourceMap: false,
    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    // 是否开启Gzip
    productionGzip: false,
    //Gzip的压缩文件类型
    productionGzipExtensions: ['js', 'css'],
    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  },
  //开发时使用的配置
  dev: {
    //webpack环境
    env: require('./dev.env'),
    // 端口
    port: 8282,
    // 主机名
    host:'localhost',
    //是否自动打开浏览器
    autoOpenBrowser: true,
    //输出的子文件夹路径
    assetsSubDirectory: 'static',
    //发布路径
    assetsPublicPath: '/',
    //配置代理表
    proxyTable: {},
    // CSS Sourcemaps off by default because relative paths are "buggy"
    // with this option, according to the CSS-Loader README
    // (https://github.com/webpack/css-loader#sourcemaps)
    // In our experience, they generally work as expected,
    // just be aware of this issue when enabling this option.
    cssSourceMap: false
  }
}
```