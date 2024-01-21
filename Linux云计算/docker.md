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

