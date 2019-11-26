---
title: RabbitMQ 进阶
date: 2019-11-26 15:48:00
tags: [RabbitMQ]
---

## 消息何去何从

`mandatory` 和 `immediate` 是 `channel.basicPublish` 方法中的两个参数，它们都有当消息传递过程中不可达目的地时将消息返回给生产者的功能。
RabbitMQ 提供的备份交换器（Alternate Exchange）可以将未能被交换器路由的消息（没有绑定队列或者没有匹配的绑定）存储起来，而不用返回给客户端。


### `mandatory` 参数

当 `mandatory` 参数设为 true 时，交换器无法根据自身的类型和路由键找到一个符合条件的队列，那么 RabbitMQ 会调用 `Basic.Return` 命令将消息
返回给生产者。当 `mandatory` 参数设置为 false 时，出现上述情形，则消息直接被丢弃。

那么生产者如何获取到没有被正确路由到合适队列的消息呢？这时候可以通过调用 `channel.addReturnListener` 来添加 `ReturnListener` 监听器实现。

使用 `mandatory` 参数的关键代码如下：

```
channel.basicPublish(EXCHANGE_NAME, "", true,
        MessageProperties.PERSISTENT_TEXT_PLAIN,
        "mandatory test".getBytes());
channel.addReturnListener(new ReturnListener() {
    public void handleReturn(int replyCode, String, replyText,
                            String exchange, String routingKey,
                            AMQP.BasicProperties basicProperties,
                            byte[] body) throws IOException {
        String message = new String(body);
        System.out.println("Basic.Return 返回的结果是:" + message);
    }
});
```

> 测试的时候并没有如期输出。可能姿势不对

上面代码中生产者没有成功地将消息路由到队列，此时 RabbitMQ 会通过 Basic.Return 返回 "mandatory test" 这条消息，
之后生产者客户端通过 ReturnListener 监听到了这个事件，上面代码的最后输出应该是 "Basic.Return 返回的结果是: mandatory test"。

从 AMQP 协议层面来说，其对应的流转过程如下图所示：

![12](/images/rabbitmq/12.png)


### `immediate` 参数

当 `immediate` 参数设为 true 时，如果交换器在将消息路由到队列时发现队列上并不存在任何消费者，那么这条消息将不会存入队列中。
当与路由键匹配的所有队列都没有消费者时，该消息会通过 `Basic.Return` 返回至生产者。

概括来说，`mandatory` 参数告诉服务器至少将该消息路由到一个队列中，否则将消息返回给生产者。
`immediate` 参数告诉服务器，如果该消息关联的队列上有消费者，则立刻投递；
如果所有匹配的队列上都没有消费者，则直接将消息返还给生产者，不用将消息存入队列等待消费者了。

RabbitMQ 3.0 版本开始去掉了对 `immediate` 参数的支持，对此 RabbitMQ 官方解释是：
`immediate` 参数会影响镜像队列的性能，增加代码复杂性，建议采用 TTL 和 DLX 的方法替代。


### 备份交换器

备份交换器，英文名称为 `Alternate Exchange`，简称 AE，或者更直白地称之为 "备胎交换器"。
生产者在发送消息的时候如果不设置 `mandatory` 参数，那么消息在未被路由的情况下将会丢失；
如果设置了 `mandatory` 参数，那么需要添加 `ReturnListener` 的编程逻辑，生产者的代码将变得复杂。
如果既不想复杂化生产者的编程逻辑，又不想消息丢失，那么可以使用备份交换器，这样可以将未被路由的消息存储在 RabbitMQ 中，再在需要的时候去处理这些消息。

可以通过在声明交换器（调用 `channel.exchangeDeclare` 方法）的时候添加 `alternate-exchange` 参数来实现，也可以通过策略的方式实现。
如果两者同时使用，则前者的优先级更高，会覆盖掉 Policy 的设置。

使用备份交换器的关键代码如下：

```
Map<String, Object> args = new HashMap<String, Object>();
args.put("alternate-exchange", "myAe");
channel.exchangeDeclare("normalExchange", "direct", true, false, args);
channel.exchangeDeclare("myAe", "fanout", true, false, null);
channel.queueDeclare("normalQueue", true, false, false, null);
channel.queueBind("normalQueue", "normalExchange", "normalKey");
channel.queueDeclare("unroutedQueue", true, false, false, null);
channel.queueBind("unroutedQueue", "myAe", "");
```

