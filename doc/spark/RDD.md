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

