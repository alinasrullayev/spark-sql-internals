title: ExplodeBase

# ExplodeBase Base Generator Expression

<!---
`ExplodeBase` is the base class for <<Explode, Explode>> and <<PosExplode, PosExplode>> generator expressions.

`ExplodeBase` is a <<spark-sql-Expression-UnaryExpression.md#, unary expression>> and <<Generator, Generator>> with Expression.md#CodegenFallback[CodegenFallback].

=== [[Explode]] Explode Generator Unary Expression

`Explode` is a unary expression that produces a sequence of records for each value in the array or map.

`Explode` is a result of executing `explode` function (in SQL and [functions](../standard-functions/index.md#explode))

[source, scala]
----
scala> sql("SELECT explode(array(10,20))").explain
== Physical Plan ==
Generate explode([10,20]), false, false, [col#68]
+- Scan OneRowRelation[]

scala> sql("SELECT explode(array(10,20))").queryExecution.optimizedPlan.expressions(0)
res18: org.apache.spark.sql.catalyst.expressions.Expression = explode([10,20])

val arrayDF = Seq(Array(0,1)).toDF("array")
scala> arrayDF.withColumn("num", explode('array)).explain
== Physical Plan ==
Generate explode(array#93), true, false, [array#93, num#102]
+- LocalTableScan [array#93]
----
-->
