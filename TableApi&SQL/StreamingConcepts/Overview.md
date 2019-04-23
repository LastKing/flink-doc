# Streaming Concepts

Flink `Table Api` and `SQL support` 支持 batch 和 stream 处理的统一API。这意味着无论是`Table API`和`SQL`查询的是无边界输入还是有边界输入，它都拥有相同的语义。因为关系代数和SQL最开始都是为批处理而设计的，所以对无边界流输入的关系查询不如有边界的输入关系查询那样容易理解。

接下来的章节用来解释 Flink的流数据关系API概念、实际限制和流特定配置参数。

## Where to go next?
* Dynamic Tables: 描述动态表的概念
* Time attributes: 解释时间属性 并 如何在TableAPI和SQL中处理时间属性
* Joins in Continuous Queries: 在连续查询中支持的不同的 join 类型
* Temporal Tables: 描述时间表概念
* Query configuration: 列出 Table API & SQL 特定的配置项