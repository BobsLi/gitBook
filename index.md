# 前端自动化构建

* #### 为什么要使用自动化构建工具

现在的前端开发已经不再仅仅只是静态网页的开发了，日新月异的前端技术已经让前端代码的逻辑和交互效果越来越复杂，更加的不易于管理，模块化开发和预处理框架把项目分成若干个小模块，增加了最后发布的困难，没有一个统一的标准，让前端的项目结构千奇百怪。前端自动化构建在整个项目开发中越来越重要。

举一些例子：

HTML:

```
<div class="a">
    <div class="b">
        <div class="c"></div>
    </div>
</div>
```

CSS:

```css
.a{
    color: red;
}
.a .b{
    background-color: blue;
}
.a .b .c{
    color: green;
}
```

如果按照上面的写法，每次都要把父元素给写出来，就会感觉代码量超多的，但是，如果用sass，less等css预处理器之后呢

SASS:

```css
.a{
    color: red;
    .b{
        background-color: blue;
        .c{
            color: green;
        }
    }
}
```

这样嵌套着写看上去就会感觉很舒服又不累赘，用sass命令编译后就和上面的css代码一样的，但是呢，如果当sass文件很多，不可能都让自己手动去执行sass命令，于是乎自动化构建工具的作用就凸显出来了，比如grunt、gulp、webpack等等，通过这些工具来自动实现从sass变为css，并且还可以做到没修改一次sass文件都可以在浏览器里面看到新的效果，而不像以前那样修改一点css，然后F5刷新看效果。,下面是gulp的一个例子。

gulp:

```js
gulp.task( 'sass', function(){
    //不能加入return,否则会导致 plumber无效
    return gulp.src( host+'/scss/*.scss' )
        //.pipe( plumber() )      //添加守护进程，让他不会因为错误而中断
        .pipe( sass( sassOptions ).on('error', sass.logError) )
        .pipe( csscomb() )      //属性优先级排序
        //.pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
        //.pipe( plumber.stop() )
        .pipe( gulp.dest( host+'/css' ) )
        .pipe(reload({stream: true}));
});

/**
 * [先编译sass，在将commom.css复制到公共文件夹]
 * @param  {String} ){                 var _copyPath [description]
 * @return {[type]}     [description]
 */
gulp.task('commonSass', ['sass'], function(){
    var _copyPath = "common/assets/vendor/css";
    deleteFile(_copyPath + '/common.css');
    return gulp.src( host+'/css/common.css' )
        .pipe( gulp.dest(_copyPath) );
})

/**
 * [前台sass编译动态监听]
 * @param  {String} ){                 host [description]
 * @return {[type]}     [description]
 */
gulp.task( 'v3Front', function(){
    host = "frontend/web";
    browserSync.init({
        proxy:'http://www.v3.test',
        // proxy:'http://192.168.0.175',
    });
    gulp.watch( [ host+'/scss/**/*.scss' ], [ 'commonSass'] );       //**表示，所有子级目录
    gulp.watch("frontend/views/*.php").on('change', reload);
} );
```

运行gulp v3Front，会监听设定目录下的所有scss文件，将scss自动转换为css，还可以复制一部分的css到另外一个目录里面，一旦修改scss文件，gulp都会做出相应的改变，将这些改变重新生成css，并且自动响应到浏览器中看到新的效果。这只是一个自动化构建工具的一个简单的应用。

自动化工具的用途远不止这，还包括【流程管理】【版本管理】【资源管理】【组件化】【脚手架工具】等。

