### 了解flume

>数据采集系统
>
>3部分
>
>==1.source,channle,sink
>
>实时采集放到kafka
>
>离线采集放到hdfs

[官方网站]( http://flume.apache.org/ )

### Linux中安装

```shell
## 测试的时候 先要启动 hadoop
##集群安装
##准备两台服务器 
192.168.73.128
192.168.73.129
解压和安装
cd /usr/local/bigdata
sz  xx
tar -zxvf apache-flume-1.6.0-bin.tar.gz
## copy到其他服务器
scp -r /usr/local/bigdata/flume-1.6.0  192.168.73.129:/usr/local/bigdata/flume-1.6.0
cd /usr/local/bigdata/flume-1.6.0
## 启动 console调试 dir-hdfs.conf 为新增的当前要跑的配置文件
bin/flume-ng agent -c conf/ -f dir-hdfs.conf -n ag1 -Dflume.root.logger=INFO,console
## 生产启动
```

#### 启动报错

>原因没有hadoop中安装

[hadoop 少common jar]( https://blog.csdn.net/qq1010234991/article/details/88426564 )

```
一定要在装有hadoop中启动，否则考虑少jar如何搞定
```

### 案例

####文件采集的方式

```shell
cd /usr/local/bigdata/flume-1.6.0
## 启动 console调试 dir-hdfs.conf 为新增的当前要跑的配置文件
bin/flume-ng agent -c conf/ -f dir-hdfs.conf -n ag1 -Dflume.root.logger=INFO,console
## 生产启动
nohup bin/flume-ng agent -c conf/ -f dir-hdfs.conf -n ag1 1>/dev/null 2>&1 &
```

**配置文件**

```shell
#定义三大组件的名称
ag1.sources = source1
ag1.sinks = sink1
ag1.channels = channel1

# 配置source组件
ag1.sources.source1.type = spooldir
ag1.sources.source1.spoolDir = /log/
ag1.sources.source1.fileSuffix=.FINISHED
ag1.sources.source1.deserializer.maxLineLength=5120

# 配置sink组件
ag1.sinks.sink1.type = hdfs
ag1.sinks.sink1.hdfs.path =hdfs://192.168.73.128:9000/access_log/%y-%m-%d/%H-%M
ag1.sinks.sink1.hdfs.filePrefix = app_log
ag1.sinks.sink1.hdfs.fileSuffix = .log
ag1.sinks.sink1.hdfs.batchSize= 100
ag1.sinks.sink1.hdfs.fileType = DataStream
ag1.sinks.sink1.hdfs.writeFormat =Text

## roll：滚动切换：控制写文件的切换规则
ag1.sinks.sink1.hdfs.rollSize = 512000

## 按文件体积（字节）来切   
ag1.sinks.sink1.hdfs.rollCount = 1000000
## 按event条数切
ag1.sinks.sink1.hdfs.rollInterval = 60
## 按时间间隔切换文件

## 控制生成目录的规则
ag1.sinks.sink1.hdfs.round = true
ag1.sinks.sink1.hdfs.roundValue = 10
ag1.sinks.sink1.hdfs.roundUnit = minute

ag1.sinks.sink1.hdfs.useLocalTimeStamp = true

# channel组件配置
ag1.channels.channel1.type = memory
ag1.channels.channel1.capacity = 500000
# event条数
ag1.channels.channel1.transactionCapacity = 600
#flume事务控制所需要的缓存容量600条event

# 绑定source、channel和sink之间的连接
ag1.sources.source1.channels = channel1
ag1.sinks.sink1.channel = channel1
```

#### 文件一行一行采集动态采集

>tail -f 命令从配置文件读取

```shell
tail-hdfs.conf
用tail命令获取数据，下沉到hdfs
启动命令：
bin/flume-ng agent -c conf/ -f dir-hdfs.conf -n ag1 -Dflume.root.logger=INFO,console
bin/flume-ng agent -c conf -f  tail-hdfs.conf -n ag1 -Dflume.root.logger=INFO,console
########
while true
do
echo `date` >>app_weichat_login.log
sleep 0.1
done
```

**配置文件**

```shell

# Name the components on this agent
ag1.sources = r1
ag1.sinks = k1
ag1.channels = c1

# Describe/configure the source
ag1.sources.r1.type = exec
ag1.sources.r1.command = tail -F /root/app_weichat_login.log

# Describe the sink
ag1.sinks.sink1.type = hdfs
ag1.sinks.sink1.hdfs.path =hdfs://192.168.73.128:9000/app_weichat_login_log/%y-%m-%d/%H-%M
ag1.sinks.sink1.hdfs.filePrefix = weichat_log
ag1.sinks.sink1.hdfs.fileSuffix = .dat
ag1.sinks.sink1.hdfs.batchSize= 100
ag1.sinks.sink1.hdfs.fileType = DataStream
ag1.sinks.sink1.hdfs.writeFormat =Text

ag1.sinks.sink1.hdfs.rollSize = 100
ag1.sinks.sink1.hdfs.rollCount = 1000000
ag1.sinks.sink1.hdfs.rollInterval = 60

ag1.sinks.sink1.hdfs.round = true
ag1.sinks.sink1.hdfs.roundValue = 1
ag1.sinks.sink1.hdfs.roundUnit = minute


ag1.sinks.sink1.hdfs.useLocalTimeStamp = true



# Use a channel which buffers events in memory
ag1.channels.c1.type = memory
ag1.channels.c1.capacity = 1000
ag1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
ag1.sources.r1.channels = c1
ag1.sinks.k1.channel = c1
```

####加密串联采集

```

```

