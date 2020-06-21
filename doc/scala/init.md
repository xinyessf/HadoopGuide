### idea如何运行scal程序

```
1.idea中安装scla插件
```

### 添加依赖插件

>idea中可以不用安装scala的sdk

```
<dependencies>
        <!-- 导入scala的依赖 -->
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>${scala.version}</version>
        </dependency>
    </dependencies>


    <build>
        <pluginManagement>
            <plugins>
                <!-- 编译scala的插件 -->
                <plugin>
                    <groupId>net.alchim31.maven</groupId>
                    <artifactId>scala-maven-plugin</artifactId>
                    <version>3.2.2</version>
                </plugin>
                <!-- 编译java的插件 -->
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.5.1</version>
                </plugin>
            </plugins>
        </pluginManagement>
        <plugins>
            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <id>scala-compile-first</id>
                        <phase>process-resources</phase>
                        <goals>
                            <goal>add-source</goal>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>scala-test-compile</id>
                        <phase>process-test-resources</phase>
                        <goals>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>compile</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>


            <!-- 打jar插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

### 关键字

```
main  主函数
var  
val  常量
String 
```

#### 数据类型

```
Byte
Char
Short
Int AnyVar Any
Long 
Float
Double(无包装类型)
Boolean
Unit (类型java void)
```



### 语法

#### class和obejct

```
主方法只能写在 object定义的文件中
object和class啥区别：
回顾一个问题：java中有一个知识点 静态
分号可有可无
object 单例
scala是包级别区分，类名可以和文件名不一致
```

#### 打印

```scala
println("hello");
println("hello "+args(0)+args(1))
```

#### var 和val

```
var 可以修改
val 常量
```

#### 占位符

```
println(f"姓名：$name%s  年龄：$age")      // 该行输出有换行
printf("%s 学费 %4.2f, 网址是%s  补充%s", name, 1234.146516, "xx",2)   // 该行输出没有换行
val stu = new Student("taotao", 18)
println(s"${name}")

println(s"${stu.name}")
```

#### 定义对象

```
class Student(var name: String, var age: Int)
```

#### if else

```scala
val r = if(i < 8) i // else 没有写， 编译器会自动推测出你什么都没有返回就是Unit

val r1 = if(i<8) i else 1

println(r1)

// 最后一行代码作为返回值
 val i: Int = 12
        val s = if(i > 10) {
            i
            i * i
        } else {
            0
            100
        }
// i=12 大于8 发挥()
 val r = if(i < 8) i // else 没有写， 编译器会自动推测出你什么都没有返回就是Unit

```

#### for

[for循环使用](https://blog.csdn.net/yang735136055/article/details/84108960)

[for](https://blog.csdn.net/qq_33366229/article/details/91445469)

```scala
//第一种
for(i <- 1 to 3){
  print(i + " ")
}

//第二种
for(i <- 1 until 3) {
  print(i + " ")
}
// i!=2
for(i <- 1 to 3 if i != 2) {
  print(i + " ")
}
//控制步进scala
val  aa = 1 until 10
println(aa)
for(i<-aa){}
//步进为2
val seqs:Range.Includsive=(1 until(10,2))
println(seqs)
//守护逻辑
for(i<-seqs if(i%2==0)){
    
}
//99乘法表,嵌套循环,原始方法
for(i<- 1 to 9){
    for(j<-1 to 9){
        if(j<=i)print(s"$i * $ j= ${i*j}\t")
        if(j==i)println()
    }
}
//99 scala
for(i<- 1 to 9;j<- 1 to 9 if(j<=i)){ 
   num+=1
    if(j<=i)print(s"$i * $ j= ${i*j}\t")
        if(j==i)println()
}
println(num) //81
//收集变量,最后一行是返回
var seqs:immutable.IndexedSeq[Int]=for(i<- 1 to 10) yield {
    i*2
}
println(seqs)
//for 循环中的 yield可以当做一个看不见的临时值存储区域，每次循环结果保存在该区域中，循环结束后将返回一个集合。返回的集合和传入的集合一样（也就是你传入Array  给你返回Array，其他类似）

