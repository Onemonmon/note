### 1. 打包流程

```
1. 初始化参数：从shell命令和配置参数中获取参数
2. 开始编译：根据获取到的参数，生成Compiler对象，并加载所有插件配置，执行对象的run方法开始编译
3. 编译模块：从入口文件出发，调用配置的loader对模块进行编译，再递归找到模块依赖的模块并对其进行编译，直到所有模块都被编译
4. 完成模块编译：经过步骤4的编译后，已经获得了模块编译后的内容及其依赖关系
5. 输出资源：将编译后的内容组成chunk并添加到输出队列
6. 输出完成：将输出队列的内容写入配置的出口文件
```

### 2. 自定义loader

```
loader对模块的源代码进行转换，允许我们打包除js外的文件
loader的编写应遵循单一功能原则，一个loader实现一个功能
loader的匹配执行顺序为从下到上、从右到左
```

```javascript
// src/loaders/my-loader.js
const loaderUtils = required('loader-utils')
module.exports = function myLoader(source) {
    // 获取参数
    const options = loaderUtils.getOptions(this)
    // 异步loader
    const cb = this.async()
    setTimeout(() => {
        const content = source.replace(/console.log\(.*\)/g, '')
        cb(null, content)
    }, 3000)
    return
    // 同步loader
    return source
}
```

```
引入自定义loader
1. 直接通过loader的路径引入
   {
   		test: /\.js$/,
   		use: './src/loaders/my-loader.js'
   }
2. 通过自定义loader名引入
   resolveLoader.alias = {
   		'my-loader': path.resolve(__dirname, './src/loaders/my-loader.js')
   }
3. 在特定目录下查找loader
   resolveLoader.modules = ['node_modules', './src/loaders']
```

### 3. 自定义Plugin

```
webpack在编译过程中会暴露很多生命周期钩子函数，plugin通过调用这些钩子，能够在编译的特定环节对模块进行处理
```

```javascript
// 编译结束后创建一个文件来显示dist下的文件信息
class MyPlugin {
    constructor({ filename }) {
        this.filename = filename
    }
    apply(compiler) { // compiler是全局的打包对象
        // 同步
        compiler.hooks.emit.tap('MyPlugin', (compilation) => {
            // compilation是每个阶段的打包对象
            const assets = compilation.assets
            let content = '文件名    资源大小 \r\n'
            Object.entries(assets).forEach([filename, stat] => {
                content += `${filename}     ${stat.size()}\r\n`
            })
            assets[this.filename] = {
                source() { return content },
                size() { return content.length }
            }
        })
        // 异步
        compiler.hooks.emit.tapAsync('MyPlugin', (compilation, cb) => {
            setTimeout(() => {
                // ...
                cb()
            }, 3000)
        })
        compiler.hooks.emit.tapPromise('MyPlugin', (compilation) => {
            return new Promise((resolve) => {
                setTimeout(() => {
                    // ...
                    resolve()
                }, 3000)
            })
        })
    }
}
```

### 4. 基本使用

#### 4.1 基本配置

```javascript
module.exports = {
    entry: '', // 入口文件路径
    output: {
        path: '', // 打包输出文件的绝对路径
        filename: '', // 打包输出文件名
        clean: true, // 5.x新增配置，代替原来的CleanWebpackPlugin
    },
    module: {
        rules: [
            // loader 对模块的源代码进行转换，允许我们打包除js外的文件
        ]
    }, 
    plugins: []
}
```

#### 4.2 webpack-dev-server

> npm i webpack-dev-server -D

```javascript
module.exports = {
    //...
    devServer: {
        open: true, // 是否编译后自动打开浏览器
        compress: true, // 开启gzip
        port: 7000, // 端口号
        hot: true, // 开启热模块更替（不用每次全量编译，而是以补丁形式）
    }
}
```

内部实现：使用了express和webpack-dev-middleware

```javascript
const express = reuqire('express')
const webpackDevMiddleware = require('webpack-dev-middleware')
const webpack = require('webpack')
const webpackConfig = require('./webpack.config.js')

const compiler = webpack(webpackConfig)
const app = express()
app.use(webpackDevMiddleware(compiler, {
    //...
}))

app.listen(3000, function() {
    // 服务器启动
})
```

