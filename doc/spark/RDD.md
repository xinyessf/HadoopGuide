### RDD

**什么是RDD**

>

```

```

### API

#### 面向数据集的操作

>//面向数据集操作：
>    //*，带函数的非聚合：  map，flatmap     map是合并,flatmap计算
>    //1，单元素：union，cartesion  没有函数计算
>    //2，kv元素：cogroup，join   没有函数计算
>    //3，排序
>    //4，聚合计算  ： reduceByKey  有函数   combinerByKey

#####计算大于3的

```scala
val conf: SparkConf = new SparkConf().setMaster("local").setAppName("test01")
    val sc = new SparkContext(conf)
    sc.setLogLevel("ERROR")
    val dataRDD: RDD[Int] = sc.parallelize( List(1,2,3,4,5,4,3,2,1) )
//    dataRDD.map()
//    dataRDD.flatMap()
//    dataRDD.filter((x:Int)=>{ x> 3})
    val filterRDD: RDD[Int] = dataRDD.filter(   _> 3    )
    val res01: Array[Int] = filterRDD.collect()
    res01.foreach(println)
    println("-----------------------")
 
```

##### 并集,交集,差集,join连接

>   面向数据集开发  面向数据集的API  1，基础API   2，复合API
>    RDD  （HadoopRDD,MappartitionsRDD,ShuffledRDD...）
>    map,flatMap,filter
>    distinct...
>    reduceByKey:  复合  ->  combineByKey（）
>    面向数据集：  交并差  关联 笛卡尔积
>
>面向数据集： 元素 -->  单元素，K,V元素  --> 机构化、非结构化
>

```scala
    //spark很人性，面向数据集提供了不同的方法的封装，且，方法已经经过经验，常识，推算出自己的实现方式
    //人不需要干预（会有一个算子）
   //人不需要干预（会有一个算子）
   val rdd1: RDD[Int] = sc.parallelize( List( 1,2,3,4,5)  )
    val rdd2: RDD[Int] = sc.parallelize( List( 3,4,5,6,7)  )

//差集：提供了一个方法：  有方向的
    val subtract: RDD[Int] = rdd1.subtract(rdd2)
   subtract.foreach(println)
//并集
    val intersection: RDD[Int] = rdd1.intersection(rdd2)
    intersection.foreach(println)

//    //  如果数据，不需要区分每一条记录归属于那个分区。。。间接的，这样的数据不需要partitioner。。不需要shuffle
//    //因为shuffle的语义：洗牌  ---》面向每一条记录计算他的分区号
//    //如果有行为，不需要区分记录，本地IO拉去数据，那么这种直接IO一定比先Parti。。计算，shuffle落文件，最后在IO拉去速度快！！！
   val cartesian: RDD[(Int, Int)] = rdd1.cartesian(rdd2)
   cartesian.foreach(println)
   println(rdd1.partitions.size)
   println(rdd2.partitions.size)
   val unitRDD: RDD[Int] = rdd1.union(rdd2)
   println(unitRDD.partitions.size)

   unitRDD.map(sdfsdf)
   rdd1.map()
   rdd2.map()
   unitRDD.foreach(println)
 // cogroup
    val kv1: RDD[(String, Int)] = sc.parallelize(List(
      ("zhangsan", 11),
      ("zhangsan", 12),
      ("lisi", 13),
      ("wangwu", 14)
    ))
     val kv2: RDD[(String, Int)] = sc.parallelize(List(
      ("zhangsan", 21),
      ("zhangsan", 22),
      ("lisi", 23),
      ("zhaoliu", 28)
    ))

   val cogroup: RDD[(String, (Iterable[Int], Iterable[Int]))] = kv1.cogroup(kv2)
   cogroup.foreach(println)
   val join: RDD[(String, (Int, Int))] = kv1.join(kv2)
   join.foreach(println)
   val left: RDD[(String, (Int, Option[Int]))] = kv1.leftOuterJoin(kv2)

   left.foreach(println)
   val right: RDD[(String, (Option[Int], Int))] = kv1.rightOuterJoin(kv2)
   right.foreach(println)
   val full: RDD[(String, (Option[Int], Option[Int]))] = kv1.fullOuterJoin(kv2)
   full.foreach(println)

```

#### 统计sort 

#####统计uv,pv