val array = Array[Int](1, 2, 3, 4, 5)
    val arrayAdd = for (item <- array) yield item + 100
    print(arrayAdd.mkString(","))
// 输出101,102,103,104,105

  val arrayAdd2 = changeArray(array)
    print(arrayAdd2.mkString(","))
  }

  def changeArray(array: Array[Int]) = {
    for (i <- 0 until array.length) {
      array(i) = array(i) + 100
    }
    array
  }
//
//
```

#### while

```scala
var aa=0
while(aa<10){
    println(aa)
    aa=aa+1
    
}
```



#### 运算符(重载方法)

[运算符](https://www.yiibai.com/scala/scala_operators.html)

```scala

```

#### 定义方法

![img](img/1.jpg)

####函数

```scala
// 没有返回值默认unit
def fun01(){
println("hello world")
}
fun01()
var x=3
var y=fun01()

def fun01():Int={
    return 3
}
//==list返回
def fun02():util.LinkedList[String]={
    new util.LinkedList[String]()
}
//参数,约束传参数类型
def fun03(a:Int):Unit={
    println(a)
}
//递归函数
def fun04(num:Int):Int={
    if(num==1){
        num
    }else{
        num=fun04(num-1)
    }
}
//
def fun05(a:Int=8,b:String="abc"):Unit={
    println(s"$a\t$b")
}
//该 a值
fun05(22);
//该 b 值,提示
fun05(b="ooxx")
//匿名函数,  函数体 println(w)  (a:Int,b:Int)=>{a+b}   函数体
//签名  (Int,Int)=>Int
var yy:(Int,Int)=>Int =(a:Int,b:Int)=>{
    a+b
}
var w:Int=yy(3,4)
//==嵌套函数
def 06(a:String):Unit={
    def 05(a:String):Unit={
        println(a)
    }  
    fun05("hello")
}
//偏应用函数
def fu07(date:Date,tp:String,msg:String):Unit{
    println(s"$date\t$tp\$tmsg")
}
fun07(new Date(),"info","ok")
var info=fun07(_:Date,"iinfo",_:String)
var error=fun07(_:Date,"error",_:String)
info(new Date,"ok")
error(new Date,"error...")
//可变参数
def fun08(a:Int*):Unit={
    for(e>-a){
        println(e)
    }
    //for 匿名
    a.foreach((x:Int)=>{print(x)})
    a.foreach(println(_))
}
fun08(2)
fun08(1,2,3,4,5,6,7,8 )
//==1.
a.foreach()
```

#### 高阶函数

```scala
//函数作为参数
def computer(a:Int,b:Int, f(Int,Int)=>Int):Unit={
    val res:Int=f(a,b)
    println(res)
    
}
computer(3,8 (x:Int,y:Int)=>{x+y})
computer(3,8 (x:Int,y:Int)=>{x*y})
//甜函数
computer(3,8,_*_)
//函数作为返回值
def factory(i:String):(Int,Int)=>Int={
   def plus(x:Int,y:Int)={
       x+y
   }
   if(i.equals("+")){
       plus
   }else{ 
       (x:Int,y:Int)->(x*y)
   }
}
computer(3,8,factory("+"))
//ke里化
def fun09(a:Int)(b:Int)(c:String):Unit={
    println(s"$a\t$b\t$c\t")
}
def fun09(3)(8)("sdfsdf")
def fun10(a:Int*)(b:String*):Unit={
    a.foreach(println)
    b.foreach(println)
}
fun10(1,2,3)("sdfs","sss")

//
def ooxx :Unit={
    println(1)
}
//方法不想执行,赋值给一个引用, 方法名+空格+下划线
val funa=ooxx
println(funa)
val func=ooxx _
func()
//scala 中+ 表示是函数
//语法->编译器->字节码->  <-jvm
```



####构造器

```scala
//==1.类型的构造器

