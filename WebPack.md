# webpack技术入门
webpack基本已经成为前端项目的标配构建工具了。

然而，在一个前端团队里面，除了架构师之外，其他开发者很难有机会在工作中完整的实现整个配置流程。

本篇文章是我在公司里面分享webpack及babel配置和插件开发的一个细节版本，

希望能让大家通过阅读本文，比较好的梳理webpack工具。

# webpack概述
webpack = module building system。

在webpack看来，所有的资源文件都是模块(module),只是处理的方式不同。

上面两句话就把webpack从top-level的角度讲清楚了。

关于webpack的使用，其实和我们平时开发业务产品是一个道理。

产品需求 ===> 代码设计 ===> 提供API给开发者使用。

webpack解决的需求点就是如何更好的加载前端模块。

这里我用了模块二字，是因为webpack从JS出发，将所有的文件看做它要处理的模块。

webpack本身并不关心这个模块是什么，它只是调度配置文件中对模块处理的方式来完成这一切,而不必纠结文件类型。

比如我们会在项目中使用ES6/7的语法来编写JS代码,

于是我们只需要配置babel-loader即可处理这种JS。

比如我们需要加载html文件获取html字符串,

于是我们只需要配置raw-loader即可拿到对应文件的字符串。

比如我们需要将sass/less文件预编译成css，

于是我们只需要配置sass-loader/less-loader即可处理。

# 一个简单而通用的webpack配置文件
```
var webpack = require("webpack");
var DefinePlugin = require('webpack/lib/DefinePlugin');
//module.exports是commonJS也就是nodejs把模块导出的方式
module.exports =  { 
        //context上下文，webpack编译的上下文
        //webpack运行环境是node.js
        //process.cwd()是默认值,是node.js的启动目录。
        context:process.cwd(), 
        
        //watch去观测webpack编译entry之后是否动态编译，每次文件改变的之后就会去让webpack动态的更新
        watch: true,
        
        //webpack打包的时候会去读entry，会告诉我们去哪里找。===> context ===> ',..index.js'
        //配合context，这个entry会相当灵活
        //意思是path.resolve(context,entry)最后都是生成绝对路径，不依赖于整个webpack文件在哪里定义
        entry: './index.js',
        //也可以写成
        //entry:{
        //    test：'/index.js'
        //}这样output的name就是test
        //上面三个是webpack的编译入口，context是确定了webpack编译上下文，entry如果是个相对路径，是相对于context的
        
        //devtool是开发者工具。指出我编译出来的文件是不是要满足source-map
        //'source-map'资源映射表，chrome debug时，编译的是源文件index.js而不是编译出来的文件
        //编译出来的文件是不方便调试的
        devtool: 'source-map',
        
        //output导出，描述webpack导出到哪边
        //是个对象，path是指到到哪边 运行目录的dist目录下面
        //filename是导出文件的文件名，不指定会用默认值
        output: {
            path: path.resolve(process.cwd(),'dist/'),
            filename: '[name].js'
        },
        
        //resolve给开发带来便利性
        //alias可以称为别名。
        //这里定义jquery这样一个字段是后面这个字段
        //require('jquery') ===> requere('process.cwd()+'/src/lib/jquery.js')
        resolve: {
            alias:{ jquery: process.cwd()+'/src/lib/jquery.js', }
        },
        
        //plugins插件，核心之一。
        plugins: [
            new webpack.ProvidePlugin({
                $: 'jquery',
                _: 'underscore',
                React: 'react'
            }),
            new DefinePlugin({
              'process.env': {
                'NODE_ENV': JSON.stringify('development')
              }
            })
        ],
        
        //最重要，对模块处理的核心逻辑。
        //module是一个对象。
        //loaders是加载方式，是个数组，数组的每一项都是个对象，test是正则表达式，exclude是排除
        module: {
            loaders: [{
                test: /\.js[x]?$/, //以.js或.jsx结尾，就会去调用下面的loader
                exclude: /node_modules/, //路径里有node_modules，排除在loader处理的范围
                loader: 'babel-loader' //加载器，用来处理相关的文件。如.js index.jsx。一个js文件就是一串字符串。
                //将字符串通过loader加载器转换成我要的另外一种形式字符串。
                //比如es6/7 通过这个loader就可以通过loader转化成es5.只是将一种代码形式转化成另一种代码形式。  
                //less/sass转化成css
            },  {
                test: /\.less$/,
                loaders:['style-loader', 'css-loader','less-loader']
                //这边是loaders，转化成loader这么写
                //loader:"less-loader!css-loader!style-loader" 
            }, {
                test: /\.(png|jpg|gif|woff|woff2|ttf|eot|svg|swf)$/,
                loader: "file-loader?name=[name]_[sha512:hash:base64:7].[ext]"
            }, {
                test: /\.html/,
                loader: "html-loader?" + JSON.stringify({minimize: false })
            } ]
        }
    };
```

