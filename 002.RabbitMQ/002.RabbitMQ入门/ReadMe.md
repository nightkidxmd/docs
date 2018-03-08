## RabbitMQ入门

@(RabbitMQ)

[TOC]

### 1. 管理页面

`host:15672（默认）`

`默认用户名：guest 密码：guest --- 此账号仅服务器本机可访问`

### 2. AMQP基础概念

RabbitMQ主要实现的是AMQP，而MQTT等仅作为插件，所以得了解AMQP的规则

[amqp0-9-1官方文档](http://previous.rabbitmq.com/v3_5_7/resources/specs/amqp0-9-1.pdf)

#### 2.1 Connections

> The connection class provides methods for a client to establish a network connection to a server, and for both peers to operate the connection thereafter.

#### 2.2 Channel

> The channel class provides methods for a client to establish a channel to a server and for both peers to operate the channel thereafter.

#### 2.3 Exchange

> An exchange accepts messages from a producer application and routes these to message queues according to pre-arranged criteria. These criteria are called "bindings". Exchanges are matching and routing engines. That is, they inspect messages and using their binding tables, decide how to forward these messages to message queues or other exchanges. Exchanges never store messages.

`exchange命名规则:`

- Exchange names starting with "amq." are reserved for pre-declared and standardised exchanges. The client MAY declare an exchange starting with "amq." if the passive option is set, or the exchange already exists. Error code: [access-refused](http://previous.rabbitmq.com/v3_5_7/amqp-0-9-1-reference.html#constant.access-refused)
- The exchange name consists of a non-empty sequence of these characters: letters, digits, hyphen, underscore, period, or colon. Error code: [precondition-failed](http://previous.rabbitmq.com/v3_5_7/amqp-0-9-1-reference.html#constant.precondition-failed)

`exchange创建参数`

- **durable**

  If set when creating a new exchange, the exchange will be marked as durable. Durable exchanges remain active when a server restarts. Non-durable exchanges (transient exchanges) are purged if/when a server restarts.

  - The server MUST support both durable and transient exchanges.

- **auto-delete**

  If set, the exchange is deleted when all queues have finished using it.

  - The server SHOULD allow for a reasonable delay between the point when it determines that an exchange is not being used (or no longer used), and the point when it deletes the exchange. At the least it must allow a client to create an exchange and then bind a queue to it, with a small but non-zero delay between these two actions.
  - The server MUST ignore the auto-delete field if the exchange already exists.

- **internal**

  If set, the exchange may not be used directly by publishers, but only when bound to other exchanges. Internal exchanges are used to construct wiring that is not visible to applications.

#### 2.4 Queue

> Queues store and forward messages. Queues can be configured in the server or created at runtime. Queues must be attached to at least one exchange in order to receive messages from publishers.

`queue创建注意事项:`

- The server MUST create a default binding for a newly-declared queue to the default exchange, which is an exchange of type 'direct' and use the queue name as the routing key.
- The server SHOULD support a minimum of 256 queues per virtual host and ideally, impose no limit except as defined by available resources.

`queue命名规则:`

- The queue name MAY be empty, in which case the server MUST create a new queue with a unique generated name and return this to the client in the Declare-Ok method.

- Queue names starting with "amq." are reserved for pre-declared and standardised queues. The client MAY declare a queue starting with "amq." if the passive option is set, or the queue already exists. Error code: [access-refused](http://previous.rabbitmq.com/v3_5_7/amqp-0-9-1-reference.html#constant.access-refused)

- The queue name can be empty, or a sequence of these characters: letters, digits, hyphen, underscore, period, or colon. Error code: [precondition-failed](http://previous.rabbitmq.com/v3_5_7/amqp-0-9-1-reference.html#constant.precondition-failed)

  如果队列名为空，会自动生成队列名(`?由于用的SpringBoot，不清楚这个是SpringBoot做的还是RabbitMQ做的`)

  ![](./picture1.png)



`queue创建参数`

- **durable**

  If set when creating a new queue, the queue will be marked as durable. Durable queues remain active when a server restarts. Non-durable queues (transient queues) are purged if/when a server restarts. Note that durable queues do not necessarily hold persistent messages, although it does not make sense to send persistent messages to a transient queue.

  - The server MUST recreate the durable queue after a restart.
  - The server MUST support both durable and transient queues.

- **exclusive**

  Exclusive queues may only be accessed by the current connection, and are deleted when that connection closes. Passive declaration of an exclusive queue by other connections are not allowed.

  - The server MUST support both exclusive (private) and non-exclusive (shared) queues.
  - The client MAY NOT attempt to use a queue that was declared as exclusive by another still-open connection. Error code: [resource-locked](http://previous.rabbitmq.com/v3_5_7/amqp-0-9-1-reference.html#constant.resource-locked)

- **auto-delete**

  If set, the queue is deleted when all consumers have finished using it. The last consumer can be cancelled either explicitly or because its channel is closed. If there was no consumer ever on the queue, it won't be deleted. Applications can explicitly delete auto-delete queues using the Delete method as normal.

  - The server MUST ignore the auto-delete field if the queue already exists.

### 3.Producer,Exchange,Queue,Consumer关系

`Producer,Consumer不是RabbitMQ给出的基本组件，这里用来分表示消息的生产者与消费者`

`测试环境：SpringBoot`

![](./picture2.png)

> 1. 消息发送需要指定Exchange
>
> 2. Exchange可以将消息转发至多个Queue也可以转发至多个Exchange:
>
>    `如Exchange A,Exchange B`
>
> 3. 同一个Queue可以绑定多个Exchange，即可以从多个Exchange收数据
>
>    `如Exchange A 和Exchange B的消息都可以发送到Queue A`
>
> 4. Queue中的一个消息，只能被一个Consumer消费，消费后，其他Consumer无法接收
>
>    `如消息A 进入 Queue A后，Consumer A/B/C 根据优先级消费该消息，若优先级相同，目测是随机`
>
> 5. 路由转发的同一个消息无论从什么路径到达Queue，此消息在目标Queue中只有一份
>
>    `如消息A 通过Exchange A，Exchange B转入Queue A，最终Queue A里只有一份`

### 4. 四类Exchange

#### 4.1 Topic

> routing key支持模糊匹配。
>
> 通配符规则：
>
> '*'      任意单字符匹配
>
> '#'      任意字符零次或多次匹配
>
> *.stock.# 匹配 usd.stock eur.stok.db ，不匹配stock.nasdaq

#### 4.2 Direct

> routing key仅支持完全匹配

#### 4.3 Fanout

> 绑定到此Exchange的queue，绑定关系不支持参数，
>
> 数据将无条件传输到被绑定的queue

#### 4.4 Headers

>通过header参数表匹配来进行绑定，而不是routing key
>
>发送消息时不用指定exchange，而是消息带相应header参数，通过对header参数的匹配，自动找到对应的exchange, exchange再通过header参数匹配分发到对应的queue
>
>匹配规则设置
>
>x-match : all     表示参数全匹配
>
>x-match : any  表示只要有一个参数匹配即可
>
>

