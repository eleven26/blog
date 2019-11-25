---
title: RabbitMQ 客户端开发向导
date: 2019-11-25 16:35:00
tags: [RabbitMQ]
---

## 连接 RabbitMQ

下面的代码用来在给定的参数（IP 地址、端口号、用户名、密码等）下连接 RabbitMQ：

```
ConnectionFactory factory = new ConnectionFactory();
factory.setUsername(USERNAME);
factory.setPassword(PASSSWORD);
factory.setVirtualHost(virtualHost);
factory.setHost(IP_ADDRESS);
factory.setPort(PORT);

Connection connection = factory.newConnection();
```

```
ConnectionFactory factory = new ConnectionFactory();
factory.setUri("amqp://guest:guest@localhost:5672/virtualHost");
Connection connection = factory.newConnection();

// Connection 接口被用来创建一个 Channel
Channel channel = connection.createChannel();
// 在创建 channel 之后，channel 可以用来发送或者接收消息了
```

> 注意要点: Connection 可以用来创建多个 Channel 实例，但是 Channel 实例不能在线程间共享，
> 应用程序应该为每一个线程开辟一个 Channel。某些情况下 Channel 的操作可以并发运行，但是在其它情况下会导致在网络上
> 出现错误的通信帧交替，同时也会影响发送方确认（publisher confirm）机制的运行，
> 所以多线程间共享 Channel 实例是非线程安全的。

Channel 或者 Connection 中有个 isOpen 方法可以用来检测其是否已处于开启状态。
但并不推荐在生产环境的代码上使用 isOpen 方法，这个方法的返回值依赖于 shutdownCause 的存在，有可能会产生竞争。

isOpen 方法的源码:

```
public boolean isOpen() {
    synchronized(this.monitor) {
        return this.shutdownCause == null;
    }
}
```

错误地使用 isOpen 方法:

```
public void brokenMethod(Channel channel)
{
    if (channel.isOpen()) {
        // The following code depends on the channel being in open state.
        // However there is a possibility of the change in the channel state
        // between isOpen() and basicQos(1) call
        channel.basicQos(1);
    }
}
```

通常情况下，在调用 createXXX 或者 newXXX 方法之后，我们可以简单地认为 Connection 或者 Channel 已经成功地处于开启状态，
而不会在代码中使用 isOpen 这个检测方法。如果在使用 Channel 的时候其已经处于关闭状态，那么程序会抛出一个 `com.rabbitmq.client.ShutdownSignalException`，
我们只需要捕获这个异常即可。当然同时也要试着捕获 `IOException` 或者 `SocketException`，以防 Connection 意外关闭。

```
public void validMethod(Channel channel)
{
    try {
        // ...
        // channel.basicQos(1);
    } catch (ShutdownSignalException e) {
        // possibly check if channel was closed
        // by the time we started action and reasons for
        // closing it
    } catch (IOException e) {
        // check why connection was closed
    }
}
```


## 使用交换器和队列

交换器和队列是 AMQP 中 high-level 层面的构建模块，应用程序需要确保在使用它们的时候就已经存在了，在使用之前需要先声明（declare）它们。

```
channel.exchangeDeclare(exchangeName, "direct", true);
String queueName = channel.queueDeclare().getQueue();
channel.queueBind(queueName, exchangeName, routingKey);
```

上面创建了一个持久化的、非自动删除的、绑定类型为 direct 的交换器，同时也创建了一个非持久化的、拍他的、自动删除的队列（此队列的名称由 RabbitMQ 自动生成）。
这里交换器和队列也都没有设置特殊的参数。

上面的代码也展示了如何将路由键将队列和交换器绑定起来。上面声明的队列具备如下特性:
只对当前应用中同一个 Connection 层面可用，同一个 Connection 的不同 Channel 可共用，并且也会在应用连接断开的时候自动删除。

如果要在应用中共享一个队列，可以做如下声明：

```
channel.exchangeDeclare(exchangeName, "direct", true);
channel.queueDeclare(queueName, true, false, false, null);
channel.queueBind(queueName, exchangeName, routingKey);
```

