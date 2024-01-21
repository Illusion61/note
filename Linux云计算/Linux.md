# Linux用户管理

引入:多个用户同时使用一个服务器,bob03操作数据库文件,jack01操作网站信息,amy02查看系统运行日志

## 用户管理简介

#### 用户是什么

- 用户就是你管理系统的一个账号

- 将用户分为特权用户和普通用户,防止普通用户造成实际损害保证系统安全

#### 用户分类

- 系统通过useradd创建的普通用户
- 安装软件时候自动生成的用户(eg:nginx,hadoop,mysql)
  - 防止软件在遭受入侵时候或者软件崩溃时候造成更大的损失
  - 便于查看软件用户日志追踪错误
- 系统默认的一些虚拟用户(eg:mail,news,proxy):用来规定和限制相应软件权限

#### 用户组是什么

- 用户组方便管理一类用户,用户组也有相应的权限(没法让文件归属于多个用户)
- Linux中文件既属于某个用户,也属于某个组

- 用户和用户组:
  - 单一用户的组:**用户在被创建的时候默认属于自己同名的用户组**
  - 多用户组:组中有多个用户
  - 一个用户也可以属于多个用户组享有多个组的权限
- 用户组分类:
  - 主组:用户创建的时候自动创建一个同名的组,称之为主组,主组只能由一个
  - 附加组:用户后面其他的组(用户既具有主组的权限,也具有附加组的权限)

## 用户配置文件

#### root权力为什么大

- UID:user-identity,相当于身份证号
  - 在Linux系统中,用户有自己的UID身份账号而且唯一
  - Linux当中UID为0就是超级用户,如果要设置管理员用户可以更改UID为0
  - 系统用户UID为0-999,普通用户UID从1000开始分配
- GID:group-identity,相当于户口本的家庭编号

#### 用户创建配置文件

```
/etc/passwd:存放了所有用户的信息
	用户名:密码:UID:GID:用户注释:用户家目录:用户用的解释器(/bin/bash,/sbin/nologin)
	ubuntu:x:1001:1002:,,,:/home/ubuntu:/bin/bash
/etc/shadow:存放了用户密码
	用户名:加密密码:修改日期(自1970.1.1过了几天):密码距离失效天数(0不失效):提前几天提醒失效:密码失效宽限天数....
	root:$y$j9T$w/G...ALtC:19630:0:99999:7:::
/etc/group:存放了所有组的信息
	用户名:密码:组号gid:附属组用户(不显示主组用户),用户间用逗号隔开
	ubuntu:x:1002:user1,user2,user3
/etc/gshadow:存放了组密钥
/etc/skel:skeleton骨骼,存储用户家目录默认文件;创建新用户的时候会将skel中的文件复制到用户家目录
```

## 用户操作指令

#### 组管理:

- 会影响/etc/group和/etc/gshadow两个文件

```bash
groupadd -g [gid] [组名] #创建组
groupmod -n [new_name] -g [new_gid] [组名] #修改组信息
groupdel [组名] #删除组
group [组名] #查看组信息
```

#### 用户管理

- 会影响/etc/passwd和/etc/shadow两个文件(同时还会默认创建同名的组影响两个文件),同时创建/home/[用户名]的目录

  - 常见的login-shell:

  ```shell
  /usr/sbin/nologin #不允许登录
  /bin/bash #bash
  /usr/bin/zsh #zsh
  ```

```shell
useradd (参数) [用户名]
	-u [UID] #自定义UID
	-g [主组] #设置主组,默认生成与用户同名的主组
	-G 组名1,组名2,组名3 #设置附加组
	-c [注释] #设置用户注释
	-d [home路径] #设置home路径,默认/home/[用户名]
	-s [login Shell] #登录的脚本shell
	-L #上锁用户禁止登录
	-U #解锁用户可以登录
usermod (参数) [用户名]#参数同useradd
userdel [用户名] #删除用户(同时删除同名主组),建议先注释用户信息而不是使用userdel
id [用户名] #查看用户信息
```

