# BasicWriteJobStatsTracker

`BasicWriteJobStatsTracker` is a concrete <<spark-sql-WriteJobStatsTracker.md#, WriteJobStatsTracker>>.

`BasicWriteJobStatsTracker` is <<creating-instance, created>> when <<spark-sql-LogicalPlan-DataWritingCommand.md#basicWriteJobStatsTracker, DataWritingCommand>> and Spark Structured Streaming's `FileStreamSink` are requested for one.

[[newTaskInstance]]
When requested for a new <<spark-sql-WriteJobStatsTracker.md#newTaskInstance, WriteTaskStatsTracker>>, `BasicWriteJobStatsTracker` creates a new <<spark-sql-BasicWriteTaskStatsTracker.md#, BasicWriteTaskStatsTracker>>.

=== [[creating-instance]] Creating BasicWriteJobStatsTracker Instance

`BasicWriteJobStatsTracker` takes the following when created:

* [[serializableHadoopConf]] Serializable Hadoop `Configuration`
* [[metrics]] Metrics (`Map[String, SQLMetric]`)
