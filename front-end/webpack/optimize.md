# 前言

Webpack优化并没有固定的模式，一般常见的优化就是 拆包、分块、压缩等等。但是并不是所有项目都适用所有的优化，需要针对项目本身的场景去打造。

### 分析打包速度

优化Webpack构建速度的第一步就是要知道 哪个部分、哪个阶段花费的时间多久，可以通过 `speed-measure-webpack-plugin` 测试得到：

```
const SpeedMeasurePlugin = require('speed-measure-webpack-plugin');
const smwp = new SpeedMeasurePlugin();
// ...
module.exports = smwp.wrap(prodWebpackConfig);
```

不同的项目，都有自己特定的性能瓶颈，针对打包的每个环节进行优化

### 分析影响打包速度的部分

打包就是从入口文件开始将所有的依赖模块打包到一个文件的过程，在这其中还涉及了 编译、优化过程。

常见的影响构建速度的部分：

#### 1. 开始打包，我们需要获取所有的依赖模块

搜索所有的依赖项，这需要占用一定的时间(搜索时间)。

<b>需要优化的就是搜索时间</b>



#### 2. 解析所有的依赖模块(解析成浏览器可运行的代码)

Webpack根据我们配置的loader解析相应的文件，在项目中我们需要使用loader 对 js、css、图片、文字等文件做转换操作，并且转换的文件数据量也是非常大，由于js单线程的特性使得这些转换操作不能并发处理文件，而是需要一个个文件进行处理

<b>需要优化的就是解析时间</b>



#### 3. 将所有的依赖模块打包到一个文件

当解析完所有代码时，打包到一个文件上，为了使浏览器加载的包更新(减少白屏时间)，所以Webpack会对代码进行优化。

JS压缩是发布编译的最后阶段，压缩JS需要将代码解析成AST语法树，然后需要根据复杂的规则去计算和处理AST，最后将AST转换成JS，这个过程需要涉及大量的计算，比较耗时

<b>需要优化的就是压缩时间</b>



#### 4. 再次打包

当更改项目中一个小小的文件时，我们需要重新打包，所有的文件都必须重新打包，需要花费同第一次打包相同的时间，但项目中大部分文件都没变更，尤其是第三方库。

<b>需要优化的就是再次打包时间</b>



### 优化解析时间

运行在node.js的Webpack是单线程模式的，也就是说， Webpack打包只能逐个文件处理，当Webpack需要打包大量文件时，打包的时间就会比较漫长。常见的解决手段：开启多线程打包

#### 1. thread-loader (webpack4)

把这个loader放在其他loader之前，放置在这个loader之后的loader就会在一个单独的worker(worker pool)里面运行，一个worker就是一个node.js进程，每个单独进程处理时间上限为600ms，各个进程的数据交换也会限制在这个时间内。

```
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: ['thread-loader', 'babel-loader']
      },
      {
        test: /\.s?css$/,
        exclude: /node_modules/,
        use: [
          'style-loader',
          'thread-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true,
              localIdentName: '[name]__[local]--[hash:base64:5]',
              importLoaders: 1
            }
          },
          'postcss-loader'
        ]
      }
    ]
  }
}
```

+ 其中 thread-loader 放在了style-loader之后，这是因为thread-loader没法存取文件也没法获取Webpack的选项位置。
+ 根据官方描述，每个worker大概都要花费600ms，所以为了防止启动worker时的高延迟，提供了对worker池的优化 - 预热