上面的代码中声明了两个交换器 `normalExchange` 和 `myAe`，分别绑定了 `normalQueue` 和 `unroutedQueue` 这两个队列，
同时将 `myAe` 设置为 `normalExchange` 的备份交换器。注意 `myAe` 的交换器类型为 `fanout`。

参考下图，如果此时发送一条消息到 `normalExchange` 上，当路由键等于 `normalKey` 的时候，消息能正确路由到 `normalQueue` 这个队列中。
如果路由键设置为其他值，此时就会发送给 `myAe`，进而发送到 `unroutedQueue` 这个队列。

![13](/images/rabbitmq/13.png)

备份交换器其实和普通的交换器没有太大的区别，为了方便使用，建议设置为 `fanout` 类型，如若读者想设置为 `direct` 或者 `topic` 的类型也没有什么不妥。
需要注意的是，消息被重新发送到备份交换器时的路由键和从生产者发出的路由键是一样的。

考虑这样一种情况，如果备份交换器的类型是 `direct`，并且有一个与其绑定的队列，假设绑定的路由键是 key1，当某条携带路由键为 key2 的消息被
转发到这个备份交换器的时候，备份交换器没有匹配到合适的队列，则消息丢失。如果消息携带的路由键为 key1，则可以存储到队列中。

对于备份交换器，总结以下几种特殊情况：

* 如果设置的备份交换器不存在，客户端和 RabbitMQ 服务端都不会有异常出现，此时消息丢失。

* 如果备份交换器没有绑定任何队列，客户端和 RabbitMQ 服务端都不会有异常出现，此时消息丢失。

* 如果备份交换器没有任何匹配的队列，客户端和 RabbitMQ 服务端都不会有异常出现，此时消息丢失。

* 如果备份交换器和 `mandatory` 参数一起使用，那么 `mandatory` 参数无效。


## 过期时间（TTL）

TTL，Time to Live 的简称，即过期时间。RabbitMQ 可以对消息和队列设置 TTL。


### 设置消息的 TTL

目前有两种方法可以设置 TTL。第一种方法是通过队列属性设置，队列中所有消息都有相同的过期时间。第二种方法是对消息本身进行单独设置，每条消息的 TTL
可以不同。如果两种方法一起使用，则消息的 TTL 以两者之间较小的那个数值为准。消息在队列中的生存时间一旦超过设置的 TTL 值时，就会变成 "死信"（Dead Message），消费者将无法再收到该消息（不是绝对的）。

通过队列属性设置 TTL 的方法是在 `channel.queueDeclare` 方法中加入 `x-message-ttl` 参数实现的，这个参数的单位是毫秒。

示例代码：

```
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-message-ttl", 6000);
channel.queueDeclare(queueName, durable, exclusive, autoDelete, args);
```

如果不设置 TTL，则表示此消息不会过期；如果将 TTL 设置为 0，则表示除非此时可以直接将消息投递到消费者，否则该消息会被立即丢弃，
这个特性可以部分代替 RabbitMQ 3.0 版本之前的 `immediate` 参数，之所以部分代替，是因为 `immediate` 参数在投递失败时会用
`Basic.Return` 将消息返回（这个功能可以用死信队列来实现）。

针对每条消息设置 TTL 的方法是在 `channel.basicPublish` 方法中加入 `expiration` 的属性参数，单位为毫秒。

关键代码如下：

```
AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
builder.deliveryMode(2); // 持久化消息
builder.expiration("60000"; // 设置 TTL=60000ms
AMQP.BasicProperties properties = builder.build();
channel.basicPublish(exchangeName, routingKey, mandatory, properties, "ttlTestMessage".getBytes());
```

也可以如下：

```
AMQP.BasicProperties properties = new AMQP.BasicProperties();
properties.setDeliveryMode(2);
properties.setExpiration("60000");
channel.basicPublish(exchangeName, routingKey, mandatory, properties, "ttlTestMessage".getBytes());
```

