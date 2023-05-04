
## Flink基本架构

Flink 的 Master 节点包含了三个组件: Dispatcher、ResourceManager 和 JobManager。

- Dispatcher: 负责接收用户提供的作业，并且负责为这个新提交的作业拉起一个新的 JobManager 服务；
- ResourceManager: 负责资源的管理，在整个 Flink 集群中只有一个 ResourceManager，资源相关的内容都由这个服务负责；
- JobManager: 负责管理具体某个作业的执行，在一个 Flink 集群中可能有多个作业同时执行，每个作业都会有自己的 JobManager 服务。

![pic](https://pan.zeekling.cn/flink/basic/00000.png)

当用户开始提交一个作业，首先会将用户编写的代码转化为一个JobGraph。

- Standalone 这种 Session 模式（对于 YARN 模式来说），这种情况下 Client 可以直接与 Dispatcher 建立连接并提交作业；
-  Per-Job 模式，这种情况下 Client 首先向资源管理系统 （如 Yarn）申请资源来启动 ApplicationMaster，然后再向 
ApplicationMaster 中的 Dispatcher 提交作业。


当作业到 Dispatcher 后，Dispatcher 会首先启动一个 JobManager 服务，然后 JobManager 会向 ResourceManager 申请资
源来启动作业中具体的任务。ResourceManager 选择到空闲的 Slot （Flink 架构-基本概念）之后，就会通知相应的 TM 将
该 Slot 分配给指定的 JobManager。

## Master 启动整体流程


Flink 集群 Master 节点在初始化时，会先调用 ClusterEntrypoint 的 runClusterEntrypoint() 方法启动集群，其整体流程如下图所示：


![pic](https://pan.zeekling.cn/flink/basic/Flink%20Master%E8%AF%A6%E8%A7%A30001.png)


