## 认识hbase

### 什么是hbase

>HBASE是一个数据库----可以提供数据的实时随机读写
>
>HBASE与mysql、oralce、db2、sqlserver等关系型数据库不同，它是一个NoSQL数据库（非关系型数据库）
>
>l  Hbase的表模型与关系型数据库的表模型不同：
>
>l  Hbase的表没有固定的字段定义；
>
>l  Hbase的表中每行存储的都是一些key-value对
>
>l  Hbase的表中有列族的划分，用户可以指定将哪些kv插入哪个列族
>
>l  Hbase的表在物理存储上，是按照列族来分割的，不同列族的数据一定存储在不同的文件中
>
>l  Hbase的表中的每一行都固定有一个行键，而且每一行的行键在表中不能重复
>
>l  Hbase中的数据，包含行键，包含key，包含value，都是byte[ ]类型，hbase不负责为用户维护数据类型
>
>l  HBASE对事务的支持很差
>
>HBASE相比于其他nosql数据库(mongodb、redis、cassendra、hazelcast)的特点：
>
>Hbase的表数据存储在HDFS文件系统中
>
>从而，hbase具备如下特性：存储容量可以线性扩展；数据存储的安全性可靠性极高！

### 安装habase

[hbase]([https://blog.csdn.net/u010416101/article/details/89182941?utm_medium=distribute.pc_relevant.none-task-blog-baidujs-6](https://blog.csdn.net/u010416101/article/details/89182941?utm_medium=distribute.pc_relevant.none-task-blog-baidujs-6))

```shell
##先启动zookeeper 128 上
/usr/local/zookeeper/bin/zkServer.sh start
/usr/local/zookeeper/bin/zkServer.sh restart
/usr/local/zookeeper/bin/zkServer.sh stop
/usr/local/zookeeper/bin/zkServer.sh status
##启动hadoop 128 129 130 在128启动
http://hdp-01:50070
##集群启动 ,需要3台,配置文件了3台
/usr/local/hadoop-2.8.1/sbin/start-dfs.sh 
##集群关闭
/usr/local/hadoop-2.8.1/sbin/stop-dfs.sh
##准备两台服务器
192.168.73.134  
192.168.73.135  
##==上传
rz xx
## 解压
tar -zxvf hbase-1.2.1-bin.tar.gz
## 修改配置 hbase-env.sh
export JAVA_HOME=/usr/local/java/jdk/jdk1.8.0_144
export HBASE_MANAGES_ZK=false
##修改 regionservers
hdp-01
hdp-02
hdp-03
##
scp -r /usr/local/bigdata/hbase-1.2.1  192.168.73.135:/usr/local/bigdata/hbase-1.2.1
## 配置免密登录
scp /root/.ssh/authorized_keys root@192.168.73.135:~/.ssh
##启动
cd /usr/local/bigdata/hbase-1.2.1
bin/start-hbase.sh
## http://192.168.73.134:16010/master-status
##启动完后，还可以在集群中找任意一台机器启动一个备用的master
bin/hbase-daemon.sh start master
新启的这个master会处于backup状态
##4. 启动hbase的命令行客户端
bin/hbase shell
list     // 查看表
status   // 查看集群状态
version  // 查看集群版本
```

**habase配置** hbase-site.xml

```xml
<configuration>
		<!-- 指定hbase在HDFS上存储的路径 -->
        <property>
                <name>hbase.rootdir</name>
                <value>hdfs://192.168.73.128:9000/hbase</value>
        </property>
		<!-- 指定hbase是分布式的 -->
        <property>
                <name>hbase.cluster.distributed</name>
                <value>true</value>
        </property>
		<!-- 指定zk的地址，多个用“,”分割 -->
        <property>
                <name>hbase.zookeeper.quorum</name>
                <value>192.168.73.128:2181</value>
        </property>
	</configuration>


<property>
<name>hbase.master.port</name>
<value>16000</value>
</property>

<property>
<name>hbase.master.info.port</name>
<value>16010</value>
</property>

<property>
<name>hbase.regionserver.port</name>
<value>16201</value>
</property>

<property>
<name>hbase.regionserver.info.port</name>
<value>16301</value>
</property>
```

### hbase命令

**特性**

```
插入到hbase中去的数据，hbase会自动排序存储：
排序规则：  首先看行键，然后看列族名，然后看列（key）名； 按字典顺序

Hbase的这个特性跟查询效率有极大的关系
比如：一张用来存储用户信息的表，有名字，户籍，年龄，职业....等信息
然后，在业务系统中经常需要：
查询某个省的所有用户
经常需要查询某个省的指定姓的所有用户

思路：如果能将相同省的用户在hbase的存储文件中连续存储，并且能将相同省中相同姓的用户连续存储，那么，上述两个查询需求的效率就会提高！！！

做法：将查询条件拼到rowkey内

```

**命令**

```mysql
## hbase
create 't_user_info1','base_info','extra_info'
## 插入语句，id为1分别插入
put 't_user_info1','001','base_info:username','zhangsan'
put 't_user_info1','001','base_info:age','18'
put 't_user_info1','001','extra_info:career','it'
## id为2 分别插入
put 't_user_info','002','extra_info:career','actoress'
put 't_user_info','002','base_info:username','liuyifei'
## 查询方式一
scan 't_user_info1'
scan 'user_info'
## 查询方式二 get 单行数据
 get 't_user_info','001'
## 删除一个key value
delete 't_user_info','001','base_info:sex'
## 删除整行数据：
deleteall 't_user_info','001' 
get 't_user_info','001'
## 删除整个表
disable 't_user_info1'
drop 't_user_info'
list
### java api 
put 'user_info','001','base_info:username','zhangsan'
```

### hbase原理

#### 棘手的bug

```
要修改zookeeper 的ip为hostname才可以
```

[hbase原理]([https://blog.csdn.net/oschina_41140683/article/details/82752115](https://blog.csdn.net/oschina_41140683/article/details/82752115))

### java操作API

```
.//版本一定要对应
https://blog.csdn.net/az9996/article/details/88914892
```

[hbase版本对应](https://blog.csdn.net/az9996/article/details/88914892)

[远程操作]([https://blog.csdn.net/qq_31987649/article/details/85342295](https://blog.csdn.net/qq_31987649/article/details/85342295))

[java aip 操作失败，修改host](https://blog.csdn.net/ty497122758/article/details/75010726/)

[habse和hadoop版本对应]([https://blog.csdn.net/u014236541/article/details/78714669](https://blog.csdn.net/u014236541/article/details/78714669))

```
NETWORKING=yes
HOSTNAME=hbase-master
```

**hbase报错**