对于第一种设置队列 TTL 属性的方法，一旦消息过期，就会从队列中抹去，而在第二种方法中，即使消息过期，也不会马上从队列中抹去，
因为每条消息是否过期是在即将投递到消费者之前判定的。

为什么这两种方法处理的不一样？因为第一种方法里，队列中已过期的消息肯定在队列头部，RabbitMQ 只要定期从队头开始扫描是否有过期的消息即可。
而第二种方法里，每条消息的过期时间不同，如果要删除所有过期消息势必要扫描整个队列，所以不如等到此消息即将被消费时再判定是否过期，如果过期再进行删除即可。


### 设置队列的 TTL

通过 `channel.queueDeclare` 方法中的 `x-expires` 参数可以控制队列被自动删除前处于未使用状态的时间。
未使用的意思是队列上没有任何的消费者，队列也没有被重新声明，并且在过期时间段内也未调用过 `Basic.Get` 命令。

设置队列里的 TTL 可以应用于类似 RPC 方式的回复队列，在 RPC 中，许多队列会被创建出来，但是却是未被使用的。

RabbitMQ 会确保在过期时间到达后将队列删除，但是不保障删除的动作有多及时。在 RabbitMQ 重启后，持久化的队列的过期时间会被重新计算。

用于表示过期时间的 `x-expires` 参数以毫秒为单位，并且服从和 `x-message-ttl` 一样的约束条件，不过不能设置为 0。
比如该参数设置为 1000，则表示该队列如果在 1 秒钟之内未使用则会被删除。

```
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-expires", 1800000);
channel.queueDeclare("myqueue", false, false, false, args);
```


## 死信队列

DLX，全称为 Dead-Letter-Exchange，可以称之为死信交换器，也有人称之为死信邮箱。
当消息在一个队列中变成死信（Dead Message），它能被重新发送到另一个交换器中，这个交换器就是 DLX，绑定 DLX 的队列就称之为死信队列。

消息变成死信一般是由于以下几种情况：

* 消息被拒绝（Basic.Reject/Basic.Nack），并且设置 requeue 参数为 false

* 消息过期

* 队列达到最大长度

DLX 也是一个正常的交换器，和一般的交换器没有区别，它能在任何的队列上被指定，实际上就是设置某个队列的属性。
当这个队列中存在死信时，RabbitMQ 就会自动地将这个消息重新发布到设置的 DLX 上去，进而被路由到另一个队列，即死信队列。
可以监听这个队列中的消息进行相应的处理，这个特性与将消息的 TTL 设置为 0 配合使用可以弥补 immediate 的功能。

通过在 `channel.queueDeclare` 方法中设置 `x-dead-letter-exchange` 参数来为这个队列添加 DLX。

```
channel.exchangeDeclare("dlx_exchange", "direct"); // 创建 DLX: dlx_exchange
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-dead-letter-exchange", "dlx_exchange");
// 为队列 myqueue 添加 DLX
channel.queueDeclare("myqueue", false, false, false, args);
```

也可以为这个 DLX 指定路由键，如果没有指定，则使用原队列的路由键：

```
args.put("x-dead-letter-routing-key", "dlx-routing-key");
```

下面创建一个队列，为其设置 TTL 和 DLX 等：

```
channel.exchangeDeclare("exchange.dlx", "direct", true);  // 定义名称为 "exchange.dlx"，类型为 "direct"，持久化的交换器
channel.exchangeDeclare("exchange.normal", "fanout", true); // 定义名称为 "exchange.narmal"，类型为 "fanout"，持久化的交换器
Map<String, Obejct> args = new HashMap<String, Object>();
args.put("x-message-ttl", 10000);
args.put("x-dead-letter-exchange", "exchange.dlx");  // 绑定的死信交换器为 "exchange.dlx"
args.put("x-dead-letter-routing-key", "routingKey"); // 定义死信队列的路由键
channel.queueDeclare("queue.normal", true, false, false, args); // 定义名称为 "queue.normal"，持久化的，非排他的，非自动删除的队列。绑定超时时间为 10000ms，绑定死信队列为 "exchange.dlx"，绑定死信队列路由键为 "routingKey"
channel.queueBind("queue.normal", "exchange.normal", "");  // 将队列 "queue.normal" 绑定到交换器 "exchange.normal"
channel.queueDeclare("queue.dlx", true, false, false, null); // 定义名称为 "queue.dlx"，持久化、非排他、非自动删除的队列
channel.queueBind("queue.dlx", "exchange.dlx", "routingKey"); // 将队列 "queue.dlx" 绑定到交换器 "exchange.dlx"，路由键为 "routingKey"
channel.basicPublish("exchange.normal", "rk",
    MessageProperties.PERSISTENT_TEXT_PLAIN, "dlx",getBytes());  // 将消息 "dlx" （路由键为 rk）发送到交换器 "exchange.normal" 中
```

