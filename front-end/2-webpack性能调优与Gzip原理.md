# webpack性能调优与Gzip原理

## webpack 的性能瓶颈

- 构建时间长

- 打包体积太大

## webpack 优化方案

### 构建过程提升策略

#### 不要让loader做太多事情 - 以babel-loader为例：

```js
module: {
  rules: [
    {
      test: /\.js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env']
        }
      }
    }
  ]
}
```

利用exclude将庞大的node_modules排除在外。除此之外，可以开启缓存，提升babel-loader的工作效率。

```js
loader: 'babel-loader?cacheDirectory=true'
```

#### 第三方库

处理第三方库的插件有很多，Externals一般会引发重复打包，CommonsChunkPlugin每次都重新构建一次vendor。推荐DllPlugin插件，会将第三方库单独打包到一个文件中，这个文件是一个单纯的依赖库。这个文件不会随着业务代码一起重新打包，只有依赖自身版本发生变化才会重新打包。

具体配置如下：

```js
const path = require('path')
const webpack = require('webpack')

module.exports = {
    entry: {
      // 依赖的库数组
      vendor: [
        'prop-types',
        'babel-polyfill',
        'react',
        'react-dom',
        'react-router-dom',
      ]
    },
    output: {
      path: path.join(__dirname, 'dist'),
      filename: '[name].js',
      library: '[name]_[hash]',
    },
    plugins: [
      new webpack.DllPlugin({
        // DllPlugin的name属性需要和libary保持一致
        name: '[name]_[hash]',
        path: path.join(__dirname, 'dist', '[name]-manifest.json'),
        // context需要和webpack.config.js保持一致
        context: __dirname,
      }),
    ],
}
```

编译完成会生成如下两个文件：

```
vendor-manifest.json
vendor.js
```

vendor.js是第三方打包的结果，vendor-manifest.json是第三方库对象的具体路径，内容如下：

```
{
  "name": "vendor_397f9e25e49947b8675d",
  "content": {
    "./node_modules/core-js/modules/_export.js": {
      "id": 0,
        "buildMeta": {
        "providedExports": true
      }
    },
    "./node_modules/prop-types/index.js": {
      "id": 1,
        "buildMeta": {
        "providedExports": true
      }
    },
    ...
  }
}
```

随后只需在webpack.config中针对dll进行配置：

```js
const path = require('path');
const webpack = require('webpack')
module.exports = {
  mode: 'production',
  // 编译入口
  entry: {
    main: './src/index.js'
  },
  // 目标文件
  output: {
    path: path.join(__dirname, 'dist/'),
    filename: '[name].js'
  },
  // dll相关配置
  plugins: [
    new webpack.DllReferencePlugin({
      context: __dirname,
      // manifest就是我们第一步中打包出来的json文件
      manifest: require('./dist/vendor-manifest.json'),
    })
  ]
}
```

Happypack - 将loader转换为多进程

webpack是单线程的，Happypack会将任务分解为多个子进程去并发执行，提升打包效率。

```js
const HappyPack = require('happypack')
// 手动创建进程池
const happyThreadPool =  HappyPack.ThreadPool({ size: os.cpus().length })

module.exports = {
  module: {
    rules: [
      ...
      {
        test: /\.js$/,
        // 问号后面的查询参数指定了处理这类文件的HappyPack实例的名字
        loader: 'happypack/loader?id=happyBabel',
        ...
      },
    ],
  },
  plugins: [
    ...
    new HappyPack({
      // 这个HappyPack的“名字”就叫做happyBabel，和楼上的查询参数遥相呼应
      id: 'happyBabel',
      // 指定进程池
      threadPool: happyThreadPool,
      loaders: ['babel-loader?cacheDirectory']
    })
  ],
}
```

### 构建体积压缩

#### 文件结构可视化，找出体积过大原因

webpack-bundle-analyzer是一个包可视化工具。可以以插件形式引入：

```js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
 
module.exports = {
  plugins: [
    new BundleAnalyzerPlugin()
  ]
}
```

#### 拆分资源

删除冗余代码，利用Tree-Shaking技术识别未被使用的代码，并在打包的时候去掉。

Tree-Shaking是适合处理模块级别的代码，粒度更细的可以用UglifyJsPlugin去除。配置如下：

```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
module.exports = {
 plugins: [
   new UglifyJsPlugin({
     // 允许并发
     parallel: true,
     // 开启缓存
     cache: true,
     compress: {
       // 删除所有的console语句    
       drop_console: true,
       // 把使用多次的静态值自动定义为变量
       reduce_vars: true,
     },
     output: {
       // 不保留注释
       comment: false,
       // 使输出的代码尽可能紧凑
       beautify: false
     }
   })
 ]
}
```

webpack4默认使用uglifyjs-webpack-plugin对代码压缩，可以配置optimization.minimize 与 optimization.minimizer来自定义。

#### 按需加载

webpack配置：

```js
output: {
    path: path.join(__dirname, '/../dist'),
    filename: 'app.js',
    publicPath: defaultSettings.publicPath,
    // 指定 chunkFilename
    chunkFilename: '[name].[chunkhash:5].chunk.js',
}
```

路由代码：

```js
const getComponent => (location, cb) {
  require.ensure([], (require) => {
    cb(null, require('../pages/BugComponent').default)
  }, 'bug')
},
...
<Route path="/bug" getComponent={getComponent}>
```

核心方法就是`require.ensure(dependencies, callback, chunkName)`，会将组件单独打包，只有跳转这个路由时，才会加载这个文件内容。

## Gzip压缩

Http压缩是指网页服务器和客户端之间传输的数据进行压缩，改进传输速度和带宽。开启方式是在headers中加上：

```
accept-encoding:gzip
```

开启Gzip，虽然压缩和解压需要时间，但如果文件很大，还是很有效。Gzip的压缩原理是替换文件中的重复内容，随意重复越多，压缩效果越好。