1. ##### 【流程管理】本地开发，调试，上线，优化整个开发流程，降低时间成本。
2. ##### 【版本管理】项目进行版本控制和管理。
3. ##### 【资源管理】以最优的方式去处理资源的合并和依赖，从而提高性能。
4. ##### 【组件化】组件化方案，是以页面的小部件为单位进行开发，在系统内可复用。如何以最优化方式实现组件化（js、css、html片段，以就近原则进行文件组织，以数据绑定方式进行代码开发，业务逻辑上相对外部独立，暴露约定的接口）；并且随着组件化的程度加深，如何对组件库进行管理，合并打包以及多人共同维护等，都是无法避免的问题。
5. ##### 【脚手架工具】不同团队不同项目之间除了可能有【可复用的模块】或者【组件】，还可能用相同的完整目录结构，如果吧这些相同的项目结构通过自动化工具做成【脚手架工具】，通过一键安装来复用，这样可以减少配置项目结构的时间，把时间多用于业务逻辑代码的编写上。

如今，前端自动化构建工具有很多，比如：angular,grunt,gulp,vue,react,avalon,yeoman,webpack,fis3等等，但是很多的自动化工具侧重的核心点不相同：

【webpack】【fis3】作为当前最优秀的打包工具，以模块为设计出发点，所有资源都可以作为模块自动化进行合并打包以及依赖处理。目前而言，是解决【资源管理】以及【版本管理】的最佳方案。

【gulp】【grunt】是一款优秀的构建工具，通过流式是文件管理，以及定制化的任务管理，可完美兼容任何形式的【流程管理】。

【vue】与【avalon】作为数据驱动的 js 框架，都拥有其优秀的【组件化】方案。vue拥有其强大的技术生态，可驾驭不同复杂度的项目，在移动端上性能也尤为卓越；而avalon在兼容性方面作了最大化地努力，可兼容ie6的亮点，则让其在传统项目中拥有先天的优势。

【yeoman】提供了一个强大的generator构建系统，让开发者可以搭建定制化的【脚手架工具】，快速启动任何类型的项目。

#### ![](/assets/111.png)

### 自动化构建工具就是用来让我们不再做机械重复的事情，解放我们的双手的。

* #### gulp

#### [gulp中文文档：https://www.gulpjs.com.cn/docs/](#gulp中文文档：httpswwwgulpjscomcndocs)

1. #### gulp的特点
2. 易于使用：采用代码优于配置策略，Gulp让简单的事情继续简单，复杂的任务变得可管理。

3. 高效：通过利用Node.js强大的流，不需要往磁盘写中间文件，可以更快地完成构建。

4. 高质量：Gulp严格的插件指导方针，确保插件简单并且按你期望的方式工作。

5. 易于学习：通过把API降到最少，你能在很短的时间内学会Gulp。构建工作就像你设想的一样：是一系列流管道。

6. #### gulp和grunt的区别
7. ##### I/O流程的不同，使用Grunt的I/O过程中会产生一些中间态的临时文件，一些任务生成临时文件，其它任务可能会基于临时文件再做处理并生成最终的构建后文件，而使用Gulp的优势就是利用流的方式进行文件的处理，使用管道（pipe）思想，前一级的输出，直接变成后一级的输入，通过管道将多个任务和操作连接起来，因此只有一次I/O的过程，流程更清晰，更纯粹。Gulp去除了中间文件，只将最后的输出写入磁盘，整个过程因此变得更快
8. ##### 插件，Gulp的插件更纯粹，单一的功能，并坚持一个插件只做一件事。
9. #### gulp API
10. ##### gulp.src\(globs\[, options\]\)，输出符合所提供的匹配模式或者匹配模式的数组的文件。将返回一个文件流，可以被pipe到别的插件中。
11. ##### gulp.dest\(path\[, options\]\)，能被 pipe 进来，并且将会写文件。并且重新输出（emits）所有数据，因此你可以将它 pipe 到多个文件夹。如果某文件夹不存在，将会自动创建它。
12. ##### gulp.task\(name\[, deps\], fn\)，定义一个gulp task任务。
13. ##### gulp.watch\(glob\[, opts\], tasks\)，一个 glob 字符串，或者一个包含多个 glob 字符串的数组，用来指定具体监控哪些文件的变动。

