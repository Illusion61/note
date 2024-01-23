# Spark环境搭建

## Spark安装

1. 上[Spark官网](https://spark.apache.org/)下载Spark安装包(有包含Hadoop版本和用户提供Hadoop版本)

2. Spark需要自己事先搭建好jdk(JAVA环境)

3. 配置环境变量

   ```shell
   export JAVA_HOME=/usr/local/jdk-20.0.1
   export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib
   export PATH=$PATH:$JAVA_HOME/bin
   export HADOOP_HOME=/usr/local/hadoop-3.3.6
   export PATH=$PATH:$HADOOP_HOME/bin
   export SPARK_HOME=/usr/local/spark
   export PYSPARK_PYTHON=/usr/bin/python3
   export PATH=$PATH:$SPARK_HOME/bin
   ```

4. 运行Spark的python版本:pyspark

   ```shell
   spark-submit  ./spark-demo.py
   ```

## Local模式



## StandAlone模式

StandAlone是完整的Spark运行环境,其中StandAlone意味着独立(spark独立于hadoop组件就能运行)

#### 配置文件清单($SPARK_HOME/conf)

- spark-env.sh:配置SPARK运行时候的环境变量
- spark-defaults.conf:spark默认配置
- log4j2.properties

- workers:配置workers主机

```shell
先在各个主机上配置好spark的环境变量(.bachrc,.zshrc,/etc/profile)
```

#### spark-env.sh

```shell
export JAVA_HOME=/usr/local/jdk-20.0.1
export HADOOP_CONF_DIR=/usr/local/hadoop-3.3.6/etc/hadoop
export YARN_CONF_DIR=/usr/local/hadoop-3.3.6/etc/hadoop
export SPARK_MASTER_HOST=node1
export SPARK_MASTER_PORT=7077
export SPARK_MASTER_WEBUI_PORT=8080

SPARK_WORKER_CORES=2
SPARK_WORKER_MEMORY=2g
SPARK_WORKER_PORT=7078
SPARK_WORKER_WEBUI_PORT=8081

SPARK_HISTORY_OPTS="-Dspark.history.fs.logDirectory=hdfs://node1:8020/sparklog/ 
-Dspark.history.fs.cleaner.enabled=true"
```

```shell
hadoop fs -mkdir /sparklog
hadoop fs -chmod 777 /sparklog
hadoop fs -mkdir /input
hadoop fs -chmod 777 /input
```

#### spark-defaults.conf

```conf
spark.eventLog.enabled	true
spark.eventLog.dir	hdfs://node1:8020/sparklog/
spark.eventLog.compress	true
```

#### log4j2.properties

```properties
修改19行INFO为WARN
```

#### workers

```
node1
node2
node3
```

#### 分发spark文件到其他机器上

``` bash
scp -r /usr/local/spark root@node2:/usr/local
scp -r /usr/local/spark/conf root@node2:/usr/local/spark
scp -r /usr/local/spark/conf root@node3:/usr/local/spark
```

#### 启动操作

查看WEBUI:`http://node1:8080`

$SPARK/sbin

- **启动历史服务器:start-history-server.sh**
- **启动所有服务器节点:start-all.sh**
- 启动本地master节点:start-master.sh
- 启动本地worker节点:start-worker.sh

- 启动pyspark连接master节点:`pyspark --master spark://node1:7077`(后面的url在WEBUI上查看)

- 测试代码

```python
sc.parallelize([1,2,3,4,5]).map(lambda x:x*10).collect()
```

启动spark-submit连接master节点:

```bash
spark-submit --master spark://node1:7077 --num-executors 6 $SPARK_HOME/examples/src/main/python/pi.py 1000

sc.textFile('/wordcount/input/hello.txt').flatMap(lambda line:line.split(' ')).map(lambda x:(x,1)).reduceByKey(lambda a,b:a+b).collect()
```

## spark任务运行层次划分

#### 常见的网页端界面

``` 
HDFS:			http://node:9870
HADOOP_app:		http://node:8088/cluster
PYSPARK(Application关了就没了):   http://node:4040/
SPARK_MASTER:	http://node:8080/
SPARK_HISTORY:	http://node:18080/
```

#### 层次划分

- Application(应用):pyspark等,是整个应用程序
- Job(工作):一个应用中有多个job,比如单词计数,计算pi等
- Stage(阶段):一个job中有多个stage,比如reduce,collect等
- Task(线程):一个stage中有多个task,这些task分配给多个workers进行具体执行

## 路径

- 本地路径:`file:///home/kali/Desktop/hello.txt`
- HDFS路径:`hdfs://node1:8020/wordcount/input/hello.txt`
- 网络路径:`https://sciform.one/hello.txt`

- **注意事项**:
  - 文件最好是每个主机都能访问到的**HDFS路径**,集群模式运行,本地路径会导致只有本地程序运行
  - default.fs会在没有开头协议的路径前面加上内容(eg:hdfs://node1:8020/)

# Spark编程初上手

## Spark原理

#### pyspark VS spark

|             pyspark              |             Spark             |
| :------------------------------: | :---------------------------: |
|         **不能独立运行**         |       **可以独立运行**        |
|              python              |      python,java,scala,R      |
| Driver底层JVM,Execotor底层python | Driver底层JVM,Execotor底层JVM |
|        本地开发python程序        |      生产环境集群化运行       |

#### spark集群架构

- Master进程(resourcemanager):**集群大管家**,整个集群的资源管理和分配
- Worker进程(nodemanager):**单个机器的管家**,负责单个机器上提供运行容器,管理当前机器的资源
- Driver:负责协调整个工作的进行,分配资源和安排任务(所有不是RDD计算的代码)
- Executor:在Worker提供的容器中,负责工作的具体实行(所有RDD计算的代码)
- HistoryServer进程(可选):保存日志信息到HDFS,启动HistoryServer可以查看历史信息

(driver提供容器,master管理容器,driver利用容器,executor在容器里面)

#### pyspark运行原理

- Driver上:python代码由py4j翻译成JVM指令进行执行
- Executor上:python代码

## RDD概念

#### 什么是RDD

Resilient Distributed Datasets

- Resilient(弹性):RDD的数据可以存储在内存或者硬盘中
- Distributed(分布式):RDD数据分布式存储,可用于分布式计算
- Dataset:一个数据集合,用于存放数据

抽象上RDD是一个完整的数据集合,具体而言每个分区存储一部分数据

#### RDD特性

- RDD是有分区的(逻辑上RDD是一个整体,物理上每个分区存储一部分数据)
- **计算方法**会作用到**每一个分区**之上
- RDD之间是有相互依赖关系的
- KV型RDD可以有分区器
- RDD分区数据的读取会尽量靠近数据的所在地(移动数据不如移动计算)

```python
sc.parallelize([1,2,3,4,5,6,7,8,9],3).glom().collect()
```

## RDD初始化

#### 构建Spark环境

```python
from pyspark import SparkConfig,SparkContext
conf = SparkConfig().setAppName("wordcount").setMasster('local[*]')
sc = SparkContext(conf=conf)
```

#### RDD初始化

并行化创建:将本地集合转向分布式RDD(本地转分布式)

- 本地对象构建:**parallelize(本地对象,分区数)**
- 读取文本创建:**textFile(文件路径,分区数参考)**,构建成一个列表并且文件中的每一行是列表中的一个元素

## RDD算子

#### RDD算子分类

- Transform算子:由RDD->RDD,构建执行计划
- Action算子:由RDD->非RDD,没有action transform**不会执行**

工厂中:transform算子是工厂中的**工序**,action是工厂中的**开关**

#### 常见Transform算子

map:

- 功能:将RDD中的数据一条条(根据给定函数)进行处理,返回新的RDD

- 语法:`rdd.map(func)`(输入和输出数据格式没有任何限制)

- 用例:`rdd.map(lambda x:x*10+8)`

flatMap:

- 功能:对RDD进行Map操作,然后进行**解除嵌套**操作
- 语法:`rdd.flatMap(func)`
- 用例:`sc.textFile('/wordcount/hello.txt').flatMap(lambda x:x.split(" ")`
- 解除嵌套:`[[1,2,3],[4,5,6]]->[1,2,3,4,5,6]`

`sortBy(func,ascending=[True,False],numPartitions=3)`:对rdd数据进行排序

- func: `lambda x:(x[1],x[0])` 告诉对哪几个数据项进行排序,**在前面的优先级更高?????**
- `ascending=[False,True]` x[1]降序排序,x[1]相等时x[0]降序排序
- numPartitions=6 用6个分区进行排序
- 多项排序??????

`reduceByKey`

- Key可以是元组,但是不能是列表