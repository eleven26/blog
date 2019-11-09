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


