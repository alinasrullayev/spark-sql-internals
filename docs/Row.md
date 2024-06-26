# Row

`Row` is a generic object that represents a table `row` (_record_).

`Row` is referred to as **Catalyst Row**.

<!---
## Review Me

with an ordered collection of fields that can be accessed by an <<apply-index, ordinal / an index>> (aka _generic access by ordinal_), a name (aka _native primitive access_) or using <<pattern-matching-on-row, Scala's pattern matching>>.

`Row` may have an optional <<schema, schema>>.

The traits of `Row`:

* `length` or `size` - `Row` knows the number of elements (columns).
* `schema` - `Row` knows the schema

`Row` belongs to `org.apache.spark.sql.Row` package.

[source, scala]
----
import org.apache.spark.sql.Row
----

=== [[field-access]][[get]][[apply-index]] Field Access by Index -- `apply` and `get` methods

Fields of a `Row` instance can be accessed by index (starting from `0`) using `apply` or `get`.

[source, scala]
----
scala> val row = Row(1, "hello")
row: org.apache.spark.sql.Row = [1,hello]

scala> row(1)
res0: Any = hello

scala> row.get(1)
res1: Any = hello
----

NOTE: Generic access by ordinal (using `apply` or `get`) returns a value of type `Any`.

=== [[getAs]] Get Field As Type -- `getAs` method

You can query for fields with their proper types using `getAs` with an index

[source, scala]
----
val row = Row(1, "hello")

scala> row.getAs[Int](0)
res1: Int = 1

scala> row.getAs[String](1)
res2: String = hello
----

[NOTE]
====
FIXME
[source, scala]
----
row.getAs[String](null)
----
====

=== [[schema]] Schema

A `Row` instance can have a schema defined.

NOTE: Unless you are instantiating `Row` yourself (using <<row-object, Row Object>>), a `Row` has always a schema.

!!! note
    It is [RowEncoder](RowEncoder.md) to take care of assigning a schema to a `Row` when `toDF` on a [Dataset](dataset/index.md) or when instantiating [DataFrame](DataFrame.md) through [DataFrameReader](DataFrameReader.md).

=== [[row-object]] Row Object

`Row` companion object offers factory methods to create `Row` instances from a collection of elements (`apply`), a sequence of elements (`fromSeq`) and tuples (`fromTuple`).

[source, scala]
----
scala> Row(1, "hello")
res0: org.apache.spark.sql.Row = [1,hello]

scala> Row.fromSeq(Seq(1, "hello"))
res1: org.apache.spark.sql.Row = [1,hello]

scala> Row.fromTuple((0, "hello"))
res2: org.apache.spark.sql.Row = [0,hello]
----

`Row` object can merge `Row` instances.

[source, scala]
----
scala> Row.merge(Row(1), Row("hello"))
res3: org.apache.spark.sql.Row = [1,hello]
----

It can also return an empty `Row` instance.

[source, scala]
----
scala> Row.empty == Row()
res4: Boolean = true
----

=== [[pattern-matching-on-row]] Pattern Matching on Row

`Row` can be used in pattern matching (since <<row-object, Row Object>> comes with `unapplySeq`).

[source, scala]
----
scala> Row.unapplySeq(Row(1, "hello"))
res5: Some[Seq[Any]] = Some(WrappedArray(1, hello))

Row(1, "hello") match { case Row(key: Int, value: String) =>
  key -> value
}
----
-->
