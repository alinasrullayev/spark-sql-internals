---
title: DataFrame
---

# DataFrame &mdash; Dataset of Rows with RowEncoder

Spark SQL introduces a tabular functional data abstraction called *DataFrame*. It is designed to ease developing Spark applications for processing large amount of structured tabular data on Spark infrastructure.

DataFrame is a data abstraction or a domain-specific language (DSL) for working with *structured* and *semi-structured data*, i.e. datasets that you can specify a schema for.

`DataFrame` is a collection of [Row](Row.md)s with a [schema](types/index.md) that is the result of executing a structured query (once it will have been executed).

DataFrame uses the immutable, in-memory, resilient, distributed and parallel capabilities of spark-rdd.md[RDD], and applies a structure called schema to the data.

[NOTE]
====
In Spark *2.0.0* `DataFrame` is a _mere_ type alias for `Dataset[Row]`.

[source, scala]
----
type DataFrame = Dataset[Row]
----

See https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/package.scala#L45[org.apache.spark.package.scala].
====

`DataFrame` is a distributed collection of tabular data organized into *rows* and *named columns*. It is conceptually equivalent to a table in a relational database with operations to project (`select`), `filter`, `intersect`, `join`, `group`, `sort`, `join`, `aggregate`, or `convert` to a RDD (consult https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrame[DataFrame API])

[source, scala]
----
data.groupBy('Product_ID).sum('Score)
----

Spark SQL borrowed the concept of DataFrame from http://pandas.pydata.org/pandas-docs/stable/dsintro.html[pandas' DataFrame] and made it *immutable*, *parallel* (one machine, perhaps with many processors and cores) and *distributed* (many machines, perhaps with many processors and cores).

NOTE: Hey, big data consultants, time to help teams migrate the code from pandas' DataFrame into Spark's DataFrames (at least to PySpark's DataFrame) and offer services to set up large clusters!

DataFrames in Spark SQL strongly rely on spark-rdd.md[the features of RDD] - it's basically a RDD exposed as structured DataFrame by appropriate operations to handle very big data from the day one. So, petabytes of data should _not_ scare you (unless you're an administrator to create such clustered Spark environment - book-intro.md[_contact me when you feel alone with the task_]).

[source, scala]
----
val df = Seq(("one", 1), ("one", 1), ("two", 1))
  .toDF("word", "count")

scala> df.show
+----+-----+
|word|count|
+----+-----+
| one|    1|
| one|    1|
| two|    1|
+----+-----+

