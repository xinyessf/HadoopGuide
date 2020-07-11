#### 环境

```
netstat -ntlp 
00:50:56:28:7D:E4
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
##集群启动 ,需要3台,配置文件了3台
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

### 命令文件操作

```shell
## 列表查看
hadoop fs -ls /aaaa
##上传文件到hdfs中
hadoop fs -put /本地文件  /aaa
hadoop fs -put /demo/a.txt /wordcount/input
##下载文件到客户端本地磁盘
hadoop fs -get /hdfs中的路径   /本地磁盘目录
##在hdfs中创建文件夹
hadoop fs -mkdir  -p /wordcount/input
##移动hdfs中的文件（更名）
hadoop fs -mv /hdfs的路径1  /hdfs的另一个路径2
##复制hdfs中的文件到hdfs的另一个目录
hadoop fs -cp /hdfs路径_1  /hdfs路径_2
hadoop fs -cp /wordcount/input/a.txt  /wordcount/input/b.txt
##删除hdfs中的文件或文件夹
hadoop fs -rm -r /ccc

##查看hdfs中的文本文件内容
hadoop fs -cat /out/bb.log
hadoop fs -cat /res.dat
hadoop fs -tail -f /demo.txt
##修改文件的权限
hadoop fs -chown user:group /aaa
hadoop fs -chmod 700 /aaa
##追加内容到已存在的文件
hadoop fs -appendToFile /本地文件   /hdfs中的文件
```

### windows安装hadoop

```shell
在windows开发环境中做一些准备工作：
1、在windows的某个路径中解压一份windows版本的hadoop安装包
2、将解压出的hadoop目录配置到windows的环境变量中：HADOOP_HOME
```

### mapduce

>map阶段: 对maptask读到的一行数据如何处理
>
>reduce阶段: 对reducetask拿到的一组相同key的kv数据如何处理

**配置启动**

```

## 修改yarn-site.xml
<property>
<name>yarn.resourcemanager.hostname</name>
<value>hdp-01</value>
</property>

<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>

<property>
<name>yarn.nodemanager.resource.memory-mb</name>
<value>2048</value>
</property>

<property>
<name>yarn.nodemanager.resource.cpu-vcores</name>
<value>2</value>
</property>

```

### yarn启动

```shell
scp yarn-site.xml hdp-02:$PWD
scp yarn-site.xml hdp-03:$PWD
scp yarn-site.xml hdp-04:$PWD
### 在hadp-01 中启动和关闭
/usr/local/hadoop-2.8.1/sbin/start-yarn.sh
/usr/local/hadoop-2.8.1/sbin/stop-yarn.sh
```

### 内存不够

```shell
##
lvs pvcreate /dev/sda3
lvextend -L+1020Mib /dev/mapper/cl-root /dev/sda3
## 
vgextend cl-root /dev/sda4
##
lvextend -L+2G /dev/mapper/cl-root /dev/sda4
##
e2fsck -a /dev/mapper/cl-root

##
resize2fs /dev/mapper/cl-root
xfs_info /dev/sda4
##
xfs_growdfs /dev/sda3
##  查看整体
du -h
## 查看文件
du -ah --max-depth=1  / 
du -lh --max-depth=1
```