###### 总之：gulp任务就是先找匹配的文件，通过管道流pipe到别的插件里面，处理好之后通过dest将结果写入文件夹，如果中途要执行监听操作，就watch需要监听的task就好，另外，task之间是可以有依赖关系的。

* #### webpack

#### webpack中文网 [https://www.webpackjs.com/concepts/](https://www.webpackjs.com/concepts/)

* ##### **什么是webpack?**

       **本质上，webpack是一个现代 JavaScript 应用程序的静态模块打包器，当 webpack 处理应用程序时，它会递归地构建一个**

**依赖关系图，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个bundle。**

##### ![](/assets/webpac23131k.png)

上图已经表达清楚了：

1. **需要把各种资源（js/ts/css/html/ejs/img/fonts等等）都看成 module；**
2. **module 之间必须是互相依赖的，在 js 里 import 模板、图片、样式文件等等，样式文件通过 url\(\) 和图片字体关联；这种依赖关系必须是 webpack 既定的或者是通过插件可以理解的关系。**

**Webpack 的核心就是模块化地组织，模块化地依赖，然后模块化地打包。在webpack看来一切都是模块，包括JavaScript代码，以及css、fonts和图片等等，通过合适的loader转换为webpack可以有效处理的模块（webpack自身只能处理javascript）。**

* ##### webpack使用

1. ##### 前提条件：安装node.js的最新版本
2. ##### 本地安装： cnpm install --save-dev webpack
3. ##### 创建webpack配置文件 webpack.config.js