#### 4.3 loaders

加载文件资源并对其进行编译、压缩、转换等操作

##### 4.3.1 打包css文件

css-loader：解析.css文件

style-loader：将前一步解析的结果通过内联样式的形式写入HTML中

```javascript
module.exports = {
    //...
    module: {
        rules: [
            {
                test: '\.css$',
                use: ['style-loader', 'css-loader'], // 从右到左执行
            }
        ]
    }
}
```

##### 4.3.2 css预处理器

```javascript
module.exports = {
    //...
    module: {
        rules: [
            {
                test: '\.css$',
                use: ['style-loader', 'css-loader'], // 从右到左执行
            },
            // less less-loader
            { 
            	test: '\.less$',
            	use: ['style-loader', 'css-loader', 'less-loader']
            },
            // scss sass-loader
            {
            	test: '\.s(a|c)ss$',
            	use: ['style-loader', 'css-loader', 'sass-loader']
            }
        ]
    }
}
```

##### 4.3.3 打包图片和字体图标

```javascript
module.exports = {
    //...
    module: {
        rules: [
            // file-loader url-loader
            {
                test: '\.(jpg|jpeg|bmp|png|gif)$',
                // 5.x新写法
                type: "asset", // 通用资源类型，根据一定的条件在 inline 和 resource 之间切换
        		// type: "asset/resource", // 发送一个单独的文件并导出 URL
        		// type: "asset/inline", // 导出一个资源的 data URI (base64)
        		// type: "asset/source", // 导出资源的源代码
                parser: {
                  dataUrlCondition: {
                    maxSize: 10 * 1024,
                  },
                },
                generator: {
                  filename: "static/images/[hash][ext][query]",
                },
                
                // 老写法
                use: {
                    loader: 'url-loader',
                    options: {
                        limit: 5 * 1024, // 不超过5kb的图片转换成base64，减少网络请求
                        outputPath: 'images', // 图片输出的目录
                        name: '[name]-[hash:8].[ext]'
                    }
                }
            },
            {
                test: '\.(woff|woff2|svg|ttf)$',
                // 5.x新写法
                type: "asset/resource",
                generator: {
                  filename: "static/media/[hash][ext][query]",
                },
                // 老写法
                use: 'url-loader'
            },
            // 打包直接在html中引入的图片
            {
            	test: '\.(htm|html)$',
            	use: 'html-withimg-loader'
            }
        ]
    }
}
```

##### 4.3.4 babel

```javascript
module.exports = {
    //...
    module: {
        rules: [
            // babel-loader @babel/core @babel/preset-env
            {
                test: '\.js$',
                use: {
                    loader: 'babel-loader',
                    options: { // 建议抽到.babelrc文件中
                        presets: ['@babel/env'], // 用于处理语法的转换
                        plugins: [], // 一些更新的高级语法需要用特定的插件
                    }
                },
                exclude: /node_modules/
                include: /src/
            }
        ]
    }
}
```

当代码中使用了原型的新方法时，需要`import '@babel/polyfill'`，他会为原型上添加需要的新方法，或者修改`entry: ['@babel/polyfill', './src/index.js']`

#### 4.4 source map 

追踪错误和警告在源代码中的位置，尽量不在生产环境中使用

```javascript
module.exports = {
    devtool: 'cheap-module-eval-source-map'
}
```

#### 4.5 Plugins

在编译过程中会暴露很多生命周期钩子函数，plugin通过调用这些钩子，能够在编译的特定环节对模块进行处理

```javascript
module.exports = {
    //...
    plugins: [
        new HtmlWebpackPlugin({
            filename: '',
            template: '',
        }),
        new CleanWebpackPlugin(),
        new CopyWebpackPlugin([{
        	from: path.join(__dirname, 'assets'),
            to: 'assets'
        }]),
        // webpack内置插件
        new webpack.BannerPlugin('right by onemonmon'), // 添加一些版权注释信息
    ]
}
```

#### 4.6 多页面应用打包