这里的队列被声明为持久化的，非排他的、非自动删除的，并且也被分配另一个确定的已知的名称（由客户端分配而非 RabbitMQ 自动生成）。

注意：Channel 的 API 方法都是可以重载的，比如 exchangeDeclare、queueDeclare。
根据参数不用，可以有不同的重载形式，根据自身的需要进行调用。

生产者和消费者都可以声明一个交换器或者队列。
如果尝试声明一个已经存在的交换器或者队列，只要声明的参数完全匹配现存的交换器或者队列，
RabbitMQ 就可以什么都不做，并成功返回。如果声明的参数不匹配则会抛出异常。


### exchangeDeclare 方法详解

exchangeDeclare 方法有多个重载方法，这些重载方法都是由下面这个方法中缺省的某些参数构成的。

```
Exchange.DeclareOk exchangeDeclare(String exchange,
    String type, boolean durable,
    boolean autoDelete, boolean internal,
    Map<String, Object> arguments) throws IOException;
```

这个方法的返回值是 Exchange.DeclareOk，用来标识成功声明了一个交换器。

各个参数详细说明如下所述：

* exchange：交换器的名称

* type：交换器的类型，常见的如 fanout、direct、topic

* durable：设置是否持久化。durable 设置为 true 表示持久化，反之是非持久化。持久化可以将交换器存盘，在服务器重启的时候不会丢失相关信息。

* autoDelete：设置是否自动删除。autoDelete 设置为 true 则表示自动删除。自动删除的前提是至少有一个队列或者交换器与这个交换器绑定，之后
所有与这个交换器绑定的队列或者交换器与此解绑。注意不能错误地把这个参数理解为："当与此交换器连接的客户端都断开时，RabbitMQ 会自动删除本交换器"

* internal：设置是否是内置的。如果设置为 true，则表示是内置的交换器，客户端程序无法直接发送消息到此交换器中，只能通过交换器路由到交换器这种方式。

* arguments：其它一些结构化参数，比如 alternate-exchange。


exchangeDeclare 的其它重载方法如下：

(1) `Exchange.DeclareOk exchangeDeclare(String exchange, String type) throws IOException;`

(2) `Exchange.DeclareOk exchangeDeclare(String exchange, String type, boolean durable) throws IOException;`

(3) `Exchange.DeclareOk exchangeDeclare(String exchange, String type, boolean durable, boolean autoDelete, Map<String, Object> arguments) throws IOException;`

与此对应的，将第二个参数 `String type` 换成 `BuiltinExchangeType type` 对应的几个重载方法：

(1) `Exchange.DeclareOk exchangeDeclare(String exchange, BuiltinExchangeType type) throws IOException;`

(2) `Exchange.DeclareOk exchangeDeclare(String exchange, BuiltinExchangeType type, boolean durable) throws IOException;`

(3) `Exchange.DeclareOk exchangeDeclare(String exchange, BuiltinExchangeType type, boolean durable, Map<String, Object> arguments) throws IOException;`

与 exchangeDeclare 师出同门的还有几个方法，比如 exchangeDeclareNoWait 方法，具体定义如下（当然也有 BuiltinExchangeType 版）：

```
void exchangeDeclareNoWait(String exchange,
                            String type,
                            boolean durable,
                            boolean autoDelete,
                            boolean internal,
                            Map<String, Object> arguments) throws IOException;
```

这个 `exchangeDeclareNoWait` 比 `exchangeDeclare` 多设置了一个 nowait 参数，这个 nowait 参数指的是 AMQP 中 `Exchange.Declare` 命令的参数，
意思是不需要服务器返回，注意这个方法的返回值是 void，而普通的 exchangeDeclare 方法的返回值是 `Exchange.DeclareOk`，意思是在客户端声明了一个交换器之后，
需要等待服务器的返回（服务器会返回 `Exchange.Declare-Ok` 这个 AMQP 命令）。

针对 "`exchangeDeclareNoWait` 不需要服务器返回任何值" 这一点，考虑这样一种情况，在声明完一个交换器之后（实际服务器还未完成交换器的创建），
那么此时客户端紧接着使用这个交换器，必然会发生异常。如果没有特殊的缘由和应用场景，并不建议使用这个方法。

