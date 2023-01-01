
<a title="Hits" target="_blank" href="https://github.com/zeekling/hits"><img src="https://hits.b3log.org/zeekling/redis_book.svg"></a>

# RocksDB 介绍

RocksDB 是嵌入式的 Key-Value 数据库，在 Flink 中被用作 RocksDBStateBackend 的底层存储。如下图所示，RocksDB 持久化的 SST
文件在本地文件系统上通过多个层级进行组织，不同层级之间会通过异步 Compaction 合并重复、过期和已删除的数据。在 RocksDB 的
写入过程中，数据经过序列化后写入到 WriteBuffer，WriteBuffer 写满后转换为 Immutable Memtable 结构，再通过 RocksDB 的
flush 线程从内存 flush 到磁盘上；读取过程中，会先尝试从 WriteBuffer 和 Immutable Memtable 中读取数据，如果没有找到，则会
查询 Block Cache，如果内存中都没有的话，则会按层级查找底层的 SST 文件，并将返回的结果所在的 Data Block 加载到 Block 
Cache，返回给上层应用。

![pic](./RocksDB001.png)


# RocksDBKeyedStateBackend增量快照介绍

这里介绍一下大家在大状态场景下经常需要调优的 RocksDBKeyedStateBackend 增量快照。RocksDB 具有 append-only 特性，Flink 利
用这一特性将两次 checkpoint 之间 SST 文件列表的差异作为状态增量上传到分布式文件系统上，并通过 JobMaster 中的 
SharedStateRegistry 进行状态的注册和过期。

![pic](./RocksDB002.png)

如上图所示，Task 进行了 3 次快照（假设作业设置保留最近 2 次 Checkpoint）：
- CP-1：RocksDB 产生 sst-1 和 sst-2 两个文件，Task 将文件上传至 DFS，JM 记录 sst 文件对应的引用计数
- CP-2：RocksDB 中的 sst-1 和 sst-2 通过 compaction 生成了 sst-1,2，并且新生成了 sst-3 文件，Task 将两个新增的文件上传
  至 DFS，JM 记录 sst 文件对应的引用计数
- CP-3：RocksDB 中新生成 sst-4 文件，Task 将增量的 sst-4 文件上传至 DFS，且在 CP-3 完成后，由于只保留最近 2 次 CP，
  JobMaster 将 CP-1 过期，同时将 CP-1 中的 sst 文件对应的引用计数减 1，并删除引用计数归 0 的 sst 文件（sst-1 和 sst-2）


增量快照涉及到 Task 多线程上传/下载增量文件，JobMaster 引用计数统计，以及大量与分布式文件系统的交互等过程，相对其他的 
StateBackend 要更为复杂，在 100+GB 甚至 TB 级别状态下，作业比较容易出现性能和稳定性瓶颈的问题。




# RocksDb大状态优化


截至当前，Flink 作业的状态后端仍然只有 Memory、FileSystem 和 RocksDB 三种可选，且 RocksDB 是
状态数据量较大（GB 到 TB 级别）时的选择。RocksDB 的性能发挥非常仰赖调优，如果全部采用默认配置，读写性能有可能会很差。

但是，RocksDB 的配置也是极为复杂的，可调整的参数多达百个，没有放之四海而皆准的优化方案。如果仅考虑 Flink 状态存储这一
方面，我们仍然可以总结出一些相对普适的优化思路。本文先介绍一些基础知识，再列举方法。



# CheckPoint设置