这里创建了两个交换器 `exhange.normal` 和 `exchange.dlx`，分别绑定两个队列 `queue.normal` 和 `queue.dlx`。

由 Web 管理界面可以看出，两个队列都被标记了 "D"，这个是 durable 的缩写。即设置了队列持久化。
`queue.normal` 这个队列还设置了 TTL、DLX 和 DLK，其中 DLX 指的是 `x-dead-letter-routing-key` 这个属性。

![14](/images/rabbitmq/14.png)

参考下图，生产者首先发送一条携带路由键为 "rk" 的消息，然后经过交换器 `exchange.normal` 顺利地存储到队列 `queue.normal` 中。
由于队列 `queue.normal` 设置了过期时间为 10s，在这 10s 内没有消费者消费这条消息，那么判定这条消息为过期。
由于设置了 DLX，过期之时，消息被丢给交换器 `exchange.dlx` 中，这时找到与 `exchange.dlx` 匹配的队列 `queue.dlx`，最后消息被存储在 `queue.dlx` 这个死信队列中。

![15](/images/rabbitmq/15.png)

对于 RabbitMQ 来说，DLX 是一个非常有用的特性。它可以处理异常情况下，消息不能够被消费者正确消费（消费者调用了 Basic.Nack 或者 Basic.Reject）而被
置入死信队列中的情况，后续分析程序可以通过消费这个死信队列中的内容来分析当时所遇到的异常情况，进而可以改善和优化系统。
DLX 配合 TTL 使用还可以实现延迟队列的功能。



## 延迟队列

延迟队列存储的对象是对应的延迟消息，所谓 "延迟消息" 是指当消息被发送以后，并不想让消费者立刻拿到消息，而是等待特定时间后，消费者才能拿到这个消息消费。

延迟队列的使用场景有很多，比如：

* 在订单系统中，一个用户下单之后通常有 30 分钟的时间进行支付，如果 30 分钟之内没有支付成功，那么这个订单将进行异常处理，这时就可以使用延迟队列来处理这些订单了。

* 用户希望通过手机远程遥控家里的智能设备在指定的时间进行工作。这时候就可以将用户指令发送到延迟队列，当指令设定的时间到了就再将指令推送到智能设备。

在 AMQP 协议中，或者 RabbitMQ 本身没有直接支持延迟队列的功能，当时可以通过前面所介绍的 DLX 和 TTL 模拟出延迟队列的功能。

在下图中，不仅展示的是死信队列的用法，也是延迟队列的用法，对于 `queue.dlx` 这个死信队列来说，同样可以看作延迟队列。
假设一个应用中需要将每条消息都设置为 10 秒延迟，生产者通过 `exchange.normal` 这个交换器将发送的消息存储在 `queue.normal` 这个队列中。
消费者订阅的并非是 `queue.normal` 这个队列，而是 `queue.dlx` 这个队列。
当消息从 `queue.normal` 这个队列中过期之后被存入 `queue.dlx` 这个队列中，消费者就恰巧消费到了延迟 10 秒的这条消息。

在真实应用中，对于延迟队列可以根据延迟时间的长短分为多个等级，一般分为 5秒、10秒、30秒、1分钟、5分钟等。

参考下图，根据应用需求的不同，生产者在发送消息的时候通过设置不同的路由键，以此将消息发送到与交换器绑定的不同的队列中。
这里队列分别设置了过期时间为 5秒、10秒、30秒、1分钟，同时也分别配置了 DLX 和相应的死信队列。
当相应的消息过期时，就会转存到相应的死信队列（即延迟队列）中，这样消费者根据业务自身的情况，分别选择不同延迟等级的延迟队列进行消费。

![16](/images/rabbitmq/16.png)


