
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


## 适用场景

微批处理通过增加延迟换取高吞吐，如果有超低延迟的要求，不建议开启微批处理。通常对于聚合的场景，微批处理可以显
著的提升系统性能，建议开启。



