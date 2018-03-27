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

```
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

```
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

```
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



