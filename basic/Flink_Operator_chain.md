
## 简介

Operator Chain 字面意思就是操作链，在Flink中就是将满足一定条件的Operator放到一个算子里面执行，这样就能有效减少
算子之间的数据传输，从而达到作业性能优化的目的。

例如常见的算子：`source->map->filter` 可以合并到一个算子里面。

## Operator Chain

### 条件

- 下游节点的入度为1.
- 上下游节点都在同一个slot group。
- 前后算子不为空。
- 上游节点的chain策略为ALWAYS或HEAD（只能与下游下游链接，不能和下游链接，Source默认为HEAD）
- 下游节点和chain策略为ALWAYS（可以与下游链接，map、flatmap、filter等默认是ALWAYS）
- 两个节点之间的物理分区逻辑是ForwardPartitioner
- 两个算子之间的shuffle方式不等于批处理模式。
- 上下游并行度一致。
- 用户没有禁用chain: `stream.isChainingEnabled()`或者配置为：`pipeline.operator-chaining=true`


