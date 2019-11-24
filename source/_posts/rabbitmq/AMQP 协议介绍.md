---
title: AMQP 协议介绍
date: 2019-11-24 23:51:00
tags: [RabbitMQ]
---

RabbitMQ 就是 AMQP 协议的 Erlang 的实现（当然 RabbitMQ 还支持 STOMP、MQTT 等协议）。
AMQP 的模型架构和 RabbitMQ 的模型架构是一样的，生产者将消息发送给交换器，交换器和队列绑定。
当生产者发送消息时所携带的 RoutingKey 与绑定时的 BindingKey 相匹配时，消息即被存入相应的队列之中。
消费者可以订阅相应的队列来获取消息。

RabbitMQ 中的交换器、交换器类型、队列、绑定、路由键等都是遵循 AMQP 中相应的概念。
目前 RabbitMQ 最新版本默认支持的使 AMQP 0-9-1。

AMQP 协议本身包括三层：

* Module Layer：位于协议最高层，主要定义了一些供客户端调用的命令，客户端可以利用这些命令实现自己的业务逻辑。
例如，客户端可以使用 Queue.Declare 命令声明一个队列或者使用 Basic.Consume 订阅消费一个队列中的消息。

* Session Layer：位于中间层，主要负责将客户端的命令发送给服务器，再将服务端的应答返回给客户端，
主要为客户端与服务器之间的通信提供可靠性同步机制和错误处理。

* Transport Layer：位于最底层，主要传输二进制数据流，提供帧的处理、信道复用、错误检测和数据表示等。

AMQP 说到底还是一个通信协议，通信协议都会涉及报文交互，从 low-level 举例来说，
AMQP 本身是应用层的协议，其填充于 TCP 协议的数据层部分。而从 high-level 来说，AMQP 是通过协议命令进行交互的，
AMQP 协议可以看作一系列结构化命令的集合，这里的命令代表一种操作，类似于 HTTP 中的方法（GET、POST 等）。


## AMQP 生产者流转过程

简洁版生产者代码

```
Connection connection = factory.newConnection(); // 创建连接
Channel channel = connection.createChannel(); // 创建信道
String message = "Hello World";
channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY,
        MessageProperties.PERSISTENT_TEXT_PLAIN,
        message.getBytes());
// 关闭资源
channel.close();
connection.close();
```

当客户端与 Broker 建立连接的时候，会调用 `factory.newConnection` 方法，这个方法会进一步封装成 Protocol Header 0-9-1 的报文发送给 Broker，
以此通知 Broker 本次交互采用的是 AMQP 0-9-1 协议，紧接着 Broker 返回 `Connection.Start` 来建立连接，在连接的过程中涉及
`Connection.Start/.Start-Ok`、`Connection.Tune/.Tune-Ok`、`Connection.Open/.Open-Ok` 这 6 个命令的交互。

当客户端调用 `connection.createChannel` 方法准备开启信道的时候，其包装的 `Channel.Open` 命令发送给 Broker，等待 `Channel.Open-Ok` 命令。

当客户端发送消息的时候，需要调用 `channel.basicPblish` 方法，对应的 AMQP 命令为 `Basic.Publish`，注意这个命令和前面涉及的命令略有不同，
这个命令还包含了 Content Header 和 Content Body。Content Header 里面包含的是消息体的属性，例如，投递模式、优先级等，而 Content Body 包含消息体本身。

当客户端发送消息需要关闭资源时，涉及 `Channel.Close/.Close-Ok` 与 `Connection.Close/.Close-Ok` 的命令交互，详细如下图所示。

![8](/images/rabbitmq/8.png)


## AMQP 消费者流转过程

简洁版消费者代码

```
Connection connection = factory.newConnection(addresses); // 创建连接
final Channel channel = connection.createChannel(); // 创建信道
Consumer consumer = new DefaultConsumer(channel) {} // ... 省略实现
channel.basicQos(64);
channel.basicConsume(QUEUE_NAME, consumer);
// 等待回调函数执行完毕之后，关闭资源
TimeUnit.SECONDS.sleep(5);
channel.close();
connection.close();
```

![9](/images/rabbitmq/9.png)

消费者客户端同样需要与 Broker 建立连接，与生产者客户端一样，协议交互同样涉及 `Connection.Start/.Start-Ok`、`Connection.Tune/.Tune-Ok`
和 `Connection.Open/.Open-Ok` 等。

紧接着也少不了在 Connection 之上建立 Channel，和生产者客户端一样，协议涉及 `Channel.Open/.Open-Ok`。

如果在消费之前调用了 `channel.basicQos(int prefetchCount)` 的方法来设置消费者客户端最大能 "保持" 的未确认的消息数，
那么协议流转会涉及 `Basic.Qos/.Qos-Ok` 这两个 AMQP 命令。

在真正消费之前，消费者客户端需要向 Broker 发送 `Basic.Consume` 命令（即调用 `channel.basicConsume` 方法）将 Channel 置为接收模式，
之后 Broker 回执 `Basic.Consume-Ok` 以告诉消费者客户端准备好消费消息。紧接着 Broker 向消费者客户端推送 (Push) 消息，
即 `Basic.Deliver` 命令，有意思的是这个和 `Basic.Publish` 命令一样会携带 Content Header 和 Content Body。

消费者接收到消息并正确消费之后，向 Broker 发送确认，即 `Basic.Ack` 命令。

在消费者停止消费的时候，主动关闭连接，这点和生产者一样，涉及 `Channel.Close/.Close-Ok` 和 `Connection.Close/.Close-Ok`。
