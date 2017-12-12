---
categories: elasticsearch

tags: 
  - Elasticsearch

title: Elasticsearch 对Apache Spark支持

date: 2017-11-15
---

[Apache Spark](http://spark.apachecn.org/)是一个快速和通用的集群计算系统。 它提供了Java，Scala和Python中的高级API，以及支持通用执行图的优化引擎。
-- Spark website

Spark通过将数据缓存在内存中，在大型数据集上提供了类似于快速迭代/函数式的功能。 与本文档中提到的其他库不同，Apache Spark是一种计算框架，它不与Map / Reduce绑定，但是它与Hadoop集成，主要是HDFS（Spark也可以单独部署）。 elasticsearch-hadoop允许Elasticsearch在Spark中以两种方式使用：通过2.1以来的专用支持或从2.0开始通过Map/Reduce桥接(bridge)。ES从5.0以后，elasticsearch-hadoop 支持2.0 。

# 安装

就像其他库一样，elasticsearch-hadoop 需要在 Spark 的 classpath 中可用。 由于Spark具有多种部署模式，因此可以将其转换为目标 classpath ，无论它是否仅在一个节点上（就像本地模式的情况那样 - 将在整个文档中使用）或者根据所需的每个节点基础设施。

# 本地支持

**注意：2.1中添加**

elasticsearch-hadoop 提供了 Elasticsearch 和 Apache Spark 之间的本地集成，以RDD（Resilient Distributed Dataset，弹性分布式数据集）（或准RDD RDD）的形式，可以从 Elasticsearch 读取数据。 RDD提供了两种风格：一种是Scala（它以Scala集合作为Tuple2返回数据），另一种是Java（以包含 java.util.collections 的Tuple2的形式返回数据）。

> 只要有可能，最好考虑使用本地集成，因为它提供了最好的性能和最大的灵活性。

## 配置

要为Apache Spark配置elasticsearch-hadoop，可以设置 [SparkConf](http://spark.apache.org/docs/1.6.2/programming-guide.html#initializing-spark) 对象的“[配置](https://www.elastic.co/guide/en/elasticsearch/hadoop/current/configuration.html)”一章中描述的各种属性：

```Scala
import org.apache.spark.SparkConf

val conf = new SparkConf().setAppName(appName).setMaster(master)
conf.set("es.index.auto.create", "true")
```

```Java
SparkConf conf = new SparkConf().setAppName(appName).setMaster(master);
conf.set("es.index.auto.create", "true");
```

**命令行.** 如果想要通过命令行（或直接通过从文件加载配置）来设置属性，请注意， Spark 只接受以“spark.”前缀开头的属性，并且会忽略其余部分（取决于版本 可能会引发警告）。 要解决此问题，请通过在属性名前追加spark来定义elasticsearch-hadoop属性。则前缀为 spark.es 的属性elasticsearch-hadoop会自动解析它们：

```
$ ./bin/spark-submit --conf spark.es.resource=index/type ...
```

> 注意：es.resource属性变成 spark.es.resource

# 将数据写入Elasticsearch

通过 elasticsearch-hadoop，任何RDD都可以保存到 Elasticsearch 中，只要 RDD 内容可以转换成ES文档。 实际上，这意味着RDD类型需要是Map（无论是Scala还是Java），[JavaBean](http://docs.oracle.com/javase/tutorial/javabeans/) 或 Scala [case class](http://docs.scala-lang.org/tutorials/tour/case-classes.html)。 如果 RDD 类型并非如此，则可以轻松 transform Spark中的数据或插入自己的自定义 [ValueWriter](https://www.elastic.co/guide/en/elasticsearch/hadoop/current/configuration.html#configuration-serialization)。

## Scala

使用Scala时，只需导入org.elasticsearch.spark包，通过 [pimp my library](http://www.artima.com/weblogs/viewpost.jsp?thread=179766) 模式，使用saveToEs方法丰富了任何RDD API：

```Scala
import org.apache.spark.SparkContext    // 1
import org.apache.spark.SparkContext._

import org.elasticsearch.spark._        // 2

// ...

val conf = ...
val sc = new SparkContext(conf)         // 3

val numbers = Map("one" -> 1, "two" -> 2, "three" -> 3)
val airports = Map("arrival" -> "Otopeni", "SFO" -> "San Fran")

sc.makeRDD/** 4 **/(Seq(numbers, airports)).saveToEs/** 5 **/("spark/docs")
```

1. Spark Scala 导入

2. elasticsearch-hadoop Scala 导入

3. 通过Scala API启动Spark

4. makeRDD根据指定的集合创建一个ad-hoc RDD; 任何其他RDD（Java或Scala）都可以通过

5. 在spark/docs下的Elasticsearch索引内容（即两个文档（数字和机场））

> Scala用户可能会试图使用Seq和→符号来声明 root 对象（即JSON文档），而不是使用Map。 虽然类似，但 Seq 会产生与JSON文档无法匹配的稍微不同的类型：Seq是一个顺序序列（换句话说，是一个列表），而 → 创建一个Tuple，它或多或少是一个有序的固定数量的元素。 因此，lists列表不能用作文档，因为它不能映射到JSON对象; 然而它可以在一个内部自由使用。 因此，为什么在上面的例子中使用Map（k→v）代替Seq（k→v）


作为上述隐式导入的替代方案，可以使用 org.elasticsearch.spark.rdd 包中的 EsSpark 在 Scala 中使用 elasticsearch-hadoop Spark 支持，该包充当允许显式方法调用的实用程序类。 此外，代替Map（这是方便的，但由于在结构上的差异，每个实例需要一个映射），请使用一个 case 类：

```Scala
import org.apache.spark.SparkContext
import org.elasticsearch.spark.rdd.EsSpark                        // 1

// define a case class
case class Trip(departure: String, arrival: String)               // 2

val upcomingTrip = Trip("OTP", "SFO")
val lastWeekTrip = Trip("MUC", "OTP")

val rdd = sc.makeRDD(Seq(upcomingTrip, lastWeekTrip))             // 3
EsSpark.saveToEs(rdd, "spark/docs")                               // 4
```

1. EsSpark 导入

2. 定义一个名为Trip的 case 类

3. 以Trip实例创建一个RDD

4. 通过EsSpark明确索引RDD


对于需要指定文档的id（或其他元数据字段，如ttl或timestamp）的情况，可以通过设置适当的 [mapping](https://www.elastic.co/guide/en/elasticsearch/hadoop/current/configuration.html#cfg-mapping) es.mapping.id来实现。 在前面的示例之后，为了向Elasticsearch指示使用字段id作为文档ID，更新RDD配置（也可以在SparkConf 上设置该属性，由于其全局效果所以不鼓励使用）：

```Scala
EsSpark.saveToEs(rdd, "spark/docs", Map("es.mapping.id" -> "id"))
```

## Java

Java用户有一个专门的类，为EsSpark提供类似的功能，即org.elasticsearch.spark.rdd.api.java（类似于Spark的 [Java API](https://spark.apache.org/docs/1.0.1/api/java/index.html?org/apache/spark/api/java/package-summary.html) 的包）中的JavaEsSpark：

```Java
import org.apache.spark.api.java.JavaSparkContext;                              // 1
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.SparkConf;

import org.elasticsearch.spark.rdd.api.java.JavaEsSpark;                        // 2
...

SparkConf conf = ...
JavaSparkContext jsc = new JavaSparkContext(conf);                              // 3

Map<String, ?> numbers = ImmutableMap.of("one", 1, "two", 2);                   // 4
Map<String, ?> airports = ImmutableMap.of("OTP", "Otopeni", "SFO", "San Fran");

JavaRDD<Map<String, ?>> javaRDD = jsc.parallelize(ImmutableList.of(numbers, airports));
JavaEsSpark.saveToEs(javaRDD, "spark/docs");                                    // 6
```

1. Spark Java的导入

2. elasticsearch-hadoop Java的导入

3. 通过其Java API启动Spark

4. 为了简化示例，使用 [Guava](https://code.google.com/p/guava-libraries/)（Spark的依赖项）Immutable* 方法来创建简单的Map，List

5. 在这两个集合上创建一个简单的RDD; 任何其他RDD（Java或Scala）都可以通过

6. 在spark / docs下的Elasticsearch索引内容（即两个文档（数字和机场））

通过使用Java 5静态导入，代码可以进一步简化。 另外，映射（由于结构松散，映射是动态的）可以用JavaBean来替换：

```Java
public class TripBean implements Serializable {
   private String departure, arrival;

   public TripBean(String departure, String arrival) {
       setDeparture(departure);
       setArrival(arrival);
   }

   public TripBean() {}

   public String getDeparture() { return departure; }
   public String getArrival() { return arrival; }
   public void setDeparture(String dep) { departure = dep; }
   public void setArrival(String arr) { arrival = arr; }
}
```

```Java
import static org.elasticsearch.spark.rdd.api.java.JavaEsSpark;                // 1
...

TripBean upcoming = new TripBean("OTP", "SFO");
TripBean lastWeek = new TripBean("MUC", "OTP");

JavaRDD<TripBean> javaRDD = jsc.parallelize(
                            ImmutableList.of(upcoming, lastWeek));        // 2
saveToEs(javaRDD, "spark/docs");                                          // 3
```
1. 静态导入JavaEsSpark

2. 定义一个包含TripBean实例的RDD（TripBean是一个JavaBean）

3. 调用saveToEs方法而不必再次输入JavaEsSpark

设置文档id（或其他元数据字段，如ttl或timestamp）类似于Scala，尽管可能稍微冗长些，这取决于您使用的是JDK类还是一些其他实用程序（如Guava）：

```Java
JavaEsSpark.saveToEs(javaRDD, "spark/docs", ImmutableMap.of("es.mapping.id", "id"));

```
# 将现有的JSON写入Elasticsearch

对于RDD中的数据已经在JSON中的情况，elasticsearch-hadoop允许直接索引而不用任何转换。 数据按原样并直接发送到Elasticsearch。 因此，在这种情况下，假设每个条目都代表一个JSON文档，elasticsearch-hadoop期望包含字符串或字节数组（byte []/Array[Byte]）的RDD。 如果RDD没有正确的签名，则不能应用saveJsonToEs方法（在Scala中它们将不可用）。

## Scala

```Scala
val json1 = """{"reason" : "business", "airport" : "SFO"}"""      // 1
val json2 = """{"participants" : 5, "airport" : "OTP"}"""

new SparkContext(conf).makeRDD(Seq(json1, json2))
                      .saveJsonToEs("spark/json-trips")   // 2
```

1. RDD中条目的示例 - JSON按原样写入，不进行任何转换

2. 通过专用的saveJsonToEs方法索引JSON数据

## Java

```Java
String json1 = "{\"reason\" : \"business\",\"airport\" : \"SFO\"}";  // 1
String json2 = "{\"participants\" : 5,\"airport\" : \"OTP\"}";

JavaSparkContext jsc = ...
JavaRDD<String> stringRDD = jsc.parallelize(ImmutableList.of(json1, json2));
JavaEsSpark.saveJsonToEs(stringRDD, "spark/json-trips");             // 2
```
1. RDD中条目的示例 - JSON按原样写入，不进行任何转换

2. 注意RDD<String>签名

3. 通过专用的saveJsonToEs方法索引JSON数据

# 写入动态/多资源

对于写入 Elasticsearch 的数据需要在不同桶(buckets)（基于数据内容）下编制索引的情况，可以使用 es.resource.write 字段，该字段在运行时接受从文档内容解析的模式。 遵循上述示例，可以如下配置它：

## Scala

```Scala
val game = Map("media_type"/* 1 */->"game","title" -> "FF VI","year" -> "1994")
val book = Map("media_type" -> "book","title" -> "Harry Potter","year" -> "2010")
val cd = Map("media_type" -> "music","title" -> "Surfing With The Alien")

sc.makeRDD(Seq(game, book, cd)).saveToEs("my-collection/{media_type}")  // 2
```

1. 用于分割数据的文档键。 任何字段都可以声明（但要确保它在所有文档中都可用）

2. 根据资源模式保存每个对象，在本例中基于media_type

对于将要写入的每个文档/对象，elasticsearch-hadoop 将提取 media_type 字段并使用其值来确定目标资源。

## Java

Java中也是相似的：

```Java
Map<String, ?> game =
  ImmutableMap.of("media_type", "game", "title", "FF VI", "year", "1994");
Map<String, ?> book = ...
Map<String, ?> cd = ...

JavaRDD<Map<String, ?>> javaRDD =
                jsc.parallelize(ImmutableList.of(game, book, cd));
saveToEs(javaRDD, "my-collection/{media_type}");  // 1
```

1. 根据资源模式保存每个对象，在本例中为media_type

# 处理文档 metadata

Elasticsearch允许每个文档都有自己的 [metadata](http://www.elasticsearch.org/guide/en/elasticsearch/guide/current/_document_metadata.html)。 如上所述，通过各种 [mapping](http://www.elasticsearch.org/guide/en/elasticsearch/guide/current/_document_metadata.html) 选项，可以定制这些参数，以便从其所属文档中提取它们的值。 此外，甚至可以 include/exclude 将数据的哪些部分发送回Elasticsearch。 在Spark中，elasticsearch-hadoop扩展了这个功能，允许元数据通过使用[RDD对](http://spark.apache.org/docs/latest/programming-guide.html#working-with-key-value-pairs)在文档之外提供。 换句话说，对于包含键值元组的RDD，可以从键和用作文档源的值中提取元数据。

元数据通过org.elasticsearch.spark.rdd包中的元数据Java[枚举](http://spark.apache.org/docs/latest/programming-guide.html#working-with-key-value-pairs)来描述，它标识了它的类型 - id，ttl，version等等。因此，RDD键可以是包含每个文档及其相关值的元数据的Map。 如果RDD键不是Map类型，则elasticsearch-hadoop会将该对象视为表示文档ID并相应地使用它。 这听起来比现在更复杂，所以让我们看看一些例子。

## Scala

Pair RDD或者简单地将RDDs与RDD [(K，V)]签名，可以利用saveToEsWithMeta方法，这些方法可以通过隐式导入org.elasticsearch.spark包或EsSpark对象。 要手动指定每个文档的ID，只需传入RDD中的Object（不是Map类型）即可：

```Scala
val otp = Map("iata" -> "OTP", "name" -> "Otopeni")
val muc = Map("iata" -> "MUC", "name" -> "Munich")
val sfo = Map("iata" -> "SFO", "name" -> "San Fran")

// instance of SparkContext
val sc = ...

val airportsRDD/* 1 */ = sc.makeRDD(Seq((1, otp), (2, muc), (3, sfo)))  // 2
airportsRDD.saveToEsWithMeta/* 3 */("airports/2015")
```

1. citiesRDD是关键值对RDD; 它是从元组的Seq创建的

2. Seq中每个元组的关键字代表其关联值/文档的ID; 换句话说，文件otp有id 1，muc 2和sfo 3

3. 由于 airportsRDD 是一对RDD，因此它具有可用的 saveToEsWithMeta 方法。 这告诉 elasticsearch-hadoop 要特别注意RDD密钥并将它们用作元数据，在这种情况下用作文档ID。 如果使用saveToE，那么 elasticsearch-hadoop 会将RDD元组（即键和值）视为文档的一部分。

当不止需要指定id时，应该使用类型为 org.elasticsearch.spark.rdd.Metadata 的键的 scala.collection.Map：

```Scala
import org.elasticsearch.spark.rdd.Metadata._          

val otp = Map("iata" -> "OTP", "name" -> "Otopeni")
val muc = Map("iata" -> "MUC", "name" -> "Munich")
val sfo = Map("iata" -> "SFO", "name" -> "San Fran")

// metadata for each document
// note it's not required for them to have the same structure
val otpMeta = Map(ID -> 1, TTL -> "3h")                
val mucMeta = Map(ID -> 2, VERSION -> "23")            
val sfoMeta = Map(ID -> 3)                             

// instance of SparkContext
val sc = ...

val airportsRDD = sc.makeRDD(Seq((otpMeta, otp), (mucMeta, muc), (sfoMeta, sfo)))
airportsRDD.saveToEsWithMeta("airports/2015") 
```
