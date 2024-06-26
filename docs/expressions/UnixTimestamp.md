# UnixTimestamp

`UnixTimestamp` is a [binary](Expression.md#BinaryExpression) expression with [timezone](Expression.md#TimeZoneAwareExpression) support that represents [unix_timestamp](../standard-functions//datetime.md#unix_timestamp) function (and indirectly [to_date](../standard-functions//datetime.md#to_date) and [to_timestamp](../standard-functions//datetime.md#to_timestamp)).

```text
import org.apache.spark.sql.functions.unix_timestamp
val c1 = unix_timestamp()

scala> c1.explain(true)
unix_timestamp(current_timestamp(), yyyy-MM-dd HH:mm:ss, None)

scala> println(c1.expr.numberedTreeString)
00 unix_timestamp(current_timestamp(), yyyy-MM-dd HH:mm:ss, None)
01 :- current_timestamp()
02 +- yyyy-MM-dd HH:mm:ss

import org.apache.spark.sql.catalyst.expressions.UnixTimestamp
scala> c1.expr.isInstanceOf[UnixTimestamp]
res0: Boolean = true
```

<!---
## Review Me

NOTE: `UnixTimestamp` is `UnixTime` expression internally (as is `ToUnixTimestamp` expression).

[[inputTypes]][[dataType]]
`UnixTimestamp` supports `StringType`, [DateType](../types/DataType.md#DateType) and `TimestampType` as input types for a time expression and returns `LongType`.

```text
scala> c1.expr.eval()
res1: Any = 1493354303
```

[[formatter]]
`UnixTimestamp` uses `DateTimeUtils.newDateFormat` for date/time format (as Java's [java.text.DateFormat]({{ java.api }}/java/text/DateFormat.html)).
-->
