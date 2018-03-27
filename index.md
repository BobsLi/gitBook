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

![](/assets/111.png)

### 自动化构建工具就是用来让我们不再做机械重复的事情，解放我们的双手的。



