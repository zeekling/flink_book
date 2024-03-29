
# 简介

<p align="center"><a title="flink book" target="_blank" href="https://github.com/zeekling/flink_book"><img src="https://img.shields.io/github/last-commit/zeekling/flink_book.svg?style=flat-square&color=FF9900"></a>
<a title="GitHub repo size in bytes" target="_blank" href="https://github.com/zeekling/flink_book"><img src="https://img.shields.io/github/repo-size/zeekling/flink_book.svg?style=flat-square"></a>
<a title="Hits" target="_blank" href="https://github.com/zeekling/hits"><img src="https://hits.b3log.org/zeekling/flink_book.svg"></a></p>

本项目用于学习Flink所做的笔记，便于以后查看复习。此项目适合我这种虽然比较菜，且爱学习的人学习。

# 内存调优

## 目录

- [内存等资源调优](./调优/Resource.md)
- [状态和CheckPoint 调优](./调优/CheckPoint.md)
- [如何分析及处理 Flink 反压](./调优/backpress.md)
- [Flink SQL 调优](./调优/flinkSql.md)

# 常见故障排除

## 目录

- [常见问题总结](./常见问题)


# Flink SQL 

Flink SQL学习笔记提纲。持续更新。Hive SQL 离线Join VS Flink SQL 双流Join。

| 对比维度 | Hive SQL离线Join | Flink SQL双流Join |
| ---| ----|----|
| 数据源 | 有界(离线数据)| 无界(实时数据)  |
| 计算次数 | 一次 | 持续计算 |
| 计算结果 | 有界(离线数据)  | 无界(实时数据) |
| 计算驱动 | 单边驱动  | 双边驱动  |

## 目录

- [Flink SQL双流Join底层原理](./Flink_SQL/双流Join底层原理.md)
- [时间区间Join](./Flink_SQL/时间区间Join.md)

# 基础知识

## Flink基础知识

- [Flink CEP](./basic/CEP.md)
- [旁路输出](./basic/旁路输出.md)
- [Flink Operator Chain](./basic/Flink_Operator_chain.md)
- [slot相关](./basic/slot相关.md)
- [Checkpoint相关](./basic/checkpoint.md)
- [Flink基本架构](./basic/Flink基本架构.md)
- [SavePoint相关](./basic/savepoint.md)
- [BlobServer相关](./basic/blobServer.md)
- [RocksDB相关](./rocksdb/README.md)


## Flink On Hudi

- [Flink On Hudi 简介](./hudi/README.md)


# Flink 源码

源码编译可以使用下面命令

```sh
mvn install -DskipTests -Dfast -Dpmd.skip=true -Dcheckstyle.skip=true \
-Dmaven.javadoc.skip=true -Dmaven.compile.fork=true

```

缺的包可以在 https://conjars.org/repo/org/pentaho/pentaho-aggdesigner-core/5.1.5-jhyde/

https://packages.confluent.io/maven/io/confluent/kafka-schema-registry-client/7.2.2/ 下面找到。

## 目录

- [作业提交流程](./source_code/作业提交.md)
- [Flink组件间通信](./source_code/Flink组件通信.md)
- [per-job模式启动流程](./source_code/per-job启动.md)
- [yarn-application模式启动](./source_code/application启动.md)
- [yarn-session启动](./source_code/yarn-session启动.md)

