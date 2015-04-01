2015年03月31日
=====

## docker安装和镜像设置

在[官方文档](http://docs.docker.com/installation/ubuntulinux/)中详细的介绍了在
ubuntu 14.04中安装docker的方式.

在ubuntu中,可以使用

	$sudo apt-get install docker.io
来安装docker,但是会发现其安装的版本是很低的.

所以官方使用了一个自己的源. 这个文档中实际上是使用了一个脚本来安装.

安装完成之后,运行

	$ docker pull ubuntu
会从官方拉下来ubuntu这个repository中的所有镜像.

一个问题就是docker hub在国外,导致下载这个镜像需要很久的时间.

在国内,daoCloud这个公司提供了一个docker hub的镜像.

[这个链接](https://dashboard.daocloud.io/mirror)中介绍了怎么使用daoCloud提供的
镜像来加速我们的镜像下载过程.

安装了之后,会发现下载的过程明显的加快了.

其实际进行的操作就是修改`/etc/default/docker`文件,这个文件中设置的值会在使用

	$ sudo service docker restart

这样实际启动的时候运行的命令就是


	docker --registry-mirror=http://bf2350bc.m.daocloud.io -dD

## docker开发环境的建立
docker的核心开发人员在开发docker的时候,使用了像编译器开发人员一样的模式,就是用一个
稳定版本的docker来开发下一个版本的docker.

fork一个docker的源代码
---
首先需要在github上fork并且clone一个docker的源代码下来.

编译Dockerfile
---
上面我们已经安装了一个稳定版本的docker了.所以可以用来编译我们的开发环境的镜像了.
切换路径到clone下来的docker目录下

	$ docker build -t my-tag .
会发现并不能很成功的通过每一个命令.

修改原始的Dockerfile
---
1.在开始安装基本的软件之前,更换源

```
# CHANGE: 在国内需要使用另外一个源,ubuntu官方源太慢了
RUN echo "deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse" >/etc/apt/sources.list
RUN echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse" >>/etc/apt/sources.list
RUN echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse" >>/etc/apt/sources.list
RUN echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse" >>/etc/apt/sources.list
RUN echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse" >>/etc/apt/sources.list
```
因为这个Dockerfile使用的是ubuntu 14.04开始构建的,其默认的软件源再国内会很慢,
换成阿里云的源.


2.安装golang的时候,默认下载源代码是从

	curl -sSL https://golang.org/dl/go${GO_VERSION}.src.tar.gz

同样的,golang.org是被墙了的,需要换成国内的

	curl -sSL http://golangtc.com/static/go/go${GO_VERSION}.src.tar.gz

3.编译golang的工具的时候,是先将当前平台的工具连构造出来,然后又构造了所有支持的其他
平台的工具连,如果不想要这些工具连的话,可以注释掉.

4.安装完了golang之后,又会安装以前版本的gofmt和go cover,还会使用gem下载一个ruby的包,
这些网址都是被墙了,所以注释掉.

5.安装busybox和hello-world两个镜像的地方可以注释掉,因为现在使用的是docker hub,下载
起来会相当慢.

总之,Dockerfile文件并不复杂,如果发现不是必须的东西,而且因为网络的原因还下载不了,那么
就只能跳过一些步骤了.
但是其中对golang的安装和docker源代码的下载是必须的,因为docker是使用go写的,而且我们是
基于现在的源代码来修改docker的.

2015年04月01日
===

首先,在一个terminal中,将当前的路径切换到我们的docker源文件所在的路径,我的是

	cd /home/cxy/code/docker
这个路径中的代码也就是和我在github上面的docker源代码同步的路径.

运行

	make
在这个路径下面,有一个Makefile,docker使用make来完成生成镜像的工作,后面会仔细分析这个
镜像的内容. 其在build的时候,使用的也是这个目录下面的Dockerfil文件. 完成之后,会生成一个
`docker:master`的镜像

	docker run --privileged --rm -ti -v `pwd`:/go/src/github.com/docker/docker docker:master /bin/bash

这里也是使用了我们的my-test这个mirror来启动一个container,其中的

	-v `pwd`:/go/src/github.com/docker/docker
表示将我们本地的docker源文件目录挂载到容器中的`/go/src/github.com/docker/docker`,
这样我们只需要改变本地的代码,在容器中就同步的更新了.

进入容器之后,保证当前的目录是`/go/src/github.com/docker/docker`,也就是本地源代码
映射过来的路径.

	cp bundles/1.5.0-dev/binary/docker /usr/bin
将上面make生成的docker执行文件放到PATH中,这样就可以直接运行了.

	docker --registry-mirror=http://bf2350bc.m.daocloud.io -dD
在这个容器中使用我们正在开发的docker建立一个daemon.

在本地再打开一个终端,
	
	docker exec -ti xxx bash
其中的xxx可以使用`docker ps`看到,其为docker:master对应的容器.
这样就在另外一个终端中也进入了这个容器了.

	docker run hello-world
在正在开发的docker生成的容器中,运行hello-world这个用于测试的镜像,如果成功了,那么说明
现在是可以使用的了.
	
>需要注意的是,在dcker:master这个镜像中,`/go/src/github.com/docker/docker`中本来也是有
代码的,像这样挂载之后,里面的代码就看不到了.

>注意,当我们修改Dockerfile的时候,最好能够做到增量式的修改,就是总是在最后的地方添加,
这样就能保证下一次生成一个镜像的时候更多的使用前面已经使用过的缓存.


