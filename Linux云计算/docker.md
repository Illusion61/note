# Docker

## Docker原理

#### **为什么要用docker?**

- **环境(配置/移植麻烦)**:
  - 企业中有开发环境(写代码),测试环境和生产环境(对外使用),每个环境都要使用相同的配置,需要**可移植性**比较好
  - 开发环境用的windows,测试环境用的MacOS,生产环境用的Linux.需要一个统一的配置能够在各个平台上不会切换
  - 程序出现BUG很多时候都是因为环境不一致,他用的node v3.0,我用的node v3.1,我就会报错
- **应用隔离**:多个环境之间可能会产生冲突,比如nodev3.0和nodev3.1,以及PHP和.NET不兼容等
- 伸缩服务:在大集群中,可以使用k8s进行伸缩服务,在需求多的时候多运行几个docker容器

#### Docker和虚拟机差别

- 虚拟机:内核+Linux发行版,核心是指令的翻译

- 容器:(宿主机内核)+Linux发行版

**容器体积小效率高**,虚拟机必须和硬件资源绑定(分配CPU核和内存)而且会有多余的内核和图形化界面,可能只是一个很小的nginx服务就占用大量无关的资源

容器在运行的时候以**进程的形式**存在:一台物理机可以有几百个容器(相互调度),但是只能有为数不多个虚拟机(占用资源)

#### docker基本概念

- 镜像(images):一个静态模板,不能更改(食谱)
- 容器(containers):模板运行的实例,一个模板可以运行多个实例(按照食谱做出来的菜),可以再更改然后重新提交为新镜像(创造新食谱)
- 仓库(registry):分享镜像(分享食谱),比较有名的是docker hub

#### docker架构

- client客户端:与用户交互接收各种命令并发送给docker server
- server服务端:包括一个docker daemon守护进程负责接收各种客户端命令,还包括images和containers
- 远程仓库:可以用来下载镜像

## docker安装配置

#### docker engine安装

- 参考docker官网:**docker engine**进行安装
  - 设置好docker的apt仓库(密钥,软件源下载列表,apt update)
  - 安装docker packages:`apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`
- 启动docker服务:`systemctl restart docker`

