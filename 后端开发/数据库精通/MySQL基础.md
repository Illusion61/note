# SQL

## SQL基础

#### 通用语法

- SQL可以单行或者多行书写,以**分号结尾**
- SQL语句可以使用空格/缩进来增强语句的可读性
- SQL语句不区分大小写,建议关键字使用大写方便区分
- 注释:
  - 单行注释:--或者#
  - 多行注释:`/*注释内容*/`

#### 分类

- DDL:数据定义语言,用来定义数据库对象(数据库,表及其字段)
- DML:数据操作语言,用来增删改表中的数据
- DQL:数据查询语言,用来查询数据库中表的记录
- DCL:数据控制语言,用来操作数据库用户,控制数据库的访问权限

## DDL语言

#### mysql数据类型

- 数值类型(后面可以加上UNSIGNED)

```mysql
TINYINT			1BYTEs
SMALLINT		2BYTEs
MEDIUMINT		3BYTEs
INT(INTEGER)	4BYTEs
BIGINT			8BYTEs
FLOAT			4BYTEs
DOUBLE			8BYTEs
DECIMAL(P,D)	DECIMAL(6,2):-9999.99~9999.99
TINYINT UNSIGNED
```

- 字符串类型

```mysql
CHAR(x):定长字符串,剩下的用空格补齐-------------->存储利用率低,但是性能好
VARCHAR(x):变长字符串,根据字符串内容确定存储长度--->存储利用率高,但是性能差
```

- 日期时间类型

```mysql
DATE:日期 YYYY-MM-DD	3BYTEs
TIME:时间 HH:MM:SS	3BYTEs
DATETIME YYYY-MM-DD HH:MM:SS(1000-01-01 00:00:00至9999-12-31 23:59:59)	8BYTEs
YEAR:年份 YYYY		1BYTEs
TIMESTAMP YYYY-MM-DD HH:MM:SS(1970-01-01 0:0:01至2038-01-19 03:14:07)	4BYTEs
```

#### 约束

- 作用:作用与表上的field用于限制表中的数据,保证数据的正确性,有效性和一致性
- 分类

```mysql
NOT NULL:非空约束
UNIQUE:唯一约束
PRIMARY KEY:主键约束
AUTO INCREMENT:自动增长(代理键)
DEFAULT:默认约束(default '缺失')
CHECK 条件:检查约束(check age>0 && age<120)
FOREIGN KEY:外键约束(CONSTRAINT fk_emp_dept_id foreign key(dept_id) references dept(id))
```

- 外键删除/更新行为:当主表删除/更新记录时,检查记录是否有对应外键并采取相应措施

```mysql
NO ACTION / RESTRICT:不允许删除/更新行为
CASCADE:删除/更新外表在子表中的记录
SET NULL:设置子表中外键值为NULL
SET DEFAULT:设置子表中外键值为DEFAULT
```

#### 数据库操作

```mysql
# 列出所有数据库
SHOW DATABASES;
# 查询当前数据库是什么
SELECT DATABASE();
# 使用数据库
USE 数据库名;
```

```mysql
# 创建数据库
CREATE DATABASE [IF NOT EXISTS] 数据库名 [DEFAULT CHARSET 字符集] [COLLATE 排序规则];
# 删除数据库
DROP DATABASE [IF EXISTS] 数据库名;
```

#### 表操作

- 查询表信息

```mysql
#查看当前数据库所有的表
SHOW TABLES;
#查询表结构
DESC 表名;
#查询指定表的建表语句
SHOW CREATE TABLE 表名;
```

- 创建表

```mysql
CREATE TABLE 表名(
	字段1 字段1类型 [约束] [COMMENT 注释1],
    字段2 字段2类型 [约束] [COMMENT 注释2],
    ......
    字段n 字段n类型 [约束] [COMMENT 注释n]
    [,CONSTRAINT 外键名称 FOREIGN KEY(外键字段名) REFERENCES 主表(主表列名)]
)[COMMENT 表注释];
```

```mysql
CREATE TABLE employee(
	id int comment '编号' PRIMARY KEY,
    workno varchar(10) comment '工号' UNIQUE,
    name varchar(10) comment '姓名',
    gender char(1) comment '性别',
    age tinyint unsigned comment '年龄',
    idcard char(18) comment '身份证号',
    enrolldate date comment '入职时间'
)comment '员工信息表';
```

- 修改表

```mysql
#添加字段
ALTER TABLE 表名 ADD 字段名 类型 [COMMENT 注释] [约束];
#修改字段名称和类型
ALTER TABLE 表名 CHANGE 旧字段名 新字段名 类型 [COMMENT 注释] [约束];
#删除字段
ALTER TABLE 表名 DROP 字段名;
#重命名表
ALTER TABLE 旧表名 RENAME TO 新表名;
```

```mysql
ALTER TABLE emp ADD nickname varchar(20) comment '昵称';
ALTER TABLE emp CHANGE nickname username varchar(30) comment '用户名';
ALTER TABLE emp DROP username;
ALTER TABLE emp RENAME TO employee;
```

- 删除表

```mysql
#删除表
DROP TABLE [IF EXISTS] 表名;
#清空表的内容
TRUNCATE TABLE 表名;
```

## DML 语言

#### INSERT

- 给指定的字段添加数据

```mysql
INSERT INTO 表名(字段1,字段2,...)VALUES(值1,值2...);
```

- 给所有的字段添加数据

```mysql
INSERT INTO 表名 VALUES (值1,值2...);
```

- 批量添加数据

```mysql
INSERT INTO 表名(字段1,字段2,...)VALUES(值1,值2...),(值1,值2...),(值1,值2...);
INSERT INTO 表名 VALUES (值1,值2...),(值1,值2...),(值1,值2...);
```

- 注意点:
  - 值的顺序必须和字段顺序**一一对应**
  - **字符串**和**日期类型数据**必须要**包含在引号中**

```mysql
insert into emp(id,workno,name,gender,age,idcard,enrolldate) values(2,'2','梁惠文王','男',18,'123456789012345679','2023-11-30');
insert into emp values(1,'1','楚庄王','男',19,'123456789012345678','2023-10-31');
insert into emp values(3,'3','齐桓公','男',23,'123456789012345678','2023-10-31'),(4,'4','宋襄公','男',20,'123456789012345678','2023-10-31');
```



#### UPDATE(SET)

```mysql
UPDATE 表名 SET FIELD1=VALUE1,FIELD2=VALUE2... [WHERE 条件];

update emp SET name='晋文公',age=18 where id=1;
```

#### DELETE(FROM)

```mysql
DELETE FROM 表名 [WHERE 条件];
```

## DQL语言

```mysql
SELECT field1 [as name1],field2 [as name2]...,fieldn [as namen]
FROM 表名
WHERE 条件

```

- 注意:SELECT*不推荐使用

#### 基础查询



## DCL语言

#### 创建用户





# SQL使用技巧

## 数据库迁移

- 导出

```
mysqldump -u <用户名> -p <数据库名> > <导出的文件名>.sql
```

- 导入

```
mysql -u <用户名> -p <目标数据库名> < <导出的文件名>.sql
```

## 本地用户密码

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Password@123';
```