这里还有师出同门的另一个方法 `exchangeDeclarePassive`，这个方法的定义如下：

```
Exchange.DeclareOk exchangeDeclarePassive(String name) throws IOException;
```

这个方法在实际应用过程中还是非常有用的，它主要用来检测相应的交换器是否存在。如果存在则正常返回；如果不存在则抛出异常：404 channel exception，同时 Channel 也会被关闭。

有声明创建交换器的方法，当然也有删除交换器的方法，相应的方法如下：

(1) `Exchange.DeleteOk exchangeDelete(String exchange) throws IOException;`

(2) `void exchangeDeleteNoWait(String exchange, boolean ifUnused) throws IOException;`

(3) `Exchange.DeleteOk exchangeDelete(String exchange, boolean ifUnUsed) throws IOException;`

其中 exchange 表示交换器的名称，而 ifUnused 用来设置是否在交换器没有被使用的情况下删除。
如果 ifUnused 设置为 true，则只有在此交换器没有被使用的情况下才会被删除；如果设置为 false，则无论如何这个交换器都要被删除。


### queueDeclare 方法详解

queueDeclare 相对于 exchangeDeclare 方法而言，重载的个数就少很多，它只有两个重载方法：

(1) `Queue.DeclareOk queueDeclare() throws IOException;`

(2) `Queue.DeclareOk queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments) throws IOException;`

不带任何参数的 queueDeclare 方法默认创建一个由 RabbitMQ 命名的（也称为匿名队列）、排他的、自动删除的、非持久化的队列。

方法的参数详细说明如下所述：

* queue：队列的名称

* durable：设置是否持久化。为 true 则设置队列为持久化。持久化的队列会存盘，在服务器重启的时候可以保证不丢失相关信息。

* exclusive：设置是否排他。为 true 则设置队列为排他的。如果一个队列被声明为排他队列，该队列仅对首次声明它的连接可见，并在连接断开时自动删除。
这里需要注意三点：排他队列是基于连接（Connection）可见的，同一个连接的不同信道（Channel）是可以同时访问同一连接创建的排他队列：
"首次" 是指如果一个连接已经声明了一个排他队列，其他连接是不允许建立同名的排他队列，这个与普通队列不同：即使该队列是持久化的，
一旦连接关闭或者客户端退出，该排他队列都会被自动删除，这种队列适用于一个客户端同时发送和读取消息的应用场景。

* autoDelete：设置是否自动删除。为 true 则设置为自动删除。自动删除的前提是：至少有一个消费者连接到这个队列，之后所有与这个队列连接的消费者
都断开时，才会自动删除。不能把这个参数错误地理解为："当连接到此队列的所有客户端断开时，这个队列自动删除"。因为生产者客户端创建这个队列，
或者没有消费者客户端与这个队列连接时，都不会自动删除这个队列。

* arguments：设置队列的其它一些参数，如 `x-message-ttl`、`x-expires`、`x-max-length`、`x-max-priority` 等

> 注意要点：生产者和消费者都能够使用 `queueDeclare` 来声明一个队列，但是如果消费者在同一个信道上订阅了另一个队列，就无法再声明队列了。
> 必须先取消订阅，然后将信道设置为 "传输" 模式，之后才能声明队列。

对应于 `exchangeDeclareNoWait` 方法，这里也有一个 `queueDeclareNoWait` 方法：

```
void queueDeclareNoWait(String queue, boolean durable, boolean exclusive,
    boolean autoDelete, Map<String, Object> arguments) throws IOException;
```

方法的返回值也是 void，表示不需要服务端的任何返回。同样也需要注意，在调用完 `queueDeclareNoWait` 之后，紧接着使用声明的队列时有可能会发生异常情况。

同样这里还有一个 `queueDeclarePassive` 的方法，也比较常用。这个方法用来检测相应的队列是否存在。
如果存在则正常返回，如果不存在则抛出异常：404 channel exception，同时 Channel 也会被关闭。方法定义如下：

