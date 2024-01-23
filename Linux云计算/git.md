# Git

## Git概念

- Git拥有**版本控制系统**,追踪所有对文件的改变

  - 防止文件被错误修改
  - 备份防止文件丢失
  - 协同开发:只能一个同时修改,如果两个人在一个文件上添加代码,就比较麻烦
  - 追溯责任:追溯问题代码发布的时间和编写人

- **仓库:整个项目的历史记录**

- 分布式:每个开发者的电脑都存储项目的整个历史记录

  - 集中式缺点:中央服务器一旦挂机,所有开发者均无法获取代码

- 分布式结构:

  - 远程代码仓库:方便所有开发者之间传输(github)

  - 本地代码仓库(每个开发者电脑上都有):方便代码备份

- 交互动作:

  - push:从本地上传仓库到远程代码仓库
  - pull:从远程代码仓库获取更新,更新本地代码仓库
  - clone:复制远程代码仓库中完整的仓库到本地

## Git命令

### 工作区-->暂存区-->本地仓库-->远程仓库

#### 初始化版本配置

```shell
git -v : 显示git版本
git config --global user.name "illusion"
git config --global user.email "jiang_sicheng@126.com"
git config --global init.defaultBranch main #修改默认分支名称
git config --global core.quotepath false #解决中文乱码(options->Text->Locale选择zh_CN/UTF-8)
```

#### 创建仓库

```shell
git init : [在当前目录]初始化新建一个git仓库(可以看到新建隐藏的git文件)
git init 仓库名 : 在当前目录创建一个名为仓库名的子目录用来存储仓库
git clone [仓库地址] : 
```

#### 添加和提交文件

- 文件状态:	

  - untracked:未被跟踪(新建的文件,还没有被git管理起来)
  - unmodified:未修改(被git管理起来)
  - modified:已修改(修改之后还没有添加进入暂存区)
  - staged:已暂存(被git暂存)

- 不同区域:

  - **工作区**(.git所在目录):**生产产品**(代码)
  - **暂存区**(.git/index):**运输车辆**(暂存代码,**等待统一提交运输**)
  - **本地仓库**(.git/objects):工厂里面的**仓库**(**历史版本**)

```shell
git status:显示当前工作区状态
git add [文件名正则表达式]:将文件添加进入暂存区
git add [目录] : 将目录下所有文件添加进入暂存区(git add ./)
git commit -m [提交信息名称] : 提交暂存区中的所有文件
git commit -am [提交信息名称] : -a将文件添加进暂存区,并且提交进入仓库
git log [目录]:查看所有提交(commit)记录
git log --oneline
```

#### 回退版本

- 回退版本场景:
  - 连续提交多个版本,想要合并为一个版本,使用soft/mixed
  - 提交版本有问题,放弃这个提交版本所有修改内容

```shell
git reset [版本号] : 本地仓库回退到指定版本状态
git reset --soft [版本号]: 保存工作区域和暂存区内容
git reset --mixed [版本号] (默认) : 保存工作区,丢弃暂存区内容
git reset --hard [版本号] : 丢弃工作区和暂存区内容
git ls-files : 显示处于跟踪状态的文件列表(暂存区+工作区)
```

- 撤销操作`git reflog`之后再`git reset [版本号]`
- 已跟踪文件切换回仓库最新版本: `git checkout .`
- 删除未跟踪文件:`git clean -f -d`

#### 查看差异

`git diff [版本1] [版本2] [文件名] `

```shell
git diff: 默认比较工作区与暂存区之间的差异
git diff HEAD: 比较工作区与本地仓库之间的差异
git diff --cached: 比较暂存区与版本库之间的差异
git diff [之前版本] [之后版本]: 比较之后版本相比之前版本改变了哪些内容
git diff HEAD^|HEAD~ HEAD:比较上一个版本相比这个版本多出了什么内容
git diff HEAD~3 HEAD:比较之前的第三个版本相比这个版本多出了什么内容
git diff HEAD~3 HEAD file3.txt:比较file3.txt上,之前的第三个版本相比这个版本多出了什么内容
```

#### 删除文件

