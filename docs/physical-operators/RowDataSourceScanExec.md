title: RowDataSourceScanExec

# RowDataSourceScanExec Leaf Physical Operator

`RowDataSourceScanExec` is a spark-sql-SparkPlan-DataSourceScanExec.md[DataSourceScanExec] (and so indirectly a SparkPlan.md#LeafExecNode[leaf physical operator]) for scanning data from a <<relation, BaseRelation>>.

`RowDataSourceScanExec` is <<creating-instance, created>> to represent a spark-sql-LogicalPlan-LogicalRelation.md[LogicalRelation] with the following scan types when [DataSourceStrategy](../execution-planning-strategies/DataSourceStrategy.md) execution planning strategy is executed:

* `CatalystScan`, `PrunedFilteredScan`, `PrunedScan` (indirectly when `DataSourceStrategy` is requested to [pruneFilterProjectRaw](../execution-planning-strategies/DataSourceStrategy.md#pruneFilterProjectRaw))

* `TableScan`

`RowDataSourceScanExec` marks the <<filters, filters>> that are included in the <<handledFilters, handledFilters>> with `*` (star) in the <<metadata, metadata>> that is used for a simple text representation.

[source, scala]
----
// DEMO RowDataSourceScanExec with a simple text representation with stars
----

=== [[doProduce]] Generating Java Source Code for Produce Path in Whole-Stage Code Generation -- `doProduce` Method

[source, scala]
----
doProduce(ctx: CodegenContext): String
----

`doProduce`...FIXME

`doProduce` is part of the [CodegenSupport](CodegenSupport.md#doProduce) abstraction.

=== [[creating-instance]] Creating RowDataSourceScanExec Instance

`RowDataSourceScanExec` takes the following when created:

* [[fullOutput]] Output schema spark-sql-Expression-Attribute.md[attributes]
* [[requiredColumnsIndex]] Indices of required columns
* [[filters]] spark-sql-Filter.md[Filter predicates]
* [[handledFilters]] Handled spark-sql-Filter.md[filter predicates]
* [[rdd]] RDD of spark-sql-InternalRow.md[internal binary rows]
* [[relation]] spark-sql-BaseRelation.md[BaseRelation]
* [[tableIdentifier]] `TableIdentifier`

NOTE: The input <<filters, filter predicates>> and <<handledFilters, handled filters predicates>> are used exclusively for the <<metadata, metadata>> property that is part of spark-sql-SparkPlan-DataSourceScanExec.md#metadata[DataSourceScanExec Contract] to describe a scan for a spark-sql-SparkPlan-DataSourceScanExec.md#simpleString[simple text representation (in a query plan tree)].

=== [[metadata]] `metadata` Property

[source, scala]
----
metadata: Map[String, String]
----

NOTE: `metadata` is part of spark-sql-SparkPlan-DataSourceScanExec.md#metadata[DataSourceScanExec Contract] to describe a scan for a spark-sql-SparkPlan-DataSourceScanExec.md#simpleString[simple text representation (in a query plan tree)].

`metadata` marks the <<filters, filter predicates>> that are included in the <<handledFilters, handled filters predicates>> with `*` (star).

NOTE: Filter predicates with `*` (star) are to denote filters that are pushed down to a relation (aka _data source_).

In the end, `metadata` creates the following mapping:

. *ReadSchema* with the <<output, output>> converted to [catalog representation](../StructType.md#catalogString)

. *PushedFilters* with the marked and unmarked <<filters, filter predicates>>