```
const threadLoader = require('thread-loader');
const jsWorkerPool = {
  workers: 2,
  poolTimeout: 2000
};
const cssWorkerPool = {
  workerParallelJobs: 2,
  poolTimeout: 2000
};
threadLoader.warmup(jsWorkerPool, ['babel-loader']);
threadLoader.warmup(cssWorkerPool, ['css-loader', 'postcss-loader']);

// ...上述省略
{
  test: /\.js$/,
  exclude: /node_modules/,
  use: [
    {
      loader: 'thread-loader',
      options: jsWorkerPool
    },
    'babel-loader'
  ]
},
{
  test: /\.s?css$/,
  exclude: /node_modules/,
  use: [
    'style-loader',
    {
      loader: 'thread-loader',
      options: cssWorkerPool
    },
    {
      loader: 'css-loader',
      options: {
        modules: true,
        localIdentName: '[name]__[local]--[hash:base64:5]',
        importLoaders: 1
      }
    },
    'postcss-loader'
  ]
}
```



#### 2. HappyPack

在Webpack构建过程中，实际上耗费时间大多数用在loader解析转换以及代码的压缩中，HappyPack可利用多进程对文件进行打包，对多核cpu利用率更高，HappyPack可以让Webpack同一时间处理多个任务，发挥多核CPU的能力，将任务分解给多个子进程去并发执行。

注意：当项目比较小型时，多进程打包反而会使打包速度变慢。

```
const path = require('path');
const webpack = require('webpack');
const HappyPack = require('happypack');
const os = require('os');

const happyThreadPool = HappyPack.ThreadPool({ size: os.cpus().length })
const createHappyPlugin = (id, loaders) => new HappyPack({
  id: id,
  loaders: loaders,
  threadPool: happyThreadPool,
  verbose: true
});
module.exports = {
  // ...
  module: {
    roles: [
      {
        test: /\.(js|jsx)$/,
        use: ['happypack/loader?id=happy-babel'],
        exclude: /node_modules/,
      }
    ]
  },
  plugins: [
    createHappyPlugin('happy-babel', [{
      loader: 'babel-loader',
      options: {
        presets: ['@babel/preset-env', '@babel/preset-react'],
        plugins: [
          ['import', { 'libraryName': 'antd', ‘style': true }],
          ['@babel/plugin-proposal-class-properties', {loose: true}]
        ],
        cacheDirectory: true,
        cacheCompression: true,
        compact: true
      }
    }]),
  ]
}

```



### 合理利用缓存

使用Webpack缓存的方法有几种。比如 css-loader, HardSourceWebpackPlugin 或者babel-loader的cacheDirectory， 所有这些缓存方法都有启动的开销，重新运行期间在本地节省的时间很大，但是初始运行会变慢。

如果项目生产版本每次都必须进行初始构建德华，缓存会增加构建时间，减慢速度。如果不是，则会大大缩减你二次构建的时间。

#### 1. cache-loader

cache-loader和thread-loader一样，使用起来也很简单，仅需要在一些开销大的loader之前添加该loader，将结果缓存到磁盘里，明显提升再次构建速度。

```
module.exports = {
  module: {
    rules: [
      {
        test: /\.ext$/,
        use: ['cache-loader', ...loaders],
        include: path.resolve('src')
      }
    ]
  }
}
```



#### 2. HardSourceWebpackPlugin

+ 第一次构建将花费正常的时间
+ 第二次构建则效果明显

```
const HardSourceWebpackPlugin = require('hard-source-webpack-plugin');
const clientWebpackConfig = {
  plugins: [
    new HardSourceWebpackPlugin({
      cacheDirectory: path.join(__dirname, './lib/cache/hard-source/[confighash]'),
      configHash: function(webpackConfig) {
        return require('node-object-hash')({sort: false}).hash(webpackConfig);
      },
      environmentHash: {
        root: process.cwd(),
        directories: [],
        files: ['package-lock.json', 'yarn-lock']
      },
      info: {
        mode: 'none',
        level: 'debug'
      },
      cachePrune: {
        maxAge: 2 * 24 * 60 * 60 * 1000,
        sizeThreshold: 50 * 1024 * 1024
      }
    }),
    new HardSourceWebpackPlugin.ExcludeModulePlugin([
      { test: /.*\.DS_Store/ }
    ])
  ]
}
```



### 优化压缩时间

#### 1. webpack3

