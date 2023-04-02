
# 简介

该模式下，集群管理框架（例如YARN或Kubernetes）会为每个提交的作业启动一个单独Flink集群，该集群仅对该作业可用。
作业运行完成，集群将被关闭，所有用到的资源（例如文件）也将被清理。此模式提供更好的资源隔离。一个作业故障不会
影响其他作业的运行。此外，由于每个作业都有自己的JobManager，负载被分散到多个实体上。考虑到Session模式的资源隔
离问题，对于需要长时间运行的作业，通常选择Per-Job模式，这些作业愿意承受增加启动延迟以提升作业的恢复能力。


Per-Job模式为每个提交的作业启动一个集群，以提供更好的资源隔离保证。在这种情况下，集群的生命周期与作业的生命周期绑定。


# 启动分析

启动类：`YarnJobClusterEntrypoint.java`


## 加载配置文件以及命令行参数解析

在per-job启动的时候，需要先加载配置文件。

从`ApplicationConstants.Environment.PWD`中加载`flink-conf.yml`中的配置项。

### `_KEYTAB_PRINCIPAL`

从系统变量里面获取当前参数，用于kerberos认证的keytab文件。

### `NM_HOST`

从系统变量里面获取当前参数，为ApplicationMaster的主机名。当前参数不能为空。将当前参数的值设置到参数
`jobmanager.rpc.address`、`rest.address`、`rest.bind-address`里面。

### `web.port`



## 启动YarnJobClusterEntrypoint


jobMaster里面启动TaskManager

```java
private void startJobExecution() throws Exception {

}
```