- 手动删除文件提交:
  - 工作区删除文件: rm [删除文件名]
  - 暂存区中删除文件:git add [删除文件名]
  - 提交更改: git commit
- 使用`git rm`:同时删除工作区和暂存区中的文件(之后再提交)

```shell
git rm [文件名]:同时删除工作区和暂存区
git rm -rf [文件夹名]:递归删除文件夹下面所有子目录
git rm --cached [文件名]:从暂存区中删除文件,保留工作区
删除之后不要忘记提交
```

## .gitignore忽略文件

#### 哪些文件应该被ignore?()

- 系统或者软件自动生成的文件(可以由代码再生成)
- 编译产生的中间文件(.lib)或者结果文件(.exe)
- 运行时生成的日志文件,缓存文件,临时文件(频繁更改且与项目代码无关)
- 涉及身份,密码,口令,密钥等敏感信息的文件(泄露隐私)

#### .gitignore实例

- **代表中间目录
- *代表任意字符任意次
- !代表取反
- /代表当前目录
- 目录名/ 代表目录

```gitignore
* 忽略所有文件和目录
!*/ 但是保留所有目录(为了防止无法获取到目录中的.md文件)
!/**/*.md 不忽略md文件
!/**/*.png 不忽略png文件
build/ 忽略任何目录下build文件夹下面内容
```

#### 注意事项

- 已经在暂存区中的文件(被追踪的文件)无法被ignore
- 去github的gitignore仓库下载示例.gitignore文件https://github.com/github/gitignore
- .gitignore后面的行具有更高优先级

## 分支

#### 为什么需要分支

- 在**团队协作**的时候,多个开发人员可以在**自己的分支**上进行开发,最后再合并
- 在问题修复或者**开发新功能**的时候,使用分支可以保证**main项目稳定运行**

#### 分支要点

- 分支名:i 指定分支的第i次提交
- 切换分支之后工作区也会发生变化

#### 分支操作

```shell
git branch: 查看所有分支
git branch [分支名]: 创建新的分支
git branch -D [分支名]: 删除指定分支
git switch [分支名]: 切换到指定的分支
(main分支下) git merge dev: 将dev合并到main分支中
```

- 解决分支冲突(难点)

```shell
git merge --abort: 退出merging状态(完全撤销merge操作)
修改冲突文件, git add . , git commit
```



# GitHub

## SSH

- 在仓库中查看仓库的SSH地址
- 在设置中添加电脑的SSH公钥(不要添加私钥)
- 配置默认登录使用的密钥(~/.ssh/confiig)

```
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/git
```

## 管理仓库

- 关联本地和远程仓库的两种方式
  - 先创建远程仓库再git clone
  - 在本地创建仓库然后git pull 远程仓库

#### 关联本地仓库和远程仓库

```shell
git remote add [远程仓库别名] [远程仓库地址]: 添加远程仓库
git remote -v:查看本地仓库
git push -u [远程仓库名] [本地分支名]: 将本地仓库分支内容更新到remote(-u记忆仓库和分支,之后只需要git push即可)
git pull [远程仓库名] [远程分支名]:[本地分支名](相同即可省略冒号后面部分): 拉取远程仓库内容
```

之后关联本地和远程仓库的两种方式

- 先创建远程仓库再git clone到本地,之后再git remote add,,,
- 在本地创建仓库,然后git remote add...,最后git pull 远程仓库

#### git pull

- `git fetch`:只会更新本地的远程仓库的分支引用,不会执行merge操作
- `git pull = git fetch+git merge`:会更新远程仓库的本地引用并且执行merge操作
  - `git pull`失败之后,建议单独使用git fetch和git merge寻找原因

![2024-01-22 221930](..\others\markdown\2024-01-22 221930.png)

```BUG
BUG1:
$ git merge origin/main
fatal: refusing to merge unrelated histories
原因: 如图所示,由于有不相关的commit存在,无法进行merge(没有公共祖先)
解决方法: 使用git merge --allow-unrelated-histories origin/main

BUG2:
$ git push -u origin main
To github.com:Illusion61/note.git
 ! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'github.com:Illusion61/note.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. If you want to integrate the remote changes,
hint: use 'git pull' before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

