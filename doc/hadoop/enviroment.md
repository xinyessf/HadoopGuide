#### 环境

```
netstat -ntlp 
```

**windows**

```
c:/windows/system32/drivers/etc/hosts
192.168.73.128	hdp-01
192.168.73.129	hdp-02
192.168.73.130	hdp-03
192.168.73.131	hdp-04
```

**linux**

```mysql
##关闭防火墙
systemctl stop firewalld.service
**修改hosts**
vi /etc/hosts
192.168.73.128	hdp-01
192.168.73.129	hdp-02
192.168.73.130	hdp-03
192.168.73.131	hdp-04
###环境配置
export HADOOP_HOME=/usr/local/hadoop-2.8.1
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

**jdk**

```shell
/usr/local/java/jdk/jdk1.8.0_144	
##环境变量复制到其他节点
scp -r /etc/profile hdp-02:/etc/profile
scp -r /etc/profile hdp-03:/etc/profile
scp -r /etc/profile hdp-04:/etc/profile
```

**安装hadoop**

[安装](https://blog.csdn.net/lv_hulk/article/details/82949327)

```shell
##1.修改hadoop-env.sh
export JAVA_HOME=/usr/local/java/jdk/jdk1.8.0_144
##2.修改core-site.xml
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://hdp-01:9000</value>
</property>
</configuration>
##3.修改hdfs-site.xml
<configuration>
<property>
<name>dfs.namenode.name.dir</name>
<value>/root/dfs/name</value>
</property>

<property>
<name>dfs.datanode.data.dir</name>
<value>/root/dfs/data</value>
</property>

</configuration>
##4.拷贝整个hadoop安装目录到其他机器
scp -r /usr/local/hadoop-2.8.1  hdp-02:/usr/local/hadoop-2.8.1
scp -r /usr/local/hadoop-2.8.1  hdp-03:/usr/local/hadoop-2.8.1
scp -r /usr/local/hadoop-2.8.1  hdp-04:/usr/local/hadoop-2.8.1
##配置secory
scp -r /usr/local/hadoop-2.8.1/etc/hadoop/hdfs-site.xml  hdp-02:/usr/local/hadoop-2.8.1/etc/hadoop/hdfs-site.xml
scp -r /usr/local/hadoop-2.8.1/etc/hadoop/hdfs-site.xml  hdp-03:/usr/local/hadoop-2.8.1/etc/hadoop/hdfs-site.xml
scp -r /usr/local/hadoop-2.8.1/etc/hadoop/hdfs-site.xml  hdp-04:/usr/local/hadoop-2.8.1/etc/hadoop/hdfs-site.xml
###5.配置变量
export HADOOP_HOME=/usr/local/hadoop-2.8.1
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
###6.初始化namenode的元数据存储目录
hadoop namenode -format
###7.启动 启动namenode进程（在hdp-01上）
hadoop-daemon.sh start namenode
##starting hdp-01, logging to /usr/local/hadoop-2.8.1/logs/hadoop-root-hdp-01-pinyoyougou-docker.out
#####启动完后，首先用jps查看一下namenode的进程是否存在
##8.访问
http://hdp-01:50070
##集群启动
/usr/local/hadoop-2.8.1/sbin/start-dfs.sh 
##集群关闭
/usr/local/hadoop-2.8.1/sbin/stop-dfs.sh
## 查看日志

```

**如何配置SSH免密登录**

[免密登录设置](https://blog.csdn.net/qq_42493452/article/details/86214090)

```shell
##
/root/.ssh
##创建
ssh localhost
##复制 ,注意没有目录要先创建
scp /root/.ssh/authorized_keys root@hdp-02:~/.ssh
scp /root/.ssh/authorized_keys root@hdp-03:~/.ssh
scp /root/.ssh/authorized_keys root@hdp-04:~/.ssh
```

### 命令

```shell
##上传文件到hdfs中
hadoop fs -put /本地文件  /aaa
hadoop fs -put /aa.txt /aaa
##下载文件到客户端本地磁盘
hadoop fs -get /hdfs中的路径   /本地磁盘目录
##在hdfs中创建文件夹
hadoop fs -mkdir  -p /aaa/xxx
##移动hdfs中的文件（更名）
hadoop fs -mv /hdfs的路径1  /hdfs的另一个路径2
##复制hdfs中的文件到hdfs的另一个目录
hadoop fs -cp /hdfs路径_1  /hdfs路径_2
##删除hdfs中的文件或文件夹
hadoop fs -rm -r /out
##查看hdfs中的文本文件内容
hadoop fs -cat /out/bb.log
hadoop fs -tail -f /demo.txt

```

### hive

####hive安装

[hive安装](https://blog.csdn.net/qq_36508766/article/details/81318996)

```shell
#hive 配置变量
export HIVE_HOME=/usr/local/hive
export PATH=$PATH:$HIVE_HOME/bin
##刷新环境变量
source /etc/profile
##启动
hive
```

**hive集群配置**

```