`Queue.DeclareOk queueDeclarePassive(String queue) throws IOException;`

与交换器对应，关于队列也有删除的相应方法：

(1) `Queue.DeleteOk queueDelete(String queue) throws IOException;`

(2) `Queue.DeleteOk queueDelete(String queue, boolean ifUnused, boolean ifEmpty) throws IOException;`

(3) `void queueDeleteNoWait(String queue, boolean ifUnused, boolean ifEmpty) throws IOException;`

其中 queue 表示队列的名称，ifEmpty 设置为 true 表示在队列为空的情况下才能够删除。

与队列相关的还有一个有意思的方法 - queuePurge，区别于 queueDelete，这个方法用来清空队列中的内容，而不删除队列本身，具体定义如下：

`Queue.PurgeOk queuePurge(String queue) throws IOException;`


### queueBind 方法详解

将队列和交换器绑定的方法如下，可以与前两节中的方法定义进行类比。

(1) `Queue.BindOk queueBind(String queue, String exchange, String routingKey) throws IOException;`

(2) `Queue.BindOk queueBind(String queue, String exchange, String routingKey, Map<String, Object> arguments) throws IOException;`

(3) `void queueBindNoWait(String queue, String exchange, String routingKey, Map<String, Object> arguments) throws IOException;`

方法中涉及的参数详解。

* queue：队列名称

* exchange：交换器名称

* routingKey：用来绑定队列和交换器的路由键

* arguments：定义绑定的一些参数

不仅可以将队列和交换器绑定起来，也可以将已经被绑定的队列和交换器进行解绑。具体方法可以参考如下：

(1) `Queue.UnbindOk queueUnbind(String queue, String exchange, String routingKey) throws IOException;`

(2) `Queue.UnbindOk queueUnbind(String queue, String exchange, String routingKey, Map<String, Object> arguments) throws IOException;`


### exchangeBind 方法详解

我们不仅可以将交换器与队列绑定，也可以将交换器与交换器绑定，后者和前者的用法如出一辙，相应的方法如下：

(1) `Exchange.BindOk exchangeBind(String destination, String source, String routingKey) throws IOException;`

(2) `Exchange.BindOk exchangeBind(String destination, String source, String routingKey, Map<String, Object> arguments) throws IOException;`

(3) `void exchangeBind(String destination, String source, String routingKey, Map<String, Object> arguments) throws IOException;`

绑定之后，消息从 source 交换器转发到 destination 交换器，某种程度上来说 destination 交换器可以看作一个队列。

```
channel.exchangeDeclare("source", "direct", false, true, null);
channel.exchangeDeclare("destination", "fanout", false, true, null);
channel.exchangeBind("destination", "source", "exKey");
channel.queueDeclare("queue", false, false, true, null);
channel.queueBind("queue", "destination", "");
channel.basicPublish("source", "exKey", null, "exToExDemo".getBytes());
```

生产者发送消息至交换器 source 中，交换器 source 根据路由键找到与其匹配的另一个交换器 destination，并把消息转发到 destination 中，
进而存储在 destination 绑定的队列 queue 中。

![10](/images/rabbitmq/10.png)


### 何时创建

RabbitMQ 的消息存储在队列中，交换器的使用并不真正耗费服务器的性能，而队列会。如果要衡量 RabbitMQ 当前的 QPS 只需看队列的即可。
在实际业务应用中，需要对所创建的队列的流量、内存占用及网卡占用有一个清晰的认知，预估其平均值和峰值，以便在固定硬件资源的情况下进行合理有效的分配。

根据 RabbitMQ 官方建议，生产者和消费者都应该尝试创建（这里指声明操作）队列。
这是一个很好的建议，但不适用于所有的情况。如果业务本身在架构之初已经充分地预估了队列地使用情况，
完全可以在业务程序上线之前在服务器上创建好，这样业务程序也可以免去声明地过程，直接使用即可。

