title: UnresolvedInlineTable

# UnresolvedInlineTable Logical Operator

`UnresolvedInlineTable` is a <<spark-sql-LogicalPlan.md#UnaryNode, unary logical operator>> that represents an inline table (aka _virtual table_ in Apache Hive).

`UnresolvedInlineTable` is <<creating-instance, created>> when `AstBuilder` is requested to <<spark-sql-AstBuilder.md#visitInlineTable, parse an inline table>> in a SQL statement.

[source, scala]
----
// `tableAlias` undefined so columns default to `colN`
val q = sql("VALUES 1, 2, 3")
scala> println(q.queryExecution.logical.numberedTreeString)
00 'UnresolvedInlineTable [col1], [List(1), List(2), List(3)]

// CreateNamedStruct with `tableAlias`
val q = sql("VALUES (1, 'a'), (2, 'b') AS t1(a, b)")
scala> println(q.queryExecution.logical.numberedTreeString)
00 'SubqueryAlias t1
01 +- 'UnresolvedInlineTable [a, b], [List(1, a), List(2, b)]
----

[[resolved]]
`UnresolvedInlineTable` is never <<spark-sql-LogicalPlan.md#resolved, resolved>> (and is converted to a <<spark-sql-LogicalPlan-LocalRelation.md#, LocalRelation>> in [ResolveInlineTables](../logical-analysis-rules/ResolveInlineTables.md) logical resolution rule).

[[output]]
`UnresolvedInlineTable` uses no <<catalyst/QueryPlan.md#output, output schema attributes>>.

[[expressionsResolved]]
`UnresolvedInlineTable` uses `expressionsResolved` flag that is on (`true`) only when all the Catalyst expressions in the <<rows, rows>> are <<expressions/Expression.md#resolved, resolved>>.

=== [[creating-instance]] Creating UnresolvedInlineTable Instance

`UnresolvedInlineTable` takes the following when created:

* [[names]] Column names
* [[rows]] Rows (as <<expressions/Expression.md#, Catalyst expressions>> for every row, i.e. `Seq[Seq[Expression]]`)