Webpack3 启动打包时附加上 `--optimize-minimize`，这样webpack会自动注入一个带有默认配置的 UglifyJSPlugin

```
module.exports = {
  optimization: {
    minimize: true
  }
}
```

#### 2. webpack4

Webpack4默认内置使用terser-webpack-plugin插件压缩优化代码，而该插件使用terser来缩小Javascript

terser是用于ES6+的 JavaScript解析器，mangler/compressor(压缩器)工具包

terser启动多进程并行运行来提高构建速度，并发运行的默认数量为 os.cpus().length - 1

```
module.exports = {
  optimization: {
    minimizer: [
      new TerserPlugin({
        parallel: true
      })
    ]
  }
}
```



### 优化搜索时间

目的：缩小文件搜索范围 减少不必要的编译工作

Webpack打包时，会从配置的entry触发，解析入口文件的导入语句，再递归的解析，在遇到导入语句时，webpack会执行：

+ 根据导入语句去寻找对应的要导入的文件，
+ 根据找到的要导入文件的后缀，使用配置中的Loader去处理文件

#### 1. 优化 loader 配置

使用 Loader 时可以通过 test、include、exclude 三个配置项命中Loader要应用规则的文件

#### 2. 优化 resolve.module 配置

resolve.modules 用于配置 webpack 去哪些目录下寻找第三方模块，resolve.modules 的默认值是 ['node_module']，含义是先去当前目录下的 ./node_modules/ 目录下去找想找的模块，如果没找就去上一级 以此类推

#### 3. 优化 resolve.alias 配置

通过别名来把原导入路径映射成一个新的导入路径，减少耗时的递归解析操作。

#### 4. 优化 resolve.extensions 配置

在导入语句没带文件后缀时，Webpack会根据 resolve.extension 自动带上后缀后去尝试询问文件是否存在，所以在配置 resolve.extension 应尽可能注意以下几点：

  + resolve.extension 列表要尽可能的小，不要把项目中不可能存在的情况写到后缀列表中
  + 频率最高的后缀应该放在最前面，尽快退出寻找过程
  + 在源码中写导入语句时，要尽可能的带上后缀，从而可以避免寻找过程

#### 5. 优化 resolve.mainFields 配置

有一些第三方模块与针对不同环境提供几分代码，例如分别提供采用es5和es6的代码，写在package.json文件里

```
{
  'jsnext:main': 'es/index.js',
  'main': 'lib/index.js'
}
```

Webpack会根据mainFields的配置去决定优先采用哪部分代码

```
// 默认配置
mainFields: ['browser', 'main']
// 若想采用es6的代码，则:
mainFields: ['jsnext:main', 'browser', 'main']
```

#### 6. 优化 resolve.noParse 配置

module.noParse配置项可以让Webpack忽略对部分没采用模块化的文件的递归解析处理，好处是提高构建性能。

#### 7. 详细 配置

```
module.exports = {
  // ...
  module: {
    // jquery没有采用模块化标准，Webpack忽略
    noParse: /jquery/,
    rules: [
      {
        // 如果没用到就不要写，提升正则性能
        test: /\.(js|jsx)$/,
        use: ['bebel-loader?cacheDirectory'],
        exclude: /node_modules/,
      }
    ]
  },
  resolve: {
    modules: [
      // 设置模块导入规则，import/require时会直接在这些目录找文件
      // 可以指明存放第三方模块的绝对路径，以减少寻找
      path.resolve(`${project}/client/components`),
      path.resolve('h5_commonr/components'),
      'node_modules'
    ],
    // 尽可能减少后缀尝试的可能性
    extensions: ['.js', '.jsx', '.css', '.json'],
    // import导入别名，减少耗时的递归解析操作
    alias: {
      '@components': path.resolve(`${project}/components`)
    }
  }
}
```


以上就是关于Webpack的一些优化操作了，根据上述配置后，在构建速度上会有一定的提升。