
## 简介

Join优化可分为如下：

1. `OPTIMIZER_CHOOSES`：让优化器自己选择最优的方案。
2. `BROADCAST_HASH_FIRST`：适用于左表数据远小于右表数据，会将左表数据进行广播。
3. `BROADCAST_HASH_SECOND`： 适用于左表数据远大于右表数据，会将右表数据进行广播。
4. `REPARTITION_HASH_FIRST`：适用于左表数据比右表数据大一点点，会将左表和右表数据进行重分区，将左表进行hash。
5. `REPARTITION_HASH_SECOND`：适用于右表数据比左表数据大一点点，会将左表和右表数据进行重分区，将右表进行hash。
6. `REPARTITION_SORT_MERGE`： 将左表和右表数据进行重分区，并且进行排序合并。