#### 用户登录记录

```shell
whoami #查看自己是哪个用户
who #查看已经登陆的用户
	pts/0:远程终端,ssh
	tty1:虚拟终端
w #查看用户登录信息,以及负载信息
last #显示近期登录的终端记录
lastlog #显示用户的登录记录
```

#### 密码

```bash
passwd [用户名] #交互式设置该用户密码
echo '新密码' | chpasswd [用户名] #非交互式设置密码,从标准输入读入密码
```

- 查看当前用户密码
- 密码存储位置
- 生成密码pwgen

#### sudo

- 用处:
  - 作为超级管理员,赋予特定普通用户相应的特权文件/目录的权限
  - 帮助root更好地(细粒度的)管理普通用户的权限
  - 安全性:每次执行sudo都会留下记录,便于审计和追踪

```
su
sudo VS 权限?	
sudo记录?
```

- 配置sudo细则:编辑/etc/sudoers文件
- 

# Linux文件权限

## 权限概念

#### 什么是权限

- Linux是多用户,多任务的,权限控制用户是否有资格**读取,修改,执行**这些文件
  - **读取**:cat,more,tail,vim只读
  - **修改**:vim的wq,echo追加,cat重定向,**修改文件属性**,**mv改名字**
  - **执行**:可以运行文件中写的可执行语句,bash的脚本文件或者python的脚本文件

#### 为什么要权限

- 对文件和进程的权限加上一定限制:保护服务器数据,文件,**进程**等
- 防止别人误操作
- 防止黑客的恶意攻击

#### 权限类型

|         | 权限针对文件                                      | 权限针对目录                                           |
| ------- | ------------------------------------------------- | ------------------------------------------------------ |
| read    | 表示可以查看文件内容:cat,vim只读                  | 表示可以ls查看目录中存在的文件名称和权限               |
| write   | 表示可以更改文件中的内容:vim 的wq                 | 表示可以**删除**目录中的子文件或者**新建**文件和子目录 |
| execute | 表示是否可以执行文件,一般指二进制文件或者脚本文件 | **表示可以进入目录中(cd)**                             |

- 文件和文件夹对比:
  - 向文件夹中**新建或者删除文件**需要文件夹权限,读取修改执行**已存在文件**需要文件权限
  - 文件夹的执行是进入文件夹

#### 文件权限修改

- 查看文件权限:
  - 查看文件夹的权限:ls-l只能列出文件夹下面的目录的权限,查看文件夹需要加上参数-d
  - 一个文件只能属于一个组(一个用户可以有多个组的权限)
  - root不受任何权限的限制

```shell
命令:ll -h或者ls -l(文件夹需要加上参数-d)
-表示文件;d表示文件夹
user/group/others相应的权限:read/write/execute
文件的硬链接数量
属主:文件的user归属(默认为创建文件的user)
属组:文件的group归属(默认为创建文件的user的主组)
- rw- r-- r-- 1 kali kali 16 Oct  5 11:51 message.txt
d rwx r-x r-x 2 kali kali 4096 Sep 30 19:40 Documents
```

- **chmod和ls -l所需的权限**	
  - 只有**文件所有者(user)和root**才有资格chmod更改文件的权限,与文件本身的rwx无关
  - ls -l一个文件需要的是**上一级目录(文件夹)的r和x权限**,与文件自身的权限无关
- chmod命令详解

```shell
chmod 755 文件 #使用数字来代表权限
chmod u+r 文件 #ugo+rwx
chmod a+rw 文件 #a代表ugo
chmod 755 $PWD #修改当前目录权限
chmod -R 755 $PWD #递归修改
```



# Linux安装包管理



## 安装package

ubuntu:使用apt安装包,使用dpkg-L检查包

#### 配置ubuntu软件源

#### apt安装失败解决



centos:使用yum安装包

## 查看端口状态

```bash
ss -tulnp #ss stands for "Socket Statistics"
//netstat -tuln(不推荐)
```

