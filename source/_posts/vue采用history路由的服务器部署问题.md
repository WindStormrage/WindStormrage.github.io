---
title: vue采用history路由的服务器部署问题
date: 2018-06-21 23:21
tags: vue, vue-router
---

<!--more-->

## 发现部署问题
在部署的时候发现打开的页面是空白

## 之前部署原理
之前的页面都是作为静态文件形式打包上传到服务器上

http://www.xiedashuaige.cn/bolg2.0/#/home

就和这个页面一样，我其实上只有一个页面/bolg2.0

然后前端的路由切换都是根据后面的哈希值来变化

然后不同的哈希值指向的页面还是/bolg2.0页面

所以就放在静态目录都可以访问

## 部署问题解析
然后我用了history路由后打开的页面页面的时候发现服务器报404

http://www.xiedashuaige.cn/BolgAdmin/admin

首先我在服务器上对应的静态页面是/BolgAdmin页面

但是我前端路由的首页是/BolgAdmin/admin这个页面

但是服务器以为/BolgAdmin/admin是单独的一个页面资源

然后又找不到这个资源，所以就会报404

## 分析问题
然后我想了两种解决方法
* 一种就是在服务器上设置一个转发，把所有/BolgAdmin下面的子路由全部转发到/BolgAdmin页面下，但是对于服务器的我不太了解
* 通过node写一个后端就像http://www.xiedashuaige.cn:3000一样，然后访问http://www.xiedashuaige.cn:3000/BolgAdmin/admin的页面全部转发到http://www.xiedashuaige.cn:3000/BolgAdmin上面，这样就可以通过node来实现，比使用apache来改应该简单一些，我还在研究中。。

## 解决问题
正好在学koa就用koa搭建了一个服务器，代码如下

```
const fs = require('fs')
const Koa = require('koa')
const route = require('koa-route')
const path = require('path')
const static = require('koa-static')
const app = new Koa()

const main = ctx => {
  ctx.response.type = 'html'
  ctx.response.body = fs.createReadStream(path.join(__dirname,  '/index.html'))
}
const toMain = ctx => {
  ctx.response.redirect('/admin/')
}

const staticFile = static(path.join(__dirname, '/'))

app.use(staticFile)
app.use(route.get('/', toMain))
app.use(route.get('/admin/*', main))

app.listen(3001)

```

其实就是搭建了一个静态目录，然后把/目录重定向到/admin目录下，然后把/admin/*目录全部打开index文件

然后这样就可以打开vue  history模式的单页面应用了

## 结语
其实吧最后还是有一个问题，是针对于我这个项目的。我这个项目使用了vue的代理跨域，然后后端是用go写的跑在另外一个端口，所以最后直接把打包后的文件让go来做相同的处理，其实主要是了解了一波history模式会出现的问题咯。