# Akka 学习笔记

------

## 什么是 Akka

**Actor模型**的**scala**实现

### 什么是Actor模型

> * 一种计算粒度，本身包含

> 1. 处理(行为)
> 2. 存储(状态)
> 3. 通讯(消息)

> * 接收消息三定则

> 1. 创建actors
> 2. 给其他actors发送消息
> 3. 指定处理下条消息的行为

提供

> * 并发，并行的高阶抽象
> * 异步、非阻塞、高性能、事件驱动
> * 超轻量事件驱动处理

## Actors

### 定义

```scala
trait Actor {

  type Receive = Actor.Receive

  implicit val context: ActorContext = ...

  implicit final val self = context.self

  final def sender(): ActorRef = context.sender()

  def receive: Actor.Receive

  def supervisorStrategy: SupervisorStrategy = SupervisorStrategy.defaultStrategy

  def preStart(): Unit = ()

  def postStop(): Unit = ()

  def preRestart(reason: Throwable, message: Option[Any]): Unit = {
    context.children foreach { child ⇒
      context.unwatch(child)
      context.stop(child)
    }
    postStop()
  }

  def postRestart(reason: Throwable): Unit = {
    preStart()
  }

  def unhandled(message: Any): Unit = {
    message match {
      case Terminated(dead) ⇒ throw new DeathPactException(dead)
      case _                ⇒ context.system.eventStream.publish(UnhandledMessage(message, sender(), self))
    }
  }
}
```

### 生命周期

1. actorOf(...) -> preStart
2. Restart -> preRestart(postStop) -> postRestart(preStart)
3. stop -> postStop

### 系统消息

```scala
trait PossiblyHarmful

abstract class PoisonPill extends AutoReceivedMessage with PossiblyHarmful with DeadLetterSuppression

```

* 有害消息标识

```scala
case object PoisonPill extends PoisonPill
```

* actor接收到消息后，执行 self.stop()

```scala
abstract class Kill extends AutoReceivedMessage with PossiblyHarmful

case object Kill extends Kill
```

* actor接收到消息后，执行 throw new ActorKilledException("Kill")

```scala
final case class Identify(messageId: Any) extends AutoReceivedMessage

final case class ActorIdentity(correlationId: Any, ref: Option[ActorRef])
```

* actor接收到Identify后， 将自己的ActorRef 作为ActorIdentity 参数发送给sender()

```scala
final case class Terminated private[akka] (@BeanProperty actor: ActorRef)
```

* 当actor terminated后, 给所有监听该 actor Death 的 actor 发送Terminated

```scala
abstract class ReceiveTimeout extends PossiblyHarmful

case object ReceiveTimeout extends ReceiveTimeout
```

* When using ActorContext.setReceiveTimeout, the singleton instance of ReceiveTimeout will be sent to the Actor when there hasn't been any message for that long

```scala
object Status {
  sealed trait Status extends Serializable

  final case class Success(status: Any) extends Status

  final case class Failure(cause: Throwable) extends Status
}
```

* Used for internal ACKing protocol. But exposed as utility class for user-specific ACKing protocols as well.

### Become/Unbecome

```scala
private[akka] class ActorCell {
  private var behaviorStack: List[Actor.Receive] = emptyBehaviorStack

  def become(behavior: Actor.Receive, discardOld: Boolean = true): Unit =
      behaviorStack = behavior :: (if (discardOld && behaviorStack.nonEmpty) behaviorStack.tail else behaviorStack)

  def unbecome(): Unit = {
    val original = behaviorStack
    behaviorStack =
      if (original.isEmpty || original.tail.isEmpty) actor.receive :: emptyBehaviorStack
      else original.tail
  }

  final def receiveMessage(msg: Any): Unit = actor.aroundReceive(behaviorStack.head, msg)

}
```

### Stash

在actor当前无法处理消息时(如等待网络连接)，将消息暂存起来(`stash`)，等时机成熟, 再将消息加入到处理队列(`unstash`, `unstashAll`)


## Dispatchers

归根结底，Akka 的 Locale actor只是对线程操作的一种封装，所以，他的实现是基于 java.util.concurrent的。

### ActorSystem初始化流程

> 1.

### Actor实例化流程

> 1. 

### 消息处理流程

> 1. 