//类里，裸露的代码是默认构造中的。有默认构造
//个性化构造！！
//类名构造器中的参数就是类的成员属性，且默认是val类型，且默认是private
//只有在类名构造其中的参数可以设置成var，其他方法函数中的参数都是val类型的，且不允许设置成var类型
object GouZao {
  def main(args: Array[String]): Unit = {
    val oxxx1:oxxx = new oxxx("aaa")
    println(oxxx1.sex)
    println(oxxx1.name)
  }

class oxxx(val sex:String){

var name="class name"

def this(xname:Int){
  this("abc")
}

var a:Int=3
//伴生关系
    
}
```

#### 集合

```scala
//list
val listjava =new util.LinkedList[String]();
listJava.add("hello");
//1.数组
// java泛型是<> scala 是[] ,所以数组用(n)
// val 约等于final 不可变的是val 指定引用的值
varl arr02=Array[Int](1,2,3,4)
val arr01=Array(1,2,3);
arr01(1)=99
println(arr01(1))
for(elem<-arr01){
    println(elem)
}
// 遍历元素,需要函数接收元素
arr01.foreach(println)
//2.链表
//scala 中collection中有2个包, immutable,mutable  默认的是不可变的immutable
val list01=List(1,2,3,4,5,4,3,2,1);
list01.foreach(println)
var list02=new ListBuffer[]()
list02.+=(33)
list02.+=(34)
list02.+=(35)
list.foreach(println)
// println("---------------set---------")
//默认不可变,无序,去重,
var set01:Set[Int]=Set(1,2,3,4,5);
for(e<-set01)println(e)

import scala.collection.mutable.Set
val set02:mutable.Set[Int]=Set(1,2,3,34,5)
set02.add(11)
set02.foreach(println)
val set03:Predef.Set[Int]=scala.collection.immutable.Set(33,23,44)
//set 03.add
//============tupble==============
val t2=new Tupble(11,"sdf")
val t3=Tuple3(22,"dfdsf","3")
//上限是22个
val t4:(Int,Int,Int,Int)=(1,2,34)
//取元素
println(t2._1)
println(t4._3)
((a:Int))=>a+8
//==tuple 签名 补充

var tIter:Iterator[Any] =t22.producerInterator
while(tIter.hasNext()){
    
}
//==================map============
val map01=Map(("a",33),"b"->22,("c",33),("a",3333))
val keys:Iterator[String] =map01.keys
//option: none some
println(map01.get("a").get)
println(map01.get("w").get)
println(map01.getOrElse("w"))
println(map01.get("a").getOrElse("hello world"))
for(elem<-keys){
      println(s" key=${elem} value=${map01.get(elem).get}")

}
    println(map01.get("a"))
    println(map01.get("q"))
    println(map01.get("a").getOrElse("hello world"))
    println(map01.get("q").getOrElse("hello world"))
//可变
 val map02: mutable.Map[String, Int] = scala.collection.mutable.Map(("a",11),("b",22))
    map02.put("c",22)


val map02:mutable.Map[String,Int] =scala.collection.mutable.Map(("a",11),("b",22))
map02.put();

//==========变化===========
val list = List(1,2,3,4,5,6)

list.foreach(println)

val listMap: List[Int] = list.map( (x:Int) => x+10  )
listMap.foreach(println)
val listMap02: List[Int] = list.map(  _*10 )

list.foreach(println)
listMap02.foreach(println)

//运算
//==========变化 生化===========
val listStr = List(
      "hello world",
      "hello msb",
      "good idea"
    )
//        val listStr = Array(
//      "hello world",
//      "hello msb",
//      "good idea"
//    )
//        val listStr = Set(
//      "hello world",
//      "hello msb",
//      "good idea"
//    )

//== flatMap
val flatMap:List[String] =listStr.flatMap((x:String)=>x.spilt(" "))

val mapList:List[(String,Int)]=flatMap.map((_,1))

//==以上代码执行的复杂度, 内存扩大了N倍, 每一步计算内存都留有对象数据,有没有现成技术解决计算中间状态占用内存问题,迭代器计算
//=================升化=============

