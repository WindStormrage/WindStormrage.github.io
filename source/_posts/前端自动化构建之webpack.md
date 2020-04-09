---
title: 前端自动化构建之webpack
date: 2018-03-12 20:16
tags: [前端, webpack]
---


### 前言
学了gulp后马上就开始学了一下webpack，所以马上来谈一下感受，感觉webpack有人说是一个模块化工具，用来和browserify来做比较，我感觉webpack牛逼多了，不但可以把复杂的文件结构打包在一起，而且还可以预编译各种js，css语言，同时可以帮你处理好html，感觉和gulp有的一拼
<!--more-->
### 基本介绍
本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器，把模块需要的文件打包成一个或多个bundle
### 装载
首先install好webpack-cli和webpack，然后在你的根目录新建一个webpack.config.json的配置文件，webpack的主要配置都在这个里面
### 使用
#### webpack来打包简单的js文件
首先先建两个互相有联系的js文件，
然后在配置文件中创建一个module.exports把配置类容引出
定义一个入口参数entry，一个入口文件为一个值，多个入口文件为一个数组
值就是对应的地址
然后添加一个出口参数output，为一个对象，里面有一个filename值为输出的文件
如：
``` javascript
module.exports = {
    //入口文件
    entry: ['./src/script/main.js', './src/script/a.js'],
    //打包输出
    output: {
        filename: './dist/js/bundle.[chunkhash].js'
    }
}
```
其中chunkhash是编译当前文件的随机哈希值
然后你可以在js中引入css文件，如

``` javascript
require('./world.js')
//css-loader是让webpack处理.css文件
//style-loader是让css文件插入到html里面
//require('style-loader!css-loader!./style.css')
//用了 --module-bind 'css=style-loader!css-loader'后可以不要每次都处理了
require('./style.css')

function hello (argument) {
    alert(argument);
}

hello('hello world');
```
最后只要在html中引入打包后的文件就好了
还有如果在packge.json里面的scripts里面加入运行webpack的命令如
``` javascript
    "webpack": "webpack --module-bind 'css=style-loader!css-loader'"
```
	以后在命令行中只要运行npm run webpack就好了
	
#### webpack对html页面的生成
前面的demo需要你自己把打包后的文件引入html中，其实也有自动生成html打包后文件的一种方法，加一个plugin（插件）
``` javascript
    plugins: [
        new htmlWebpackPlugin({
            filename: './dist/bundle.index.html',
            template: 'index.html',
            inject: 'head',
            title: 'demo'
        })
    ]
```
htmlWebpackPlugin是打包后可以直接重新组合html的一个插件
然后如果是单页面的程序，你可以new一个htmlWebpackPlugin，
如果是多页面的程序你就需要多new几个htmlWebpackPlugin
你也可以从配置文件传参到html页面，
htmlWebpackPlugin的构造方法中的参数可以在html通过模板语言获取
``` jsp
<%= htmlWebpackPlugin.options.title %>
```
你还可以在output中设置publicPath，从而使上线后html中引入外部地址而不只是相对位置
htmlWebpackPlugin也有很多配置，如压缩html
如果是多页面多个htmlWebpackPlugin，可以把entry写成一个对象，然后配置chunks，来指定打包哪个js
	
#### webpack打包预处理其他语言
前面只是对html，js，css的打包，而接下来我要介绍的是对各种语言各种文件进行预处理
这里用到的是webpack的Loader
这里先示范对js的处理
你需要把es6代码编译成es5的
所以你需要用到babel
	npm install babel-loader babel-core babel-preset-env webpack --save-dev
	然后你需要配置webpack
``` javascript
    module: {
        loaders: [
            {
                test: /\.js$/,
                loader: 'babel-loader',
				query: {
					presets: ['latest']
				}
                //path.resolve(__dirname, )
                //项目运行路径
                exclude: path.resolve(__dirname, 'node_modules')
            }
        ]
    },
```
这句配置的意思就是把后缀名为js的文件用babel-loader处理
然后其他配置也是一样的
### 结语
我这里只是简单的对webpack进行简单的配置，其实还有很多牛逼的配置，以后慢慢学