```scala
val conf: SparkConf = new SparkConf().setMaster("local").setAppName("sort")
    val sc = new SparkContext(conf)
    sc.setLogLevel("ERROR")

    //PV,UV
    //需求：根据数据计算各网站的PV,UV，同时，只显示top5
    //解题：要按PV值，或者UV值排序，取前5名

    val file: RDD[String] = sc.textFile("file///E:\\MySummaryStudy\\big-data-soa\\big-data-spark\\data\\pvuvdata",5)

    //pv：
    //  187.144.73.116	浙江	2018-11-12	1542011090255	3079709729743411785	www.jd.com	Comment
    println("----------PV:-----------")
    val pair: RDD[(String, Int)] = file.map(line=>  (line.split("\t")(5),1)    )
    val reduce: RDD[(String, Int)] = pair.reduceByKey(_+_)
    val map: RDD[(Int, String)] = reduce.map(_.swap)
    val sorted: RDD[(Int, String)] = map.sortByKey(false)
    val res: RDD[(String, Int)] = sorted.map(_.swap)
    val pv: Array[(String, Int)] = res.take(5)
    pv.foreach(println)

    println("----------UV:-----------")

    //  187.144.73.116	浙江	2018-11-12	1542011090255	3079709729743411785	www.jd.com	Comment

    val keys: RDD[(String, String)] = file.map(
      line => {
        val strs: Array[String] = line.split("\t")
        (strs(5), strs(0))
      }
    )
    val key: RDD[(String, String)] = keys.distinct()
    val pairx: RDD[(String, Int)] = key.map(k => (k._1,1))
    val uvreduce: RDD[(String, Int)] = pairx.reduceByKey(_+_)
    val uvSorted: RDD[(String, Int)] = uvreduce.sortBy(_._2,false)
    val uv: Array[(String, Int)] = uvSorted.take(5)
    uv.foreach(println)
    while(true){
    }
  }

(www.jd.com,171.82.4.154)
```

### 分区器的使用

>如/何定义一个分区器使用
>
>按照指定的分区器去使用,按照分区规则进行分区即可

```scala
//自定义分区器
class SubjectParitioner(sbs: Array[String]) extends Partitioner {

  //相当于主构造器（new的时候回执行一次）
  //用于存放规则的一个map
  val rules = new mutable.HashMap[String, Int]()
  var i = 0
  for(sb <- sbs) {
    //rules(sb) = i
    rules.put(sb, i)
    i += 1
  }

  //返回分区的数量（下一个RDD有多少分区）
  override def numPartitions: Int = sbs.length

  //根据传入的key计算分区标号
  //key是一个元组（String， String）
  override def getPartition(key: Any): Int = {
    //获取学科名称
    val subject = key.asInstanceOf[(String, String)]._1
    //根据规则计算分区编号
    rules(subject)
  }
}
```

### 排序

####1.使用类实现排序接口

```
  def main(args: Array[String]): Unit = {

    val conf = new SparkConf().setAppName("CustomSort3").setMaster("local[*]")

    val sc = new SparkContext(conf)

    //排序规则：首先按照颜值的降序，如果颜值相等，再按照年龄的升序
    val users= Array("laoduan 30 99", "laozhao 29 9999", "laozhang 28 98", "laoyang 28 99")

    //将Driver端的数据并行化变成RDD
    val lines: RDD[String] = sc.parallelize(users)

    //切分整理数据
    val tpRDD: RDD[(String, Int, Int)] = lines.map(line => {
      val fields = line.split(" ")
      val name = fields(0)
      val age = fields(1).toInt
      val fv = fields(2).toInt
      (name, age, fv)
    })

    //排序(传入了一个排序规则，不会改变数据的格式，只会改变顺序)
    val sorted: RDD[(String, Int, Int)] = tpRDD.sortBy(tp => Man(tp._2, tp._3))

    println(sorted.collect().toBuffer)

    sc.stop()

  }

}


case class Man(age: Int, fv: Int) extends Ordered[Man] {

  override def compare(that: Man): Int = {
    if(this.fv == that.fv) {
      this.age - that.age
    } else {
      -(this.fv - that.fv)
    }
  }
}
```

#### 2.使用RDD自带属性

