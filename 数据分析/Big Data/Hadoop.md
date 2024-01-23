# Hadoop简介

## Hadoop简介

#### Hadoop功能

Hadoop是一个开源的技术框架;提供解决分布式存储,计算,资源调度的解决方案

- 可以部署在1台乃至成千上万台服务器节点上协同工作,完成**海量数据的存储,索引和计算**

- 单台服务器能力有上限,无法存储计算数十亿网页的数据

#### Hadoop组成

- 分布式数据存储:HDFS(Hadoop-Distributed -File-System)
- 分布式数据计算:MapReduce
- 分布式资源调度:YARN

#### Hadoop集群

- HDFS集群:namenode,secondarynamenode,datanode
- YARN集群:resourcemanager,nodemanager

## Hadoop单机安装

#### 服务器基础环境配置

- 修改主机名

- 修改/etc/hosts

  ```
  192.168.88.12 node1
  192.168.88.15 node2
  192.168.88.16 node3
  ```

- 配置ssh免密登录

- 设置默认使用bash而不是zsh:`chsh -s /bin/bash`

- 关闭防火墙

- 集群时间同步

- 安装java

- 安装hadoop

#### 安装jdk和hadoop

1. 去[Hadoop官网](https://hadoop.apache.org/releases.html)下载binary包(.tar.gz),上传到服务器并解压,移动到/usr/local目录

2. 去[Oracle官网](https://www.oracle.com/)下载JDK安装包,上传到服务器并解压,移动到/usr/local目录

3. 配置环境变量(/etc/profile)

   ```shell
   ;配置jdk环境变量
   export JAVA_HOME=/usr/local/jdk-20.0.1
   export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib
   export PATH=$PATH:$JAVA_HOME/bin
   ;配置hadoop环境变量
   export HADOOP_HOME=/usr/local/hadoop-3.3.6
   export PATH=$PATH:$HADOOP_HOME/bin
   ```

4. 格式化hdfs文件系统

   ```bash
   hdfs namenode -format
   ```

## Hadoop环境部署

https://hadoop.apache.org/docs/r3.3.6/

#### 配置文件(./etc/hadoop)

- 配置hadoop**环境变量**:hadoop-env.sh
- 配置各个模块:
  - **核心**模块配置:core-site.xml
  - **HDFS**模块配置:hdfs-site.xml
- **MapReduce**模块配置:mapred-site.xml
  - **YARN**模块配置:yarn-site.xml
  
- 配置**从节点(DataNode)**:workers

#### hadoop-env.sh文件

```shell
export JAVA_HOME=/usr/local/jdk-20.0.1 #jdk安装目录
export HADOOP_HOME=/usr/local/hadoop-3.3.6 #Hadoop安装位置
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop #hadoop配置文件目录位置
export HADOOP_LOG_DIR=$HADOOP_HOME/logs #hadoop运行日志目录位置

export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
export HADOOP_SSH_OPTS="-p 53269" #自定义ssh登录端口
```

#### core-site.xml文件

```xml
<configuration>
    <property>
        <!--配置HDFS文件系统的通讯路径-->
        <name>fs.defaultFS</name>
        <!--协议:hdfs,namenode为node1,端口为8020-->
        <!--表明datanode将会与node1:8020进行通讯,即指定node1为namenode(node1必须启动NameNode进程)-->
        <value>hdfs://node1:8020</value>
    </property>
    <property>
        <!--配置io操作缓冲区大小为131072bit(不是byte)-->
        <name>io.file.buffer.size</name>
        <value>131072</value>
    </property>
</configuration>
```

#### hdfs-site.xml文件

- NameNode配置:
  - NameNode元数据位置:dfs.namenode.name.dir
  - NameNode连接白名单:dfs.namenode.hosts
  - NameNode并发线程数:dfs.namenode.handler.count
- DataNode配置:
  - DataNode数据存储位置:dfs.datanode.data.dir
  - DataNode创建文件权限:dfs.datanode.data.dir.perm
  - DataNode创建的副本数量:dfs.replication
  - DataNode文件块大小(bytes):dfs.blocksize

```xml
<configuration>
    <!--配置NameNode元数据位置-->
    <property>
        <name>dfs.namenode.name.dir</name>
        <!--在node1节点的/data/nn目录下-->
        <value>/usr/local/hadoop-3.3.6/data/nn</value>
    </property>
    <!--配置NameNode允许哪几个节点的连接(白名单)-->
    <property>
        <name>dfs.namenode.hosts</name>
        <value>node1,node2,node3</value>
    </property>
    <!--配置namenode处理的并发线程数-->
    <property>
        <name>dfs.namenode.handler.count</name>
        <value>3</value>
    </property>
    
    <!--配置DataNode文件位置-->
    <property>
    	<name>dfs.datanode.data.dir</name>
        <value>/usr/local/hadoop-3.3.6/data/dn</value>
	</property>
    <!--配置默认创建的文件权限(700,rwx------)-->
    <property>
        <name>dfs.datanode.data.dir.perm</name>
        <value>700</value>
    </property>
    <!--指定hdfs保存数据的副本数量-->
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <!--配置hdfs默认块大小(单位:bytes)-->
    <property>
        <name>dfs.blocksize</name>
        <value>134217728</value>
    </property>
</configuration>
```

#### mapred-site.xml

```xaml
<configuration>
    <!-- MapReduce Framework specific configuration properties -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

    <!-- Configuration for MapReduce JobHistory Server -->
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>node1:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>node1:19888</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.intermediate-done-dir</name>
        <value>/usr/local/hadoop-3.3.6/mr-history/tmp</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.done-dir</name>
        <value>/usr/local/hadoop-3.3.6/mr-history/done</value>
    </property>
	
    
    <property>
      <name>yarn.app.mapreduce.am.env</name>
      <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>

    <property>
      <name>mapreduce.map.env</name>
      <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>

    <property>
      <name>mapreduce.reduce.env</name>
      <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
    
    <!-- Configuration for MapReduce Shuffle -->
    <property>
        <name>mapreduce.shuffle.port</name>
        <value>13562</value>
    </property>
    <property>
        <name>mapreduce.shuffle.ssl.enabled</name>
        <value>false</value>
    </property>

    <!-- Configuration for MapReduce Task Logs -->
    <property>
        <name>mapreduce.task.userlog.limit.kb</name>
        <value>8192</value>
    </property>
</configuration>
```

#### yarn-site.xml

```xml
<configuration>
    <!-- Configuration for ResourceManager -->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>node1</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>node1:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>node1:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>node1:8088</value>
    </property>

    <!-- Configuration for NodeManager -->
    <property>
        <name>yarn.nodemanager.local-dirs</name>
        <value>/usr/local/hadoop-3.3.6/yarn/local-dirs</value>
    </property>
    <property>
        <name>yarn.nodemanager.log-dirs</name>
        <value>/usr/local/hadoop-3.3.6/yarn/log-dirs</value>
    </property>
    <property>
        <name>yarn.scheduler.minimum-allocation-mb</name>
        <value>512</value>
     </property>
    <property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>2048</value>
     </property>
    <property>
        <name>yarn.scheduler.maximum-allocation-vcores</name>
        <value>1</value>
     </property>
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>2048</value>
     </property>
     <property>
            <name>yarn.nodemanager.resource.cpu-vcores</name>
            <value>1</value>
     </property>

    <!-- Configuration for Resource Localization -->
    <property>
        <name>yarn.nodemanager.remote-app-log-dir</name>
        <value>/usr/local/hadoop-3.3.6/yarn/app-logs</value>
    </property>
    <property>
        <name>yarn.nodemanager.remote-app-log-dir-suffix</name>
        <value>logs</value>
    </property>

    <!-- Configuration for Application History Server -->
    <property>
        <name>yarn.timeline-service.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>yarn.timeline-service.address</name>
        <value>node1:10200</value>
    </property>
    <property>
        <name>yarn.timeline-service.webapp.address</name>
        <value>node1:8188</value>
    </property>
    
    <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>

<!-- 是否将对容器实施物理内存限制 -->
<property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
</property>

<!-- 是否将对容器实施虚拟内存限制。 -->
<property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
</property>

<!-- 开启日志聚集 -->
<property>
  <name>yarn.log-aggregation-enable</name>
  <value>true</value>
</property>
    <!-- 设置yarn历史服务器地址 -->
<property>
    <name>yarn.log.server.url</name>
    <value>http://node1:19888/jobhistory/logs</value>
</property>

<!-- 历史日志保存的时间 7天 -->
<property>
  <name>yarn.log-aggregation.retain-seconds</name>
  <value>604800</value>
    </property>
</configuration>
```

```
mkdir yarn
cd yarn
mkdir local-dirs
mkdir log-dirs
mkdir app-logs
cd ../
mkdir mr-history
cd mr-history
mkdir tmp
mkdir done
scp -r /usr/local/hadoop-3.3.6/etc/hadoop root@node2:/usr/local/hadoop-3.3.6/etc
scp -r /usr/local/hadoop-3.3.6/etc/hadoop root@node3:/usr/local/hadoop-3.3.6/etc
```

#### workers文件

```
node1 #Datanode1
node2 #Datanode2
node3 #Datanode3,在hosts文件中有配置node3对应的ip
```


#### 分发hadoop文件夹

1. 使用scp将配置好的hadoop文件夹分发到其他机器中的相同目录下
2. 在其他机器上面配置好JDK等基础依赖项
3. 配置好环境变量,创建好相应的文件夹
4. 开启相应端口的防火墙

#### 启动&关闭hadoop

启动操作和关闭操作只需要在namenode上面运行就可以管理整个集群,会自动远程启动datanode节点上面的datanode进程,不需要在每个datanode上面单独运行start-dfs.sh

```bash
$HADOOP_HOME/sbin/start-all.sh #启动hadoop集群(dfs和yarn)
$HADOOP_HOME/sbin/stop-all.sh #关闭hadoop集群(dfs和yarn)
```

查看正在运行的hadoop进程:

```shell
jps #查看java进程
```

#### 启动失败原因:

- 必须先格式化`hdfs namenode -format`hdfs系统才能正常启动
- 安装jdk1.8(java8)版本,才能正常启动resourcemanager和nodemanager

## 网页端界面

- hdfs:`namenode:9870`
- yarn:`resourcemanager:8088`



# HDFS分布式存储系统

## 2.1 分布式存储简介

#### 为什么需要分布式存储

- 文件体积太大,单个服务器无法存放:使用多台服务器进行存储
- 单个服务器**传输速率有上限**:三台服务器就有**3倍的传输效率(带宽)和写入/读取效率**,以及**3倍CPU内存等各方面的综合性能**
- 多个服务器提升**容错能力**,一个服务器坏了还有其他服务器提供**备份**

#### 分布式的基础架构

去中心化模式:	如区块链和P2P

中心化模式:一个Master节点其他都是Slave节点,**每个节点都是一个独立的进程**

- 主节点(**NameNode**):**存储元数据**,负责管理整个HDFS文件系统,管理DataNode
  - 文件自身的属性信息:文件名称,权限,修改时间,文件大小,数据块大小
  - 文件块位置映射信息:文件块位于哪个DataNode节点上
  - NameSpace:统一的抽象目录树(文件目录和路径树)
- 从节点(**DataNode**):主要负责数据的存储,即**存入数据和取出数据**
- 辅助角色(SecondaryNameNode):帮助NameNode完成元数据的管理工作

## 2.2 HDFS特点

#### 文件分块存储

- **提高存储利用率**:例如三台服务器都只剩99G,有一个100G的文件,将它分块之后就能更好地**平均分配**到三台服务器上
- **提高并行度**:文件只有分块才能存储在多台服务器上,从而**同时被处理**
- **不支持编辑文件**:只能上传/下载/读取,不能再HDFS上面实时编辑文件

#### 分成大小相等的块

- 提高容错度:块大小相等便于namenode平均分配到服务器上,便于管理,增强冗余度和容错性
- 数据并行度:块的大小相同使得块在各个节点分配平衡,便于进行并行处理

#### 副本机制

- 数据备份:便于在其中一些服务器坏掉之后进行数据恢复

## 2.3 HDFS应用场景

#### 适合

- 大文件
- 高容错
- 数据流式访问(一次写入多次读取)
- 低成本部署,廉价PC

#### 不适合

- 小文件
- 数据交互式访问,频繁修改
- 低延迟处理

## 2.4 HDFS操作

#### 文件url: 协议+路径

- `file:///`:本地文件系统

- `hdfs://node1:8020`:hdfs文件系统

- 默认系统:未指定协议的时候,使用fs.defaultFs

#### 命令行参数

`hadoop fs [options]`

- `-mkdir [-p] <path>`:创建文件夹
  
  - -p:会自动创建父目录
- `-ls [-h] [-R] [<path>]`:查看指定目录下的内容,
  - -h:人性化显示文件size
  - -R:递归查看指定目录及其子目录
- `-put [-f] [-p] <localsrc> <dst>`:上传文件到hdfs
  - localsrc: file:///home/ubuntu/...(不能使用相对路径,因为默认文件系统是hdfs)
  - -f: 覆盖目标文件
  - -p: 保留访问时间和修改时间,所有权和权限
- `-get [-f] [-p] <src> <localsrc>`:
  
  - localsrc:本地文件系统路径
- `-cat <src>`:查看hdfs文件内容

- `-cp [-f] <src> <dst>`:拷贝HDFS文件

- `-appendToFile <localsrc> <dst>`:追加文件写入文件中(合并文件)

  - 将所有给定的文件追加到dst文件中,换行然后追加到末尾
  - dst文件如果不存在,将创建该文件
  - 如果localsrc为空,则输入从标准输入中读取

  