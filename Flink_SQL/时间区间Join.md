
# 简介

时间区间Join语法：

![时间区间Join](https://pan.zeekling.cn/flink/akka/%E6%97%B6%E9%97%B4%E5%8C%BA%E9%97%B4join_001.png)

上图只能Join前后十分钟的数据。

![Join结果](https://pan.zeekling.cn/flink/akka/%E6%97%B6%E9%97%B4%E5%8C%BA%E9%97%B4join_002.png)

如上图所示，表payment_flow中09:50的数据只能和表user_order中09:40到10:00之间的数据做时间区间Join。不在这个时间
区间内的数据关联不上。

# 执行流程