```javascript
module.exports = {
    // 修改为多入口
    entry: {
        index: './src/index.js',
        other: './src/other.js'
    },
    output: {
        path: path.join(__dirname, 'dist'),
        // 多入口无法对应同一个固定的出口
        filename: '[name]-bundle.js'
    },
    plugins: [
        // 配置多个html
        new HtmlWebpackPlugin({
            filename: 'index.html',
            template: './public/index.html',
            chunks: ['index'] // index => entry.index
        }),
        new HtmlWebpackPlugin({
            filename: 'other.html',
            template: './public/other.html',
            chunks: ['other']
        }),
    ]
}
```

#### 4.7 引入第三方库的方法

##### 4.7.1 expose-loader 

将库加入全局作用域

```javascript
module.exports = {
    module: {
        rules: [
            test: require.resolve('jquery'), // 解析jquery模块的绝对路径
            use: {
            	loader: 'expose-loader',
            	options: '$' // 全局变量名称
            },
        ]
    }
}
```

##### 4.7.2 ProvidePlugin

为每个模块自动注入

```javascript
module.exports = {
    plugins: [
        new webpack.ProvidePlugin({
            $: 'jquery',
            jQuery: 'jquery'
        })
    ]
}
```

#### 4.8 不同环境的配置文件

-build/webpack.base.config.js

-build/webpack.prod.config.js

-build/webpack.dev.config.js

此时应注意配置中的绝对路径要对应改变（多了/build/），相对路径时针对项目根目录所以无需改变

```javascript
const baseConfig = require('./webpack.base.config.js')
const webpackMerge = require('webpack-merge')

module.exports = webpackMerge(baseConfig, {
    //...
})
```

#### 4.9 定义环境变量

```javascript
module.exports = {
    plugins: [
        new webpack.DefinePlugin({
            IS_DEV: 'true', // ''里面的代码会被当作js表达式执行
        })
    ]
}
```

#### 4.10 跨域问题解决

##### 4.10.1 CORS

##### 4.10.2 http proxy

```javascript
module.exports = {
    devServer: {
		proxy: {
            '/api': {
                target: 'http://10.20.30.40:8000',
                pathRewrite: {
                    '^/api': ''
                }
            }
        }
    }
}
```

#### 4.11 性能优化

##### 4.11.1 production模式自带的打包优化

1）tree shaking：依赖于ES6的`import/export`，打包时移除js中未使用的代码（dead-code）

2）scope hoisting：webpack.ModuleConcatenationPlugin，提前分析模块间的关系，尽可能把打散的模块合并到一个函数中

3）代码压缩：uglifyjs-webpack-plugin

##### 4.11.2 css优化

1）MiniCssExtractPlugin

多个模块需要复用改样式时，不用每个模块都通过style标签引入

只能在webpack 4中使用，将css代码提取到一个文件中

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: '\.css$',
                use: [MiniCssExtractPlugin.loader, 'css-loader']
            }
        ]
    },
    plugins: [
        new MiniCssExtractPlugin({
            filename: '[name].css'
        })
    ]
}
```

2）postcss

自动为样式属性添加前缀

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: '\.css$',
                use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader']
            }
        ]
    },
    plugins: [
        new MiniCssExtractPlugin({
            filename: '[name].css'
        })
    ]
}
```

> 根目录下添加 .postcss.config.js

```javascript
module.exports = {
    plugins: [require('autoprefixer')]
}
```

3）css压缩 optimize-css-assets-webpack-plugin

```javascript
const TerserJsWebpackPlugin = require('terser-webpack-plugin')
const OptimizeCssAssetsWebpackPlugin = required('optimize-css-assets-webpack-plugin')

module.exports = {
    optimization: { 
        // webpack默认提供了一些minimizer配置，重写之后会覆盖，所以默认的js压缩会失效
        minimizer: [new TerserJsWebpackPlugin(), new OptimizeCssAssetsWebpackPlugin()]
    }
}
```

##### 4.11.3 js优化

使用entry配置多入口手动分离代码？

如果不同入口chunk引入了同一个模块，那这个模块会被分别打包各个bundle中

抽取公共代码，使用`SplitChunkPlugin`去重和分离chunk