```
def main(args: Array[String]): Unit = {

    val conf = new SparkConf().setAppName("CustomSort5").setMaster("local[*]")

    val sc = new SparkContext(conf)

    //排序规则：首先按照颜值的降序，如果颜值相等，再按照年龄的升序
    val users= Array("laoduan 30 99", "laozhao 29 9999", "laozhang 28 98", "laoyang 28 99")

    //将Driver端的数据并行化变成RDD
    val lines: RDD[String] = sc.parallelize(users)

    //切分整理数据
    val tpRDD: RDD[(String, Int, Int)] = lines.map(line => {
      val fields = line.split(" ")
      val name = fields(0)
      val age = fields(1).toInt
      val fv = fields(2).toInt
      (name, age, fv)
    })

    //充分利用元组的比较规则，元组的比较规则：先比第一，相等再比第二个
    val sorted: RDD[(String, Int, Int)] = tpRDD.sortBy(tp => (-tp._3, tp._2))

    println(sorted.collect().toBuffer)

    sc.stop()

  }

```

### jdbc使用

####1.jdbc的案例

>如何将数据写入到mysql当中
>
>

```
object JdbcRddDemo {

  val getConn = () => {
    DriverManager.getConnection("jdbc:mysql://localhost:3306/bigdata?characterEncoding=UTF-8", "root", "123")
  }


  def main(args: Array[String]): Unit = {

    val conf = new SparkConf().setAppName("JdbcRddDemo").setMaster("local[*]")

    val sc = new SparkContext(conf)

    //创建RDD，这个RDD会记录以后从MySQL中读数据

    //new 了RDD，里面没有真正要计算的数据，而是告诉这个RDD，以后触发Action时到哪里读取数据
    val jdbcRDD: RDD[(Int, String, Int)] = new JdbcRDD(
      sc,
      getConn,
      "SELECT * FROM logs WHERE id >= ? AND id < ?",
      1,
      5,
      2, //分区数量
      rs => {
        val id = rs.getInt(1)
        val name = rs.getString(2)
        val age = rs.getInt(3)
        (id, name, age)
      }
    )
    //触发Action
    val r = jdbcRDD.collect()

    println(r.toBuffer)
    sc.stop()
  }
}
```

### rdd 写入到mysql

>1.将迭代器中的数据,遍历写入到mysql
>
>2.按照分区的规则写

```
reduced.foreachPartition(it => MyUtils.data2MySQL(it))


def data2MySQL(it: Iterator[(String, Int)]): Unit = {
    //一个迭代器代表一个分区，分区中有多条数据
    //先获得一个JDBC连接
    val conn: Connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/bigdata?characterEncoding=UTF-8", "root", "toortoor")
    //将数据通过Connection写入到数据库
    val pstm: PreparedStatement = conn.prepareStatement("INSERT INTO access_log VALUES (?, ?)")
    //将分区中的数据一条一条写入到MySQL中
    it.foreach(tp => {
      pstm.setString(1, tp._1)
      pstm.setInt(2, tp._2)
      pstm.executeUpdate()
    })
    //将分区中的数据全部写完之后，在关闭连接
    if(pstm != null) {
      pstm.close()
    }
    if (conn != null) {
      conn.close()
    }
  }
```

### DataSet

#### DataFrame注册临时表使用

```SCALA
object SQLDemo3 {

  def main(args: Array[String]): Unit = {

    var path="E:\\wordcount\\spark\\man.txt"

    //提交的这个程序可以连接到Spark集群中
    val conf = new SparkConf().setAppName("SQLDemo3").setMaster("local[*]")

    //创建SparkSQL的连接（程序执行的入口）
    val sc = new SparkContext(conf)
    //sparkContext不能创建特殊的RDD（DataFrame）
    //将SparkContext包装进而增强
    val sqlContext = new SQLContext(sc)
    //创建特殊的RDD（DataFrame），就是有schema信息的RDD

    //先有一个普通的RDD，然后在关联上schema，进而转成DataFrame

    val lines = sc.textFile(path)
    //将数据进行整理
    val rowRDD: RDD[Row] = lines.map(line => {
      val fields = line.split(",")
      val id = fields(0).toLong
      val name = fields(1)
      val age = fields(2).toInt
      val fv = fields(3).toDouble
      Row(id, name, age, fv)
    })
    //结果类型，其实就是表头，用于描述DataFrame
    val sch: StructType = StructType(List(
      StructField("id", LongType, true),
      StructField("name", StringType, true),
      StructField("age", IntegerType, true),
      StructField("fv", DoubleType, true)
    ))
    //将RowRDD关联schema
    val bdf: DataFrame = sqlContext.createDataFrame(rowRDD, sch)
    //不使用SQL的方式，就不用注册临时表了
    val df1: DataFrame = bdf.select("name", "age", "fv")
    import sqlContext.implicits._
    val df2: DataFrame = df1.orderBy($"fv" desc, $"age" asc)
    df2.show()
    sc.stop()
  }
}
```