预先创建好资源还有一个好处是，可以确保交换器和队列之间正确地绑定匹配。
很多时候，由于人为因素、代码缺陷等，发送消息地交换器并没有绑定任何队列，那么消息将会丢失；
或者交换器绑定了某个队列，但是发送消息的时候路由键无法与现存的队列匹配，那么消息也会丢失。
当然可以配合 mandatory 参数或者备份交换器来提高程序的健壮性。

与此同时，预估好队列的使用情况非常重要，如果在后期运行过程中超过预期的阈值，可以根据实际情况对当前集群进行扩容或者将相应的队列
迁移到其他集群。迁移的过程也可以对业务程序完全透明。此种方法也更有利于开发和运维分工，便于相应的资源管理。

如果集群资源充足，而即将使用的队列所占用的资源又在可控的范围之内，为了增加业务程序的灵活性，也完全可以在业务程序中声明队列。


## 发送消息

如果要发送一个消息，可以使用 Channel 类的 basicPublish 方法，比如发送一条内容为 "Hello World!" 的消息，参考如下：

```
byte[] messageBodyBytes = "Hello, World!".getBytes();
channel.basicPublish(exchangeName, routingKey, null, messageBodyBytes);
```

为了更好地控制发送，可以使用 mandatory 这个参数，或者可以发送一些特定属性的信息：

```
channel.basicPublish(exchangeName, routingKey, mandatory,
        MessageProperties.PERSISTENT_TEXT_LPAIN,
        messageBodyBytes);
```

上面这行代码发送了一条消息，这条消息的投递模式（delivery mode）设置为 2，即消息会被持久化（即存入磁盘）在服务器中。
同时这条消息的优先级（priority）设置为 1，content-type 为 "text/plain"。可以自己设定消息的属性：

```
channel.basicPublish(exchangeName, routingKey,
        new AMQP.BasicProperties.Builder()
        .contentType("text/plain")
        .deliveryMode(2)
        .priority(1)
        .userId("hidden")
        .build(),
        messageBodyBytes);
```

也可以发送一条带有 headers 的消息：

```
Map<String, Object> headers = new HashMap<String, Object>();
headers.put("location", "here");
headers.put("time", "today");
channel.basicPublish(exchangeName, routingKey,
    new AMQP.BaiscProperties.Builder()
    .builder(headers)
    .build(),
    messageBodyBytes);
```

还可以发送一条带有过期时间（expiration）的消息：

```
channel.basicPublish(exchangeName, routingKey,
    new AMQP.BasicProperties.Builder()
    .expiration("60000")
    .build()
    .messageBodyBytes);
```

对于 basicPublish 而言，有几个重载方法：

(1) `void basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body) throws IOException;`

(2) `void basicPublish(String exchange, String routingKey, boolean mandatory, BasicProperties props, byte[] body) throws IOException;`

(3) `void basicPublish(String exchange, String routingKey, boolean mandatory, boolean immediate, BasicProperties props, byte[] body) throws IOException;`

对应的具体参数解释如下所述。

* exchange：交换器名称，指明消息发送到哪个交换器中。如果设置为空字符串，则消息会被发送到 RabbitMQ 默认的交换器中。

* routingKey：路由键，交换器根据路由键将消息存储到相应的队列之中。

* props：消息的基本属性集，其包含 14 个属性成员，分别有 contentType、contentEncoding、headers(Map<String, Object>)、
deliveryMode、priority、correlationId、replyTo、expiration、messageId、timestamp、type、userId、appId、clusterId。

* byte[] body：消息体（payload），真正需要发送的消息。


## 消费消息

RabbitMQ 的消费模式分两种：推（push）和拉（pull）模式。推模式采用 Basic.Consume 进行消费，而拉模式是调用 Basic.Get 进行消费。

### 推模式

在推模式中，可以通过持续订阅的方式来消费消息，使用到的相关类有：

```
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
```

接收消息一般通过实现 Consumer 接口或者继承 DefaultConsumer 类来实现。当调用与 Consumer 相关的 API 方法时，不同的订阅采用不同的消费者标签（consumerTag）
来区分彼此，在同一个 Channel 中的消费者也需要通过唯一的消费者标签以作区分，关键消费者代码如下：


