## 前端工程化——Webpack原理和应用
### Entry的用法
Entry是打包的入口支持**单入口**和**多入口**两种模式，单入口模式的entry字段是一个字符串，多入口则是一个**对象**。
```javascript
module.exports = {
    entry: {
        app: './src/index.js',
        admin: './src/admin.js'
    }
}
```

### Output的用法
对于多入口的情况，`filename`字段通过占位符`[name]`来指定文件名；对于单入口的情况，则直接指定文件名即可。
```javascript
module.exports = {
    output: {
        path: path.join(__dirname, 'dist'),
        filename: '[name].js'
    },
}
```
### Loaders的概念和用法
Webpack原生只支持**Javascript**和**JSON**两种类型的资源，若要打包其他类型，需要loaders帮助。Loaders的可以把相关文件转换为**有效的模块**，并添加到**依赖图**中。它本身是个函数，以文件为参数，返回转换结果。

#### 常见的loader类型
- babel-loader 转换ES6以上的新语法 (也可以解析React的JSX语法)
- css-loader 加载和解析css文件
- less-loader 将less文件转为css
- ts-loader 将typescript转换为javascript
- file-loader 打包图片、字体等资源
- raw-loader 将文件以字符串形式导入
- thread-loader 多线程打包

举例：解析css需要css-loader和style-loader，前者负责加载css文件，并将其**转换成commonjs对象**，后者将样式通过`<style>`标签插入到页面的head中。注意，**loader是链式调用的且是从右向左的**，所以要先写style-loader，再写css-loader，保证css已经被解析和加载，再执行插入。

#### 简单示例
在`module.rules`数组中配置文件名的正则表达式以及相应的loader类型。
```javascript
module.exports = {
    module: {
        rules: [
            { test: /\.txt$/, use: 'raw-loader' }
        ]
    },
}
```

### Plugins的概念和用法
Plugins用于完成loaders不能完成的事情，比如**bundle文件的优化、资源管理和环境变量注入**等，它可以应用于整个构建过程。

#### 常见的plugins
- CommonChunkPlugin 将chunk相同的模块代码提取成公共的js文件
- CleanWebpackPlugin 清理构建目录
- HtmlWebpackPlugin 创建html文件去承载输出的bundle文件
- UglifyjsWebpackPlugin 压缩js文件
- ZipWebpackPlugin 将打包后的资源存储在zip中

#### 简单示例
```javascript
module.exports = {
    plugins: [
        new HtmlWebpackPlugin({ template: './src/index.html' }),
    ]
}
```

### Webpack的mode
`mode`支持`production`、`development`、`none`三个值，默认为`production`，Webpack通过mode来为内部函数开启特定的参数（**process.env.NODE_ENV**），并开启相应的默认插件。比如：development下的NameChunksPlugin、production下的FlagDependencyUsagePlugin等。

### Webpack的watch
通过指定`--watch`命令参数，或者在配置中指定`watch: true`均可以实现对文件改动的监控，并自动重新编译（但不会自动刷新页面）。原理是webpack会通过node.js的文件接口**轮询文件的修改时间**，如果发生变化，就执行重新编译。轮询的频率和检查的文件都是可以配置的。

### Webpack 热更新
通过`webpack-dev-server`以及`HotModuleReplacementPlugin`可以实现网站的热更新。原理如下：
- Webpack编译器打包生成输出的bundle文件
- Bundle Server会提供文件在浏览器中的访问
- HMR Server负责将文件的更新输出给HMR Runtime
- 在开发时，一个HMR Runtime代码会被注入到生成的bundle.js文件中，用于与HMR Server建立WebSocket连接，更新文件的变化

### 文件指纹
文件指纹可以用来做版本管理，优化缓存加速访问。
Webpack有三种文件指纹:
- hash：项目文件有修改就会重新做hash
- chunk hash：不同的entry生成不同的chunk hash
- content hash：根据文件内容定义hash，文件内容不变，hash不变，如css文件

### 代码压缩
Javascript：Webpack内置uglifyjs-webpack-plugin，提供对JS文件的压缩。
CSS: 对于webpack 5 可以使用CssMinimizerPlugin。
HTML：可以使用HtmlWebpackPlugin。

