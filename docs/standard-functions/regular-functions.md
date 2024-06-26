---
title: Non-aggregate (Normal)
---

# Non-Aggregate (Normal) Functions

## named_struct { #named_struct }

```scala
named_struct(
  cols: Column*): Column
```

`named_struct`...FIXME

<!---
## Review Me

=== [[broadcast]] `broadcast` Function

[source, scala]
----
broadcast[T](df: Dataset[T]): Dataset[T]
----

`broadcast` function marks the input Dataset.md[Dataset] as small enough to be used in broadcast join.

TIP: Read up on spark-sql-joins-broadcast.md[Broadcast Joins (aka Map-Side Joins)].

[source, scala]
----
val left = Seq((0, "aa"), (0, "bb")).toDF("id", "token").as[(Int, String)]
val right = Seq(("aa", 0.99), ("bb", 0.57)).toDF("token", "prob").as[(String, Double)]

scala> left.join(broadcast(right), "token").explain(extended = true)
== Parsed Logical Plan ==
'Join UsingJoin(Inner,List(token))
:- Project [_1#123 AS id#126, _2#124 AS token#127]
:  +- LocalRelation [_1#123, _2#124]
+- BroadcastHint
   +- Project [_1#136 AS token#139, _2#137 AS prob#140]
      +- LocalRelation [_1#136, _2#137]

== Analyzed Logical Plan ==
token: string, id: int, prob: double
Project [token#127, id#126, prob#140]
+- Join Inner, (token#127 = token#139)
   :- Project [_1#123 AS id#126, _2#124 AS token#127]
   :  +- LocalRelation [_1#123, _2#124]
   +- BroadcastHint
      +- Project [_1#136 AS token#139, _2#137 AS prob#140]
         +- LocalRelation [_1#136, _2#137]

== Optimized Logical Plan ==
Project [token#127, id#126, prob#140]
+- Join Inner, (token#127 = token#139)
   :- Project [_1#123 AS id#126, _2#124 AS token#127]
   :  +- Filter isnotnull(_2#124)
   :     +- LocalRelation [_1#123, _2#124]
   +- BroadcastHint
      +- Project [_1#136 AS token#139, _2#137 AS prob#140]
         +- Filter isnotnull(_1#136)
            +- LocalRelation [_1#136, _2#137]

== Physical Plan ==
*Project [token#127, id#126, prob#140]
+- *BroadcastHashJoin [token#127], [token#139], Inner, BuildRight
   :- *Project [_1#123 AS id#126, _2#124 AS token#127]
   :  +- *Filter isnotnull(_2#124)
   :     +- LocalTableScan [_1#123, _2#124]
   +- BroadcastExchange HashedRelationBroadcastMode(List(input[0, string, true]))
      +- *Project [_1#136 AS token#139, _2#137 AS prob#140]
         +- *Filter isnotnull(_1#136)
            +- LocalTableScan [_1#136, _2#137]
----

NOTE: `broadcast` standard function is a special case of dataset/index.md[Dataset.hint] operator that allows for attaching any hint to a logical plan.

## <span id="coalesce"> coalesce

```scala
coalesce(
  e: Column*): Column
```

`coalesce` gives the first non-``null`` value among the given columns or `null`.

`coalesce` requires at least one column and all columns have to be of the same or compatible types.

Internally, `coalesce` creates a [Column](Column.md#apply) with a `Coalesce` expression (with the children being the [expressions](Column.md#expr) of the input `Column`).

### <span id="coalesce-example"> Demo: coalesce

```text
val q = spark.range(2)
  .select(
    coalesce(
      lit(null),
      lit(null),
      lit(2) + 2,
      $"id") as "first non-null value")
scala> q.show
+--------------------+
|first non-null value|
+--------------------+
|                   4|
|                   4|
+--------------------+
```

=== [[col]][[column]] Creating Columns -- `col` and `column` Functions

```scala
col(colName: String): Column
column(colName: String): Column
```

`col` and `column` methods create a [Column](Column.md) that you can later use to reference a column in a dataset.

```text
import org.apache.spark.sql.functions._

scala> val nameCol = col("name")
nameCol: org.apache.spark.sql.Column = name

scala> val cityCol = column("city")
cityCol: org.apache.spark.sql.Column = city
```

=== [[expr]] `expr` Function

[source, scala]
----
expr(expr: String): Column
----

`expr` function parses the input `expr` SQL statement to a `Column` it represents.

[source, scala]
----
val ds = Seq((0, "hello"), (1, "world"))
  .toDF("id", "token")
  .as[(Long, String)]

scala> ds.show
+---+-----+
| id|token|
+---+-----+
|  0|hello|
|  1|world|
+---+-----+

val filterExpr = expr("token = 'hello'")

scala> ds.filter(filterExpr).show
+---+-----+
| id|token|
+---+-----+
|  0|hello|
+---+-----+
----

Internally, `expr` uses the active session's SessionState.md[sqlParser] or creates a new  sql/SparkSqlParser.md[SparkSqlParser] to call spark-sql-ParserInterface.md#parseExpression[parseExpression] method.

=== [[struct]] `struct` Functions

[source, scala]
----
struct(cols: Column*): Column
struct(colName: String, colNames: String*): Column
----

`struct` family of functions allows you to create a new struct column based on a collection of `Column` or their names.

NOTE: The difference between `struct` and another similar `array` function is that the types of the columns can be different (in `struct`).

[source, scala]
----
scala> df.withColumn("struct", struct($"name", $"val")).show
+---+---+-----+---------+
| id|val| name|   struct|
+---+---+-----+---------+
|  0|  1|hello|[hello,1]|
|  2|  3|world|[world,3]|
|  2|  4|  ala|  [ala,4]|
+---+---+-----+---------+
----

=== [[monotonically_increasing_id]] `monotonically_increasing_id` Function

[source, scala]
----
monotonically_increasing_id(): Column
----

`monotonically_increasing_id` returns monotonically increasing 64-bit integers. The generated IDs are guaranteed to be monotonically increasing and unique, but not consecutive (unless all rows are in the same single partition which you rarely want due to the amount of the data).

[source, scala]
----
val q = spark.range(1).select(monotonically_increasing_id)
scala> q.show
+-----------------------------+
|monotonically_increasing_id()|
+-----------------------------+
|                  60129542144|
+-----------------------------+
----

The [current implementation](expressions/MonotonicallyIncreasingID.md) uses the partition ID in the upper 31 bits, and the lower 33 bits represent the record number within each partition. That assumes that the data set has less than 1 billion partitions, and each partition has less than 8 billion records.

```text
// Demo to show the internals of monotonically_increasing_id function
// i.e. how MonotonicallyIncreasingID expression works

// Create a dataset with the same number of rows per partition
val q = spark.range(start = 0, end = 8, step = 1, numPartitions = 4)

// Make sure that every partition has the same number of rows
q.mapPartitions(rows => Iterator(rows.size)).foreachPartition(rows => assert(rows.next == 2))
q.select(monotonically_increasing_id).show

// Assign consecutive IDs for rows per partition
import org.apache.spark.sql.expressions.Window
// count is the name of the internal registry of MonotonicallyIncreasingID to count rows
// Could also be "id" since it is unique and consecutive in a partition
import org.apache.spark.sql.functions.{row_number, shiftLeft, spark_partition_id}
val rowNumber = row_number over Window.partitionBy(spark_partition_id).orderBy("id")
// row_number is a sequential number starting at 1 within a window partition
val count = rowNumber - 1 as "count"
val partitionMask = shiftLeft(spark_partition_id cast "long", 33) as "partitionMask"
// FIXME Why does the following sum give "weird" results?!
val sum = (partitionMask + count) as "partitionMask + count"
val demo = q.select(
  $"id",
  partitionMask,
  count,
  // FIXME sum,
  monotonically_increasing_id)
scala> demo.orderBy("id").show
+---+-------------+-----+-----------------------------+
| id|partitionMask|count|monotonically_increasing_id()|
+---+-------------+-----+-----------------------------+
|  0|            0|    0|                            0|
|  1|            0|    1|                            1|
|  2|   8589934592|    0|                   8589934592|
|  3|   8589934592|    1|                   8589934593|
|  4|  17179869184|    0|                  17179869184|
|  5|  17179869184|    1|                  17179869185|
|  6|  25769803776|    0|                  25769803776|
|  7|  25769803776|    1|                  25769803777|
+---+-------------+-----+-----------------------------+
```

Internally, `monotonically_increasing_id` creates a [Column](Column.md#apply) with a [MonotonicallyIncreasingID](expressions/MonotonicallyIncreasingID.md) non-deterministic leaf expression.
-->