webpack不会做代码分析。在webpack看来，entry里的都是字符串而已。 
会对entry里的代码进行操作，经过loaders处理，输出到output
babel里面的preset是一个plugins集合

loader是从哪里找来的：
- webpack.config的运行环境是node.js
- 依赖包的寻址方式是node_modules递归往上查找(也是node.js里的寻址方式)，查不到会报错
- package.json里的所有的依赖包都有写清  
  
webpack里面有一套自己的require逻辑，和node.js类似，是node.js模块引入的超集合  

style-loader在js里面创建style标签，然后将结果输出到style标签里面

webpack.config.js:
```
module.exports = {
    entry: './main.js',
    output: {
        filename: 'bundle.js',
    },
    module: {
        loaders:[
        {
            test: /\.(png|jpg)$/,//会读取main.js里require的png和jpg文件，满足正则就进行loader的操作
            loader:'url-loader?;imit=8192'     
            //要这个图片大小要小于8192，url-loader的限制limit产生的效果，会变成base 64,大于的就直接按照hash算法修改名字 
        }
        ]
    }
}
```
### plugin
plugins是在webpack的不同编译阶段去做不同的事情，一般是生产环境做而不是开发环境。  
一般是做一些个性化工作，是个性化webpack的行为。   
比如uglifyJsPlugin会压缩和混淆代码。像jquery.min.js  
混淆代码mangle,hello world===>h  这样代码量就变少了
```
plugins:[
    new uglifyJsPlugin({
      compress:{
        warnings: false
    })
]
```


#### gulp与webpack
gulp和webpack定位是不同的，一个是steam building system 一个是module building  system    
gulp是工作流，webpack是模块加载器  
uglifyjs的源码是一样的，都用了gulp或者webpack不同的包装方式，套了个外壳。  
gulp是流的模式.  
webpack某种程度上是为前端打造量身打造的。很多功能都是通的，只不过webpack就是前端加载器，gulp是制定工作流，    
比如打包代码，上传服务器cdn之类的。   
gulp的插件都会引用through2模块，这个模块是node中的stream实例。  

### loader的加载顺序
loader: 'style-loader!css-loader'  
loaders:['style','css']  
这两者是一致的，  
加载是从右往左的  


### require.js
require.js是一个AMD规范的加载器，是AMD的一个子集，和webpack是两码事。
webpack能够兼容AMD规范，比require.js强大很多

### 优化缓存及懒加载
在生产环境中，将输出文件名添加hash值，目的是在文件更改时强制客户端重新加载这个文件，而未改变的文件继续使用缓存文件。常用的有hash和chunkhash，二者的区别请参考hash与chunkhash。配置文件中的[chunkhash:8]即截取8位chunkhash值。　　
webpack的编译理念：webpack将style视为js的一部分，所以在计算chunkhash时，会把所有的js代码和style代码混合在一起计算。比如entry.js引用了main.css：
```
import 'main.css'; 
alert('I am main.js');
```
