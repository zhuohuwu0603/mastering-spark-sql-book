title: SortMergeJoinExec

# SortMergeJoinExec Binary Physical Operator for Sort Merge Join

`SortMergeJoinExec` is a SparkPlan.md#BinaryExecNode[binary physical operator] to <<doExecute, execute>> a *sort merge join*.

`ShuffledHashJoinExec` is <<creating-instance, selected>> to represent a spark-sql-LogicalPlan-Join.md[Join] logical operator when [JoinSelection](../execution-planning-strategies/JoinSelection.md) execution planning strategy is executed for joins with <<leftKeys, left join keys>> that are <<orderable, orderable>>, i.e. that can be ordered (sorted).

[[orderable]]
[NOTE]
====
A join key is *orderable* when is of one of the following spark-sql-DataType.md[data types]:

* `NullType`
* spark-sql-DataType.md#AtomicType[AtomicType] (that represents all the available types except `NullType`, `StructType`, `ArrayType`, `UserDefinedType`, `MapType`, and `ObjectType`)
* `StructType` with orderable fields
* `ArrayType` of orderable type
* `UserDefinedType` of orderable type

Therefore, a join key is *not* orderable when is of the following data type:

* `MapType`
* `ObjectType`
====

[NOTE]
====
spark-sql-properties.md#spark.sql.join.preferSortMergeJoin[spark.sql.join.preferSortMergeJoin] is an internal configuration property and is enabled by default.

That means that [JoinSelection](../execution-planning-strategies/JoinSelection.md) execution planning strategy (and so Spark Planner) prefers sort merge join over [shuffled hash join](ShuffledHashJoinExec.md).
====

[[supportCodegen]]
`SortMergeJoinExec` supports [Java code generation](CodegenSupport.md) (aka _codegen_) for inner and cross joins.

[TIP]
====
Enable `DEBUG` logging level for `org.apache.spark.sql.catalyst.planning.ExtractEquiJoinKeys` logger to see the join condition and the left and right join keys.
====

[source, scala]
----
// Disable auto broadcasting so Broadcast Hash Join won't take precedence
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)

val tokens = Seq(
  (0, "playing"),
  (1, "with"),
  (2, "SortMergeJoinExec")
).toDF("id", "token")

// all data types are orderable
scala> tokens.printSchema
root
 |-- id: integer (nullable = false)
 |-- token: string (nullable = true)

// Spark Planner prefers SortMergeJoin over Shuffled Hash Join
scala> println(spark.conf.get("spark.sql.join.preferSortMergeJoin"))
true