##### webpack.config.js:

    const path = require('path');
    const webpack = require('webpack');

    module.exports = {
            //起点入口
            entry: "./app/entry", // string | object | array
            entry: ["./app/entry1", "./app/entry2"],
            entry: {
                a: "./app/entry-a",
                vendor: ['react', 'react-dom']
            },
            // 这里应用程序开始执行
            // webpack 开始打包

            output: {
                // webpack 如何输出结果的相关选项

                path: path.resolve(__dirname, "dist"), // string
                // 所有输出文件的目标路径
                // 必须是绝对路径（使用 Node.js 的 path 模块）

                filename: "bundle.js", // string
                filename: "[name].js", // 用于多个入口点(entry point)（出口点？）
                filename: "[chunkhash].js", // 用于长效缓存
                // 「入口分块(entry chunk)」的文件名模板（出口分块？）

                publicPath: "/assets/", // string
                publicPath: "",
                publicPath: "https://cdn.example.com/",
                // 输出解析文件的目录，url 相对于 HTML 页面

                library: "MyLibrary", // string,
                // 导出库(exported library)的名称

                libraryTarget: "umd", // 通用模块定义
                libraryTarget: "umd2", // 通用模块定义
                libraryTarget: "commonjs2", // exported with module.exports
                libraryTarget: "commonjs-module", // 使用 module.exports 导出
                libraryTarget: "commonjs", // 作为 exports 的属性导出
                libraryTarget: "amd", // 使用 AMD 定义方法来定义
                libraryTarget: "this", // 在 this 上设置属性
                libraryTarget: "var", // 变量定义于根作用域下
                libraryTarget: "assign", // 盲分配(blind assignment)
                libraryTarget: "window", // 在 window 对象上设置属性
                libraryTarget: "global", // property set to global object
                libraryTarget: "jsonp", // jsonp wrapper
                // 导出库(exported library)的类型

                /* 高级输出配置（点击显示） */

                pathinfo: true, // boolean
                // 在生成代码时，引入相关的模块、导出、请求等有帮助的路径信息。

                chunkFilename: "[id].js",
                chunkFilename: "[chunkhash].js", // 长效缓存(/guides/caching)
                // 「附加分块(additional chunk)」的文件名模板

                jsonpFunction: "myWebpackJsonp", // string
                // 用于加载分块的 JSONP 函数名

                sourceMapFilename: "[file].map", // string
                sourceMapFilename: "sourcemaps/[file].map", // string
                // 「source map 位置」的文件名模板

                devtoolModuleFilenameTemplate: "webpack:///[resource-path]", // string
                // 「devtool 中模块」的文件名模板

                devtoolFallbackModuleFilenameTemplate: "webpack:///[resource-path]?[hash]", // string
                // 「devtool 中模块」的文件名模板（用于冲突）

                umdNamedDefine: true, // boolean
                // 在 UMD 库中使用命名的 AMD 模块

                crossOriginLoading: "use-credentials", // 枚举
                crossOriginLoading: "anonymous",
                crossOriginLoading: false,
                // 指定运行时如何发出跨域请求问题

                /* 专家级输出配置（自行承担风险） */
            },

            module: {
                // 关于模块配置

                rules: [
                    // 模块规则（配置 loader、解析器等选项）

                    {
                        test: /\.jsx?$/,
                        include: [
                            path.resolve(__dirname, "app")
                        ],
                        exclude: [
                            path.resolve(__dirname, "app/demo-files")
                        ],
                        // 这里是匹配条件，每个选项都接收一个正则表达式或字符串
                        // test 和 include 具有相同的作用，都是必须匹配选项
                        // exclude 是必不匹配选项（优先于 test 和 include）
                        // 最佳实践：
                        // - 只在 test 和 文件名匹配 中使用正则表达式
                        // - 在 include 和 exclude 中使用绝对路径数组
                        // - 尽量避免 exclude，更倾向于使用 include

                        issuer: { test, include, exclude },
                        // issuer 条件（导入源）

                        enforce: "pre",
                        enforce: "post",
                        // 标识应用这些规则，即使规则覆盖（高级选项）

                        loader: "babel-loader",
                        // 应该应用的 loader，它相对上下文解析
                        // 为了更清晰，`-loader` 后缀在 webpack 2 中不再是可选的
                        // 查看 webpack 1 升级指南。

                        options: {
                            presets: ["es2015", 'react']
                        },
                        // loader 的可选项
                    },

                    //sass-loader, ->, css-loader, -> ,style-loader
                    {
                        test: /\.(css|scss)$/,
                        exclude: /(node_modules)/,
                        use: [
                            {
                                loader: "style-loader"
                            },
                            {
                                loader: "css-loader",
                                options: {
                                    modules: false,
                                    minimize: true,
                                }
                            },
                            {
                                loader: "sass-loader",
                                options: {
                                    modules: false,
                                }
                            },
                        ],
                    },

                    {
                        test: /\.html$/,
                        test: "\.html$"

                        use: [
                            // 应用多个 loader 和选项
                            "htmllint-loader",
                            {
                                loader: "html-loader",
                                options: {
                                    /* ... */
                                }
                            }
                        ]
                    },

                    { oneOf: [ /* rules */ ] },
                    // 只使用这些嵌套规则之一

                    { rules: [ /* rules */ ] },
                    // 使用所有这些嵌套规则（合并可用条件）

                    { resource: { and: [ /* 条件 */ ] } },
                    // 仅当所有条件都匹配时才匹配

                    { resource: { or: [ /* 条件 */ ] } },
                    { resource: [ /* 条件 */ ] },
                    // 任意条件匹配时匹配（默认为数组）

                    { resource: { not: /* 条件 */ } }
                    // 条件不匹配时匹配
                ],

                /* 高级模块配置（点击展示） */
            },

            resolve: {
                // 解析模块请求的选项
                // （不适用于对 loader 解析）

                modules: [
                    "node_modules",
                    path.resolve(__dirname, "app")
                ],
                // 用于查找模块的目录

                extensions: [".js", ".json", ".jsx", ".css"],
                // 使用的扩展名

                alias: {
                    // 模块别名列表

                    "module": "new-module",
                    // 起别名："module" -> "new-module" 和 "module/path/file" -> "new-module/path/file"

                    "only-module$": "new-module",
                    // 起别名 "only-module" -> "new-module"，但不匹配 "only-module/path/file" -> "new-module/path/file"

                    "module": path.resolve(__dirname, "app/third/module.js"),
                    // 起别名 "module" -> "./app/third/module.js" 和 "module/file" 会导致错误
                    // 模块别名相对于当前上下文导入
                },
                /* 可供选择的别名语法（点击展示） */

                /* 高级解析选项（点击展示） */
            },

            //实时监听代码变化
            devServer: {
                proxy: { // proxy URLs to backend development server
                    '/api': 'http://localhost:3000'
                },
                contentBase: path.join(__dirname, 'public'), // boolean | string | array, static file location
                compress: true, // enable gzip compression
                historyApiFallback: true, // true for index.html upon 404, object for multiple paths
                hot: true, // hot module replacement. Depends on HotModuleReplacementPlugin
                https: false, // true for self-signed, object for cert authority
                noInfo: true, // only errors & warns on hot reload
                // ...
            },

            plugins: [
                new webpack.optimize.CommonsChunkPlugin('vendor', 'react.bundle.js'), //这是第三方库打包生成的文件
                new webpack.optimize.UglifyJsPlugin(), //压缩js文件
                // ...
            ],
            // 附加插件列表

            performance: {
                hints: "warning", // 枚举
                maxAssetSize: 200000, // 整数类型（以字节为单位）
                maxEntrypointSize: 400000, // 整数类型（以字节为单位）
                assetFilter: function(assetFilename) {
                    // 提供资源文件名的断言函数
                    return assetFilename.endsWith('.css') || assetFilename.endsWith('.js');
                }
            },

            devtool: "source-map", // enum
            // 通过在浏览器调试工具(browser devtools)中添加元信息(meta info)增强调试
            // 牺牲了构建速度的 `source-map' 是最详细的。

            context: __dirname, // string（绝对路径！）
            // webpack 的主目录
            // entry 和 module.rules.loader 选项
            // 相对于此目录解析

            target: "web", // 枚举
            // 包(bundle)应该运行的环境
            // 更改 块加载行为(chunk loading behavior) 和 可用模块(available module)

            externals: ["react", /^@angular\//],
            // 不要遵循/打包这些模块，而是在运行时从环境中请求他们

            stats: "errors-only",
            // 精确控制要显示的 bundle 信息

            /* 高级配置（点击展示） */

            resolveLoader: { /* 等同于 resolve */ }
            // 独立解析选项的 loader

            parallelism: 1, // number
            // 限制并行处理模块的数量

            profile: true, // boolean
            // 捕获时机信息

            bail: true, //boolean
            // 在第一个错误出错时抛出，而不是无视错误。

            cache: false, // boolean
            // 禁用/启用缓存

            watch: true, //boolean
        // 启用观察

        watchOptions: {
            aggregateTimeout: 1000, // in ms
            // 将多个更改聚合到单个重构建(rebuild)

            poll: true,
            poll: 500, // 间隔单位 ms
            // 启用轮询观察模式
            // 必须用在不通知更改的文件系统中
            // 即 nfs shares（译者注：Network FileSystem，最大的功能就是可以透過網路，讓不同的機器、不同的作業系統、可以彼此分享個別的檔案 ( share file )）
        },

        node: {
            // Polyfills and mocks to run Node.js-
            // environment code in non-Node environments.

            console: false, // boolean | "mock"
            global: true, // boolean | "mock"
            process: true, // boolean
            __filename: "mock", // boolean | "mock"
            __dirname: "mock", // boolean | "mock"
            Buffer: true, // boolean | "mock"
            setImmediate: true // boolean | "mock" | "empty"
        },

        recordsPath: path.resolve(__dirname, "build/records.json"),
        recordsInputPath: path.resolve(__dirname, "build/records.json"),
        recordsOutputPath: path.resolve(__dirname, "build/records.json"),
        // TODO

    }



