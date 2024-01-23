#  运维中的监控

## 运维监控简介

#### 为什么需要监控

- 定位错误:网站故障无法访问的时候及时诊断是**哪项服务出现问题**(eg:数据库服务)
- 性能评估:在某项资源占用非常大的时候,就要尝试优化或者扩充服务器(eg:CPU占用率一达到60%就报警)
  - 浏览服务器端的**CPU**,**内存**,**带宽**占用情况
  - 查看服务响应延迟
  - 分析网站性能瓶颈(在CPU还是内存还是带宽)
- 安全审核:查看日志信息规避潜在问题,防止别人入侵(爬虫,hack等)
- 必要的时候发送**警报**

#### 监控的种类

- 硬件监控:监控硬件温度,硬盘故障等
- 系统监控:CPU,内存,硬盘占用,硬盘IO,系统负载,进程数
- 服务监控:apache2,nginx,mysql,redis,TCP连接数
- 性能监控:网站性能,服务器性能,数据库性能,存储性能
- 日志监控:访问日志,错误日志,运行日志等
- 安全监控:用户登录数,攻击次数统计,passwd文件变化,所有文件改动
- 网络监控:端口,带宽流量,网络流入流出实时流量

#### 运维岗位分类

- 基础运维:负责维护硬件修机器,网络设备管理
- 监控运维:监控服务器性能状态,及时解决问题
- 应用运维:维护各种服务的搭建,写文档等(系统管理员)
- 运维开发:负责一些运维平台的研发,使得运维更加自动化

## 选择监控软件

- 优秀软件特点:
  - **自定义监控内容**,自己通过脚本采集数据
  - 数据需要存入**数据库**,日后对数据进行分析和计算
  - 监控系统可以**简易,快速**地部署到服务器
  - 数据**可视化**直观清晰
- 异常报警通知:
  - 可以定义复杂度告警逻辑

# Zabbix安装

## 配置软件源

- 为什么要配置软件源:kali-linux当中默认的软件源是kali的里面很多包都没有,安装的时候会提示has- no -installation -candidate

- 在/etc/apt/sources.list当中添加debian官方软件源

```
deb http://deb.debian.org/debian/ buster main contrib non-free
deb-src http://deb.debian.org/debian/ buster main contrib non-free

deb http://security.debian.org/debian-security buster/updates main contrib non-free
deb-src http://security.debian.org/debian-security buster/updates main contrib non-free

deb http://deb.debian.org/debian/ buster-updates main contrib non-free
deb-src http://deb.debian.org/debian/ buster-updates main contrib non-free
```

- 更新软件源

```
apt update
```

## 安装zabbix前置

- 在zabbix官网的操作手册中查看安装zabbix所需前置
  - mysql
  - apache2
  - php指定范围内版本
  - apache2的php模块
  - 配置php时区

```bash
apt install mysql-server #安装mysql
systemctl restart mysql #启动mysql
apt install apache2 #安装apache2
apt-get install php php-pear php-cgi php-common libapache2-mod-php #安装php
php -v #检查php版本(需要符合zabbix要求)
a2enconf php8.2-cgi #apache2允许php运行
systemctl restart apache2 #重启apache2
#配置zabbix时区
vim /etc/php/8.2/apache2/php.ini
	date.timezone = "Asia/Shanghai"
```

## 使用zabbix

- 参考官网选择Server,Frontend,Agent进行安装

- 默认登录**用户名:Admin,密码:zabbix**
- 打开防火墙:
  - 80/443:HTTP(S)服务
  - 客户端:10050,服务端10051之间相互通信
  - mysql数据库:3306
  - SNMP Trap报警机制:打开162号端口而且是UDP用于接收SNMP-Trap消息

# Zabbix客户端

zabbix-agent2:被部署在被监控目标上面,用来主动监控本地相应资源

- zabbix客户端:监控采集本地数据发送到服务端
- zabbix服务端:接收zabbix客户端的数据并展示在网页上面

## Zabbix客户端安装

#### 同步时间

- 时间正确:监控一定要确保时间正确
  - zabbix依赖于不同主机之间的时间一致性来记录和分析数据,如果时间不一致会**影响数据的可靠性**
  - 触发器和报警:时间一致才能够**准时触发**(报警)

- 安装ntpdate来同步时间

```bash
apt install ntpdate
ntpdate -u ntp.aliyun.com #同步时间
```

- 查看时间和时区

```bash
date #Sat Sep 30 23:40:58 CST 2023 显示日期时间和时区
```

- 同步时区

```
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

#### 安装Agent2

- 参考Zabbix官网选择Agent2进行安装
  - 配置Zabbix源
  - 安装zabbix-agent2
  - 启动zabbix-agent2服务(开机自启)

```
systemctl restart zabbix-agent2
systemctl enable zabbix-agent2
```

- 打开10050端口的防火墙(zabbix客户端默认监听10050端口接收来自zabbix服务端通信)

#### 修改Agent2配置文件

```conf
Server=192.168.88.12
ServerActive=192.168.88.12
Hostname=主机名(hostnamectl set-hostname node01)
```

```
systemctl restart zabbix-agent2
```

#### 验证Agent2的连通性

```bash
apt install zabbix-get
zabbix_get -s 'node1' -p 10050 -k 'system.hostname'
zabbix_get -s 'node1' -p 10050 -k 'agent.ping'
```

# Zabbix使用

## 添加监控项

```shell
vim /etc/zabbix/zabbix_agent2.d/UserParameter.conf #创建用户参数配置文件
	UserParameter=login.user,who|wc -l #添加自定义参数和Linux命令
systemctl restart zabbix-agent2#重启zabbix-agent2
zabbix_get -s 'node1' -p 10050 -k 'login.user' #检测参数是否可以使用
```

## 自定义模板

#### 创建Template

#### 添加监控项item

- 参考官网的doc文档:[官网地址](https://www.zabbix.com/documentation/current/en/manual/config/items/itemtypes/zabbix_agent#net.if.totalifmode)添加合适的配置项
- 设置配置项多久更新一次(1s),同时在user-profile里面也设置更新率
- 设置合适的pre-processing(change-per-second)

#### 添加trigger

- 设置监控项在达到什么条件的前提下触发什么级别的警报
- trigger的检查频率和item一致

#### 添加Graph







## 发送电子邮件

#### 电子邮件协议

- SMTP:是用于**发送邮件**的标准协议,它定义了邮件客户端到邮件服务器的方式,并通过TCP连接将邮件传递到目标服务器
- POP3:是用于**接收邮件**的标准协议,允许客户端从邮件服务器上下载文件
- IMAP:是用于**接收邮件**的标准协议,比POP3更加高级提供更丰富的功能(多端同步等)

授权码:使用授权码可以某个用户身份发送邮件,权限等级比账户密码低

#### 设置发送提醒

- /user-settings/media:设置联络媒介(发送目标)
- /Administration/Media-Types:设置自己的media和**media-template**
- /Configuration/Actions:enable相应的actions





ssh登录后linux主机名?????/etc/hosts主机名?????

```
systemctl stop zabbix-server zabbix-agent apache2 
systemctl disable zabbix-server zabbix-agent apache2 
```

zabbix原理????

cobbler,saltstack,kubernetes,zabbix,elastic-search(日志分析),Jenkins