```

#### 创建虚表

```
touch dual.txt
echo 'X' >dual.txt
load data local inpath '/hivedemo/dual.txt' overwrite into table dual;
select 'a' from dual;
```



#### hive常用命令

```mysql
show databases;
create database test_work; //新建一个测试库
use test_work;
craete table course(id string);
insert into table course values("qq");
exit
#清屏
!clear
# hive 执行dfs 命令
!dfs -lsr / ; hive 执行dfs命令 
# 建库
create database db_order ;
#删除库
drop database db_order ;
## 建表
use db_order;
create table t_order(id string,create_time string,amount float,uid string)
row format delimited
fields terminated by ',';
## 删除
drop table t_order; ##删除表
truncate table t_order; ##删除表数据
alter table t_order drop partition (partition_name='分区名') ##按照分区删除
##看表结构
desc t_order ;
##改表

## 外部表创建
create external table t_access(ip string,url string,access_time string)
row format delimited
fields terminated by ','
location '/access/log';
##分区表创建
create table t_access(ip string,url string,access_time string)
partitioned by(dt string)
row format delimited
fields terminated by ','
location '/access/log';
##导入数据
load data local inpath '/aa.log' into table t_access partition(dt='20170804');
##统计
select count(*) from t_access where dt='20170804';
select count(*) from t_access;
##多个分区表
create table t_partition(id int,name string,age int)
partitioned by(department string,sex string,howold int)
row format delimited fields terminated by ',';

load data local inpath '/hivedemo/person.dat' into table t_partition partition(department='xiangsheng',sex='male',howold=20);
###CAST建表
create table t_user_2 like t_user;
create table t_access_user 
as
select ip,url from t_access;
#################数据导入导出########################
方式1：导入数据的一种方式：
手动用hdfs命令，将文件放入表目录；

方式2：在hive的交互式shell中用hive命令来导入本地数据到表目录
load data local inpath '/root/order.data.2' into table t_order;

方式3：用hive命令导入hdfs中的数据文件到表目录
load data inpath '/aaa/aa.log' into table t_access partition(dt='20170806');
#################数据导入导出########################
```

#### 导入和导出

```mysql
#################数据导入导出########################
方式1：导入数据的一种方式：
手动用hdfs命令，将文件放入表目录；

方式2：在hive的交互式shell中用hive命令来导入本地数据到表目录
load data local inpath '/root/order.data.2' into table t_order;

方式3：用hive命令导入hdfs中的数据文件到表目录
load data inpath '/aaa/aa.log' into table t_access partition(dt='20170806');
#################数据导入导出########################
##############导出########################
5.3.2.	将hive表中的数据导出到指定路径的文件
1、将hive表中的数据导入HDFS的文件
insert overwrite directory '/root/access-data'
row format delimited fields terminated by ','
select * from t_access;

2、将hive表中的数据导入本地磁盘文件
insert overwrite local directory '/root/access-data'
row format delimited fields terminated by ','
select * from t_access limit 100000;
##############导出########################
create table dual(dummy String);

```

#### 常用内置函数

```shell
##类型转换函数
select cast("5" as int) from dual;
select cast("2017-08-03" as date) ;
select cast(current_timestamp as date);
##时间函数
select current_timestamp; 
select current_date;

```

#### 自定义函数

```

```

#### 案例

```
create table t_access_times(username string,month string,counts int)
row format delimited fields terminated by ',';

load data local inpath '/hivedemo/accumulate.dat' into table t_access_times;

A,2015-01,5
A,2015-01,15
B,2015-01,5
A,2015-01,8
B,2015-01,25
A,2015-01,5
C,2015-01,10

1、第一步，先求个用户的月总金额
select username,month,sum(salary) as salary from t_access_times group by username,month

+-----------+----------+---------+--+
| username  |  month   | salary  |
+-----------+----------+---------+--+
| A         | 2015-01  | 33      |
| A         | 2015-02  | 10      |
| B         | 2015-01  | 30      |
| B         | 2015-02  | 15      |
+-----------+----------+---------+--+

2、第二步，将月总金额表自己连接 自己连接
select A.*,B.* FROM
(select username,month,sum(salary) as salary from t_access_times group by username,month) A 
inner join 
(select username,month,sum(salary) as salary from t_access_times group by username,month) B
on
A.username=B.username
where B.month <= A.month
+-------------+----------+-----------+-------------+----------+-----------+--+
| a.username  | a.month  | a.salary  | b.username  | b.month  | b.salary  |
+-------------+----------+-----------+-------------+----------+-----------+--+
| A           | 2015-01  | 33        | A           | 2015-01  | 33        |
| A           | 2015-01  | 33        | A           | 2015-02  | 10        |
| A           | 2015-02  | 10        | A           | 2015-01  | 33        |
| A           | 2015-02  | 10        | A           | 2015-02  | 10        |
| B           | 2015-01  | 30        | B           | 2015-01  | 30        |
| B           | 2015-01  | 30        | B           | 2015-02  | 15        |
| B           | 2015-02  | 15        | B           | 2015-01  | 30        |
| B           | 2015-02  | 15        | B           | 2015-02  | 15        |
+-------------+----------+-----------+-------------+----------+-----------+--+


第3步：
select auname,amonth,acnts,sum(bcnts)
from t_tmp2
group by auname,amonth,acnts;
得到最终结果

当然，也可以把整个逻辑过程写成一个SQL语句：

select A.username,A.month,max(A.salary) as salary,sum(B.salary) as accumulate
from 
(select username,month,sum(salary) as salary from t_access_times group by username,month) A 
inner join 
(select username,month,sum(salary) as salary from t_access_times group by username,month) B
on
A.username=B.username
where B.month <= A.month
group by A.username,A.month
order by A.username,A.month;

```