#### dataset使用

```java
	public static boolean filter(String whiteString) throws Exception{		
		//mysql链接
		String jdbcUrl = Starter.props.getStr("mysql.jdbc.url");
		Properties connectionProperties = new Properties();
		connectionProperties.put("user", Starter.props.getStr("mysql.jdbc.username"));
		connectionProperties.put("password",Starter.props.getStr("mysql.jdbc.password"));
		connectionProperties.put("driver", Starter.props.getStr("mysql.jdbc.driver"));
		String antiInfo = Starter.props.getStr("mysql.table.t_anti_info");
		//SparkSession（固定写法）
		//un_anti_mvtech为spark任务的名称，通过这个名称可以查看任务细腻
     /*   SparkSession sparkSession = SparkSession.builder().appName("un_anti_mvtech").enableHiveSupport().getOrCreate();
        sparkSession.conf().set("spark.sql.parquet.binaryAsString", "true");
        sparkSession.conf().set("spark.hadoop.hive.metastore.uris", "thrift://h146:9083");
        sparkSession.conf().set("spark.hadoop.hive.metastore.warehouse.dir", "hdfs://maxc/apps/hive/warehouse");
        sparkSession.conf().set("spark.debug.maxToStringFields", 2000);*/
		SparkSession sparkSession = SparkSession.builder().appName("un_anti_mvtech").master("local[*]").enableHiveSupport().getOrCreate();
		//将url格式化成域名
		UDF1 getDomainInUrl = (UDF1<String, String>) url1 -> getHost(url1);
		//将getDomainInUrl注册到sparkSession里
		sparkSession.udf().register("format_url_to_domain", getDomainInUrl, DataTypes.StringType);

		//6、执行查询hive语句获得要保存到mysql的数据,查表original.t_qxrz
		String urlSQL = "select -1 as diaoyutype, from_unixtime(c_capturetime, 'yyyy-MM-dd HH:mm:ss') as creat_time, "
				+ "'' as name, c_msisdn as msisdn, 0 as count, c_url as url2, c_dip AS ip, '' as ua, now() as r_time, "
				+ "MD5(CONCAT(c_url, '', c_msisdn)) as name_md5, c_ydz_uli, c_extern_content['host'] as host "
				+ "from original.t_qxrz "
				+ "where day='" + Starter.TASK_DAY + "' and hour='" + String.format("%02d",Starter.TASK_HOUR)  + "' and c_ydz_spcode=3 "
				+ "and array_contains(map_keys(c_extern_content),'host') and c_extern_content['host'] != '' "
				+ "and c_ydz_uli != '' and c_url != '' and c_msisdn != '' and c_dip != '' ";
		if (!Strings.isNullOrEmpty(whiteString)) {
			urlSQL = urlSQL + " and find_in_set(c_extern_content['host'], '" + whiteString + "') = 0 ";
		}
		logger.info("hive查询访问日志,sql="+urlSQL);
		Dataset<Row> urlDs = sparkSession.sql(urlSQL);
		if (null == urlDs || urlDs.count() ==0L) {
			logger.info("查询结束,hive查询访问日志结果为空！");
			return false;
		}
		logger.info("hive查询访问日志完成,结果数量="+urlDs.count());

		//增加一列将url转换成domain
		Dataset<Row> urlAddDomainDs = urlDs.withColumn("url", functions.callUDF("format_url_to_domain", urlDs.col("host")));

		int showCount = Starter.props.getInt("debug.result.show.lines");
		int truncate = Starter.props.getInt("debug.result.show.line.characters");
		if(isDebug()){
			logger.info("开始打印访问日志查询结果");
			logger.info(urlAddDomainDs.showString(showCount,truncate));
			logger.info("结束打印访问日志查询结果");
		}

        //联合查询过滤域名
		String tacticsSql = "select domain as tactics_domain,label as labels from original.t_tactics where day ='" + Starter.TASK_DAY +"'";
		Dataset<Row> tacticsDs = sparkSession.sql(tacticsSql);
		logger.info("hive查询策略文件,sql="+tacticsSql);

		if (null == tacticsDs || tacticsDs.count() ==0L) {
			logger.info("查询结束,hive查询策略文件为空！");
			return false;
		}
		logger.info("hive查询策略文件完成,结果数量="+tacticsDs.count());
		if(isDebug()){
			logger.info("开始打印策略文件查询结果");
			logger.info(tacticsDs.showString(showCount,truncate));
			logger.info("结束打印策略文件查询结果");
		}

		Dataset<Row> filterDs = urlAddDomainDs.join(tacticsDs,
				urlAddDomainDs.col("url").contains(tacticsDs.col("tactics_domain")),
				"inner");


		logger.info("经策略文件过滤后的结果数量="+filterDs.count());
		if(isDebug()){
			logger.info("开始打印策略文件过滤后的查询结果");
			logger.info(filterDs.showString(showCount,truncate));
			logger.info("结束打印策略文件过滤后的查询结果");
		}


		//hive查询地区信息
		String odsSQL = "select c_uli, substr(c_areacode,1,4) as city, c_uli_addr as street, "
				+ "c_areacode as fjareaid, '' as fjareaname from original.t_ods_uli_info2";
		logger.info("hive查询地区信息,sql="+odsSQL);

		Dataset<Row> odsDataSet = sparkSession.sql(odsSQL);
		if(null == odsDataSet || odsDataSet.count() ==0L){
			logger.info("查询结束,hive查询地区信息为空");
			return false;
		}

		if(isDebug()){
			logger.info("开始打印地区信息");
			logger.info(odsDataSet.showString(showCount,truncate));
			logger.info("结束打印地区信息");
		}


		//联合查询filterAddLabelsDs和odsDataSet
		Dataset<Row> lastDataSet = filterDs.join(odsDataSet,
				filterDs.col("c_ydz_uli").equalTo(odsDataSet.col("c_uli")),
				"inner");

		if(isDebug()){
			logger.info("开始打印地区联合查询后的结果");
			logger.info(lastDataSet.showString(showCount,truncate));
			logger.info("结束打印地区联合查询后的结果");
		}
		//删除列
		lastDataSet = lastDataSet.drop("c_ydz_uli").drop("c_uli").drop("host").drop("tactics_domain");
		//将查询结果存储到mysql中 先删除原有数据防止重复
		DBQuery.deleteCurrentHourAntiInfo();
		lastDataSet.write().mode(SaveMode.Append).jdbc(jdbcUrl, antiInfo, connectionProperties);
		//关闭spark应用
		sparkSession.stop();
		sparkSession.close();
		logger.info("hive数据入库mysql结束");
		return true;
	}
```

