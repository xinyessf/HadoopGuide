### 整合hive库

####修改配置

```
1.安装MySQL（hive的元数据库）并创建一个普通用户，并且授权
	CREATE USER 'xiaoniu'@'%' IDENTIFIED BY '123568'; 
	GRANT ALL PRIVILEGES ON hivedb.* TO 'xiaoniu'@'%' IDENTIFIED BY '123568' WITH GRANT OPTION;
	FLUSH PRIVILEGES;


#在spark的conf目录下创建一个hive的配置文件
2.添加一个hive-site.xml

<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
   Licensed to the Apache Software Foundation (ASF) under one or more
   contributor license agreements.  See the NOTICE file distributed with
   this work for additional information regarding copyright ownership.
   The ASF licenses this file to You under the Apache License, Version 2.0
   (the "License"); you may not use this file except in compliance with
   the License.  You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
-->
<configuration>
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://node-6:3306/hivedb?createDatabaseIfNotExist=true</value>
    <description>JDBC connect string for a JDBC metastore</description>
  </property>

   <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
  </property>

  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>bigdata</value>
    <description>username to use against metastore database</description>
  </property>

  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>123568</value>
    <description>password to use against metastore database</description>
  </property>

</configuration>

3.上传一个mysql连接驱动（sparkSubmit也要连接MySQL，获取元数据信息）
./spark-sql --master spark://node-4:7077,node-5:7077 --driver-class-path /home/xiaoniu/mysql-connector-java-5.1.7-bin.jar

4.sparkSQL会在mysql上创建一个database，需要手动改一下DBS表中的DB_LOCATION_UIR改成hdfs的地址

5.要在/etc/profile中配置一个环节变量(让sparkSQL知道hdfs在哪里，其实就是namenode在哪里)
  exprot HADOOP_CONF_DIR

6.重新启动SparkSQL的命令行
```

#### 依赖

```
	<dependency>
	    <groupId>org.apache.spark</groupId>
	    <artifactId>spark-hive_2.11</artifactId>
	    <version>2.2.0</version>
	</dependency>
```



###整合mysql库

```

```

### 本地跑spark

[配置spark]([https://blog.csdn.net/weixin_34315485/article/details/93311143](https://blog.csdn.net/weixin_34315485/article/details/93311143))

```
//报错

```