```
University of Electronic Science and Technology of China
实习:内推
简历:建议
C++语言学到什么程度比较合适:需不需要学深入的框架,是否需要背诵常见的函数
科研经历:思路,填不了华为?
规划:申请时间,不同学校看重什么,明确的时间节点
读研读博状态:导师怎么找,读研升级PHD?

交换建议,导师?学生?
升级?
马克思?

语言学到什么程度,学深入框架,底层原理(存储机制,编译)?背诵常见的语法和函数?
语言要学很多吗?
机试准备?面试准备?哪个更好过?
字节跳动?华为?字节跳动内推?
开源项目?

简历投递?学校和直接官网投递区别?
校招和社招?
怎么快速提升职位?转型业务?
有没有时间谈恋爱找女朋友?
直属上级(p)
```



为什么想参加伯克利项目:

```
broaden-my-horizon:
Experience the education style of American universities and immerse yourself in American life, while also having the opportunity to learn about cutting-edge technologies.
enlarge-my-social-circle:

enlarge-my-social-circle:have the opportunity to know many excellent students in UC Berkeley

Improve my English language skills, especially in speaking and listening.
```

打算选择哪些课程:

- foundamentalcoursesofCS:操作系统,data-structure
- DS-and-Math-cant-meet-pre-requirement

问题:

```
课程负担重吗,怎么合理安排时间:curious about the workload in UC Berkeley,how much free time will a student have in a week in avarage?How to arange time better?
如果我在英语上遇到某些困难,应该怎么办?:meet difficulty in English?
```

```
没有没有,还有很多需要学习
我也差不多,最近学英语和准备实习面试
那Alex你是一般什么时候在线呢?按照
有时间就说几句嘛
```

申请F-1签证

写知识产权论文

问成电讲坛

问清除学分兑换



## 产品

#### 功能

- 

#### 优势

- 使用Redis缓存进行加速,使得数据存储在内存中
- 使用哈希表,支持O(1)时间的查询
- api限制用户访问频率,避免机器爬虫
- 注册之后使用邮箱验证,防止僵尸用户批量注册
- 完善的监控机制和容错,有错就发送给邮箱及时提醒
- Mysql常用的键使用索引(B+树),加快查询速度(O(N)-->O(logN))
- 负载均衡
- 容错和恢复:
  - 数据,网页代码本地有备份
  - 备份:使用crontab每隔1小时执行一次备份文件,将数据库中的信息导出成文件进行备份
- 安全性:
  - HTTPS加密数据传输
  - 用户密码在数据库中哈希加密存储,数据库泄露之后也不会泄露用户密码
  - 云端配置防火墙,只允许访问特定端口,SSH只允许特定IP登录并且禁用密码登录只能密钥登陆

#### 结合理论

- OS:Linux Ubuntu,Linux命令配置,vim
- 数据库:使用Mysql和Redis
- 软件工程:分析需求(分析竞争对手),测试注册登录的时候考虑边界条件,使用错误监控机制(发邮件)进行维护
- 计算机网络:
  - 使用cookie,分析HTTP头根据IP屏蔽频繁访问的用户,然后根据用户登录情况屏蔽频繁访问的用户
  - 负载均衡
  - HTTPS和HTTP不同版本(HTTP2)
- 编译原理:使用正则表达式检测用户名密码和邮箱格式是否合法,使用正则表达式匹配路由
- 计算机体系结构:内存速度比外存快,用Redis替代Mysql
- AI:LSTM

## 软件工程

 本报告总结了我们团队设计与实现的________项目。报告详细介绍了针对复杂工程问题的方案设计与实现、系统测试、知识技能学习情况以及分工协作与交流情况。我们的分布式文件系统通过对现有技术和理论的研究，提出了创新性的设计方案，通过推理分析验证了设计的可行性，并通过实际实现和测试证明了系统的功能和性能。通过整个项目的开发过程，团队成员提高了自己的知识技能，加深了对分布式系统的理解，并在分工协作与交流中发挥了积极的作用。

## 方案设计

#### 前端

- F12,分析pixiv登录界面的样式,写出登陆页面和注册页面
- 使用vue写出网站主界面

#### 后端

- 购买服务器,购买域名
- 申请免费HTTPS证书
- 安装nginx,编写nginx网站配置文件
- 安装Mysql数据库和Redis数据库,搭建好Mysql表
- 找到音乐和图片数据文件
- 安装node.js
- 编写node.js后端脚本文件
- 运行并维护node.js



- 听新生欢迎课程
- 上传照片



## 资源

- BGA student handbook

## 选课

#### 硬性要求

- waitlist只反映伯克利本校学生,不包括交换生
- berkeley times是本校学生运营
- **12 学分**,最好多于15学分
- 申请15门课程,bmail检查
- **查看是否满员**
- ![image-20240107153452804](C:\Users\nightmare00\AppData\Roaming\Typora\typora-user-images\image-20240107153452804.png)

- Open Seats:一定要有Open Seats
- 评分:**Letter Graded**,deadline confirm check bmail
- **additional information**:
  - BGA student:**I am a concurrent enrollment student in the BGA program**
  - prerequisites:taken similar course
  - not tooooo long
- department chairs decide
- graduate student == TA

![image-20240107155406523](C:\Users\nightmare00\AppData\Roaming\Typora\typora-user-images\image-20240107155406523.png)

#### 申请成功要点

- pending:talking to assitants,impact grade(showing up出勤)
- **withdraw if not want(2月9日之前是drop deadline)**
- deadline(different from matriculated students):
  - **important deadlines:**extension.berkeley.edu/static/studentservices/concurrent
- more units(no:
  - decal:taught by graduate students
  - 

```
CS61A:
	MATH 1A(微积分):
	充足的编程经验(nodejs编写过网站后端,熟悉C/C++)
CS61B:
	数据结构基础(算法和事件复杂度)
	COMPSCI 61A:JAVA语言程序设计(类和对象),编程基础(C语言)
	ENGIN 7:计算机导论
CS61C:
	COMPSCI 61A:
	COMPSCI 61B:数据结构基础(算法和事件复杂度),JAVA语言程序设计(类和对象)
	希望底层机制,cuda
```

**Berkeley**:数学,英语,历史

- Connect数学,英语,历史

Seminar