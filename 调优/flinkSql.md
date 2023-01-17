
<a title="Hits" target="_blank" href="https://github.com/zeekling/hits"><img src="https://hits.b3log.org/zeekling/flink_book.svg"></a>

# 设置空闲状态保留时间

不设置空闲状态保留时间会导致状态爆炸。

- FlinkSQL 的 regular join inner 、 left 、 right ），左右表的数据都会一直保存在状态里，不会清理！要么设置 
TTL ，要么使用 Flink SQL 的 interval join 。
- 使用 Top N 语法进行去重，重复数据的出现一般都位于特定区间内（例如一小时或一天内），过了这段时间之后，对应的
状态就不再需要了。


Flink SQL可以指定空闲状态（即未更新的状态）被保留的最小时间 当状态中某个 key对应的 状态未更新的时间达到阈值时，
该条状态被自动清理。

API 设置：

```java 
tableEnv.getConfig().setIdleStateRetention(Duration.ofHours(1));
```
配置参数设置：
```java 
Configuration configuration = tableEnv.getConfig().getConfiguration();
configuration.setString("table.exec.state.ttl", " 1 h" );
```

# 开启MiniBatch


MiniBatch是微批处理，原理是 缓存一定的数据后再触发处理，以减少对 State 的访问从而提升吞吐并减少数据的输出量。MiniBatch主要依靠在每个Task上注册的Timer线程来触发微批，需要消耗一定的线程调度性能。

## 开启方式

MiniBatch 默认关闭，开启方式如下:

```java 
Configuration configuration = tEnv.getConfig().getConfiguration();
configuration.setString(" table.exec.mini batch.enabled ", true);
configuration.setString(" table.exec.mini batch.allow latency ", 5 s);
configuration.setString(" table.exec.mini batch.size ", 20000);

```

- table.exec.mini batch.enabled: 开启 miniBatch的参数。
- table.exec.mini batch.allow latency： 批量输出的间隔时间。
- table.exec.mini batch.size： 防止 OOM 设置每个批次最多缓存数据的条数 ，可以设为2 万条。


注意：

- 目前上述样例中的key value 配置项仅被 Blink planner支持。
- 1.12 之前的版本有 bug ，开启 miniBatch ，不会清理过期状态，也就是说如果设置状态的 TTL ，无法清理过期状态。
1.12 版本才修复这个问题 。

参考ISSUE：https://issues.apache.org/jira/browse/FLINK_17096


## 适用场景

微批处理通过增加延迟换取高吞吐，如果有超低延迟的要求，不建议开启微批处理。通常对于聚合的场景，微批处理可以显
著的提升系统性能，建议开启。


# 开启 LocalGlobal

## 原理介绍

LocalGlobal优化将原先的 Aggregate 分成 Local+Global 两阶段聚合，即MapReduce 模型中的 Combine+Reduce 
处理模式。第一阶段在上游节点本地攒一批数据进行聚合（ localAgg ），并输出这次微批的增量值 A ccumulator ）。第
二阶段再将收到的 Accumulator 合并（ Merge ），得到最终的结果 GlobalAgg ）。


LocalGlobal本质上能够靠 LocalAgg 的聚合筛除部分倾斜数据，从而降低 GlobalAgg的热点，提升性能。结合下图理解 
LocalGlobal 如何解决数据倾斜的问题。

![pic](./flinksql0001.png)

- 未开启 LocalGlobal 优化，由于流中的数据倾斜， Key 为红色的聚合算子实例需要处理更多的记录，这就导致了热点问题。
- 开启 LocalGlobal 优化后，先进行本地聚合，再进行全局聚合。可大大减少 GlobalAgg的热点，提高性能。

## 开启方式

- LocalGlobal 优化需要先开启 MiniBatch ，依赖于 MiniBatch 的参数。
- table.optimizer.agg phase strategy : 聚合策略。默认 AUTO ，支持参数 AUTO 、TWO_PHASE( 使用 LocalGlobal 两阶
段聚合 、 ONE_PHASE( 仅使用 Global 一阶段聚合）。