    val iter: Iterator[String] = listStr.iterator  //什么是迭代器，为什么会有迭代器模式？  迭代器里不存数据！

    val iterFlatMap= iter.flatMap(  (x:String)=> x.split(" ") )
//    iterFlatMap.foreach(println)

    val iterMapList = iterFlatMap.map( (_,1) )

    while(iterMapList.hasNext){
      val tuple: (String, Int) = iterMapList.next()
      println(tuple)
    }
//为什么使用迭代器,不存数据,放的指针--->指向集合,迭代一次,指针被清空了
```

#### train

>类似java的继承和接口

```scala
trait  God{
  def say(): Unit ={
    println("god...say")
  }
}

trait Mg{
  def ku(): Unit ={
    println("mg...say")
  }
  def haiRen():Unit
}

class Person(name:String)  extends   God with Mg{

  def hello(): Unit ={
    println(s"$name say hello")
  }

  override def haiRen(): Unit = {
    println("ziji shixian ....")
  }
}


object Lesson04_trait {

  def main(args: Array[String]): Unit = {

    val p = new Person("zhangsan")
    p.hello()
    p.say()
    p.ku()
    p.haiRen()

  }
```

**demo1**

```scala
trait God{
   def hello(name String):Unit={
        println("say")
    }  
}
trait Mg{
   def ku(name String):Unit={
        println(s"$name say hello")
    }  
    def haiRen():Unit
}

class  person(name:String) extends God with Mg{
    def hello(name String):Unit={
        println(s"$name say hello")
    }
}
```

#### match

```scala
def main(args: Array[String]): Unit = {
    val tup: (Double, Int, String, Boolean, Int) = (1.0,88,"abc",false,44)

    val iter: Iterator[Any] = tup.productIterator

    val res: Iterator[Unit] = iter.map(
      (x) => {
        x match {
          case 1 => println(s"$x...is 1")
          case 88 => println(s"$x ...is 88")
          case false => println(s"$x...is false")
          case w: Int if w > 50 => println(s"$w...is  > 50")
          case _ => println("wo ye bu zhi dao sha lei xing ")
        }
      }
    )
    while(res.hasNext)  res.next()

  }
```

#### case

>case 修饰的对象会去比较内容
>
>不用case 修饰比较的地址值

```scala
case class Dog(name:String,age:Int){
}
object Lesson05_case_class {
  def main(args: Array[String]): Unit = {
    val dog1 =  Dog("hsq",18)
    val dog2 =  Dog("hsq",18)
    println(dog1.equals(dog2))
    println(dog1 == dog2)
  }
}
```

#### 偏函数

>PartialFunction 
>
>

```scala

    def xxx:PartialFunction[  Any,String] ={
      case "hello"  => "val is hello"
      case x:Int => s"$x...is int"
      case _ => "none"
    }

    println(xxx(44))
    println(xxx("hello"))
    println(xxx("hi"))
```

#### 隐式转换(工具类)

>linkedList 在scala中没有 add
>
>//必须先承认一件事情：  list有foreach方法吗？  肯定是没有的~！ 在java里这么写肯定报错。。

**类**

```scala
// 偏函数使用,隐式转换的功能

  def main(args: Array[String]): Unit = {
    var linkedtLinked = new util.LinkedList[Int]()
    linkedtLinked.add(1)
    linkedtLinked.add(2)
    linkedtLinked.add(3)
    linkedtLinked.add(4)
    var unit = new XXX[Int](linkedtLinked)
    unit.foreach(println)

  }

