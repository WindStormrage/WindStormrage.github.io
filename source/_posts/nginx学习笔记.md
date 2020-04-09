---
title: nginx学习笔记
date: 2018-12-08 15:05
tags: file
---

<!--more-->

## nginx 安装
直接用brew安装，没有brew的自行安装
> brew install nginx
## nginx 基本命令
可以通过
> nginx -h

查看
> sudo nginx -t

查看nginx配置文件有没有语法错误，切记一定要用管理员权限运行，不然会出现错误

> sudo nginx

这个就可以开启你的nginx
> sudo nginx -s reload

可以重启你的nginx
你还可以把reload替换成stop, quit, reopen, reload
> sudo nginx -c filename

可以设置nginx的配置文件的目录，就是==nginx.conf==的位置

## nginx 简单配置

```
server{
	# 监听的端口
	listen 80;
	# 域名,当两个项目共用一个端口的时候就可以通过不同的server_name来访问到不同的服务器资源
	server_name test.cn;
	# 资源的根目录,如果资源在用户的文件夹里面的话可能会没有访问资源的权限,需要给文件设置777权限
	# 你可以通过pwd目录查看当前的目录
	root /html/demo;
	# 如果网址上没有对应的资源地址打开的页面
    index index.js index.html index.htm;
}

```
这是一个简单的nginx配置，你需要把配置写在你对应的==nginx.conf==里面，你也可以在nginx文件夹下新建一个文件夹servers然后在nginx.conf中引入这个文件夹下的文件,至于文件名你可以用test.conf
```
include servers/*;
```

你只需要放一个html文件放在 ==/html/demo== 下就可以通过==127.0.0.1==访问了，如果你想通过 ==test.cn== 访问的话你需要设置一个hosts
> 127.0.0.1 test.cn

如果发现出现403的错误的话可能有两种情况
* 可能在你的root配置的地址里面找不到对应的index
* 还有一种可能就是没有对应的权限了。

## nginx 坑之403错误
你可以查看一下你要访问的文件夹的对应权限==ll==命令就可以查看当前目录下的所有的权限了，关于权限的修改和设置你可以查看这个文章

[mac 查看、修改文件权限的命令](https://blog.csdn.net/x1876631/article/details/70162009/)

然后之前我是在桌面上建的文件夹，然后就算修改了权限后也不可以访问，是因为他的上一级文件夹没有权限进不来，如果你在最外面的 ==/== 目录下建立文件夹然后nginx就可以访问到，如果你想访问你这个用户下面的文件，你可以用node在你目录下开个服务然后用nginx反向代理过去。

## nginx 反向代理
nginx的简单的反向代理就很简单

而且还不会出现资源访问的问题

首先你需要开一个node服务在8888端口

可以不需要之前写的root

然后在server里面加上一段

然后你就可以直接通过==test.cn==访问到你的node服务了

记得重启一下nginx
```
server {
    # ...前面的代码
    location / {
        proxy_pass http://127.0.0.1:8888;
    }
}
```
然后我理解反向代理的作用是，如果你的服务器上有两个项目都需要跑在80端口上，然后你就可以通过nginx配置不同的==server_name==来访问不同的服务，然后还可以解决负载均衡等等的问题

关于具体的配置你可以看看[Nginx服务器的反向代理proxy_pass配置方法讲解
](https://www.jb51.net/article/78746.htm)

## nginx 解决跨域
nginx 解决跨域其实就是nginx反向代理的一个实现，他的实现原理是，你的后端跑在==localhost:81==端口，然后你的前端是跑在==localhost:82==端口，当你前端调用后端接口的时候由于端口不同浏览器会报跨域的错误，然后你需要把两个项目跑在同一个端口上才可以，这样你就会有疑问，同样的域名，一个端口怎么能跑两个项目，我们实现的方法是监听一下83端口然后如果你的请求的url是由 ==/api== 开头的就把它发送到81上去，否则就发送到82端口去。

具体的配置就是

```
server{
	# 监听的端口
	listen 83;
	server_name localhost;
    location /api {
    	rewrite ^/api$ / break;
        proxy_pass http://localhost:81;
    }
    location / {
        proxy_pass http://localhost:82;
    }
}
```
然后你的ajax可以这么写

```
	$.get({
		url: 'http://localhost:83/api',
		success: (data) => {
			console.log(data);
		} 
	})
```
nginx的配置要说明的是location后面的 **/** 和 **/api** 的意思是拦截url为这两个开头的请求，然后 **rewrite** 是Nginx的URL重写

```
    	rewrite ^/api$ / break;
```
* ==rewrite==是关键字
* 第二项是一个正则，我这里是匹配 ==/api== 这个字符串
* 然后第三项是第二项匹配到的正则需要替换成的字符串，我这里的意思是吧 ==/api== 替换成 ==/==
* 然后第四项是一个flag标记
* flag标记说明：

    last  #本条规则匹配完成后，继续向下匹配新的location URI规则
    
    break  #本条规则匹配完成即终止，不再匹配后面的任何规则
    
    redirect  #返回302临时重定向，浏览器地址会显示跳转后的URL地址
    
    permanent  #返回301永久重定向，浏览器地址栏会显示跳转后的URL地址

这样我就可以通过==http://localhost:83/index.html==访问到我的前端界面然后跨域请求到对应的接口了

参考资料

[nginx反向代理-解决前端跨域问题](https://www.cnblogs.com/renjing/p/6394725.html)
[nginx配置location总结及rewrite规则写法](https://segmentfault.com/a/1190000002797606#articleHeader1)

后来做项目的时候发现

```
rewrite ^/api$ / break;
```
只会吧/api替换成/,/api/data还是请求的/api/data的接口

然后改成了
```
rewrite ^/api/(.*)$ /$1 break;
```
如果/api/后面还有东西，就替换成/加上之前api后面的东西


[我的博客](http://www.xiedashuaige.cn/bolg2.0/#)