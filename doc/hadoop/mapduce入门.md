### 了解mapreduce

>mapreduce程序应该是在很多机器上并行启动，而且先执行map task，当众多的maptask都处理完自己的数据后，还需要启动众多的reduce task，这个过程如果用用户自己手动调度不太现实，需要一个自动化的调度平台——hadoop中就为运行mapreduce之类的分布式运算程序开发了一个自动化调度平台——YARN
>
>map阶段: 对maptask读到的一行数据如何处理
>
>reduce阶段: 对reducetask拿到的一组相同key的kv数据如何处理



### 集群搭建

```shell
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

**copy到其它机器**

```shell
scp yarn-site.xml hdp-02:$PWD
scp yarn-site.xml hdp-03:$PWD
scp yarn-site.xml hdp-04:$PWD
### 在hadp-01 中启动和关闭
/usr/local/hadoop-2.8.1/sbin/start-yarn.sh
/usr/local/hadoop-2.8.1/sbin/stop-yarn.sh
## 查看启动
netstat -tpnl | grep java 
## 访问
http://hdp-01:8088
```



### 项目demo

**wordcount**

```shell
### 计算word count,项目
big-data-mapreduce
###  
##方式一 本地调试连接linux

##方式二 linux中 启动
hadoop jar big-data-mapreduce.jar com.huanyu.mapreduce.wordcount.linux.JobSubmitterLinuxToYarn
## 方式三 windows方式 本地调试
JobSubmitterWindowsToYarn
```

