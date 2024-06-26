---
title: DataSourceV2ScanExecBase
---

# DataSourceV2ScanExecBase Leaf Physical Operators

`DataSourceV2ScanExecBase` is an [extension](#contract) of [LeafExecNode](SparkPlan.md#LeafExecNode) abstraction for [leaf physical operators](#implementations) that [track number of output rows](#metrics) when executed ([with](#doExecuteColumnar) or [without](#doExecute) support for [columnar reads](#supportsColumnar)).

## Contract

### Input Partitions { #inputPartitions }

```scala
partitions: Seq[InputPartition]
```

See:

* [BatchScanExec](BatchScanExec.md#partitions)

Used when:

* `DataSourceV2ScanExecBase` is requested for the [partitions](#partitions), [groupedPartitions](#groupedPartitions), [supportsColumnar](#supportsColumnar)

### Input RDD { #inputRDD }

```scala
inputRDD: RDD[InternalRow]
```

See:

* [BatchScanExec](BatchScanExec.md#inputRDD)

### keyGroupedPartitioning { #keyGroupedPartitioning }

```scala
keyGroupedPartitioning: Option[Seq[Expression]]
```

Optional partitioning [expression](../expressions/Expression.md)s

See:

* [BatchScanExec](BatchScanExec.md#keyGroupedPartitioning)

Used when:

* `DataSourceV2ScanExecBase` is requested to [groupedPartitions](#groupedPartitions), [groupPartitions](#groupPartitions), [outputPartitioning](#outputPartitioning)

### PartitionReaderFactory { #readerFactory }

```scala
readerFactory: PartitionReaderFactory
```

[PartitionReaderFactory](../connector/PartitionReaderFactory.md) for partition readers (of the [input partitions](#inputPartitions))

See:

* [BatchScanExec](BatchScanExec.md#readerFactory)

Used when:

* `BatchScanExec` physical operator is requested for an [input RDD](BatchScanExec.md#inputRDD)
* `ContinuousScanExec` and `MicroBatchScanExec` physical operators (Spark Structured Streaming) are requested for an `inputRDD`
* `DataSourceV2ScanExecBase` physical operator is requested to [outputPartitioning](#outputPartitioning) or [supportsColumnar](#supportsColumnar)

### Scan

```scala
scan: Scan
```

[Scan](../connector/Scan.md)

## Implementations

* [BatchScanExec](BatchScanExec.md)
* `ContinuousScanExec` ([Spark Structured Streaming]({{ book.structured_streaming }}/physical-operators/ContinuousScanExec))
* `MicroBatchScanExec` ([Spark Structured Streaming]({{ book.structured_streaming }}/physical-operators/MicroBatchScanExec))

## <span id="doExecute"> Executing Physical Operator

```scala
doExecute(): RDD[InternalRow]
```

`doExecute` is part of the [SparkPlan](SparkPlan.md#doExecute) abstraction.

---

`doExecute`...FIXME

## <span id="doExecuteColumnar"> doExecuteColumnar

```scala
doExecuteColumnar(): RDD[ColumnarBatch]
```

`doExecuteColumnar` is part of the [SparkPlan](SparkPlan.md#doExecuteColumnar) abstraction.

---

`doExecuteColumnar`...FIXME

## <span id="metrics"> Performance Metrics

```scala
metrics: Map[String, SQLMetric]
```

`metrics` is part of the [SparkPlan](SparkPlan.md#metrics) abstraction.

---

`metrics` is the following [SQLMetric](../SQLMetric.md)s with the [customMetrics](#customMetrics):

Metric Name | web UI
------------|--------
 `numOutputRows` | number of output rows

## <span id="outputPartitioning"> Output Data Partitioning Requirements

```scala
outputPartitioning: physical.Partitioning
```

`outputPartitioning` is part of the [SparkPlan](SparkPlan.md#outputPartitioning) abstraction.

---

`outputPartitioning`...FIXME

## <span id="simpleString"> Simple Node Description

```scala
simpleString(
    maxFields: Int): String
```

`simpleString` is part of the [TreeNode](../catalyst/TreeNode.md#simpleString) abstraction.

---

`simpleString`...FIXME

## <span id="supportsColumnar"> supportsColumnar

```scala
supportsColumnar: Boolean
```

`supportsColumnar` is part of the [SparkPlan](SparkPlan.md#supportsColumnar) abstraction.

---

`supportsColumnar` is `true` if the [PartitionReaderFactory](#readerFactory) can [supportColumnarReads](../connector/PartitionReaderFactory.md#supportColumnarReads) for all the [input partitions](#inputPartitions). Otherwise, `supportsColumnar` is `false`.

---

`supportsColumnar` makes sure that either all the [input partitions](#inputPartitions) are [supportColumnarReads](../connector/PartitionReaderFactory.md#supportColumnarReads) or none, or throws an `IllegalArgumentException`:

```text
Cannot mix row-based and columnar input partitions.
```

## <span id="customMetrics"> Custom Metrics

```scala
customMetrics: Map[String, SQLMetric]
```

??? note "Lazy Value"
    `customMetrics` is a Scala **lazy value** to guarantee that the code to initialize it is executed once only (when accessed for the first time) and the computed value never changes afterwards.

    Learn more in the [Scala Language Specification]({{ scala.spec }}/05-classes-and-objects.html#lazy).

`customMetrics` requests the [Scan](#scan) for [supportedCustomMetrics](../connector/Scan.md#supportedCustomMetrics) that are then converted to [SQLMetric](../SQLMetric.md)s.

---

`customMetrics` is used when:

* `DataSourceV2ScanExecBase` is requested for the [performance metrics](#metrics)
* `BatchScanExec` is requested for the [inputRDD](BatchScanExec.md#inputRDD)
* `ContinuousScanExec` is requested for the `inputRDD`
* `MicroBatchScanExec` is requested for the `inputRDD` (that creates a [DataSourceRDD](../DataSourceRDD.md))

## <span id="verboseStringWithOperatorId"> verboseStringWithOperatorId

??? note "Signature"

    ```scala
    verboseStringWithOperatorId(): String
    ```

    `verboseStringWithOperatorId` is part of the [QueryPlan](../catalyst/QueryPlan.md#verboseStringWithOperatorId) abstraction.

`verboseStringWithOperatorId` requests the [Scan](#scan) for one of the following (`metaDataStr`):

* [Metadata](#getMetaData) when [SupportsMetadata](../connector/SupportsMetadata.md)
* [Description](../connector/Scan.md#description), otherwise

In the end, `verboseStringWithOperatorId` is as follows (based on [formattedNodeName](../catalyst/QueryPlan.md#formattedNodeName) and [output](../catalyst/QueryPlan.md#output)):

```text
[formattedNodeName]
Output: [output]
[metaDataStr]
```
