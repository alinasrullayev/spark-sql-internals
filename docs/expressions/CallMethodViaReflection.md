title: CallMethodViaReflection

# CallMethodViaReflection Expression

`CallMethodViaReflection` is an Expression.md[expression] that represents a static method call in Scala or Java using `reflect` and `java_method` functions.

NOTE: `reflect` and `java_method` functions are only supported in SparkSession.md#sql[SQL] and dataset/index.md#selectExpr[expression] modes.

.CallMethodViaReflection's DataType to JVM Types Mapping
[cols="1,2",options="header",width="100%"]
|===
| DataType
| JVM Type

| `BooleanType`
| `java.lang.Boolean` / `scala.Boolean`

| `ByteType`
| `java.lang.Byte` / `Byte`

| `ShortType`
| `java.lang.Short` / `Short`

| `IntegerType`
| `java.lang.Integer` / `Int`

| `LongType`
| `java.lang.Long` / `Long`

| `FloatType`
| `java.lang.Float` / `Float`

| `DoubleType`
| `java.lang.Double` / `Double`

| `StringType`
| `String`
|===

[source, scala]
----
import org.apache.spark.sql.catalyst.expressions.CallMethodViaReflection
import org.apache.spark.sql.catalyst.expressions.Literal
scala> val expr = CallMethodViaReflection(
     |   Literal("java.time.LocalDateTime") ::
     |   Literal("now") :: Nil)
expr: org.apache.spark.sql.catalyst.expressions.CallMethodViaReflection = reflect(java.time.LocalDateTime, now)
scala> println(expr.numberedTreeString)
00 reflect(java.time.LocalDateTime, now)
01 :- java.time.LocalDateTime
02 +- now

// CallMethodViaReflection as the expression for reflect SQL function
val q = """
  select reflect("java.time.LocalDateTime", "now") as now
  """
val plan = spark.sql(q).queryExecution.logical
// CallMethodViaReflection shows itself under "reflect" name
scala> println(plan.numberedTreeString)
00 Project [reflect(java.time.LocalDateTime, now) AS now#39]
01 +- OneRowRelation$
----

`CallMethodViaReflection` supports a Expression.md#CodegenFallback[fallback mode for expression code generation].

[[properties]]
.CallMethodViaReflection's Properties
[width="100%",cols="1,2",options="header"]
|===
| Property
| Description

| [[dataType]] `dataType`
| `StringType`

| [[deterministic]] `deterministic`
| Disabled (i.e. `false`)

| [[nullable]] `nullable`
| Enabled (i.e. `true`)

| [[prettyName]] `prettyName`
| `reflect`
|===

NOTE: `CallMethodViaReflection` is very similar to spark-sql-Expression-StaticInvoke.md[StaticInvoke] expression.
