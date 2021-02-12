#### Spark SQL入门

#### 说明：

**使用pyspark或scala**

**不同于RDD编程的命令式编程范式，SparkSQL编程是声明式编程范式，我们可以通过SQL语句或者调用DataFrame的相关API描述我们想要实现的操作。**

**然后Spark会将我们的描述进行语法解析，找到相应的执行计划并对其进行流程优化，然后调用相应的基础命令进行执行。**

**我们使用pyspark进行RDD编程时，在Executor上跑的很多时候是Python代码，当然，少数时候也会跑java字节码**

但我们使用pyspark进行SparkSQL编程时，在Executor上跑的全部是java字节码，pyspark在Driver端就将相应的Python代码转换成了java任务然后放到Executor上执行。

因此，使用SparkSQL的编程范式进行编程，我们能够取得几乎和直接使用scala/java进行编程相当的效率（忽律语法解析时间差异）。此外SparkSQL提供了非常方便的数据读写API，我们可以用它和Hive表，HDFS，mysql表，Cassandra，Hbase等各种存储媒介进行数据交换。

#### 本节主要介绍以下内容：

* RDD和DataFrame对比

* 创建DataFrame

* DataFrame保存成文件

* DataFrame的API交互

* DataFrame的SQL交互

* DataFrame和DataSet区别

#### 一、RDD，DataFrame和DataSet对比

* DataFrame参照了python中的pandas的思想，在RDD基础上增加了schema，能够获取列名信息。

* DataSet在DataFrame基础上进一步增加了数据类型信息，可以在编译时发现错误类型。

* DataFrame可以看成DataSet\[Row\]，两者的API接口完全相同。

* DataFrame和DataSet都支持SQL交互式查询，可以和Hive无缝衔接。

* DataSet只有Scala语言和Java语言接口中才支持，在Python和R语言接口只支持DataFrame。

* DataFrame数据结构本质上是通过RDD来实现的，但是RDD是一种行存储的数据结构，而DataFrame是一种列存储的数据结构

#### 二、创建DataFrame

* **通过toDF方法转换成DataFrame**

  可以将RDD用toDF方法转换成DataFrame

```scala
#将RDD转换成DataFrame
    rdd = sc.parallelize([("LiLei",15,88),("HanMeiMei",16,90),("DaChui",17,60)])
    df = rdd.toDF(["name","age","score"])
    df.show()
    df.printSchema()
```

![](file:///Users/wuzhibo/Library/Application Support/typora-user-images/image-20210211162810848.png?lastModify=1613125365 "image-20210211162810848")

**people.txt**

```
Michael,29
Andy,30
Justin,19
```

### RDD转DataFrames

```scala
scala> val rdd=sc.textFile("people.txt")
    rdd: org.apache.spark.rdd.RDD[String] = people.txt MapPartitionsRDD[44] at textFile at <console>:24
```

#### 方式一：直接指定列名和数据类型

```scala
scala> val ds=rdd.map(_.split(",")).map(x=>(x(0),x(1).trim().toInt)).toDF("name","age")
    ds: org.apache.spark.sql.DataFrame = [name: string, age: int]
```

#### 方式二：通过反射转换

```scala
scala> case class people(name:String,age:Long)
    defined class people

    scala> rdd.map(_.split(",")).map(x=>(people(x(0),x(1).trim.toInt))).toDF()
    res44: org.apache.spark.sql.DataFrame = [name: string, age: bigint]
```

#### 方式三：通过编程设置Schema（StructType）

```scala
   # 在一些时候不能直接定义case类，就用这种方法
    scala> val rdd=sc.textFile("people.txt")
    rdd: org.apache.spark.rdd.RDD[String] = people.txt MapPartitionsRDD[97] at textFile at <console>:27

    scala> val schemaString = "name age"
    schemaString: String = name age

    scala> import org.apache.spark.sql.types._
    import org.apache.spark.sql.types._

    scala> val fields = schemaString.split(" ").map(fieldName => StructField(fieldName, StringType, nullable = true))
    fields: Array[org.apache.spark.sql.types.StructField] = Array(StructField(name,StringType,true), StructField(age,StringType,true))

    scala> val schems=StructType(fields)
    schems: org.apache.spark.sql.types.StructType = StructType(StructField(name,StringType,true), StructField(age,StringType,true))

    scala> import org.apache.spark.sql._
    import org.apache.spark.sql._

    scala> val rowrdd=rdd.map(_.split(",")).map(attributes => Row(attributes(0), attributes(1).trim))
    rowrdd: org.apache.spark.rdd.RDD[org.apache.spark.sql.Row] = MapPartitionsRDD[99] at map at <console>:35

    scala> spark.createDataFrame(rowrdd,schems)
    res46: org.apache.spark.sql.DataFrame = [name: string, age: string]
```

### RDD转DataSet

```scala
scala> rdd.map(_.split(",")).map(x=>(x(0),x(1).trim().toInt)).toDS()
    res17: org.apache.spark.sql.Dataset[(String, Int)] = [_1: string, _2: int]
```

### DataFrame/Dataset转RDD

```
scala> ds.rdd
    scala> df.rdd
```

### DataFrame转Dataset

```scala
scala> case class people(name:String,age:Long)
    defined class people
    
    scala> df.as[people]
    res39: org.apache.spark.sql.Dataset[people] = [age: bigint, name: string]
```

### Dataset转DataFrame

```
scala> ds.toDF()
```



