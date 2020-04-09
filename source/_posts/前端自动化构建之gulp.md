---
title: 前端自动化构建之gulp
date: 2018-03-07 20:40
tags: [前端, gulp]
---


## 前言
之前学完html的基础后就去学js框架了，每次都是用脚手架搭好的文件，在无形中体验了一波前端自动化带来的方便。然后前一段时间才开始学习前端自动化。
<!--more-->

## 基本介绍
gulp说得简单一点就是一个自动化把前端的各种工具以流的方式一步一步的按照设置的规定加载的一个管理工具

## 装载
首先全局安装gulp

npm install --global gulp
然后你需要在根目录下创建一个名为gulpfile.js的文件
这个文件是gulp命令需要加载的文件，通过这个文件来处理预设的构建

## 使用
首先你可以在你的项目里面创建一些html和js文件，然后你的文件可以通过browserify来处理js的模块的文件，如果你每次修改js文件你需要每次都要在命令行里面运行browserify的处理。

## gulp来自动化browserify
如果通过gulp来自动化构建的话，你可以在配置文件gulpfile.js里面添加一个task（任务）
```
gulp.task('mainjs', ()=>{
  console.log('处理渲染')
  browserify().add('./assets/js/index.js').bundle().pipe(fs.createWriteStream('./js/main.js'))
})
```
在此之前你需要引入gulp和browerifi同时引入fs来找到文件

作用是使用browerify来把index.js打包成main.js在html页面只需要加载main.js就好
然后你每次都只要运行gulp就可以了。

## gulp来自动监听代码的变动
 你需要通过gulp自带的watch方法
新建一个task来调用watch，然后你需要在监听到后重新渲染mainjs这个任务，
可以通过runSequence来在一个task中调用另一个task

 
```
gulp.task('watch', () => {
  gulp.watch(['./assets/js/*.js'], ()=>{
  //监听到后就重新渲染
    runSequence('mainjs')
  })
})
```
监听到了assets中的js中的所有的js的变动就会调用后面的箭头函数

## 结语
这里只是初步的尝试了一下gulp的作用，gulp是通过流一样的任务的模式来处理你之前定义的加载方式
同时你还可以处理css和构建第三方库
gulp也有许多的方法可以灵活的调用