---
title: RabbitMQ 相关概念介绍
date: 2019-11-09 20:14:00
tags: [RabbitMQ]
---

RabbitMQ 整体上是一个生产者与消费者模型，主要负责接收、存储和转发消息。
可以把消息传递的过程想象成：当你将一个包裹送到邮局，邮局会暂存并最终将邮件通过邮递员送到收件人的手上，
RabbitMQ 就好比邮局、邮箱和邮递员组成的一个系统。
从计算机术语层面来说，RabbitMQ 模型更像是一种交换机模型。

![rabbitmq_structure](/images/rabbitmq/rabbitmq_structure.png)


## 生产者和消费者

* Producer：生产者，就是投递消息的一方。

生产者创建消息，然后发布到 RabbitMQ 中。消息一般可以包含 2 个部分：消息体和标签（Label）。
消息体也可以称之为 payload，在实际应用中，消息体一般是一个带有业务逻辑结构的数据，比如一个 JSON 字符串。
当然可以进一步对这个消息体进行序列化操作。消息的标签用来表述这条消息，比如一个交换器的名称和一个路由键。
生产者把消息交由 RabbitMQ，RabbitMQ 之后会根据标签把消息发送给感兴趣的消费者（Consumer）。


* Consumer：消费者，就是接收消息的一方。

消费者连接到 RabbitMQ 服务器，并订阅到队列上。当消费者消费一条消息时，只是消费消息的消息体（payload）。
在消息路由的过程中，消息的标签会丢弃，存入队列中的消息只有消息体，消费者也只会消费到消息体，也就不知道
消息的生产者是谁，当然消费者也不需要知道。

* Broker：消息中间件的服务节点。

对于 RabbitMQ 来说，一个 RabbitMQ Broker 可以简单地看作一个 RabbitMQ 服务节点，或者 RabbitMQ 服务实例。
大多数情况下也可以将一个 RabbitMQ Broker 看作一台 RabbitMQ 服务器。

下图展示了生产者将消息存入 RabbitMQ Broker，以及消费者从 Broker 中消费数据的整个流程：

![rabbitmq1](/images/rabbitmq/rabbitmq1.png)

首先生产者将业务方数据进行可能的包装，之后封装成消息，发送（AMQP 协议里这个动作对应的命令为 Basic.Publish）到 Broker 中。
消费者订阅并接收消息（AMQP 协议里这个动作对应的命令为 Basic.Consume 或者 Basic.Get），经过可能的解包处理得到原始的数据，
之后再进行业务处理逻辑。这个业务处理逻辑并不一定需要和接收消息的逻辑使用同一个线程。
消费者进程可以使用一个线程去接收消息，存入到内存中，比如使用 Java 中的 BlockingQueue。
业务处理逻辑使用另一个线程从内存中读取数据，这样可以将应用进一步解耦，提高整个应用的处理效率。


## 队列

Queue：队列，是 RabbitMQ 的内部对象，用于存储消息。

RabbitMQ 中消息都只能存储在队列中，
