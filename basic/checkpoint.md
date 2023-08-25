
# 常见报错

## The maximum number of queued checkpoint requests exceeded 

未完成的Checkpoint排队超过了1000个。需要查看作业是否存在被压等。一般情况下作业被压会导致checkpoint失败。


## Periodic checkpoint scheduler is shut down 



## The minimum time between checkpoints is still pending 


## Not all required tasks are currently running 

部分算子任务已经完成，但是如果在维表join场景下，flink 1.13版本之前可能无法恢复checkpoint 



## An Exception occurred while triggering the checkpoint. 


## Asynchronous task checkpoint failed.


## The checkpoint was aborted due to exception of other subtasks sharing the ChannelState file 


## Checkpoint expired before completing 


## Checkpoint has been subsumed


## Checkpoint was declined


## Checkpoint was declined (tasks not ready) 


## Checkpoint was declined (task is closing) 


## Checkpoint was canceled because a barrier from newer checkpoint was received


## Task received cancellation from one of its inputs 


## Checkpoint was declined because one input stream is finished 


## CheckpointCoordinator shutdown 


## Checkpoint Coordinator is suspending 


## FailoverRegion is restarting 


## Task has failed 


## Task local checkpoint failure 


## Unknown task for the checkpoint to notify 


## Failure to finalize checkpoint 


## Trigger checkpoint failure 



