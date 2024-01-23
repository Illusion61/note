# 存储引擎

## MySQL体系结构

- 连接层:

	- 客户端:连接器;服务器端:连接池
	- 用途:处理建立连接以及客户端验证
- 服务层

	- 组成:SQL接口,解析器,查询优化器,缓存
- 存储引擎:
  - **索引**,存储管理
- 系统文件/日志

## 存储引擎简介

- 创建表的时候,指定存储引擎:

```mysql
CREATE TABLE xxx(
...
)ENGINE=INNODB;

CREATE TABLE my_memory(
    id int,
    name varchar(20)
)ENGINE=MEMORY;
```

- 查看当前数据库支持的存储引擎:

```mysql
SHOW ENGINES;
```

## InnoDB引擎

- 兼顾高可靠性和高性能的存储引擎,默认存储引擎
- 特点:
  - 遵循ACID模型,支持事物
  - 提高并发访问性能
  - 保证数据的完整性和正确性
- 文件路径:

```mysql
/var/lib/mysql:  数据库名/表名.ibd(用来存储表结构,数据和索引)
```

## MyISAM

- 特点:
  - 不支持事物
  - 支持行级锁

## Memory




# 索引优化

## B-树和B+树

- B-树:多路平衡查找树,**叶子节点和根节点都存储数据**
- B+树:多路平衡查找树
  - **只有叶子节点存储数据**,内部节点只存储指针和key
  - 叶子节点构成链表
- 单页最大为16KB,一个节点对应一个页:
  - 为什么使用B+树不使用红黑树?:B+树单个节点(磁盘页)存储的key比较多,使得树比较浅(调用磁盘的次数比较少)
  - 为什么使用B+树不使用B-树?:B-树内部节点额外存储数据,使得单个节点的key和指针比较少(单个页只能16KB),降低存储效率
  - 为什么不使用哈希表:哈希表不支持范围查询

## 索引分类

- 索引定义:
  - 主键索引(PRIMARY):针对表中主键创建的索引,默认在PRIMARY KEY列自动创建且只能有一个
  - 唯一索引(UNIQUE):防止索引数据列重复,默认在UNIQUE列自动生成
  - 常规索引:快速定位特定数据
  - 全文索引(FULL TEXT):查找文本中的关键词,而不是比较索引中的值

- 索引存储形式:

  - 聚集索引(Clustered-Index):必须有而且只能有一个

    - 选取规则:
      - 如果存在主键,主键索引就是聚集索引
      - 否则选择第一个唯一索引
      - 没有主键和唯一索引就自动生成一个隐藏的聚集索引
    - 子节点存储的是真实数据

  - 二级索引(Secondary-Index):

    - 子节点存储的是聚集索引的"主键"
    - 二级索引查询过程:先用二级索引值查询对应的主键,然后再用主键在聚集索引上查询数据行位置

    ```mysql
    SELECT * FROM my_db1 WHERE name="xxx";
    #假设name上建立二级索引,id是聚集索引
    #首先根据xxx在二级索引的B+树上查找到数据对应的id,再在聚集索引上根据id查找到数据行位置
    ```

## 索引语法

- 联合索引和单列索引区别?

  - 单列索引:适用于只根据一个字段进行查询

  ```mysql
  select * from tb1 where age>=30 and age<=40;
  ```

  - 联合索引:适用于根据多个字段进行查询,找到满足多个字段条件对应的主键id,(https://bbs.huaweicloud.com/blogs/332783)

    - 构建B+树的时候先比较第一个字段,相同的情况下比较第二个字段...
    - 联合索引字段的顺序很重要,联合索引(A,B)可以只根据A查询但是不能只根据B查询
    - 创建(A,B)的联合索引之后也可以再创建A和B的单列索引

    ```mysql
    select * from tb1 where age<30 and enroll_date<2002-11-01;
    ```

```mysql
#创建索引
CREATE [UNIQUE|FULLTEXT] INDEX index_name ON table_name(index_col_name,...);
#查看索引
SHOW INDEX FROM table_name;
#删除索引
DROP INDEX index_name ON table_name;
```

