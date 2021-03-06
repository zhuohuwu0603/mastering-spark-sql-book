# FileFormat &mdash; Data Sources to Read and Write Data In Files

`FileFormat` is the <<contract, contract>> for <<implementations, data sources>> that <<buildReader, read>> and <<prepareWrite, write>> data stored in files.

[[contract]]
.FileFormat Contract
[cols="1m,2",options="header",width="100%"]
|===
| Method
| Description

| buildReader
a| [[buildReader]]

[source, scala]
----
buildReader(
  sparkSession: SparkSession,
  dataSchema: StructType,
  partitionSchema: StructType,
  requiredSchema: StructType,
  filters: Seq[Filter],
  options: Map[String, String],
  hadoopConf: Configuration): PartitionedFile => Iterator[InternalRow]
----

Builds a Catalyst data reader, i.e. a function that reads a <<spark-sql-PartitionedFile.md#, PartitionedFile>> file as <<spark-sql-InternalRow.md#, InternalRows>>.

`buildReader` throws an `UnsupportedOperationException` by default (and should therefore be overriden to work):

```
buildReader is not supported for [this]
```

Used exclusively when `FileFormat` is requested to <<buildReaderWithPartitionValues, buildReaderWithPartitionValues>>

| <<buildReaderWithPartitionValues-internals, buildReaderWithPartitionValues>>
a| [[buildReaderWithPartitionValues]]

[source, scala]
----
buildReaderWithPartitionValues(
  sparkSession: SparkSession,
  dataSchema: StructType,
  partitionSchema: StructType,
  requiredSchema: StructType,
  filters: Seq[Filter],
  options: Map[String, String],
  hadoopConf: Configuration): PartitionedFile => Iterator[InternalRow]
----

