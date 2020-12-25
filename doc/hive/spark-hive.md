##spark-hive

###hive地址

```java
  /*本地测试*/
      /*return SparkSession.builder()
                .appName("HiveOnSpark2")
                .master("local[*]")
                .enableHiveSupport()
                .getOrCreate();*/
        SparkSession sparkSession = SparkSession.builder().appName("unfraud-lac").enableHiveSupport().getOrCreate();
        sparkSession.conf().set("spark.sql.parquet.binaryAsString", "true");
        sparkSession.conf().set("spark.hadoop.hive.metastore.warehouse.dir", "hdfs://hadoop:8020/apps/hive/warehouse");
        return sparkSession;
```

### hiveSQL处理

```
   Dataset<Row> data = sparkSession.sql(querySQL);
```

###hive-rdd方式

```java
{    
logger.info(PropUtils.TASK_DAY + "进入推送数据到电信准备:==== ");
        try {
            String querySQL = getQuerySQL(PropUtils.CTCC_QUERY_SQL);
            if (!Strings.isNullOrEmpty(querySQL)) {
                logger.info("电信sql " + querySQL);
                Integer firstPushValue = IsEnum.Yes.getValue();
                Dataset<Row> originalData = sparkSession.sql(querySQL);
                logger.info("推送电信RDD");
                JavaRDD<Row> rowJavaRDD = originalData.javaRDD();
                logger.info("电信查询数据数量"+rowJavaRDD.count());
                rowJavaRDD.foreach(row -> {
                    String c_usernum = row.getString(0);
                    String c_relatenum = row.getString(1);
                    String c_time = row.getString(2);
                    String c_content = row.getString(3);
                    String c_type = row.getString(4);
                    String c_keywords = row.getString(5);
                    String c_hour = row.getString(6);
                    CtccSms ctccSms = new CtccSms(c_usernum, c_relatenum, c_time, c_content, c_type, c_keywords, firstPushValue,c_hour);
                    String json = null;
                    try {
                        json = JsonUtils.serialize(ctccSms);
                        if (json != null && json != "") {
                            //2.==写入到队列
                            broadcast.getValue().sendData("ctccsmsqueue", json);
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                        logger.info("电信写消息错误 " + ctccSms.toString());
                    }
                });
                logger.info("==推送数据到电信结束:==== ");
            }
        } catch (Exception e) {
            logger.error(e.getMessage());
            e.printStackTrace();
        }
    }
```

### 写入到mysql方式

```java
public static void insertCallForwarding(List<CallForwarding> list) {
        if (list == null || list.size() <= 0) {
            return;
        }
        String sql = "INSERT INTO `secure_model_call_fording_result` ( `callingnum`, `forward_count`,  `open_date`, `open_city_code`, `product_name`, `credit_class`, `qqnet`, `groupuser`, `grouppay`, `wideband`, `develop_staff_id`,`is_many_card`, `insert_time`) " +
                "VALUES ( ?, ?,  ?, ?, ?, ?, ?, ?, ?, ?, ?,? ,now());";
        Object[][] callFordings = new Object[list.size()][];
        for (int i = 0; i < list.size(); i++) {
            try {
                CallForwarding callFording = list.get(i);
                callFordings[i] = new String[12];
                int j=0;
                callFordings[i][j] = callFording.getCallingnum();
                callFordings[i][++j] = callFording.getForward_count();
                Date open_date = callFording.getOpen_date();
                String date=null;
                if(open_date!=null){
                     date = DateUtil.date2Str(open_date, DateUtil.YYYY_MM_DD);
                }
                callFordings[i][++j] =date;
                callFordings[i][++j] = callFording.getOpen_city_code();
                callFordings[i][++j] = callFording.getProduct_name();
                callFordings[i][++j] = callFording.getCredit_class();
                callFordings[i][++j] = callFording.getQqnet();
                callFordings[i][++j] = callFording.getGroupuser();
                callFordings[i][++j] = callFording.getGrouppay();
                //10
                callFordings[i][++j] = callFording.getWideband();

                callFordings[i][++j] = callFording.getDevelop_staff_id();
                callFordings[i][++j] = callFording.getIs_many_card();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        dbUtils.batchUpdate(sql, callFordings);
    }
```

### 写入到kafka方式