val counted = df.groupBy('word).count

scala> counted.show
+----+-----+
|word|count|
+----+-----+
| two|    1|
| one|    2|
+----+-----+
----

You can create DataFrames by <<read, loading data from structured files (JSON, Parquet, CSV), RDDs, tables in Hive, or external databases (JDBC)>>. You can also create DataFrames from scratch and build upon them (as in the above example). See https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrame[DataFrame API]. You can read any format given you have appropriate Spark SQL extension of [DataFrameReader](DataFrameReader.md) to format the dataset appropriately.

CAUTION: FIXME Diagram of reading data from sources to create DataFrame

You can execute queries over DataFrames using two approaches:

* <<query-using-sql, the good ol' SQL>> - helps migrating from "SQL databases" world into the world of DataFrame in Spark SQL
* <<query-using-dsl, Query DSL>> - an API that helps ensuring proper syntax at compile time.

`DataFrame` also allows you to do the following tasks:

* <<filter, Filtering>>

DataFrames use the [Catalyst logical query optimizer](catalyst/Optimizer.md) to produce efficient queries (and so they are supposed to be faster than corresponding RDD-based queries).

NOTE: Your DataFrames can also be type-safe and moreover further improve their performance through [specialized encoders](Encoder.md) that can significantly cut serialization and deserialization times.

You can enforce types on [generic rows](Row.md) and hence bring type safety (at compile time) by <<as, encoding rows into type-safe `Dataset` object>>. As of Spark 2.0 it is a preferred way of developing Spark applications.

## Features of DataFrame

A `DataFrame` is a collection of "generic" [Row](Row.md) instances (as `RDD[Row]`) and a [schema](types/index.md).

NOTE: Regardless of how you create a `DataFrame`, it will always be a pair of `RDD[Row]` and [StructType](types/StructType.md).

## SQLContext, spark, and Spark shell

You use https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.SQLContext[org.apache.spark.sql.SQLContext] to build DataFrames and execute SQL queries.

The quickest and easiest way to work with Spark SQL is to use spark-shell.md[Spark shell] and `spark` object.

```
scala> spark
res1: org.apache.spark.sql.SQLContext = org.apache.spark.sql.hive.HiveContext@60ae950f
```

As you may have noticed, `spark` in Spark shell is actually a  https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.hive.HiveContext[org.apache.spark.sql.hive.HiveContext] that integrates *the Spark SQL execution engine* with data stored in https://hive.apache.org/[Apache Hive].

> The Apache Hive™ data warehouse software facilitates querying and managing large datasets residing in distributed storage.

=== Creating DataFrames from Scratch

Use Spark shell as described in spark-shell.md[Spark shell].

==== Using toDF

After you `import spark.implicits._` (which is done for you by Spark shell) you may apply `toDF` method to convert objects to DataFrames.

[source, scala]
----
scala> val df = Seq("I am a DataFrame!").toDF("text")
df: org.apache.spark.sql.DataFrame = [text: string]
----

==== Creating DataFrame using Case Classes in Scala

This method assumes the data comes from a Scala case class that will describe the schema.

[source, scala]
----
scala> case class Person(name: String, age: Int)
defined class Person

scala> val people = Seq(Person("Jacek", 42), Person("Patryk", 19), Person("Maksym", 5))
people: Seq[Person] = List(Person(Jacek,42), Person(Patryk,19), Person(Maksym,5))

scala> val df = spark.createDataFrame(people)
df: org.apache.spark.sql.DataFrame = [name: string, age: int]

scala> df.show
+------+---+
|  name|age|
+------+---+
| Jacek| 42|
|Patryk| 19|
|Maksym|  5|
+------+---+
----

==== Custom DataFrame Creation using createDataFrame

https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.SQLContext[SQLContext] offers a family of `createDataFrame` operations.

```
scala> val lines = sc.textFile("Cartier+for+WinnersCurse.csv")
lines: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[3] at textFile at <console>:24

scala> val headers = lines.first
headers: String = auctionid,bid,bidtime,bidder,bidderrate,openbid,price

scala> import org.apache.spark.sql.types.{StructField, StringType}
import org.apache.spark.sql.types.{StructField, StringType}

scala> val fs = headers.split(",").map(f => StructField(f, StringType))
fs: Array[org.apache.spark.sql.types.StructField] = Array(StructField(auctionid,StringType,true), StructField(bid,StringType,true), StructField(bidtime,StringType,true), StructField(bidder,StringType,true), StructField(bidderrate,StringType,true), StructField(openbid,StringType,true), StructField(price,StringType,true))

scala> import org.apache.spark.sql.types.StructType
import org.apache.spark.sql.types.StructType

scala> val schema = StructType(fs)
schema: org.apache.spark.sql.types.StructType = StructType(StructField(auctionid,StringType,true), StructField(bid,StringType,true), StructField(bidtime,StringType,true), StructField(bidder,StringType,true), StructField(bidderrate,StringType,true), StructField(openbid,StringType,true), StructField(price,StringType,true))

scala> val noheaders = lines.filter(_ != header)
noheaders: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[10] at filter at <console>:33

scala> import org.apache.spark.sql.Row
import org.apache.spark.sql.Row

scala> val rows = noheaders.map(_.split(",")).map(a => Row.fromSeq(a))
rows: org.apache.spark.rdd.RDD[org.apache.spark.sql.Row] = MapPartitionsRDD[12] at map at <console>:35

scala> val auctions = spark.createDataFrame(rows, schema)
auctions: org.apache.spark.sql.DataFrame = [auctionid: string, bid: string, bidtime: string, bidder: string, bidderrate: string, openbid: string, price: string]

scala> auctions.printSchema
root
 |-- auctionid: string (nullable = true)
 |-- bid: string (nullable = true)
 |-- bidtime: string (nullable = true)
 |-- bidder: string (nullable = true)
 |-- bidderrate: string (nullable = true)
 |-- openbid: string (nullable = true)
 |-- price: string (nullable = true)

scala> auctions.dtypes
res28: Array[(String, String)] = Array((auctionid,StringType), (bid,StringType), (bidtime,StringType), (bidder,StringType), (bidderrate,StringType), (openbid,StringType), (price,StringType))

scala> auctions.show(5)
+----------+----+-----------+-----------+----------+-------+-----+
| auctionid| bid|    bidtime|     bidder|bidderrate|openbid|price|
+----------+----+-----------+-----------+----------+-------+-----+
|1638843936| 500|0.478368056|  kona-java|       181|    500| 1625|
|1638843936| 800|0.826388889|     doc213|        60|    500| 1625|
|1638843936| 600|3.761122685|       zmxu|         7|    500| 1625|
|1638843936|1500|5.226377315|carloss8055|         5|    500| 1625|
|1638843936|1600|   6.570625|    jdrinaz|         6|    500| 1625|
+----------+----+-----------+-----------+----------+-------+-----+
only showing top 5 rows
```

=== Loading data from structured files

==== Creating DataFrame from CSV file

Let's start with an example in which *schema inference* relies on a custom case class in Scala.

```
scala> val lines = sc.textFile("Cartier+for+WinnersCurse.csv")
lines: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[3] at textFile at <console>:24

scala> val header = lines.first
header: String = auctionid,bid,bidtime,bidder,bidderrate,openbid,price

scala> lines.count
res3: Long = 1349

scala> case class Auction(auctionid: String, bid: Float, bidtime: Float, bidder: String, bidderrate: Int, openbid: Float, price: Float)
defined class Auction

scala> val noheader = lines.filter(_ != header)
noheader: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[53] at filter at <console>:31

scala> val auctions = noheader.map(_.split(",")).map(r => Auction(r(0), r(1).toFloat, r(2).toFloat, r(3), r(4).toInt, r(5).toFloat, r(6).toFloat))
auctions: org.apache.spark.rdd.RDD[Auction] = MapPartitionsRDD[59] at map at <console>:35

scala> val df = auctions.toDF
df: org.apache.spark.sql.DataFrame = [auctionid: string, bid: float, bidtime: float, bidder: string, bidderrate: int, openbid: float, price: float]

scala> df.printSchema
root
 |-- auctionid: string (nullable = true)
 |-- bid: float (nullable = false)
 |-- bidtime: float (nullable = false)
 |-- bidder: string (nullable = true)
 |-- bidderrate: integer (nullable = false)
 |-- openbid: float (nullable = false)
 |-- price: float (nullable = false)

scala> df.show
+----------+------+----------+-----------------+----------+-------+------+
| auctionid|   bid|   bidtime|           bidder|bidderrate|openbid| price|
+----------+------+----------+-----------------+----------+-------+------+
|1638843936| 500.0|0.47836804|        kona-java|       181|  500.0|1625.0|
|1638843936| 800.0| 0.8263889|           doc213|        60|  500.0|1625.0|
|1638843936| 600.0| 3.7611227|             zmxu|         7|  500.0|1625.0|
|1638843936|1500.0| 5.2263775|      carloss8055|         5|  500.0|1625.0|
|1638843936|1600.0|  6.570625|          jdrinaz|         6|  500.0|1625.0|
|1638843936|1550.0| 6.8929167|      carloss8055|         5|  500.0|1625.0|
|1638843936|1625.0| 6.8931136|      carloss8055|         5|  500.0|1625.0|
|1638844284| 225.0|  1.237419|dre_313@yahoo.com|         0|  200.0| 500.0|
|1638844284| 500.0| 1.2524074|        njbirdmom|        33|  200.0| 500.0|
|1638844464| 300.0| 1.8111342|          aprefer|        58|  300.0| 740.0|
|1638844464| 305.0| 3.2126737|        19750926o|         3|  300.0| 740.0|
|1638844464| 450.0| 4.1657987|         coharley|        30|  300.0| 740.0|
|1638844464| 450.0| 6.7363195|        adammurry|         5|  300.0| 740.0|
|1638844464| 500.0| 6.7364697|        adammurry|         5|  300.0| 740.0|
|1638844464|505.78| 6.9881945|        19750926o|         3|  300.0| 740.0|
|1638844464| 551.0| 6.9896526|        19750926o|         3|  300.0| 740.0|
|1638844464| 570.0| 6.9931483|        19750926o|         3|  300.0| 740.0|
|1638844464| 601.0| 6.9939003|        19750926o|         3|  300.0| 740.0|
|1638844464| 610.0|  6.994965|        19750926o|         3|  300.0| 740.0|
|1638844464| 560.0| 6.9953704|            ps138|         5|  300.0| 740.0|
+----------+------+----------+-----------------+----------+-------+------+
only showing top 20 rows
```

==== Creating DataFrame from CSV files using spark-csv module

You're going to use https://github.com/databricks/spark-csv[spark-csv] module to load data from a CSV data source that handles proper parsing and loading.

NOTE: Support for CSV data sources is available by default in Spark 2.0.0. No need for an external module.

Start the Spark shell using `--packages` option as follows:

```
➜  spark git:(master) ✗ ./bin/spark-shell --packages com.databricks:spark-csv_2.11:1.2.0
Ivy Default Cache set to: /Users/jacek/.ivy2/cache
The jars for the packages stored in: /Users/jacek/.ivy2/jars
:: loading settings :: url = jar:file:/Users/jacek/dev/oss/spark/assembly/target/scala-2.11/spark-assembly-1.5.0-SNAPSHOT-hadoop2.7.1.jar!/org/apache/ivy/core/settings/ivysettings.xml
com.databricks#spark-csv_2.11 added as a dependency

scala> val df = spark.read.format("com.databricks.spark.csv").option("header", "true").load("Cartier+for+WinnersCurse.csv")
df: org.apache.spark.sql.DataFrame = [auctionid: string, bid: string, bidtime: string, bidder: string, bidderrate: string, openbid: string, price: string]

scala> df.printSchema
root
 |-- auctionid: string (nullable = true)
 |-- bid: string (nullable = true)
 |-- bidtime: string (nullable = true)
 |-- bidder: string (nullable = true)
 |-- bidderrate: string (nullable = true)
 |-- openbid: string (nullable = true)
 |-- price: string (nullable = true)

 scala> df.show
 +----------+------+-----------+-----------------+----------+-------+-----+
 | auctionid|   bid|    bidtime|           bidder|bidderrate|openbid|price|
 +----------+------+-----------+-----------------+----------+-------+-----+
 |1638843936|   500|0.478368056|        kona-java|       181|    500| 1625|
 |1638843936|   800|0.826388889|           doc213|        60|    500| 1625|
 |1638843936|   600|3.761122685|             zmxu|         7|    500| 1625|
 |1638843936|  1500|5.226377315|      carloss8055|         5|    500| 1625|
 |1638843936|  1600|   6.570625|          jdrinaz|         6|    500| 1625|
 |1638843936|  1550|6.892916667|      carloss8055|         5|    500| 1625|
 |1638843936|  1625|6.893113426|      carloss8055|         5|    500| 1625|
 |1638844284|   225|1.237418982|dre_313@yahoo.com|         0|    200|  500|
 |1638844284|   500|1.252407407|        njbirdmom|        33|    200|  500|
 |1638844464|   300|1.811134259|          aprefer|        58|    300|  740|
 |1638844464|   305|3.212673611|        19750926o|         3|    300|  740|
 |1638844464|   450|4.165798611|         coharley|        30|    300|  740|
 |1638844464|   450|6.736319444|        adammurry|         5|    300|  740|
 |1638844464|   500|6.736469907|        adammurry|         5|    300|  740|
 |1638844464|505.78|6.988194444|        19750926o|         3|    300|  740|
 |1638844464|   551|6.989652778|        19750926o|         3|    300|  740|
 |1638844464|   570|6.993148148|        19750926o|         3|    300|  740|
 |1638844464|   601|6.993900463|        19750926o|         3|    300|  740|
 |1638844464|   610|6.994965278|        19750926o|         3|    300|  740|
 |1638844464|   560| 6.99537037|            ps138|         5|    300|  740|
 +----------+------+-----------+-----------------+----------+-------+-----+
 only showing top 20 rows
```

==== [[read]] Reading Data from External Data Sources (read method)

You can create DataFrames by loading data from structured files (JSON, Parquet, CSV), RDDs, tables in Hive, or external databases (JDBC) using https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.SQLContext[SQLContext.read] method.

[source, scala]
----
read: DataFrameReader
----

`read` returns a [DataFrameReader](DataFrameReader.md) instance.

Among the supported structured data (file) formats are (consult [Specifying Data Format (format method)](DataFrameReader.md#format) for `DataFrameReader`):

* JSON
* parquet
* JDBC
* ORC
* Tables in Hive and any JDBC-compliant database
* libsvm

```
val reader = spark.read
r: org.apache.spark.sql.DataFrameReader = org.apache.spark.sql.DataFrameReader@59e67a18

reader.parquet("file.parquet")
reader.json("file.json")
reader.format("libsvm").load("sample_libsvm_data.txt")
```

=== Querying DataFrame

NOTE: Spark SQL offers a <<query-using-dsl, Pandas-like Query DSL>>.

==== [[query-using-dsl]] Using Query DSL

You can select specific columns using `select` method.

NOTE: This variant (in which you use stringified column names) can only select existing columns, i.e. you cannot create new ones using select expressions.

```
scala> predictions.printSchema
root
 |-- id: long (nullable = false)
 |-- topic: string (nullable = true)
 |-- text: string (nullable = true)
 |-- label: double (nullable = true)
 |-- words: array (nullable = true)
 |    |-- element: string (containsNull = true)
 |-- features: vector (nullable = true)
 |-- rawPrediction: vector (nullable = true)
 |-- probability: vector (nullable = true)
 |-- prediction: double (nullable = true)

scala> predictions.select("label", "words").show
+-----+-------------------+
|label|              words|
+-----+-------------------+
|  1.0|     [hello, math!]|
|  0.0| [hello, religion!]|
|  1.0|[hello, phy, ic, !]|
+-----+-------------------+
```

```
scala> auctions.groupBy("bidder").count().show(5)
+--------------------+-----+
|              bidder|count|
+--------------------+-----+
|    dennisthemenace1|    1|
|            amskymom|    5|
| nguyenat@san.rr.com|    4|
|           millyjohn|    1|
|ykelectro@hotmail...|    2|
+--------------------+-----+
only showing top 5 rows
```

In the following example you query for the top 5 of the most active bidders.

Note the _tiny_ `$` and `desc` together with the column name to sort the rows by.

```
scala> auctions.groupBy("bidder").count().sort($"count".desc).show(5)
+------------+-----+
|      bidder|count|
+------------+-----+
|    lass1004|   22|
|  pascal1666|   19|
|     freembd|   17|
|restdynamics|   17|
|   happyrova|   17|
+------------+-----+
only showing top 5 rows

scala> import org.apache.spark.sql.functions._
import org.apache.spark.sql.functions._

scala> auctions.groupBy("bidder").count().sort(desc("count")).show(5)
+------------+-----+
|      bidder|count|
+------------+-----+
|    lass1004|   22|
|  pascal1666|   19|
|     freembd|   17|
|restdynamics|   17|
|   happyrova|   17|
+------------+-----+
only showing top 5 rows
```

```
scala> df.select("auctionid").distinct.count
res88: Long = 97

scala> df.groupBy("bidder").count.show
+--------------------+-----+
|              bidder|count|
+--------------------+-----+
|    dennisthemenace1|    1|
|            amskymom|    5|
| nguyenat@san.rr.com|    4|
|           millyjohn|    1|
|ykelectro@hotmail...|    2|
|   shetellia@aol.com|    1|
|              rrolex|    1|
|            bupper99|    2|
|           cheddaboy|    2|
|             adcc007|    1|
|           varvara_b|    1|
|            yokarine|    4|
|          steven1328|    1|
|              anjara|    2|
|              roysco|    1|
|lennonjasonmia@ne...|    2|
|northwestportland...|    4|
|             bosspad|   10|
|        31strawberry|    6|
|          nana-tyler|   11|
+--------------------+-----+
only showing top 20 rows
```

==== [[query-using-sql]][[registerTempTable]] Using SQL

Register a DataFrame as a named temporary table to run SQL.

[source,scala]
----
scala> df.registerTempTable("auctions") // <1>

scala> val sql = spark.sql("SELECT count(*) AS count FROM auctions")
sql: org.apache.spark.sql.DataFrame = [count: bigint]
----
<1> Register a temporary table so SQL queries make sense

You can execute a SQL query on a DataFrame using `sql` operation, but before the query is executed it is optimized by *Catalyst query optimizer*. You can print the physical plan for a DataFrame using the `explain` operation.

```
scala> sql.explain
== Physical Plan ==
TungstenAggregate(key=[], functions=[(count(1),mode=Final,isDistinct=false)], output=[count#148L])
 TungstenExchange SinglePartition
  TungstenAggregate(key=[], functions=[(count(1),mode=Partial,isDistinct=false)], output=[currentCount#156L])
   TungstenProject
    Scan PhysicalRDD[auctionid#49,bid#50,bidtime#51,bidder#52,bidderrate#53,openbid#54,price#55]

scala> sql.show
+-----+
|count|
+-----+
| 1348|
+-----+

scala> val count = sql.collect()(0).getLong(0)
count: Long = 1348
```

=== [[filter]] Filtering

[source, scala]
----
scala> df.show
+----+---------+-----+
|name|productId|score|
+----+---------+-----+
| aaa|      100| 0.12|
| aaa|      200| 0.29|
| bbb|      200| 0.53|
| bbb|      300| 0.42|
+----+---------+-----+

scala> df.filter($"name".like("a%")).show
+----+---------+-----+
|name|productId|score|
+----+---------+-----+
| aaa|      100| 0.12|
| aaa|      200| 0.29|
+----+---------+-----+
----

=== Handling data in Avro format

Use custom serializer using http://spark-packages.org/package/databricks/spark-avro[spark-avro].

Run Spark shell with `--packages com.databricks:spark-avro_2.11:2.0.0` (see https://github.com/databricks/spark-avro/issues/85[2.0.0 artifact is not in any public maven repo] why `--repositories` is required).

```
./bin/spark-shell --packages com.databricks:spark-avro_2.11:2.0.0 --repositories "http://dl.bintray.com/databricks/maven"
```

And then...

```
val fileRdd = sc.textFile("README.md")
val df = fileRdd.toDF

import org.apache.spark.sql.SaveMode

val outputF = "test.avro"
df.write.mode(SaveMode.Append).format("com.databricks.spark.avro").save(outputF)
```

See https://spark.apache.org/docs/latest/api/java/index.html#org.apache.spark.sql.SaveMode[org.apache.spark.sql.SaveMode] (and perhaps https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.SaveMode[org.apache.spark.sql.SaveMode] from Scala's perspective).

```
val df = spark.read.format("com.databricks.spark.avro").load("test.avro")
```

=== Example Datasets

* http://www.modelingonlineauctions.com/datasets[eBay online auctions]
* https://data.sfgov.org/Public-Safety/SFPD-Incidents-from-1-January-2003/tmnf-yvry[SFPD Crime Incident Reporting system]
