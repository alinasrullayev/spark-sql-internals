---
title: Collection
---

# Standard Collection Functions

## filter { #filter }

```scala
filter(
  column: Column,
  f: (Column, Column) => Column): Column
filter(
  column: Column,
  f: Column => Column): Column
```

`filter` creates an [ArrayFilter](../expressions/ArrayFilter.md) expression (for the [expr](../Column.md#expr) of the given [Column](../Column.md) and a [LambdaFunction](#createLambda)).

In the end, `collect_set` wraps the `ArrayFilter` up in a [Column](../Column.md).

## str_to_map { #str_to_map }

```scala
str_to_map(
  text: Column): Column
str_to_map(
  text: Column,
  pairDelim: Column): Column
str_to_map(
  text: Column,
  pairDelim: Column,
  keyValueDelim: Column): Column
```

`str_to_map`...FIXME

<!---
## Review Me

| <<from_json, from_json>>
a|

[source, scala]
----
from_json(e: Column, schema: DataType): Column
from_json(e: Column, schema: DataType, options: Map[String, String]): Column
from_json(e: Column, schema: String, options: Map[String, String]): Column
from_json(e: Column, schema: StructType): Column
from_json(e: Column, schema: StructType, options: Map[String, String]): Column
----

Extract data from arbitrary JSON-encoded values into a [StructType](types/StructType.md) or [ArrayType](types/ArrayType.md) of `StructType` elements with the specified schema

| <<reverse-internals, reverse>>
a| [[reverse]]

[source, scala]
----
reverse(e: Column): Column
----

Returns a reversed string or an array with reverse order of elements

NOTE: Support for reversing arrays is *new in 2.4.0*.

| <<size-internals, size>>
a| [[size]]

[source, scala]
----
size(e: Column): Column
----

Returns the size of the given array or map. Returns -1 if `null`.

|===

=== [[size-internals]] `size` Collection Function

[source, scala]
----
size(e: Column): Column
----

`size` returns the size of the given array or map. Returns -1 if `null`.

Internally, `size` creates a `Column` with `Size` unary expression.

[source, scala]
----
import org.apache.spark.sql.functions.size
val c = size('id)
scala> println(c.expr.asCode)
Size(UnresolvedAttribute(ArrayBuffer(id)))
----

=== [[explode]] `explode` Collection Function

[source, scala]
----
scala> Seq(Array(0,1,2)).toDF("array").withColumn("num", explode('array)).show
+---------+---+
|    array|num|
+---------+---+
|[0, 1, 2]|  0|
|[0, 1, 2]|  1|
|[0, 1, 2]|  2|
+---------+---+
----

NOTE: `explode` function is an equivalent of dataset/index.md#flatMap[`flatMap` operator] for `Dataset`.

=== [[explode_outer]] `explode_outer` Collection Function

```scala
explode_outer(
  e: Column): Column
```

`explode_outer` generates a new row for each element in `e` array or map column.

!!! NOTE
    Unlike [explode](#explode), `explode_outer` generates `null` when the array or map is `null` or empty.

```text
val arrays = Seq((1,Seq.empty[String])).toDF("id", "array")
scala> arrays.printSchema
root
 |-- id: integer (nullable = false)
 |-- array: array (nullable = true)
 |    |-- element: string (containsNull = true)
scala> arrays.select(explode_outer($"array")).show
+----+
| col|
+----+
|null|
+----+
```

Internally, `explode_outer` creates a [Column](Column.md) with [GeneratorOuter](expressions/Generator.md#GeneratorOuter) and [Explode](expressions/Generator.md#Explode) Catalyst expressions.

```text
val explodeOuter = explode_outer($"array").expr
scala> println(explodeOuter.numberedTreeString)
00 generatorouter(explode('array))
01 +- explode('array)
02    +- 'array
```

=== [[from_json]] Extracting Data from Arbitrary JSON-Encoded Values -- `from_json` Collection Function

[source, scala]
----
from_json(e: Column, schema: StructType, options: Map[String, String]): Column // <1>
from_json(e: Column, schema: DataType, options: Map[String, String]): Column // <2>
from_json(e: Column, schema: StructType): Column // <3>
from_json(e: Column, schema: DataType): Column  // <4>
from_json(e: Column, schema: String, options: Map[String, String]): Column // <5>
----
<1> Calls <2> with `StructType` converted to `DataType`
<2> (fixme)
<3> Calls <1> with empty `options`
<4> Relays to the other `from_json` with empty `options`
<5> Uses schema as `DataType` in the JSON format or falls back to `StructType` in the DDL format

`from_json` parses a column with a JSON-encoded value into a [StructType](types/StructType.md) or [ArrayType](types/ArrayType.md) of `StructType` elements with the specified schema.

```text
val jsons = Seq("""{ "id": 0 }""").toDF("json")

import org.apache.spark.sql.types._
val schema = new StructType()
  .add($"id".int.copy(nullable = false))

import org.apache.spark.sql.functions.from_json
scala> jsons.select(from_json($"json", schema) as "ids").show
+---+
|ids|
+---+
|[0]|
+---+
```

[NOTE]
====
A schema can be one of the following:

* [DataType](types/DataType.md) as a Scala object or in the JSON format

* [StructType](types/StructType.md) in the DDL format
====

```text
// Define the schema for JSON-encoded messages
// Note that the schema is nested (on the addresses field)
import org.apache.spark.sql.types._
val addressesSchema = new StructType()
  .add($"city".string)
  .add($"state".string)
  .add($"zip".string)
val schema = new StructType()
  .add($"firstName".string)
  .add($"lastName".string)
  .add($"email".string)
  .add($"addresses".array(addressesSchema))
scala> schema.printTreeString
root
 |-- firstName: string (nullable = true)
 |-- lastName: string (nullable = true)
 |-- email: string (nullable = true)
 |-- addresses: array (nullable = true)
 |    |-- element: struct (containsNull = true)
 |    |    |-- city: string (nullable = true)
 |    |    |-- state: string (nullable = true)
 |    |    |-- zip: string (nullable = true)

// Generate the JSON-encoded schema
// That's the variant of the schema that from_json accepts
val schemaAsJson = schema.json

// Use prettyJson to print out the JSON-encoded schema
// Only for demo purposes
scala> println(schema.prettyJson)
{
  "type" : "struct",
  "fields" : [ {
    "name" : "firstName",
    "type" : "string",
    "nullable" : true,
    "metadata" : { }
  }, {
    "name" : "lastName",
    "type" : "string",
    "nullable" : true,
    "metadata" : { }
  }, {
    "name" : "email",
    "type" : "string",
    "nullable" : true,
    "metadata" : { }
  }, {
    "name" : "addresses",
    "type" : {
      "type" : "array",
      "elementType" : {
        "type" : "struct",
        "fields" : [ {
          "name" : "city",
          "type" : "string",
          "nullable" : true,
          "metadata" : { }
        }, {
          "name" : "state",
          "type" : "string",
          "nullable" : true,
          "metadata" : { }
        }, {
          "name" : "zip",
          "type" : "string",
          "nullable" : true,
          "metadata" : { }
        } ]
      },
      "containsNull" : true
    },
    "nullable" : true,
    "metadata" : { }
  } ]
}

// Let's "validate" the JSON-encoded schema
import org.apache.spark.sql.types.DataType
val dt = DataType.fromJson(schemaAsJson)
scala> println(dt.sql)
STRUCT<`firstName`: STRING, `lastName`: STRING, `email`: STRING, `addresses`: ARRAY<STRUCT<`city`: STRING, `state`: STRING, `zip`: STRING>>>

// No exception means that the JSON-encoded schema should be fine
// Use it with from_json
val rawJsons = Seq("""
  {
    "firstName" : "Jacek",
    "lastName" : "Laskowski",
    "email" : "jacek@japila.pl",
    "addresses" : [
      {
        "city" : "Warsaw",
        "state" : "N/A",
        "zip" : "02-791"
      }
    ]
  }
""").toDF("rawjson")
val people = rawJsons
  .select(from_json($"rawjson", schemaAsJson, Map.empty[String, String]) as "json")
  .select("json.*") // <-- flatten the struct field
  .withColumn("address", explode($"addresses")) // <-- explode the array field
  .drop("addresses")  // <-- no longer needed
  .select("firstName", "lastName", "email", "address.*") // <-- flatten the struct field
scala> people.show
+---------+---------+---------------+------+-----+------+
|firstName| lastName|          email|  city|state|   zip|
+---------+---------+---------------+------+-----+------+
|    Jacek|Laskowski|jacek@japila.pl|Warsaw|  N/A|02-791|
+---------+---------+---------------+------+-----+------+
```

!!! note
    `options` controls how a JSON is parsed and contains the same options as the [json](connectors/json/JsonDataSource.md) format.

Internally, `from_json` creates a [Column](Column.md) with [JsonToStructs](expressions/JsonToStructs.md) unary expression.

!!! NOTE
    `from_json` (creates a [JsonToStructs](expressions/JsonToStructs.md) that) uses a JSON parser in [FAILFAST](expressions/JsonToStructs.md#FAILFAST) parsing mode that simply fails early when a corrupted/malformed record is found (and hence does not support `columnNameOfCorruptRecord` JSON option).

```text
val jsons = Seq("""{ id: 0 }""").toDF("json")

import org.apache.spark.sql.types._
val schema = new StructType()
  .add($"id".int.copy(nullable = false))
  .add($"corrupted_records".string)
val opts = Map("columnNameOfCorruptRecord" -> "corrupted_records")
scala> jsons.select(from_json($"json", schema, opts) as "ids").show
+----+
| ids|
+----+
|null|
+----+
```

`from_json` corresponds to SQL's `from_json`.

=== [[array_contains]] `array_contains` Collection Function

```scala
array_contains(
  column: Column,
  value: Any): Column
```

`array_contains` creates a `Column` for a `column` argument as an [ArrayType](types/ArrayType.md) and the `value` of same type as the type of the elements of the array.

Internally, `array_contains` creates a [Column](Column.md#apply) with a `ArrayContains` expression.

```text
// Arguments must be an array followed by a value of same type as the array elements
import org.apache.spark.sql.functions.array_contains
val c = array_contains(column = $"ids", value = 1)

val ids = Seq(Seq(1,2,3), Seq(1), Seq(2,3)).toDF("ids")
val q = ids.filter(c)
scala> q.show
+---------+
|      ids|
+---------+
|[1, 2, 3]|
|      [1]|
+---------+
```

`array_contains` corresponds to SQL's `array_contains`.

```text
import org.apache.spark.sql.functions.array_contains
val c = array_contains(column = $"ids", value = Array(1, 2))
val e = c.expr
scala> println(e.sql)
array_contains(`ids`, [1,2])
```

!!! TIP
    Use SQL's `array_contains` to use values from columns for the `column` and `value` arguments.

```text
val codes = Seq(
  (Seq(1, 2, 3), 2),
  (Seq(1), 1),
  (Seq.empty[Int], 1),
  (Seq(2, 4, 6), 0)).toDF("codes", "cd")
scala> codes.show
+---------+---+
|    codes| cd|
+---------+---+
|[1, 2, 3]|  2|
|      [1]|  1|
|       []|  1|
|[2, 4, 6]|  0|
+---------+---+

val q = codes.where("array_contains(codes, cd)")
scala> q.show
+---------+---+
|    codes| cd|
+---------+---+
|[1, 2, 3]|  2|
|      [1]|  1|
+---------+---+

// array_contains standard function with Columns does NOT work. Why?!
// Asked this question on StackOverflow --> https://stackoverflow.com/q/50412939/1305344
val q = codes.where(array_contains($"codes", $"cd"))
scala> q.show
java.lang.RuntimeException: Unsupported literal type class org.apache.spark.sql.ColumnName cd
  at org.apache.spark.sql.catalyst.expressions.Literal$.apply(literals.scala:77)
  at org.apache.spark.sql.functions$.array_contains(functions.scala:3046)
  ... 50 elided

// Thanks Russel for this excellent "workaround"
// https://stackoverflow.com/a/50413766/1305344
import org.apache.spark.sql.Column
import org.apache.spark.sql.catalyst.expressions.ArrayContains
val q = codes.where(new Column(ArrayContains($"codes".expr, $"cd".expr)))
scala> q.show
+---------+---+
|    codes| cd|
+---------+---+
|[1, 2, 3]|  2|
|      [1]|  1|
+---------+---+
```
-->