* 单例模式

  ```java
  public class DataProducer2 implements Serializable {
  
      private static Logger logger = Logger.getLogger(DataProducer1.class);
  
      private static DataProducer2 INSTANCE;
  
      private static Producer<String, String> producer;
  
      private DataProducer2() {
      }
  
  
      static {
          Properties props = new Properties();
          props.put("bootstrap.servers", "10.10.113.21:9092,10.10.113.22:9092,10.10.113.23:9092,10.10.113.24:9092,10.10.113.25:9092,10.10.113.26:9092,10.10.113.27:9092,10.10.113.29:9092,10.10.113.30:9092");
          // props.put("bootstrap.servers", PropUtils.KAFKA_LIST);
          //props.put("acks", "all");
          props.put("request.timeout.ms", 10000);
          props.put("retries", 1);
          props.put("batch.size", 16384);
          props.put("linger.ms", 1);
          props.put("buffer.memory", 33554432);
          props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
          props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
          props.setProperty("security.protocol", "SASL_PLAINTEXT");
          props.setProperty("sasl.mechanism", "SCRAM-SHA-512");
          props.setProperty("sasl.jaas.config", "org.apache.kafka.common.security.scram.ScramLoginModule required username= \"mvtech_admin\" password=\"a123456\";");
          //props.setProperty("sasl.jaas.config", "org.apache.kafka.common.security.scram.ScramLoginModule required username=\"mvtech_admin\"  password=\"a123456\"");
          logger.info("org.apache.kafka.common.security.scram.ScramLoginModule required username=\"mvtech_admin\"  password=\"a123456\"");
          producer = new KafkaProducer<>(props);
          INSTANCE = new DataProducer2();
      }
  
      public static DataProducer2 getInstance() {
          return INSTANCE;
      }
  
      public void sendData(String topic, String data) {
          logger.info("----producer = " + producer);
          logger.info("----topic = " + topic);
          logger.info("----data = " + data);
          logger.info("---producer.send start---");
          producer.send(new ProducerRecord<>(topic, data));
          //producer.flush();
          logger.info("---producer.send end---");
  
      }
  
      public static void close() {
          if(producer!=null){
              producer.close();
          }
      }
  ```

  

* 写入kafka

```java
 //广播的方式
 sparkSession = SparkUtils.getSession();
            String flagTrue = IsEnum.Yes.getDescription();
            ClassTag<DataProducer2> tag = scala.reflect.ClassTag$.MODULE$.apply(DataProducer2.class);
            Broadcast<DataProducer2> broadcast = sparkSession.sparkContext().broadcast(DataProducer2.getInstance(), tag);
            
```

### 如何映射的方式

```

```



## 打包方式

### 依赖项目外的方式

```html
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.1.1</version>
                <configuration>
                    <archive>
                        <addMavenDescriptor>false</addMavenDescriptor>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <classpathPrefix>lib/</classpathPrefix>
                            <mainClass>com.mvtech.forward.Starter</mainClass>
                        </manifest>
                    </archive>
                    <excludes>
                        <exclude>**/assembly/</exclude>
                    </excludes>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>3.0.1</version>
                <executions>
                    <execution>
                        <id>copy</id>
                        <phase>compile</phase><inherited></inherited>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>
                                ${project.build.directory}/lib
                            </outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

###同一个包的方式

```html
 <build>
         <plugins>
             <plugin>
                 <artifactId>maven-assembly-plugin</artifactId>
                 <configuration>
                     <appendAssemblyId>false</appendAssemblyId>
                     <descriptorRefs>
                         <descriptorRef>jar-with-dependencies</descriptorRef>
                     </descriptorRefs>
                     <archive>
                         <manifest>
                             <mainClass>com.mvtech.increase.Starter</mainClass>
                         </manifest>
                     </archive>
                 </configuration>
                 <executions>
                     <execution>
                         <id>make-assembly</id>
                         <phase>package</phase>
                         <goals>
                             <goal>single</goal>
                         </goals>
                     </execution>
                 </executions>
             </plugin>
             <plugin>
                 <groupId>org.apache.maven.plugins</groupId>
                 <artifactId>maven-compiler-plugin</artifactId>
                 <configuration>
                     <source>1.8</source>
                     <target>1.8</target>
                 </configuration>
             </plugin>
         </plugins>
     </build>
```