#### spark-hive

>创建hive中表数据,
>
>执行hive的sql语句,展示hive数据

```scala
object HiveOnSpark1 {

  def main(args: Array[String]): Unit = {

    //如果想让hive运行在spark上，一定要开启spark对hive的支持
    val spark = SparkSession.builder()
      .appName("HiveOnSpark")
      .master("local[*]")
      .enableHiveSupport() //启用spark对hive的支持(可以兼容hive的语法了)
      .getOrCreate()

    //想要使用hive的元数据库，必须指定hive元数据的位置，添加一个hive-site.xml到当前程序的classpath下即可

    //有t_boy真个表或试图吗？
    //val result: DataFrame = spark.sql("SELECT * FROM niu ")
    //val result: DataFrame = spark.sql("select * from t_dm_goip_original_tactics")

    //val sql: DataFrame = spark.sql("CREATE TABLE niu (id bigint, name string)")

    //val sql: DataFrame = spark.sql("load data local inpath 'f:/wordcount/spark/t_access.log' into table t_access  partition(dt='20200513')")
    //val sql: DataFrame = spark.sql("load data local inpath 'f:/wordcount/spark/t_access.log' into table t_access  partition(dt='20200513')")
    //加载数据
    //val sql: DataFrame = spark.sql("load data local inpath 'f:/wordcount/spark/tactics.txt' into table t_dm_goip_original_tactics ")
    //新增中间表
    //val sql2: DataFrame = spark.sql("CREATE  TABLE  t_dm_goip_original_tactics(phone STRING COMMENT '被叫号码',province STRING COMMENT '省份',city STRING COMMENT '城市',call_time STRING COMMENT '通话时间',call_duration INT COMMENT '通话时长',fraud_type STRING COMMENT '违规类型') partitioned by(day string) row format delimited fields terminated by ','")
    //建库
    // val result1: DataFrame = spark.sql("create database iaf")

    //删表
   //val result1: DataFrame = spark.sql("drop table iaf.t_dm_mvtech_tactics")
    //val sql1: DataFrame = spark.sql("CREATE  TABLE iaf.t_dm_mvtech_tactics(phone STRING COMMENT '电话号码',province STRING COMMENT '省份',city STRING COMMENT '城市',start_time BIGINT COMMENT '开始时间',fraud_type STRING COMMENT '违规类型',phone_m STRING COMMENT '被叫电话号码') partitioned by(day string) row format delimited fields terminated by ','")
    // val sql2: DataFrame = spark.sql("CREATE  TABLE  t_dm_mvtech_tactics(phone STRING COMMENT '加密后电话号码',province STRING COMMENT '省份',city STRING COMMENT '城市',start_time BIGINT COMMENT '开始时间',fraud_type STRING COMMENT '违规类型',phone_m STRING COMMENT '未被加密电话号码')partitioned by(day string)row format delimited fields terminated by ','stored as textfile TBLPROPERTIES (\"parquet.compression\"=\"SNAPPY\")")
    //val sql: DataFrame = spark.sql("load data local inpath 'f:/wordcount/spark/tactics.txt'  into table iaf.t_dm_mvtech_tactics partition(day='20200511')")
    //查表
   val result: DataFrame = spark.sql("select * from iaf.t_dm_mvtech_tactics")
   // val result2: DataFrame = spark.sql("select date_add('2017-06-16 15:00:01',0.1)")
    //val result3: DataFrame = spark.sql("show databases")
    //result.collectAsList();
    result.show()
    spark.close()
  }
}
```

