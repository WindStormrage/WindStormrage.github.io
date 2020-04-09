---
title: Docker学习笔记
date: 2018-12-18 16:59
tags: docker
---

## Docker介绍
最近花了点时间学习了一下docker，发现docker还是挺好用的，学了后也在实际中使用到了，就是一个静态页面需要跑在服务器上查看效果，如果用node跑的话还需要在项目中加上其他项目中没有用到的东西，然后用nginx的话还需要写个nginx的配置文件，如果在你的文件夹里面还需要设置权限，都比较麻烦，然后我就直接用上了docker，在之前使用docker的时候也装上过nginx的docker镜像，然后就只需要一条命令创建一个nginx的容器，然后把目标目录挂载到nginx上就可以访问到了，然后每次打开只需要docker start一下就好了，非常方便。
<!--more-->
我再来讲讲我对docker的认识，docker上你可以下载多个镜像，每个镜像都是一个环境，然后镜像可以跑起来创建一个容器，创建的这个容器是镜像的一个实例，对镜像没有影响，就和我们的new操作一样，一个镜像可以创建多个不同的容器，你也可以把容器进行修改打包成一个镜像保存起来，然后你可以移植到各个客户端上去使用，这也是docker的一个用处，在以前配环境需要执行各种命令，然后又因为各种版本不同的问题出现各种莫名其妙的问题，现在你可以打包成一个对应的镜像，使用的人只需要下下来然后run一下就好了。
## Docker准备
如果你需要使用（玩）docker的话你可以去[docker官网](https://www.docker.com)下载一个安装包具体的安装不同系统可能会有点不同，可以参考一下[菜鸟教程](http://www.runoob.com/docker/windows-docker-install.html)，其实如果你现在只是想尝试一下docker，并不想下载他的话这里有一个可以云把玩docker的地方，创建一个docker的账号然后[打开这里](https://labs.play-with-docker.com/)创建一个新的实例，里面就是一个已经有docker环境的系统一样，你就可以使用docker的各种命令去学习docker。
## Docker镜像
docker镜像(Images)就可以理解成是一个打包好的环境，和github一样，docker也有一个自己的hub，[DockerHub](https://hub.docker.com/)，是一个所有人储存镜像的地方，你可以获取到别人的镜像来使用比如说你可以搜索nginx然后就可以看到搜索项中的nginx，这是一个官方的docker镜像，比如windstormrage/nginx这种就是用户自己上传的镜像，一般使用还是使用官方的比较安全，然后你也可以上传你的镜像。
## Docker容器
docker容器(Container)可以理解成是镜像实例化出的一个系统，你可以运行容器中的命令，然后也可以打自己本地的目录挂载到容器对应位置，然后也可以把容器接口映射到本地上打开。
## Docker基本使用
我们就通过一个小栗子来学习一下docker的操作，我这里使用playDocker来操作，你可以用你的电脑来操作。
### 拉取镜像
我们拉取到一个nginx的镜像
```
docker pull nginx
```
docker会自动从你的源(DockerHub)上找到名字为nginx的镜像，然后拉取下来。
然后你可以通过命令来查看你本地拥有的镜像
```
docker images
```
![img](http://123.207.39.128:8080/upload/file/6d6a18333a8bb35225d0fef1eb06b9b5)
### 运行镜像
拥有了一个镜像后你就可以运行它生成一个容器
```
docker run -d -p 8360:80 nginx
```
![img](http://123.207.39.128:8080/upload/file/e8b5fe22b6cee9109033892ab8c6a201)

其中-d是让容器在后台运行，-p是把内部的端口映射到我们的主机上面，我们这里是把nginx的80端口映射到了我们的8360端口。然后返回的哈希值是我们当前的容器的id，使用这个id我们可以操作我们的容器。比如说我们可以通过
```
docker stop cf
```
其中cf是对应id的前几位，他只要可以找到对应的容器就可以来停止这个容器，然后如果是使用的playDocker来操作的话你可以点击这里来查看对应端口的页面

![img](http://123.207.39.128:8080/upload/file/8a529bf42503ae750ac7d8fb800141db)

如果你用的是自己的电脑的话你可以打开localhost:8360看到你nginx运行起来的页面了。然后你可以通过
```
docker ps
```
查看你运行中的容器

你可以在DockerHub的[nginx](https://hub.docker.com/_/nginx)查看更多关于这个镜像的使用。

比如说我上面说的挂载项目就可以使用
```
docker run -p 8360:80 -v /html/demo:/usr/share/nginx/html -d nginx
```
这个应该就只能在本地尝试了，你第一次挂载的时候docker会找你要权限，你允许就好了

其中-v就是挂载，然后我的项目是在/html/demo上。然后你打开localhost:8360就可以看见你demo目录下的文件了。
### 修改容器
其实就是在你运行镜像的时候可以加上你需要运行容器里面的命令
```
docker run learn/tutorial apt-get install -y ping
```
在你运行learn/tutorial的时候docker发现你并没有这个镜像，然后docker会自己去hub上去找，然后下载下来，然后后面的apt-get install -y ping就是在tutorial中安装ping命令。 你可以通过
```
docker ps -a
```
来找到你当前的容器，你修改了容器后可以通过commit命令来保存你这次修改，他的保存是生成一个新的镜像。
```
docker commit fc windstormrage/ping
```
其中的fc是你容器的id，然后后面跟着的是你的镜像名称，镜像名称最好是以你的DockeHun的名称加上/然后加上镜像名组成，不然到后面你不能上传，因为镜像只能上传到自己名字的空间下。 

![image](http://123.207.39.128:8080/upload/file/ac3af2b9037ff70af51861c7a230b059?ynotemdtimestamp=1545116832831)

然后你可以通过run命令来运行你刚刚打包了的镜像
```
docker run windstormrage/ping ping www.google.com
```
然后你需要知道的是你每次run都是创建了一个新的容器，对你之前创建的容器是没有影响的。
### 上传镜像
发布镜像很简单
```
docker push windstormrage/ping
```
注意的就是你的用户名需要正确，不然会报错。
### 通过dockerfile来创建镜像
其实docker镜像一般是通过dockerfile来创建，我们这里只是简单的入门，暂时不涉及到自己手动来创建一个镜像，如果感兴趣的可以看看后面的参考资料。
## 参考资料
[菜鸟教程](http://www.runoob.com/docker/docker-tutorial.html)

[docker官方入门教程](http://www.docker.org.cn/book/docker/what-is-docker-16.html)