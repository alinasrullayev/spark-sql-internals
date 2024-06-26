---
title: Project
---

# Project Unary Logical Operator

[[creating-instance]]
`Project` is a <<spark-sql-LogicalPlan.md#UnaryNode, unary logical operator>> that takes the following when created:

* [[projectList]] Project <<expressions/NamedExpression.md#, named expressions>>
* [[child]] Child <<spark-sql-LogicalPlan.md#, logical operator>>

`Project` is <<creating-instance, created>> to represent the following:

* Dataset operators, i.e. [joinWith](../joins.md#joinWith), [select](../dataset/index.md#select) (incl. `selectUntyped`), `unionByName`
* `KeyValueGroupedDataset` operators, i.e. `keys`, `mapValues`
* `CreateViewCommand` logical command is <<CreateViewCommand.md#run, executed>> (and <<CreateViewCommand.md#aliasPlan, aliasPlan>>)
* SQL's [SELECT](../sql/AstBuilder.md#withQuerySpecification) queries with named expressions

`Project` can also appear in a logical plan after [analysis](../Analyzer.md) or [optimization](../catalyst/Optimizer.md) phases.

```text
// FIXME Add examples for the following operators
// Dataset.unionByName
// KeyValueGroupedDataset.mapValues
// KeyValueGroupedDataset.keys
// CreateViewCommand.aliasPlan

// joinWith operator
case class Person(id: Long, name: String, cityId: Long)
case class City(id: Long, name: String)
val family = Seq(
  Person(0, "Agata", 0),
  Person(1, "Iweta", 0),
  Person(2, "Patryk", 2),
  Person(3, "Maksym", 0)).toDS
val cities = Seq(
  City(0, "Warsaw"),
  City(1, "Washington"),
  City(2, "Sopot")).toDS
val q = family.joinWith(cities, family("cityId") === cities("id"), "inner")
scala> println(q.queryExecution.logical.numberedTreeString)
00 Join Inner, (_1#41.cityId = _2#42.id)
01 :- Project [named_struct(id, id#32L, name, name#33, cityId, cityId#34L) AS _1#41]
02 :  +- LocalRelation [id#32L, name#33, cityId#34L]
03 +- Project [named_struct(id, id#38L, name, name#39) AS _2#42]
04    +- LocalRelation [id#38L, name#39]

// select operator
val qs = spark.range(10).select($"id")
scala> println(qs.queryExecution.logical.numberedTreeString)
00 'Project [unresolvedalias('id, None)]
01 +- Range (0, 10, step=1, splits=Some(8))

// select[U1](c1: TypedColumn[T, U1])
scala> :type q
org.apache.spark.sql.Dataset[(Person, City)]

val left = $"_1".as[Person]
val ql = q.select(left)
scala> println(ql.queryExecution.logical.numberedTreeString)
00 'SerializeFromObject [assertnotnull(assertnotnull(input[0, $line14.$read$$iw$$iw$Person, true])).id AS id#87L, staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, assertnotnull(assertnotnull(input[0, $line14.$read$$iw$$iw$Person, true])).name, true, false) AS name#88, assertnotnull(assertnotnull(input[0, $line14.$read$$iw$$iw$Person, true])).cityId AS cityId#89L]
01 +- 'MapElements <function1>, class scala.Tuple1, [StructField(_1,StructType(StructField(id,LongType,false), StructField(name,StringType,true), StructField(cityId,LongType,false)),true)], obj#86: $line14.$read$$iw$$iw$Person
02    +- 'DeserializeToObject unresolveddeserializer(newInstance(class scala.Tuple1)), obj#85: scala.Tuple1
03       +- Project [_1#44]
04          +- Join Inner, (_1#44.cityId = _2#45.id)
05             :- Project [named_struct(id, id#32L, name, name#33, cityId, cityId#34L) AS _1#44]
06             :  +- LocalRelation [id#32L, name#33, cityId#34L]
07             +- Project [named_struct(id, id#38L, name, name#39) AS _2#45]
08                +- LocalRelation [id#38L, name#39]

// SQL
spark.range(10).createOrReplaceTempView("nums")
val qn = spark.sql("select * from nums")
scala> println(qn.queryExecution.logical.numberedTreeString)
00 'Project [*]
01 +- 'UnresolvedRelation `nums`

// Examples with Project that was added during analysis
// Examples with Project that was added during optimization
```

NOTE: spark-sql-Expression-Nondeterministic.md[Nondeterministic] expressions are allowed in `Project` logical operator and enforced by CheckAnalysis.md#deterministic[CheckAnalysis].

[[output]]
The catalyst/QueryPlan.md#output[output schema] of a `Project` is...FIXME

[[maxRows]]
`maxRows`...FIXME

[[resolved]]
`resolved`...FIXME

[[validConstraints]]
`validConstraints`...FIXME

[TIP]
====
Use `select` operator from [Catalyst DSL](../catalyst-dsl/index.md) to create a `Project` logical operator, e.g. for testing or Spark SQL internals exploration.

[source, scala]
----
import org.apache.spark.sql.catalyst.dsl.plans._  // <-- gives table and select
import org.apache.spark.sql.catalyst.dsl.expressions.star
val plan = table("a").select(star())
scala> println(plan.numberedTreeString)
00 'Project [*]
01 +- 'UnresolvedRelation `a`
----
====
