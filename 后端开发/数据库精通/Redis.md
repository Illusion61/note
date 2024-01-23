# Redis基础

## NoSQL

- 没有约束,自由度更高
- 非关联:数据之间没有关联,需要自己实现(自己写JOIN).重复性高

- 没有SQL语言统一,语法不统一很多样
- 不满足ACID
- 扩展性好(方便水平分布式扩展)

## 安装

1. 下载tar.gz包到/usr/local/src下
2. 解压后make,如果失败就安装依赖项&清除原来make的东西重新make
3. `make install`

## 修改配置文件redis.conf

- 重要的配置

1. bind 127.0.0.1 -::1:监听的地址
2. `daemonize yes`:后台运行服务器
3. `requirepass 6d2ySW2py`
4. port 6379:启用的端口
5. `database 16`:有16个数据库(16个独立的命名空间)
6. `maxmemory 128mb`
7. save持久化的规则

- 不重要的配置

1. maxclients:最大的客户端连接数量
2. `timeout 0`:客户端超时时间(0永不超时)
3. `logfile "redis.log"`:日志文件路径(开启后看不见命令行图标)
5. appendonly:每个命令追加到日志文件中

## 连接

```bash
redis-cli -h [ip地址] -p [端口号]
ping(应该返回pong)
auth [密码]
help @generic
```







# Redis常用命令

帮助:查看[redis官网](https://redis.io)

## Generic类型

1. help: help查看帮助,help @generic查看某种类别的帮助,help MSET查看某个命令用法
2. keys [pattern]:查看所有符合模板的key,不建议在生产环境中使用?
3. del [key1] [key2]...:删除指定的key
4. exists [key]:判断指定的key是否存在
5. **expire [key] [seconds]:设置有效期**
6. ttl [key]:查看key的有效期(如果是-2说明key不存在,是-1代表key永久存在没有有效期)
7. **FLUSHALL**清空所有数据

## String类型

```redis
SET key value [EX seconds|PX milliseconds|KEEPTTL] [NX|XX]:添加记录
GET [key]:获得记录
MSET [key1] [value] [key2] [value]...:添加多个记录
MGET [key1] [key2] [key3]...:获取多个记录的值

INCR [key]:key自增1
INCRBY [key] [number]:让key增加number
DECR [key]:key自减1
```

## Hash类型

- String类型将对象序列化为json存储,需要修改对象的**某个字段**的时候很不方便(**需要整体重新修改**)

  ```json
  user:info:305
  {
      name:"Jack",
      age:21,
      password:"1acba1inf2condscz"
  }
  ```

- Hash结构将对象中的每个字段独立存储,可以针对**单个字段**做CRUD
  ```json
  user:info:305
	->name-->"Jack"
  	->age-->21
  	->password-->"1acba1inf2condscz"
  ```

```redis
HSET [key] [field] [value]:添加或者修改某个key的field的值
HGET [key] [field]:获得某个key的field的值
HMSET [key] [field1] [value1] [field2] [value2]...:批量添加多个hash类型的key的field的值
HMGET [key] [field1] [field2]...:批量获得hash类型的key的多个field的值

HKEYS [key]:获得key所有field
HVALS [key]:获取一个key的所有field的value
HINCRBY [key] [field] [incr]:让一个key的field自增并指定步长
```

## List类型

List类型用来存储有序数据,本质是一个双向队列

```redis
LPUSH [key] [element]:向左侧插入元素
LPOP [key]:弹出左侧元素并返回该元素值
RPUSH [key] [element]:向右侧插入元素
RPOP [key]:弹出右侧元素并返回该元素值

LRANGE [key] [start] [end]:返回一段区间内的元素值(按照顺序,下标从0开始)
BLPOP [key] [element]:弹出并返回左侧元素,如果没有元素不等待直接返回nil
BRPOP [key] [element]:弹出并返回右侧元素,如果没有元素不等待直接返回nil
```

## Set类型

使用HashMap存储,无序且元素不可重复,支持**并集,交集,差集**等功能

```redis
SADD [key] [member1] [member2]...:向set中添加一个或者多个元素
SREM [key1] [member1] [member2]...:移除set中的指定元素
SISMEMBER [key] [member]:判断元素是否在集合中
SRANDMEMBER [key] [number]:从集合中随机选取number个元素
SCARD key:返回集合中的元素个数
SMEMBERS key:返回集合中的所有元素
```

```redis
SINTER [key1] [key2]:求key1和key2的交集
SDIFF [key1] [key2]:求key1和key2的差集
SUNION [key1] [key2]:求key1和key2的并集
```

## SortedSet类型

# 小提示

#### Mysql数据库中的内容怎么转移到redis中去?

- 使用nodejs读取mysql的内容,存储为变量
- 使用redis命令将变量存储到redis中去
- 单独运行脚本

# 分布式缓存

## 单点Redis问题

- 数据丢失问题:服务重启可能会丢失数据--->**实现Redis数据持久化**
- 故障恢复问题:需要一种恢复数据的手段--->利用**Redis哨兵**,实现健康检测和自动恢复

- 并发能力问题:单节点无法承担双11之类的高并发场景(网速+连接数)--->**搭建主从集群**
- 存储能力问题:单个节点的内存不够-->**动态扩容**

## Redis数据持久化

#### RDB持久化

- RDB全称`Redis Database Backup`(Redis数据备份),相当于把**内存中的所有数据都记录到磁盘中**,当Redis实例故障重启后,从磁盘读取快照文件,恢复数据.
- 快照文件称为RDB文件(.rdb),默认是保存在**当前运行目录**

```redis
save #由Redis主进程来执行RDB,会阻塞所有命令
bgsave #后台异步执行(子进程执行),避免主进程受到影响
```

- 默认redis停机的时候会存储数据(.rdb)在当前运行目录

- redis数据持久化配置

```conf
save 900 1 #900秒内至少1次修改数据,就执行bgsave
rdbcompression yes #是否压缩数据(会消耗CPU)
dbfilename dump.rdb #RDB文件名称
dir ./ #文件保存的路径目录
```

