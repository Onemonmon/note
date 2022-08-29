### 基本配置

```javascript
export default {
    input: "./src/index.js", // 入口文件路径
    output: { // 打包生成的文件配置
        file: "./dist/bundle.js", // 输出文件路径
        format: "es", // 输出文件的格式（amd/es/iife/cjs/umd）
        name: "bundleName", // 当format=iife/umd时，需要指定全局变量名称window.[name]
        globals: { // 配合external使用，当format=iife/umd时需要
            lodash: 'lodash'
        }
    },
    external: ['lodash'], // 告诉rollup不需要打包的包，而是作为外部依赖引入
    plugins: [], // 插件
}
```

### 插件

```javascript
import babel from "rollup-plugin-babel"
import resolve from '@rollup/plugin-node-resolve'
import commonjs from '@rollup/plugin-commonjs'
import typescript from '@rollup/plugin-typescript'
import postcss from 'rollup-plugin-postcss'
import autoprefixer from 'autoprefixer'
import vue from 'rollup-plugin-vue'

export default {
    ...
    plugins: [
        vue({
            postcssPlugins: [autoprefixer()]
        }),
        typescript(),
        resolve(),
        commonjs(),
        babel({ 
            exclude: 'node_modules/**' 
        }),
        postcss({
            plugins: [autoprefixer()],
            extract: "css/index.css", // 抽离css文件
        })
    ]
}
```

#### 配置Babel

`npm i rollup-plugin-babel @babel/core @babel/preset-env -D`

```js
// babel.config.js
module.exports = {
	presets: [
        [
            "@babel/env",
            {
                modules: false, // 否则Babel会在Rollup有机会做处理之前，将模块转成CommonJS，导致 Rollup的一些处理失败
            }
        ]
    ]
}
```

#### 引用CommonJS的包

`npm i @rollup/plugin-node-resolve @rollup/plugin-commonjs -D `

#### 使用TS

`npm i @rollup/plugin-typescript typescript tslib -D`

#### 处理CSS

`npm i rollup-plugin-postcss -D`

会将CSS代码通过JS脚本生成style标签并插入head中

###### 自动添加属性前缀

`npm i autoprfixer -D`

配置浏览器版本.browserslistrc

```
> 0.25%
last 2 version
```

###### 使用CSS预处理

`npm i less -D`

#### 使用Vue

`npm i rollup-plugin-vue @vue/compiler-sfc -D`