`buildReaderWithPartitionValues` builds a data reader with partition column values appended, i.e. a function that is used to read a single file in (as a <<spark-sql-PartitionedFile.md#, PartitionedFile>>) as an `Iterator` of <<spark-sql-InternalRow.md#, InternalRows>> (like <<buildReader, buildReader>>) with the partition values appended.

Used exclusively when `FileSourceScanExec` physical operator is requested for the <<spark-sql-SparkPlan-FileSourceScanExec.md#inputRDD, inputRDD>> (when requested for the <<spark-sql-SparkPlan-FileSourceScanExec.md#inputRDDs, inputRDDs>> and <<spark-sql-SparkPlan-FileSourceScanExec.md#doExecute, execution>>)

| inferSchema
a| [[inferSchema]]

[source, scala]
----
inferSchema(
  sparkSession: SparkSession,
  options: Map[String, String],
  files: Seq[FileStatus]): Option[StructType]
----

Infers (returns) the [schema](StructType.md) of the given files (as Hadoop's https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/fs/FileStatus.html[FileStatuses]) if supported. Otherwise, `None` should be returned.

Used when:

* `HiveMetastoreCatalog` is requested to [inferIfNeeded](hive/HiveMetastoreCatalog.md#inferIfNeeded)
* `DataSource` is requested to [getOrInferFileFormatSchema](DataSource.md#getOrInferFileFormatSchema) and [resolveRelation](DataSource.md#resolveRelation)

| isSplitable
a| [[isSplitable]]

[source, scala]
----
isSplitable(
  sparkSession: SparkSession,
  options: Map[String, String],
  path: Path): Boolean
----

Controls whether the format (under the given path as Hadoop https://hadoop.apache.org/docs/current/api/org/apache/hadoop/fs/Path.html[Path]) can be split or not.

`isSplitable` is disabled (`false`) by default.

Used exclusively when `FileSourceScanExec` physical operator is requested to <<spark-sql-SparkPlan-FileSourceScanExec.md#createNonBucketedReadRDD, create an RDD for non-bucketed reads>> (when requested for the <<spark-sql-SparkPlan-FileSourceScanExec.md#inputRDD, inputRDD>> and neither the optional [bucketing specification](HadoopFsRelation.md#bucketSpec) of the <<spark-sql-SparkPlan-FileSourceScanExec.md#relation, HadoopFsRelation>> is defined nor [bucketing is enabled](SQLConf.md#bucketingEnabled))

| prepareWrite
a| [[prepareWrite]]

[source, scala]
----
prepareWrite(
  sparkSession: SparkSession,
  job: Job,
  options: Map[String, String],
  dataSchema: StructType): OutputWriterFactory
----

Prepares a write job and returns an `OutputWriterFactory`

Used when `FileFormatWriter` is used to [write out a query result](FileFormatWriter.md#write)

| supportBatch
a| [[supportBatch]]

[source, scala]
----
supportBatch(
  sparkSession: SparkSession,
  dataSchema: StructType): Boolean
----

Flag that says whether the format supports <<spark-sql-vectorized-parquet-reader.md#, vectorized decoding>> (aka _columnar batch_) or not.

Default: `false`

Used exclusively when `FileSourceScanExec` physical operator is requested for the <<spark-sql-SparkPlan-FileSourceScanExec.md#supportsBatch, supportsBatch>>

| vectorTypes
a| [[vectorTypes]]

[source, scala]
----
vectorTypes(
  requiredSchema: StructType,
  partitionSchema: StructType,
  sqlConf: SQLConf): Option[Seq[String]]
----

Defines the fully-qualified class names (_types_) of the concrete <<spark-sql-ColumnVector.md#, ColumnVectors>> for every column in the input `requiredSchema` and `partitionSchema` schemas that are used in a columnar batch.

Default: undefined (`None`)

Used exclusively when `FileSourceScanExec` leaf physical operator is requested for the <<spark-sql-SparkPlan-FileSourceScanExec.md#vectorTypes, vectorTypes>>
|===

[[implementations]]
.FileFormats (Direct Implementations and Extensions)
[width="100%",cols="1,2",options="header"]
|===
| FileFormat
| Description

| [AvroFileFormat](spark-sql-AvroFileFormat.md)
| [[AvroFileFormat]] Avro data source

| [HiveFileFormat](hive/HiveFileFormat.md)
| [[HiveFileFormat]] Writes hive tables

| [OrcFileFormat](spark-sql-OrcFileFormat.md)
| [[OrcFileFormat]] ORC data source

| [ParquetFileFormat](spark-sql-ParquetFileFormat.md)
| [[ParquetFileFormat]] Parquet data source

| [TextBasedFileFormat](TextBasedFileFormat.md)
| [[TextBasedFileFormat]] Base for text splitable `FileFormats`
|===

=== [[buildReaderWithPartitionValues-internals]] Building Data Reader With Partition Column Values Appended -- `buildReaderWithPartitionValues` Method

[source, scala]
----
buildReaderWithPartitionValues(
  sparkSession: SparkSession,
  dataSchema: StructType,
  partitionSchema: StructType,
  requiredSchema: StructType,
  filters: Seq[Filter],
  options: Map[String, String],
  hadoopConf: Configuration): PartitionedFile => Iterator[InternalRow]
----

`buildReaderWithPartitionValues` is simply an enhanced <<buildReader, buildReader>> that appends spark-sql-PartitionedFile.md#partitionValues[partition column values] to the internal rows produced by the reader function from <<buildReader, buildReader>>.

Internally, `buildReaderWithPartitionValues` <<buildReader, builds a data reader>> with the input parameters and gives a *data reader function* (of a spark-sql-PartitionedFile.md[PartitionedFile] to an `Iterator[InternalRow]`) that does the following:

. Creates a converter by requesting `GenerateUnsafeProjection` to spark-sql-GenerateUnsafeProjection.md#generate[generate an UnsafeProjection] for the attributes of the input `requiredSchema` and `partitionSchema`

. Applies the data reader to a `PartitionedFile` and converts the result using the converter on the joined row with the spark-sql-PartitionedFile.md#partitionValues[partition column values] appended.

NOTE: `buildReaderWithPartitionValues` is used exclusively when `FileSourceScanExec` physical operator is requested for the spark-sql-SparkPlan-FileSourceScanExec.md#inputRDDs[input RDDs].
