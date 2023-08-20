
Flink中提供了一种Blob服务，用来进行包的管理。

Flink当中按照支持BLOB文件类型分为：

- jar包：被user classloader使用的jar包。
- 高负荷RPC消息。
  - RPC消息长度超出了akka.framesize的大小。
  - 在HA摸式中，利用底层分布式文件系统分发单个高负荷RPC消息，比如: TaskDeploymentDescriptor,给多个接受对象。
  - 失败导致重新部署过程中复用RPC消息
- TaskManager的日志文件。
  - 为了在web ui上展示taskmanager的日志。


按存储特性分为：
- `PERMANENT_BLOB`:生命周期和job的生命周期一致，并且是可恢复的。会上传到BlobStore分布式文件系统中。
- `TRANSIENT_BLOB`:生命周期由用户自行管理，并且是不可恢复的。不会上传到BlobStore分布式文件系统中。


![pic](https://pan.zeekling.cn/flink/basic/blobServer0001.png)


BLOB底层存储，支持多种实现`HDFS`,`S3`,`FTP`等，HA中使用BlobStore进行文件的恢复。