  class XXX[T](list: util.LinkedList[T]) {
    def foreach(f: (T) => Unit): Unit = {
      val iter: util.Iterator[T] = list.iterator()
      while (iter.hasNext) f(iter.next())
    }

  }
```

**方法里面 泛型**

```scala
 def main(args: Array[String]): Unit = {
    var linkedtLinked = new util.LinkedList[Int]()
    linkedtLinked.add(1)
    linkedtLinked.add(2)
    linkedtLinked.add(3)
    linkedtLinked.add(4)

    def foreach[T](list: util.LinkedList[T], f: (T) => Unit): Unit = {
      val iter: util.Iterator[T] = list.iterator()
      while (iter.hasNext) f(iter.next())
    }

    foreach(linkedtLinked, println)


  }

```

**方法中  泛型**

```scala
def main(args: Array[String]): Unit = {
    var linkedtLinked = new util.LinkedList[Int]()
    linkedtLinked.add(1)
    linkedtLinked.add(2)
    linkedtLinked.add(3)
    linkedtLinked.add(4)

    val listArray = new util.ArrayList[Int]()
    listArray.add(1)
    listArray.add(2)
    listArray.add(3)

    //隐式转换：  隐式转换方法
    implicit def sdfsdf[T](list: util.LinkedList[T]) = {
      val iter: util.Iterator[T] = list.iterator()
      new XXX(iter)
    }

    implicit def sldkfjskldfj[T](list: java.util.ArrayList[T]) = {
      val iter: util.Iterator[T] = list.iterator()
      new XXX(iter)
    }
    linkedtLinked.foreach(println)
    listArray.foreach(println)

  }

  class XXX[T](list: util.Iterator[T]) {

    def foreach(f: (T) => Unit): Unit = {

      while (list.hasNext) f(list.next())
    }

  }
```

**方式中仅用一次**

```scala
def main(args: Array[String]): Unit = {
    var linkedtLinked = new util.LinkedList[Int]()
    linkedtLinked.add(1)
    linkedtLinked.add(2)
    linkedtLinked.add(3)
    linkedtLinked.add(4)
    val listArray = new util.ArrayList[Int]()
    listArray.add(1)
    listArray.add(2)
    listArray.add(3)
    //spark  RDD N方法 scala
    //隐式转换类
        implicit  class KKK[T](list:util.LinkedList[T]){
         def foreach( f:(T)=>Unit): Unit ={
            val iter: util.Iterator[T] = list.iterator()
            while(iter.hasNext) f(iter.next())
         }
       }
     val unit = new KKK[Int](linkedtLinked)
    unit.foreach(println)
  }
```



#### wordcount

```
def main(args: Array[String]): Unit = {

    val conf = new SparkConf()
    conf.setAppName("wordcount")
    conf.setMaster("local")  //单击本地运行

    val sc = new SparkContext(conf)
    //单词统计
    //DATASET
//    val fileRDD: RDD[String] = sc.textFile("bigdata-spark/data/testdata.txt",16)
    //hello world

//    fileRDD.flatMap(  _.split(" ") ).map((_,1)).reduceByKey(  _+_   ).foreach(println)

    val fileRDD: RDD[String] = sc.textFile("bigdata-spark/data/testdata.txt")
    //hello world
    val words: RDD[String] = fileRDD.flatMap((x:String)=>{x.split(" ")})
    //hello
    //world
    val pairWord: RDD[(String, Int)] = words.map((x:String)=>{new Tuple2(x,1)})
    //(hello,1)
    //(hello,1)
    //(world,1)
    val res: RDD[(String, Int)] = pairWord.reduceByKey(  (x:Int,y:Int)=>{x+y}   )
    //X:oldValue  Y:value
    //(hello,2)  -> (2,1)
    //(world,1)   -> (1,1)
    //(msb,2)   -> (2,1)

    val fanzhuan: RDD[(Int, Int)] = res.map((x)=>{  (x._2,1)  })
    val resOver: RDD[(Int, Int)] = fanzhuan.reduceByKey(_+_)


    resOver.foreach(println)
    res.foreach(println)




    Thread.sleep(Long.MaxValue)


  }
```