- ss和netstat有什么区别?:
  - ss相对比较新,输出格式更容易阅读
  - ss效率更高,功能更多
  - ss跨平台通用
- tuln??
  - 不输入-t和-u将会默认输出所有协议的信息(很多无效信息)

```bash
-t:表示TCP,使用该选项命令将会显示TCP协议相关的信息
-u:表示UDP,使用该选项命令将会显示UDP协议相关的信息
-l:表示监听(listen),使用该选项命令将只会显示正在监听(打开)的端口信息
-n:表示数字(number),使用该选项命令将会显示端口数字(443)而不是服务名称(https)
-p:表示进程(process)显示进程名称
```



# Linux发行版

## Kali-Linux

#### penel设置

#### 配置代理

- 安装Clash-For-Windows
- 配置系统代理:/home/kali/.zshrc&&/root/.zshrc&&/etc/skel/.zshrc

- Start-with-Linux&&Silent-Start

#### 基础配置

- 配置root密码
- 安装软件源

```list
deb http://deb.debian.org/debian/ buster main contrib non-free
deb-src http://deb.debian.org/debian/ buster main contrib non-free

deb http://security.debian.org/debian-security buster/updates main contrib non-free
deb-src http://security.debian.org/debian-security buster/updates main contrib non-free

deb http://deb.debian.org/debian/ buster-updates main contrib non-free
deb-src http://deb.debian.org/debian/ buster-updates main contrib non-free
```

- 安装输入法:

```
apt-get install fcitx fcitx-googlepinyin
重启之后在新出现的键盘图标中添加输入法google-pinyin

```

- 安装语言

```bash
dpkg-reconfigure locales #空格键选中,TAB切换到OK,Enter确定
```

- 禁止自动锁屏:power-manager

#### 磁盘压缩vmware

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  bookworm stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

# 系统配置

## 时间配置

- 查询时间和时区

```shell
date #Sat Sep 30 23:40:58 CST 2023 显示日期时间和时区
```

- 配置时区:**timedatectl**

```shell
timedatectl -h: 查看帮助
timedatectl list-timezones: 显示出所有时区的名称
timedatectl set-timezone [ZONE]: 设置系统时区
```

- 设置自动同步时间:

```shell
timedatectl set-ntp true: 设置从ntp服务器自动同步时间
```

- 配置主机名

```shell
hostnamectl: 查看主机名
hostnamectl set-hostname: 设置主机名
```

## 设置定时任务(crontab):

```shell
crontab [-u username] {-l | -r | -e}
-u:选择用户指定日程,默认指定当前用户
-l:列出当前的日程表
-r:删除用户所有日程
-e:编辑日程表(最常用)
```

- cron任务时间表达式:
```cron
minute hour date(of month) month day-of-week command
0-59   0-23 1-31           1-12  0-6(星期天为0)
* * * * * /usr/bin/backup: 每分钟执行一次/usr/bin/backup
0 * * * * ...: 每小时的第0分钟执行
23 45 * * * ...: 每天的23:45执行
0 0 * * 0 ...: 每周日的0点0分执行(周六23:59的下一分钟)

* 取值范围内的所有数字
，散列数字
/ 每过多少个数字:6-12/3(6,9,12),*/2(是否能整除2)
- 从X到Z(包括X也包括Z)

3,15 * * * * myCommand: 每小时的第3分钟和第15分钟执行
3,15 8-11 * * * myCommand: 每天的8-11点(8,9,10,11)的第3分钟和第15分钟执行
3,15 8-11 */2 * * myCommand: 每隔两天的8点到11点的第3和第15分钟执行
```

- 原理:

  - 每分钟检测所有日程的时间表达式,查看是否匹配,如果匹配就执行相应的任务
  - crontab不会记录任务的创建日期(何时编辑的任务文件)



















# 管道符号

## |

## $()

ls-a显示所有文件信息包括隐藏文件

## Vim详细操作

查找

删除

0.0.0.0/8?

0.0.0.0/24?

ip地址的正则表达式?



# Linux基本命令

# Linux进程管理

# Linux网络管理

