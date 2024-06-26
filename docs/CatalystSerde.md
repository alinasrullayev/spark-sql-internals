# CatalystSerde Utility

## <span id="deserialize"> Creating Deserializer

```scala
deserialize[T : Encoder](
  child: LogicalPlan): DeserializeToObject
```

`deserialize` creates a [DeserializeToObject](logical-operators/DeserializeToObject.md) logical operator for the given `child` [logical plan](logical-operators/LogicalPlan.md).

---

Internally, `deserialize` creates an `UnresolvedDeserializer` for the deserializer for the type `T` first and passes it on to a `DeserializeToObject` with an `AttributeReference` (being the result of [generateObjAttr](#generateObjAttr)).

---

`deserialize` is used when:

* `Dataset` is requested for a [QueryExecution](dataset/index.md#rddQueryExecution)
* `ExpressionEncoder` is requested to [resolveAndBind](ExpressionEncoder.md#resolveAndBind)
* `MapPartitions` utility is used to [apply](logical-operators/MapPartitions.md#apply)
* `MapElements` utility is used to `apply`
* ([Catalyst DSL](catalyst-dsl/DslLogicalPlan.md)) [deserialize](catalyst-dsl/DslLogicalPlan.md#deserialize) operator is used

## <span id="serialize"> Creating Serializer

```scala
serialize[T : Encoder](
  child: LogicalPlan): SerializeFromObject
```

`serialize` tries to find the [ExpressionEncoder](ExpressionEncoder.md) for the type `T` and requests it for [serializer expressions](ExpressionEncoder.md#namedExpressions).

In the end, creates a `SerializeFromObject` logical operator with the serializer and the given `child` logical operator.