```javascript
module.exports = {
    optimization: {
        splitChunks: {
            chunks: 'async', // （默认值）只对异步加载的模块进行分割 'all' | 'initial'
            minSize: 30000, // 超过30kb的模块才会被拆分
            maxSize: 0, // 0无上限，否则超过maxSize的模块会被进一步拆分
            maxAsyncRequests: 5, // 异步加载时同时发送的请求数量不超过5，超出部分不拆分
            maxInitialRequests: 3, // 页面初始化异步加载时发送的请求数量不超过3，超出部分不拆分
            automaticNameDelimiter: '~', // 文件命名连接符
            name: true, // 拆分的chunk名，true为使用模块名加cacheGroups的key
            cacheGroups: {
                vendors: {
                    test: /[\\/]node_modules[\\/]/,
                    priority: -10
                },
                default: {
                    minChunk: 2,
                    priority: -20
                }
            }
        }
    }
}
```

动态导入（懒加载）

.babelrc 使用@babel/plugin-syntax-dynamic-import

```json
{
	"plugins": ["@babel/plugin-syntax-dynamic-import"]    
}
```

```javascript
import('jquery').then((res) => {
    //...
})
```

##### 4.11.4 noParse

跳过解析一些库（该库没有其他依赖），加快构建速度

```javascript
module.exports = {
    module: {
        noParse: /jquery|bootstrap/
    }
}
```

##### 4.11.5 IgnorePlugin

忽略一些第三方库的语言包或者不需要使用的文件，并自行手动引入需要的语言包

```javascript
module.exports = {
    plugins: [
        new webpack.IgnorePlugin(/\.\/locale/, /moment/),
    ]
}
```

```javascript
import moment from 'moment'
import 'moment/locale/zh-cn'
moment.locale('zh-cn')
```

##### 4.11.6 DllPlugin

将一些不做修改的库（vue、react）提前打包，在开发打包时不需要再进行打包

将vue项目的库抽取成dll，需要单独的webpack配置文件，注意不要使用CleanWebpackPlugin

```javascript
// webpack.vue.config.js
module.exports = {
    mode: 'production',
    entry: {
        vue: [ // 将需要做成dll的库全放进来
            'vue/dist/vue.js',
            'vue-router'
        ]
    },
    output: {
        path: path.resolve(__dirname, '../dist'),
        filename: '[name]_dll.js',
        library: '[name]_dll', // 最终会在全局暴露出名为[name]_dll的对象
    },
    plugin: [
        // 将打包后的bundle关联起来
        new Webpack.DllPlugin({
            name: '[name]_dll',
            path: path.resolve(__dirname, '../dist/manifest.json'), // 生成清单文件的路径
        })
    ]
}
```

`DllReferencePlugin`使用这个dll，指定该dll的manifest.json的路径

```javascript
// webpack.base.config.js
modules.exports = {
    plugins: [
        new webpack.DllReferencePlugin({
            manifest: path.resolve(__dirname, '../dist/manifest.json')
        })
    ]
}
```

自动添加vue_dll.js到HTML中

```javascript
module.exports = {
    plugins: [
        new HtmlWebpackPlugin(),
        new AddAssetHtmlWebpackPlugin({
            filepath: path.resolve(__dirname, '../dist/vue_dll.js')
        }), // 在HtmlWebpackPlugin之后
    ]
}
```

##### 4.11.7 利用浏览器缓存

浏览器会缓存之前加载过的js等资源文件，提高加载速度，但是当业务代码修改重新编译后（服务器未重启），浏览器还是会去获取之前缓存的资源，这时可以通过将bundle的命名添加上hash值`[contenthash]`，这样就能使代码更新后浏览器能正确从服务器获取新资源。

```javascript
module.exports = {
    output: {
        path: path.resolve(__dirname, '../dist'),
        filename: '[name].[contenthash:8].bundle.js'
    }
}
```

##### 4.11.8 prefetching

通过动态导入+魔法注释的方法，在加载完首屏资源后自动加载其他资源

```javascript
import(/* webpackPrefetch: true */ 'jquery').then(() => {})
```