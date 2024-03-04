# SHELL基础

## SHELL准备

- shell: 交互式(底层是C语言)
- windows中的.bat批处理文件/Linux中的.sh脚本: 非交互式,一次性执行完毕

#### shell可以用来做什么?

- 自动化配置网站节点: 自动(或者一行代码)配置网站,申请网站证书,扩建负载均衡节点
- docker/k8s进行容器自动配置
- 定期维护网站
- 自动备份数据库数据并导出

#### Shebang

- Shebang指的是出现在文本文件第一行前两个字符:`#!`:用来**指定执行脚本的解释器**

```shell
#! /bin/bash: 调用/bin/bash作为代码解释器
#! /usr/bin/python: 代表使用python进行执行
#! /usr/bin/env: 一种在不同平台上都能找到正确解释器的方法(对平台兼容)
```

- 注意事项:
  - Shebang**替换用户默认的解释器$SHELL**
  - 如果运行时显式地指明解释器(如/bin/zsh ./test.sh),会**替换掉Shebang**
  - Shebang必须使用**绝对路径**,不能使用相对路径或者环境变量
  - 提示`bad interpreter`,说明Shebang出错

#### 注释,规范,功能

- 脚本注释:`#`
- 开发规范(代码中要提及以下内容)
  - 脚本开发日期
  - 脚本功能,脚本部分划分
- 功能:SHELL脚本方便地进行文本处理,便于处理日志,注释,网页文件等内容

#### bash配置

- history:查看输入命令的历史

```shell
history -c: 清空内存中的历史记录缓存
history -r: 恢复内存中的历史记录缓存
$HISTSIZE: 存储history在内存中存储的条目数量(最新的xx条)
!880: 执行第880条命令
!!:执行上一条命令
```

- 文件:

```txt
~/.bash_history:存储bash历史记录
~/.bashrc:存储bash配置文件
```

- bash支持的多命令执行(同一行内):

```bash
ls;cd /var/www/nodejs;ls
```

## SHELL变量

#### 变量定义

- 注意:临时存储,区分大小写

```shell
name="Harry-${PATH}-Harry" #定义/修改变量
echo $name # $变量名
echo ${name}# ${变量名}
```
#### 环境变量

- 作用域:
  - 本地变量:**针对当前的shell进程**,**更换SHELL(创建新的shell或者切换shell)变量会丢失**
  - 环境变量(全局):**针对当前shell以及其任意子进程**
  - 局部变量:只在shell函数体内生效


- 环境变量:
  - 使用export导出的变量(在当前进程和各个子进程都生效),退出终端后丢失
  - 在环境变量配置文件中定义的变量: 后加载的环境变量优先级较高
  ![2024-01-29-151515](C:\homecity\Note\others\markdown\2024-01-29-151515.png)
#### 变量命令
- 有关变量的命令:

```shell
set: 输出所有变量,包括全局变量和局部变量
env: 只显示全局变量
export: 
unset [变量名]: 删除变量
readonly 变量名=变量值: 定义只读变量
name='${PATH}':单引号不能识别特殊语法(输出${PATH})
name="${PATH}":双引号可以识别特殊语法(输出PATH具体的值)
name=`ls`:反引号可以先让内部执行命令,再将命令的结果作为值返回
```

- 调用shell脚本的不同方式
  - 使用/bin/bash调用脚本(默认):相当于**新建一个子shell**然后在**子shell中执行脚本**(本地变量作用域是分离的)
  - 使用**source**或者 **.** 调用脚本:相当于在**本地shell环境**下直接执行脚本(变量作用域和当前shell是同一个)

#### 特殊变量

```shell
特殊状态变量
$?: 上一次执行状态,成功为0,失败为1-255的状态码(正确OR失败)
$$: 当前shell脚本的PID
$!: 上一次后台进程的PID
$_: 获取上次命令的最后一个参数

特殊参数变量:
$0: 此程序名称
$1 $2 ... $9 ${10}: 第n个位置变量(参数)
$@: 参数的列表(每个参数作为独立的个体)
$*: 参数的列表(所有参数作为一个整体)
$#:参数的个数

当前信息部分:
$USER:当前用户的用户名
$HOME:当前用户的主目录路径
$PWD:当前工作目录的路径
```

- 修改脚本返回值

```shell
#! /bin/bash
[ $# -ne 2 ] && {
	echo "must be two args"
	exit 119
}
echo "没毛病,就是2个参数"
```

#### 变量子串的用法

```shell
${变量}:返回变量值
${#变量}:返回变量的长度
${变量:start}:返回变量offset(从0开始)之后的字符
${变量:start:length}:提取offset之后的length限制的字符

${变量#word}:从开头删除最小匹配word
${变量##word}:从开头删除最大匹配word
${变量%word}:从结尾删除最小匹配word
${变量%%word}:从结尾删除最大匹配word
${变量/pattern/string}:替换第一个pattern为string
${变量//pattern/string}:替换所有pattern为string
```

## 条件判断



## for循环



## 函数



## 补充:Linux常用命令

- 文本输出:

```shell
echo -n不换行 -e解析特殊符号
printf 格式化输出
eval 执行多个命令
exec source执行命令,之后自动退出当前shell
```



- 文本处理

```shell
A | B :管道符号,A命令的结果将会作为B命令的输入
	set | grep uname
grep "pattern" folder/*.txt: grep在file中查询有符合"pattern"的行
	-i:忽略大小写 -n:显示行号 -v:反向搜索(搜索不符合条件的行)
wc file.txt : 统计文本中的行数,字数和字节数
	-l:只显示行数
	wc -l file1.txt file2.txt file3.txt: 统计多个文件的总行数
awk
```

- 后台运行:nohup &缺一不可