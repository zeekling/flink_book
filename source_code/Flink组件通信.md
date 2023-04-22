
## Flink组件通信原理

![Flink组件通信原理](https://pan.zeekling.cn/flink/akka/akka_actor.png)


## Akka总结

- ActorSystem 是管理 Actor生命周期的组件，Actor是负责进行通信的组
- 每个 Actor 都有一个 MailBox，别的 Actor 发送给它的消息都首先储存在 MailBox 中，通过这种方式可以实现异步通信。
- 每个Actor 是单线程的处理方式，不断的从 Mai1Box 拉取消息执行处理，所以对于Actor的消息处理，不适合调用会阻塞的处理方法。
- Actor 可以改变他自身的状态，可以接收消息，也可以发送消息，还可以生成新的 Actor
- 每一个ActorSystem 和Actor都在启动的时候会给定一个 name．如果要从ActorSystem中，获取一个 Actor，
则通过以下的方式来进行 Actor的获取：`akka.tcp://asname＠bigdata02:9527/user/actorname`
- 如果一个 Actor 要和另外一个 Actor进行通信，则必须先获取对方 Actor的 ActorRef 对象，然后通过该对象发送消息即可。
- 通过 te11 发送异步消息，不接收响应，通过 ask 发送异步消息，得到 Future 返回，通过异步回到返回处理结果。