#### 对文件数据转换表数据

>1,读取文件
>
>2.将文件中的数据转换到映射表中
>
>3,做数据统计,写sql

```scala
object SQLFavTeacher {

  def main(args: Array[String]): Unit = {

    val topN =3

    val spark = SparkSession.builder().appName("RowNumberDemo")
      .master("local[4]")
      .getOrCreate()

    val lines: Dataset[String] = spark.read.textFile("E:\\wordcount\\spark\\teacher.log")

    import spark.implicits._

    val df: DataFrame = lines.map(line => {
      val tIndex = line.lastIndexOf("/") + 1
      val teacher = line.substring(tIndex)
      val host = new URL(line).getHost
      //学科的index
      val sIndex = host.indexOf(".")
      val subject = host.substring(0, sIndex)
      (subject, teacher)
    }).toDF("subject", "teacher")

    df.createTempView("v_sub_teacher")

    //该学科下的老师的访问次数
    val temp1: DataFrame = spark.sql("SELECT subject, teacher, count(*) counts FROM v_sub_teacher GROUP BY subject, teacher")

    //求每个学科下最受欢迎的老师的topn
    temp1.createTempView("v_temp_sub_teacher_counts")

    //val temp2 = spark.sql(s"SELECT * FROM (SELECT subject, teacher, counts, row_number() over(partition by subject order by counts desc) sub_rk, rank() over(order by counts desc) g_rk FROM v_temp_sub_teacher_counts) temp2 WHERE sub_rk <= $topN")

    //val temp2 = spark.sql(s"SELECT *, row_number() over(order by counts desc) g_rk FROM (SELECT subject, teacher, counts, row_number() over(partition by subject order by counts desc) sub_rk FROM v_temp_sub_teacher_counts) temp2 WHERE sub_rk <= $topN")

    val temp2 = spark.sql(s"SELECT *, dense_rank() over(order by counts desc) g_rk FROM (SELECT subject, teacher, counts, row_number() over(partition by subject order by counts desc) sub_rk FROM v_temp_sub_teacher_counts) temp2 WHERE sub_rk <= $topN")


    temp2.show()



    spark.stop()
  }
}

```

### 本地文件spark-写入到hive中

>读取本地文件,将文件写入到hive,rdd的方式
>
>将文件写入到kafka

```

```

