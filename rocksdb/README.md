## 简介

RocksDB是一个高性能、可扩展、嵌入式、持久化、可靠、易用和可定制的键值存储库。它采用LSM树数据结构，支持高吞吐量的写入和快速的范围查询，可被嵌入到应用程序中，实现持久化存储，支持水平扩展，可以在多台服务器上部署，实现集群化存储，具有高度的可靠性和稳定性，易于使用并可以根据需求进行定制和优化。RocksDB主要使用到了下面知识：

## LSM树

LSM树全称Log-Structured Merge Tree，是一种数据结构，常用于键值存储系统中。LSM树的优点是可以支持高吞吐量的写入，具有良好的性能和可扩展性，并且可以在磁盘上存储大量的数据。但是，由于需要定期进行合并操作，因此对查询性能和磁盘空间的使用可能会造成一定的影响。为了解决这个问题，LSM树还有许多优化，如Bloom Filter、Compaction等，可以进一步提高查询性能和减少磁盘空间的使用。

![pic](https://pan.zeekling.cn//flink/basic/state/rocksdb_0001.png)

### LSM的组成

LSM树中的层级可以分为内存和磁盘两个部分，具体分层如下：

- 内存层：内存层也被称为MemTable，是指存储在内存中的数据结构，用于缓存最新写入的数据。当数据写入时，先将其存储到MemTable中，然后再将MemTable中的数据刷写到磁盘中，生成一个新的磁盘文件。由于内存读写速度非常快，因此使用MemTable可以实现高吞吐量的写入操作。
- 磁盘层：磁盘层是指存储在磁盘中的数据文件，可以分为多个层级。一般来说，LSM树中的磁盘层可以分为以下几个层级：
  - Level-0: 是最底层的磁盘层，存储的是从内存层写到磁盘中的文件。Level-0的文件一般比较小，按照写入顺序排序。由于要保证写入速度很快，因此Level-0中的文件数量较多。
  - Level-1: 是Level-0的上一层，存储的是由多个Level-0的文件合并而来，Level-1中的文件一般比较大，按照键值排序。由于Level-0中的文件较多，因此Level-1中的文件也是比较多。
  - Level-2以上：Level-2以上的磁盘层数都是由更底层级别的文件合并而来的文件，文件大小逐渐增大，排序方式也逐渐趋向于按照键值排序。由于每个层级的文件大小和排序方式不同，因此可以根据查询的需求，会选择更适合的层级进行查询，从而提高查询效率。

LSM树的内存层和磁盘层之间存在多层级的分层结构，可以通过不同文件大小和排序方式，满足不同的查询需求。通过分层的方式，LSM树能够高效的进行写入操作，并且能够快速定位到所需要的数据。

### Memtable

Memtable是存储在内存中的数据结构，用于缓存最新写入的数据。当数据写入时，先将其存储到Memtable中，然后再将Memtable中的数据刷新到磁盘当中，生成一个新的磁盘文件。

Memtable一般采用的数据结构有有序数组、有序链表、hash表、跳表、B树，由于存储在内存中，因此读写速度非常快，支持快速高吞吐量的写入操作。

当数据达到一定量时，需要将数据刷新到磁盘当中，生成一个新的磁盘文件，Flush操作会将Memtable的所有数据按照键的大小排序，并写入到磁盘当中。

为了减少Flush操作带来的影响，通常会设置多个Memtable，当一个Memtable中的数量达到一定大小时，就将其刷写到磁盘中，并将其替换成一个新的MemTable。这个过程被称为“Compaction”。Compaction操作会将多个磁盘文件合并成一个新的磁盘文件，从而减少磁盘文件的数量，提高读取性能。在Compaction操作中，也会同时将多个MemTable合并到一起，生成一个新的MemTable，从而减少Flush操作的频率，提高写入性能。

### Immutable MemTable

Immutable MemTable是指已经被刷写到磁盘中的、不可修改的MemTable。当一个MemTable达到一定的大小后，会被Flush到磁盘中，生成一个新的SSTable文件。同时将该MemTable标记为Immutable MemTable。

在LSM树的Compaction过程中，多个Immutable MemTable会被合并成一个新的SSTable文件。Compaction操作也会将多个SSTable文件合并成一个新的SSTable文件，并将其中的重复数据进行去重。因为Immutable MemTable是只读的，所以它们在Compaction过程中是不会被修改的，这样就可以避免数据冲突和一致性问题。

### SSTable(Sorted String Table)

SSTable是LSM树中的一种数据存储结构，用于存储已经被flush到磁盘的Immutable MemTable数据。它的特点是数据按照key有序存储，并且支持快速的范围查询和迭代访问。

SSTable是由多个数据块（Data Block）和一个索引块（Index Block）组成。数据块中存储着按照key有序排列的数据，索引块中存储着数据块的位置和对应的key。

SSTable中的数据块采用了一些压缩算法，例如LZ4、Snappy等，可以有效地压缩数据，减少磁盘存储空间。同时，SSTable还支持Bloom Filter等数据结构，可以提高查询的效率。

SSTable是LSM树中非常重要的一种数据存储结构，通过有序的存储方式和快速的索引访问方式，提高了查询性能和存储空间的利用率。

![pic](https://pan.zeekling.cn//flink/basic/state/rocksdb_0002.png)



### Compaction

在LSM树中，数据的更新是通过追加日志形式完成的。这种追加方式使得LSM树可以顺序写，避免了频繁的随机写，从而提高了写性能。

在LSM树中，数据被存储在不同的层次中，每个层次对应一组SSTable文件。当MemTable中的数据达到一定的大小时，会被刷写（flush）到磁盘上，生成一个新的SSTable文件。这种以追加式的更新方式会导致数据冗余的问题。需要定期进行SSTable的合并（Compaction）操作，将不同的SSTable文件中相同Key的数据进行合并，并将旧版本的数据删除，从而减少冗余数据的存储空间。

数据在LSM树中存储的方式，读取时需要从最新的SSTable文件开始倒着查询，直到找到需要的数据。这种倒着查询的方式会降低读取性能，尤其是在存在大量SSTable文件的情况下。为了提高读取性能，LSM树通常会采用一些技术，例如索引和布隆过滤器来优化查询速度，减少不必要的磁盘访问。

## 压缩

LSM树压缩策略需要围绕三个问题进行考量：

- 读放大：在读取数据时，需要读取的数据量大于实际的数据量。在LSM树中，需要先在MemTable中查看是否存在该key，如果不存在，则需要继续在SSTable中查找，直到找到为止。如果数据被分散在多个SSTable中，则需要遍历所有的SSTable，这就导致了读放大。如果数据分布比较均匀，则读放大不会很严重，但如果数据分布不均，则可能需要遍历大量的SSTable才能找到目标数据。
- 写放大：在写入数据时，实际写入的数据量大于真正的数据量。在LSM树中写入数据时可能会触发Compact操作，这会导致一些SSTable中的冗余数据被清理回收，但同时也会产生新的SSTable，因此实际写入的数据量可能远大于该key的数据量。
- 空间放大：数据实际占用的磁盘空间比数据的真正大小更多。在LSM树中，由于数据的更新是以日志形式进行的，因此同一个key可能在多个SSTable中都存在，而只有最新的那条记录是有效的，之前的记录都可以被清理回收。这就导致了空间的浪费，也就是空间放大。



### size-tiered 策略

Size-tiered策略是一种常用的Compaction策略。它可以有效地减少SSTable的数量和大小，降低查询时的磁盘读取次数和延迟，提高LSM树的查询性能和空间利用率。

- 统计每个层级中的SSTable数量和总大小。当某个层级中的SSTable数量达到预设的阈值N后，就会触发Compaction操作。
- 将该层级中的所有SSTable按照大小分成若干组。每组的大小大致相等。
- 对于每组SSTable，选择一个合适的合并策略。常用的合并策略包括两两合并（Two-Level Merge）、级联合并（Cascade Merge）和追加合并（Append Merge）等。
- 执行合并操作，将同一组中的SSTable合并为一个更大的SSTable，并将合并后的结果写入到下一层级的队尾。这样可以保持每个层级中的SSTable大小相近，从而减少后续Compaction操作的成本。
- 更新索引和元数据信息，记录新生成的SSTable的位置、大小和版本号等信息，以便后续的查询和Compaction操作。
- 删除原有的SSTable文件，释放磁盘空间。如果需要保留一定数量的历史版本，则可以将旧的SSTable文件移动到历史版本目录中，以便后续的查询和回滚操作。

![pic](https://pan.zeekling.cn//flink/basic/state/rocksdb_0003.png)



### leveled 策略

Leveled策略是一种基于有序SSTable的高效Compaction策略。它可以有效地减小空间放大和读放大问题，提高LSM树的查询性能和空间利用率。

当一个 SSTable 中的数据量达到一定大小时，它就会被合并到上一层，这个过程被称为 L0 合并（Level 0 Merge）。在 L0 合并时，相邻的 SSTable 会被合并成一个更大的 SSTable，这样可以减少 SSTable 的数量，降低查询时需要扫描的 SSTable 的数量，从而提高查询效率。

在 L0 合并完成之后，新生成的 SSTable 会被插入到第 1 层，如果第 1 层的 SSTable 数量超过了限制，那么就会进行 L1 合并，将相邻的 SSTable 合并成一个更大的 SSTable，同样的过程会在第 2 层、第 3 层等等一直进行下去，直到最高层。

当进行查询时，LSM 树会从最底层开始查找，如果在当前层的 SSTable 中找不到需要的数据，就会往上一层查找，直到找到需要的数据或者到达最高层。由于每一层的 SSTable 都是有序的，因此可以使用二分查找等算法来加速查询。

![pic](https://pan.zeekling.cn//flink/basic/state/rocksdb_0004.png)