- 备注:
  - (参考文章:[如何在ubuntu上面安装docker](https://zhuanlan.zhihu.com/p/143156163))
  - 安装路径:/var/lib/docker
  - 验证是否安装:`docker info`

#### docker安装



# Docker操作

## 镜像管理

```bash
docker search xxx:搜索xxx的镜像
docker pull xxx:latest:下载xxx的镜像到本地
docker images:显示有哪些镜像
	docker images -q:显示所有镜像ID
	docker images centos:查找centos相关的镜像
	docker images --format "table {{.ID}}\t{{.Repository}}":格式化显示
	docker images --format "table {{.ID}}\t{{.Repository}}":表格显示
docker rmi 镜像ID:删除指定镜像(如果依赖容器正在运行需要先删除容器)
	docker rmi $(docker images -q):删除所有镜像
```

- 注意:容器ID或者镜像ID可以不全,前几位能够标识即可

## 容器管理

#### 操作正在运行的容器

```shell
docker ps:显示正在运行的容器
    docker ps -a:显示所有容器
    docker ps -q:显示容器ID
docker exec [容器名|容器ID] [命令]:执行容器
	docker exec -it [容器名] bash:进入容器交互界面(一般cd,rm,ls,cat,echo等命令都有,但是vim,systemctl等命令就没有)
docker start [容器名|容器ID]:开启容器(在停止之后)
docker stop [容器名|容器ID]: 停止容器
重启容器服务:先stop再start
docker cp /var/www/html nginx:/var/www/html: 将本地文件复制到名为nginx的容器中的/var/www/html中
```

#### **运行镜像为容器**

- 后台运行
- 路径映射:主机中配置文件夹和资源文件夹映射到docker容器中(两个对文件的修改是同步的)
  - 映射配置文件和资源文件
  - 或者使用`docker cp`将配置文件和资源文件复制进容器中
- 端口映射:网络端口进行映射
- 名称:便于识别区分众多的docker容器

```bash
docker run 镜像名称:创建容器
	-d:后台运行
	-v [本地路径]:[容器中路径]
	-p 80:80 -p 443:443:开启端口映射
	-it:开启控制台界面,i表示标准输入,t表示交互???
	--privileged=true:特权模式,可以使用systemd
	--name dnsserver:容器名称
	--rm:结束后自动删除容器
	-P:所有端口映射(注意是随机分配本地端口,映射的端口数字不一定相同)
```

```shell
docker run -d -v /etc/nginx:/etc/nginx -v /var/www/html:/var/www/html -p 80:80 --name nginx nginx
```

#### 容器提交和删除

```shell
docker commit 容器ID 新镜像名称:保存容器为新镜像
docker rm [容器ID]:删除指定容器
	docker rm $(docker ps -aq):删除所有容器
```

## 分享镜像

#### 镜像导入导出

```bash
docker image save centos:7.2.1511 > ~/centos :将镜像导出为一个文件
docker image load -i ~/centos :从单个文件中加载镜像(-i指定加载的镜像文件路径和名称)
```

#### 仓库管理



## docker配置centos镜像

启动容器:

```shell
docker run -P -it --privileged=true --name dnsserver centos /usr/sbin/init
```

```bash
docker run --name bind -d --restart=always --publish 53:53/tcp --publish 53:53/udp --publish 10000:10000/tcp --volume docker-bind:/data sameersbn/bind:9.16.1-20200524
```

初始化:配置yum(从centos官网上下载配置文件覆盖自己的配置文件)

```
cd /etc/yum.repos.d/
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
yum update -y
```

退出centos但是保持运行:ctrl+P然后ctrl+Q



# Dockerfile

## Dockerfile基本介绍

#### Dockerfile简介

- Dockerfile描述了docker容器的依赖和构建过程

- 可以根据Dockerfile中描述的构建过程自动搭建容器

#### 为什么要使用Dockerfile

- **可定制性**:相比docker容器,dockerfile允许接收方能够**更好地查看并修改**镜像中的部分
- 安全性:dockerfile允许接收方进行审查有无安全风险
- 减少传输量:一个文本文件仅仅几十KB,而相应的镜像可能多达几十GB
- 版本控制与协作:dockerfile可以使用git进行版本控制,并且可以由团队成员创建分支共同开发

## Dockerfile的指令介绍

#### 基础指令

```dockerfile
FROM 指定基础镜像
MAINTAINER 指定维护者信息,可以没有
RUN 执行命令(install之类的)
ADD 添加宿主机压缩包文件到容器内,自动解压
COPY [宿主机相对文件/目录路径] [目标镜像中路径] :拷贝宿主机文件到容器内


ENTRYPOINT ["参数1","参数2"...]: 指定容器启动程序以及参数,有了ENTRYPOINT之后CMD就会变成ENTRYPOINT的参数
	在docker run的时候输入的参数会覆盖CMD参数,并且会作为ENTRYPOINT参数的补充
CMD ["参数1","参数2"...] : 指定容器启动之后执行的运行参数
	CMD ["/bin/bash"]:在容器启动之后执行/bin/bash作为交互器
	CMD ["cat","/etc/os-release"]
	在docker run的时候输入的参数会覆盖CMD指定的参数
	
ENV 变量名=变量值 :定义环境变量,之后可以使用$变量名来进行调用.在镜像构建和容器运行时都生效
	ENV MYSQL_VERSION=5.6
	ENV NAME="illusion"
ARG 变量名=变量值 :定义环境变量,只在镜像构建的时候生效,容器运行时失效

VOLUME :容器在运行的时候
EXPOSE 在容器内暴露一个端口给主机,容器端口默认全封闭(EXPOSE 80)
WORKDIR 指定工作目录(就是cd)
USER 11
```

- COPY指令需要让被复制的文件和Docker处于相同目录或者子目录下,并且使用相对路径
- dockerfile命名需要是`Dockerfile`首字母大写其他小写
- **容器中程序一般都是前台运行,没有后台运行的概念**;容器就是为了主进程存在的,如果主进程结束,容器就应该退出因为容器就失去了存在的意义
  - 容器中不能用systemctl运行守护进程
  - 如果没有前台运行,容器就会立即退出
- 容器在运行的时候,应该保证在存储层不写入任何数据.**运行在容器内产生的数据**,我们推荐是**挂载,写入到宿主机上进行维护**
  - 日志文件
  - 用户数据

#### 使用Dockerfile创建镜像

`docker build -t [image_name] [path]`

#### Dockerfile实践

- 需求1:通过Dockerfile,构建nginx镜像.且运行容器之后,页面为welcome to dockerfile

```dockerfile
FROM nginx
RUN echo '<h1>Welcome to dockerfile!</h1>' > /usr/share/nginx/html/index.html
```

- 需求2:自定义nginx服务配置的主页

```shell
FROM nginx
RUN rm -rf /usr/share/nginx/html/*
COPY html/* /usr/share/nginx/html
```

- 需求3:获取当前IP,或者使用-I获取请求头

```dockerfile
FROM centos:7.8.2003
RUN rpm --rebuilddb && yum install epel-release -y
RUN rpm --rebuilddb && yum install curl -y
ENTRYPOINT ["curl","-s","http://ipinfo.io/ip"]

docker run myapp -I 获取请求头(相当于curl -s http://ipinfo.io/ip -I)
docker run myapp 获取当前IP(相当于curl -s http://ipinfo.io/ip)
```



