

Flink 为作业的容错提供 Checkpoint 和 Savepoint 两种机制。保存点机制（Savepoints）是检查点机制一种特殊的实现，它允许你通过手工方式来触发Checkpoint，并将结果持久化存储到指定路径中，主要用于避免Flink集群在重启或升级时导致状态丢失。

![pic](https://pan.zeekling.cn//flink/basic/state/savepoint_0001.png)

 Savepoint 是一种特殊的 Checkpoint，实际上它们的存储格式也是一致的，它们主要的不同在于定位。Checkpoint机制的目标在于保证Flink作业意外崩溃重启不影响exactly once准确性，通常是配合作业重启策略使用的。Checkpoint 是为 Flink runtime 准备的，Savepoint 是为 Flink 用户准备的。因此 Checkpoint 是由 Flink runtime 定时触发并根据运行配置自动清理的，一般不需要用户介入，而 Savepoint 的触发和清理都由用户掌控。

由于 Checkpoint 的频率远远大于 Savepoint，Flink 对 Checkpoint 格式进行了针对不同 StateBackend 的优化，因此它在底层储存效率更高，而代价是耦合性更强，比如不保证 扩容 （即改变作业并行度）的特性和跨版本兼容。

Savepoint 是全量的，不支持增量的。因为 Checkpoint 是秒级频繁触发的，两个连续 Checkpoint 通常高度相似，因此对于 State 特别大的作业来说，每次 Checkpoint 只增量地补充 diff 可以大大地节约成本，这就是 incremental Checkpoint 的由来。而 Savepoint 并不会连续地触发，而且比起效率，Savepoint 更关注的是可移植性和版本兼容性。





