
# 简介

## 时间区间Join左右时间一致

![时间区间Join](https://pan.zeekling.cn/flink/join/%E6%97%B6%E9%97%B4%E5%8C%BA%E9%97%B4join_001.png)

上图只能Join前后十分钟的数据。

![Join结果](https://pan.zeekling.cn/flink/join/%E6%97%B6%E9%97%B4%E5%8C%BA%E9%97%B4join_002.png)

如上图所示，表payment_flow中09:50的数据只能和表user_order中09:40到10:00之间的数据做时间区间Join。不在这个时间
区间内的数据关联不上。

## 时间区间左右时间不一致

![时间区间](https://pan.zeekling.cn/flink/join/%E6%97%B6%E9%97%B4%E5%8C%BA%E9%97%B4join_004.png)

Join示例

![join](https://pan.zeekling.cn/flink/join/%E6%97%B6%E9%97%B4%E5%8C%BA%E9%97%B4join_005.png)



# 执行流程


![执行流程](https://pan.zeekling.cn/flink/join/%E6%97%B6%E9%97%B4%E5%8C%BA%E9%97%B4join_003.png)


代码实现：TimeIntervalJoin.java

函数processElement1 左流处理。

函数processElement2 右流处理。


# 相关参数

`table.exec.source.idle-timeout`:

当一个source在超时时间内没有接收到任何元素时，它将被标记为临时空闲。这允许下游任务在空闲时提前其watermarks，而
无需等待来自该source的watermarks。