val q = tokens.join(tokens, Seq("id"), "inner")
scala> q.explain
== Physical Plan ==
*(3) Project [id#5, token#6, token#10]
+- *(3) SortMergeJoin [id#5], [id#9], Inner
   :- *(1) Sort [id#5 ASC NULLS FIRST], false, 0
   :  +- Exchange hashpartitioning(id#5, 200)
   :     +- LocalTableScan [id#5, token#6]
   +- *(2) Sort [id#9 ASC NULLS FIRST], false, 0
      +- ReusedExchange [id#9, token#10], Exchange hashpartitioning(id#5, 200)
----

[[metrics]]
.SortMergeJoinExec's Performance Metrics
[cols="1,2,2",options="header",width="100%"]
|===
| Key
| Name (in web UI)
| Description

| [[numOutputRows]] `numOutputRows`
| number of output rows
|
|===

.SortMergeJoinExec in web UI (Details for Query)
image::images/spark-sql-SortMergeJoinExec-webui-query-details.png[align="center"]

NOTE: The prefix for variable names for `SortMergeJoinExec` operators in [CodegenSupport](CodegenSupport.md)-generated code is **smj**.

```text
scala> q.queryExecution.debug.codegen
Found 3 WholeStageCodegen subtrees.
== Subtree 1 / 3 ==
*Project [id#5, token#6, token#11]
+- *SortMergeJoin [id#5], [id#10], Inner
   :- *Sort [id#5 ASC NULLS FIRST], false, 0
   :  +- Exchange hashpartitioning(id#5, 200)
   :     +- LocalTableScan [id#5, token#6]
   +- *Sort [id#10 ASC NULLS FIRST], false, 0
      +- ReusedExchange [id#10, token#11], Exchange hashpartitioning(id#5, 200)

Generated code:
/* 001 */ public Object generate(Object[] references) {
/* 002 */   return new GeneratedIterator(references);
/* 003 */ }
/* 004 */
/* 005 */ final class GeneratedIterator extends org.apache.spark.sql.execution.BufferedRowIterator {
/* 006 */   private Object[] references;
/* 007 */   private scala.collection.Iterator[] inputs;
/* 008 */   private scala.collection.Iterator smj_leftInput;
/* 009 */   private scala.collection.Iterator smj_rightInput;
/* 010 */   private InternalRow smj_leftRow;
/* 011 */   private InternalRow smj_rightRow;
/* 012 */   private int smj_value2;
/* 013 */   private org.apache.spark.sql.execution.ExternalAppendOnlyUnsafeRowArray smj_matches;
/* 014 */   private int smj_value3;
/* 015 */   private int smj_value4;
/* 016 */   private UTF8String smj_value5;
/* 017 */   private boolean smj_isNull2;
/* 018 */   private org.apache.spark.sql.execution.metric.SQLMetric smj_numOutputRows;
/* 019 */   private UnsafeRow smj_result;
/* 020 */   private org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder smj_holder;
/* 021 */   private org.apache.spark.sql.catalyst.expressions.codegen.UnsafeRowWriter smj_rowWriter;
...
```

[[output]]
The catalyst/QueryPlan.md#output[output schema] of a `SortMergeJoinExec` is...FIXME

[[outputPartitioning]]
The SparkPlan.md#outputPartitioning[outputPartitioning] of a `SortMergeJoinExec` is...FIXME

[[outputOrdering]]
The SparkPlan.md#outputOrdering[outputOrdering] of a `SortMergeJoinExec` is...FIXME

[[requiredChildDistribution]]
The SparkPlan.md#requiredChildDistribution[partitioning requirements] of the input of a `SortMergeJoinExec` (aka _child output distributions_) are [HashClusteredDistributions](HashClusteredDistribution.md) of <<leftKeys, left>> and <<rightKeys, right>> join keys.

.SortMergeJoinExec's Required Child Output Distributions
[cols="1,1",options="header",width="100%"]
|===
| Left Child
| Right Child

| [HashClusteredDistribution](HashClusteredDistribution.md) (per <<leftKeys, left join key expressions>>)
| [HashClusteredDistribution](HashClusteredDistribution.md) (per <<rightKeys, right join key expressions>>)
|===

[[requiredChildOrdering]]
The SparkPlan.md#requiredChildOrdering[ordering requirements] of the input of a `SortMergeJoinExec` (aka _child output ordering_) is...FIXME

NOTE: `SortMergeJoinExec` operator is chosen in [JoinSelection](../execution-planning-strategies/JoinSelection.md) execution planning strategy (after [BroadcastHashJoinExec](BroadcastHashJoinExec.md) and [ShuffledHashJoinExec](ShuffledHashJoinExec.md) physical join operators have not met the requirements).

=== [[creating-instance]] Creating SortMergeJoinExec Instance

`SortMergeJoinExec` takes the following when created:

* [[leftKeys]] Left join key expressions/Expression.md[expressions]
* [[rightKeys]] Right join key expressions/Expression.md[expressions]
* [[joinType]] spark-sql-joins.md#join-types[Join type]
* [[condition]] Optional join condition expressions/Expression.md[expression]
* [[left]] Left SparkPlan.md[physical operator]
* [[right]] Right SparkPlan.md[physical operator]
