# RabbitMQ-入门篇

官方文档地址：[RabbitMQ Tutorials — RabbitMQ](https://www.rabbitmq.com/getstarted.html)

## 安装
使用 docker 进行安装 RabbitMQ，具体步骤如下所示：

1. 拉取 RabbitMQ 镜像

   ```bash
   docker pull rabbitmq
   ```

2. 创建容器并设置自启动

   ```bash
   docker run -d --restart=always --name myRabbit -p 15672:15672 -p 5672:5672 rabbitmq
   ```

3. 服务器 -> 防火墙 -> 开放 15672 & 5672 端口

4. 使用 http://{ip}:15672 访问 RabbitMQ Web 端管理界面，发现无法访问！解决方案步骤如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311329466.png)

   1. 进入容器

      ```bash
      docker exec -it {容器id} /bin/bash
      ```

   2. 启用插件

      ```bash
      rabbitmq-plugins enable rabbitmq_management
      ```

5. 进入管理界面之后，出现 Stats in management UI are disabled on this node 错误信息！解决方案步骤如下所示：
   1. 进入容器

      ```bash
      docker exec -it {容器id} /bin/bash
      ```

   2. 进入 conf 目录下

      ```bash
      cd /etc/rabbitmq/conf.d/
      ```

   3. 修改management_agent.disable_metrics_collector.conf

      ```bash
      echo management_agent.disable_metrics_collector = false > management_agent.disable_metrics_collector.conf
      ```

   4. 退出容器

         ```bash
         exit
         ```

   5. 重启容器

      ```bash
      docker restart {容器id}
      ```

## 应用场景

### 应用解耦
>  提升系统的容错性和可维护性

如下图所示，如果库存系统出现问题，那么调用库存系统的订单系统也可能会出现问题，导致后面无法正常调用支付系统和物流系统。系统耦合度高，一处错误可能会导致后面无法正常执行。而且如果增加一个系统 X 的话，则需要修改订单系统的代码，订单系统的可维护性变低。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311339374.jpeg" style="zoom: 50%;" /><br />如下图所示，在使用 MQ 之后使得应用间解耦，提升容错性和可维护性。订单系统发送消息给 MQ，其他系统订阅 MQ 的消息，拿到消息后才执行。这样即使库存系统执行出现问题，也不会影响到其他系统的正常执行。而且，库存系统可能只是某几十秒或几分钟之内存在问题，后面好了的话，就可以继续从 MQ 中拿到未正常消息的消息， 重新执行。如果需要增加一个系统 X 的话，只需要让系统 X 从 MQ 中将消息取出进行消费即可。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311340437.jpeg" style="zoom:50%;" />
### 异步处理
> 提升系统的用户体验和吞吐量（响应速度）

如下图所示，一个下单操作需要耗时：20 + 300 + 300 + 300 = 920ms. 用户点击下单之后，需要等待 920ms 才能得到反馈，太慢了！<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311341794.jpeg" style="zoom:50%;" /><br />如下图所示，在使用 MQ 之后，用户点击下单之后，只需等待 25ms（20 + 5）就可以得到反馈，提升了用户体验和系统的吞吐量（单位时间内处理请求的数目）。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311341017.jpeg" style="zoom:50%;" />
### 削峰填谷
> 解决系统高并发问题，提升系统的稳定性

如下图所示，在没有 MQ 的情况下，如果请求数量瞬间增多，系统来不及处理可能会崩溃。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311341139.jpeg" style="zoom:50%;" /><br />如下图所示，在使用 MQ 之后，请求会先存储在消息队列中，然后系统再慢慢从队列中取出请求逐个进行处理。这样一来，高峰期产生的数据会被积压在队列中，高峰就被 "削" 掉了，但是因为消息的积压，在高峰期过后的一段时间内，消费消息的速度会维持恒定直至消费完队列中积压的消息，这就叫 "填谷"。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311341824.jpeg" style="zoom:50%;" />
## 基础架构
<img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311341507.jpeg"  /><br />AMQP 协议模型由三部分组成：生产者、消费者和服务端，执行流程如下所示：

1. 生产者连接到 Server，建立一个连接，开启一个信道；
2. 生产者声明交换机和消息队列，设置相关属性，并通过路由键将交换机与消息队列进行绑定；
3. 消费者也需要建立连接，开启信道等操作，并订阅相关消息队列；
4. 生产者发送消息到虚拟主机中的交换机；
5. 交换机接收到消息之后会根据路由键和不同的路由规则将消息分发到指定的一个或多个消息队列中；
6. 订阅了相关消息队列的消费者就可以获取到消息并消费；
## 核心概念

- Producer：生产者，消息提供者；
- Consumer：消费者，消息使用者；
- Message：消息，由生产者通过 RabbitMQ 发送给消费者，消息可以非常简单，也可以非常复杂。由 Properties 和 Body 组成，Properties 为外包装，可以对消息进行修饰，比如消息的优先级、延迟等高级特性，Body 为消息体内容；
- Broker：接收和分发消息的应用，RabbitMQ Server 就是 Message Broker；
- Virtual Host：出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组种，类似于网络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ Server 提供的服务时，可以划分出多个 vhost，每个用户在自己的 vhost 创建 Exchange / Queue 等；
- Connection：Producer / Consumer 和 Broker 之间的 <u>TCP 长连接</u>；
- Channel：信道，消息读写等操作都在信道中进行。如果每一次访问 RabbitMQ 时都建立一个 Connection 连接，在消息量大的时候建立 TCP 连接的开销将是巨大的，效率也较低。Channel 是在 Connection 内部建立的<u>逻辑连接</u>，如果应用程序支持多线程，通常每个 Thread 创建单独的 Channel 进行通讯，AMQP method 包含了 Channel Id 帮助客户端和 Broker 识别 Channel，所以 Channel 之间是完全隔离的。Channel 作为轻量级的 Connection 极大减少了操作系统建立 TCP Connection 的开销；
- Routing Key：路由键，交换机会根据路由键来决定消息应该分发到哪个队列；
- Queue：队列，消息存放的容器，消息先进先出，等待被 Consumer 消费；
- Binding：是将 Exchange 和 Queue 进行关联的过程 = Routing Key + Exchange + Queue；当 Exchange 与 Queue 创建完成之后，需要使用 Routing Key（aka Binding Key，绑定键） 将它们俩绑定起来。属于一种多对多的关系 => 一个 Exchange 可以与多个 Queue 进行绑定，一个 Queue 又可以与多个 Exchange 进行绑定，甚至，一个 Queue 可以与 同一个 Exchange 进行多次绑定，只不过每次绑定时的 Routing Key 并不相同；
- Binding Key：其实也属于路由键中的一种，官方解释为：the routing key to use for the binding，翻译：在绑定时使用的路由键。大多数时候，包括官方文档和 RabbitMQ Java API 中都会把 Binding Key 看作 Routing Key，为了避免混淆，可以这么理解：
   - 交换机与消息队列进行绑定时的路由键为 Binding Key。涉及的客户端方法为：channel.exchangeBind、channel.queueBind，对应的AMQP命令为：Exchange.Bind、Queue.Bind；
   - 客户端发送消息给交换机时的路由键是 Routing Key。涉及的客户端方法为：channel.basicPublish，对应的AMQP命令为：Basic.Publish；
- Exchange：消息（Message）到达 Broker 的第一站，主要<u>根据路由键和不同的路由规则（对应四种不同类型的交换机）将消息分发到指定的一个或多个 Queue 中，供订阅了相关 Queue 的消费者进行消费</u>。
## 四种交换机类型
![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311342422.jpeg)<br />[How RabbitMQ Works and RabbitMQ Core Concepts](https://www.javaguides.net/2018/12/how-rabbitmq-works-and-rabbitmq-core-concepts.html)
### 扇形交换机
fanout Exchange（multicast）：不需要进行路由键的匹配，即无视路由键，直接将接收到的消息路由到每一个与其绑定的消息队列中（速度最快），相当于广播 | 群发；<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311342507.jpeg" style="zoom: 67%;" /><br />如上图所示，扇形交换机 X 与 Q1、Q2 两个队列进行了绑定。无论消息中的路由键是什么，都会将消息路由到与之绑定的两个队列 Q1 和 Q2 中。

> [!DEMO|label:用于演示 Fanout Exchange 的案例构建一个简单的日志系统]
>
> 它包含两个程序：第一个程序负责发送日志消息，第二个程序负责获取消息并输出内容。在该日志系统中，所有正在运行的接收方程序都会接收消息。咱们用其中一个接收者把日志写入硬盘中，另外一个接收者把日志输出到屏幕上。最终，日志消息被广播给所有的接收者。如下图所示：<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311346946.jpeg" style="zoom:67%;" />	生产者：
>
> ```java
> public class EmitLog {
>     private static final String EXCHANGE_NAME = "logs";
> 
>     public static void main(String[] argv) throws Exception {
>         ConnectionFactory factory = new ConnectionFactory();
>         factory.setHost("42.194.233.222");
>         try (Connection connection = factory.newConnection();
>              Channel channel = connection.createChannel()) {
>             channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT);
>             String message = "info: Hello World!";
>             channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes(StandardCharsets.UTF_8));
>             System.out.println(" [x] Sent '" + message + "'");
>         }
>     }
> }
> ```
>
> 消费者1：将收到的日志消息打印在控制台
>
> ```java
> public class ReceiveLogs1 {
>     private static final String EXCHANGE_NAME = "logs";
> 
>     public static void main(String[] argv) throws Exception {
>         ConnectionFactory factory = new ConnectionFactory();
>         factory.setHost("42.194.233.222");
>         Connection connection = factory.newConnection();
>         Channel channel = connection.createChannel();
> 
>         // 声明一个名称为 "logs" 的扇形交换机
>         channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT);
>         // 声明一个临时队列，队列的名称是随机的，当没有消费者监听该队列时，队列会自动删除
>         String queueName = channel.queueDeclare().getQueue();
>         // 将队列与交换机进行绑定，此时绑定用的路由键为空字符串
>         channel.queueBind(queueName, EXCHANGE_NAME, "");
> 
>         System.out.println(" [*] Waiting for messages");
> 
>         DeliverCallback deliverCallback = (consumerTag, delivery) -> {
>             String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
>             System.out.println(" [x] Received '" + message + "'");
>         };
>         channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
>     }
> }
> ```
>
> 消费者2：将收到的日志消息存储到文件中
>
> ```java
> public class ReceiveLogs2 {
>     private static final String EXCHANGE_NAME = "logs";
> 
>     public static void main(String[] argv) throws Exception {
>         ConnectionFactory factory = new ConnectionFactory();
>         factory.setHost("42.194.233.222");
>         Connection connection = factory.newConnection();
>         Channel channel = connection.createChannel();
> 
>         // 声明一个名称为 "logs" 的扇形交换机
>         channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT);
>         // 声明一个临时队列，队列的名称是随机的，当没有消费者监听该队列时，队列会自动删除
>         String queueName = channel.queueDeclare().getQueue();
>         // 将队列与交换机进行绑定，此时绑定用的路由键为空字符串
>         channel.queueBind(queueName, EXCHANGE_NAME, "");
> 
>         System.out.println(" [*] Waiting for messages");
> 
>         DeliverCallback deliverCallback = (consumerTag, delivery) -> {
>             String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
>             System.out.println(" [x] Received '" + message + "'");
>             try (BufferedWriter bufferedWriter = new BufferedWriter(new FileWriter("F:\\logs_from_rabbit.log"))) {
>                 bufferedWriter.write(message);
>             }
>             System.out.println(" [x] write to the file succeeded");
>         };
>         channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
>     }
> }
> ```
>
> 首先运行两个消费者程序，开始等待生产者发送消息，运行生产者程序时会发送一条消息 "info: Hello World!"。运行结果如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311347880.png)![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311347294.png)![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311347844.png)

### 直连交换机
Direct Exchange（point-to-point）：<u>精准匹配</u>，当消息中的路由键和交换机与消息队列绑定关系中的绑定键完全一致时，消息才会路由到该队列；<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311348826.jpeg" style="zoom:67%;" /><br />如上图所示，直连交换机 X 与 Q1、Q2 两个队列进行了绑定，Q1 使用 orange 作为绑定键，Q2 同时使用 black 和 green 作为绑定键。<br />这样一来，当路由键为 orange 的消息发布到交换机时，就会被路由到队列 Q1；路由键为 black 或 green 的消息就会路由到 Q2；其他的消息都将会被丢弃。<br />多个绑定（Mutilple bindings）<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311348844.jpeg" style="zoom:67%;" /><br />多个队列使用相同的绑定键是合法的。在该例子中，添加一个 X 与 Q1 之间的绑定，使用 black 作为绑定键。这样一来，直连交换机就和扇形交换机的行为一样，会将消息广播到所有匹配的队列。带有 black 路由键的消息会同时发送到 Q1 和 Q2。

> [!DEMO|label:用于演示 Direct Exchange 的案例]
>
> 在前面的 Fanout Exchange 案例中，实现了一个简单的日志系统，可以把日志消息广播给多个接收者。在本案例中新增一个功能 -- 使得它能够只订阅消息的一个子集，例如，只需要把严重的错误日志信息写入日志文件中，但同时把所有的日志信息输出到控制台中。如下图所示：<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311349567.jpeg" style="zoom:67%;" /><br />	生产者：
>
> ```java
> public class EmitLogDirect {
>     private static final String EXCHANGE_NAME = "direct_logs";
> 
>     public static void main(String[] args) throws Exception {
>         ConnectionFactory connectionFactory = new ConnectionFactory();
>         connectionFactory.setHost("42.194.233.222");
>         try (Connection connection = connectionFactory.newConnection();
>              Channel channel = connection.createChannel()) {
>             // 声明一个名称为 "direct_logs" 的直连交换机
>             channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
>             Map<String, String> map = new HashMap<>(4);
>             map.put("info", "普通 info 信息");
>             map.put("warning", "警告 warning 信息");
>             map.put("error", "错误 error 信息");
>             // 由于找不到绑定键=debug的队列，那么该消息将会被丢弃
>             map.put("debug", "调试 debug 信息");
>             for (Map.Entry<String, String> entry: map.entrySet()) {
>                 String routingKey = entry.getKey();
>                 String message = entry.getValue();
>                 channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes(StandardCharsets.UTF_8));
>                 System.out.println(" [x] Sent '" + routingKey + "':'" + message + "'");
>             }
>         }
>     }
> }
> ```
>
> 消费者1：将收到的日志消息打印在控制台
>
> ```java
> public class ReceiveLogsDirect1 {
>     private static final String EXCHANGE_NAME = "direct_logs";
> 
>     public static void main(String[] argv) throws Exception {
>         ConnectionFactory factory = new ConnectionFactory();
>         factory.setHost("42.194.233.222");
>         Connection connection = factory.newConnection();
>         Channel channel = connection.createChannel();
> 
>         // 声明一个名称为 "direct_logs" 的扇形交换机
>         channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
>         // 声明一个临时队列，队列的名称是随机的，当没有消费者监听该队列时，队列会自动删除
>         String queueName = channel.queueDeclare().getQueue();
>         // 将队列与交换机进行绑定，此时绑定用的路由键分别为 "info", "warning", "error"
>         for (String bindingKey : new String[]{"info", "warning", "error"}) {
>             channel.queueBind(queueName, EXCHANGE_NAME, bindingKey);
>         }
> 
>         System.out.println(" [*] Waiting for messages");
> 
>         DeliverCallback deliverCallback = (consumerTag, delivery) -> {
>             String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
>             System.out.println(" [x] Received '" +
>                     delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
>         };
>         channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
>     }
> }
> ```
>
> 消费者2：将收到的错误日志信息存储到文件中
>
> ```java
> public class ReceiveLogsDirect2 {
>     private static final String EXCHANGE_NAME = "direct_logs";
> 
>     public static void main(String[] argv) throws Exception {
>         ConnectionFactory factory = new ConnectionFactory();
>         factory.setHost("42.194.233.222");
>         Connection connection = factory.newConnection();
>         Channel channel = connection.createChannel();
> 
>         // 声明一个名称为 "direct_logs" 的扇形交换机
>         channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
>         // 声明一个临时队列，队列的名称是随机的，当没有消费者监听该队列时，队列会自动删除
>         String queueName = channel.queueDeclare().getQueue();
>         // 将队列与交换机进行绑定，此时绑定用的路由键为 "error"
>         channel.queueBind(queueName, EXCHANGE_NAME, "error");
> 
>         System.out.println(" [*] Waiting for messages");
> 
>         DeliverCallback deliverCallback = (consumerTag, delivery) -> {
>             String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
>             System.out.println(" [x] Received '" +
>                     delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
>             try (BufferedWriter bufferedWriter = new BufferedWriter(new FileWriter("F:\\logs_from_rabbit.log"))) {
>                 bufferedWriter.write(message);
>             }
>             System.out.println(" [x] write to the file succeeded");
>         };
>         channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
>     }
> }
> ```
>
> 首先运行两个消费者程序，开始等待生产者发送消息，运行生产者程序时总共会发送四条日志消息，其中的 debug 日志信息将会被丢弃。运行结果如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311351361.png)![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311351826.png)![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311351311.png)

### 主题交换机
Topic Exchange：<u>模糊匹配</u>，当消息中的路由键和交换机与消息队列绑定关系中的绑定键满足<u>通配符</u>匹配时（最为灵活），消息将会路由到该队列；<br />规定如下所示：

- Routing Key：必须是一个由`.`分隔开的词语列表，如 "stock.usd.nyse","nyse.vmw","quick.orange.rabbit"，词语的个数可以随意，但是不要超过 255 字节
- Binding Key：与 Routing Key 一样也是由`.`分隔开的词语列表，存在如下两种特殊的字符串`*` 和`#`，用作模糊匹配；
   - `*`(星号) 用来表示一个单词；
   - `#` (井号) 用来表示任意数量（零个或多个）单词；

<img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311351990.jpeg" style="zoom:67%;" /><br />发送的所有消息都是用来描述小动物的，消息中的 Routing Key = <celerity>.<colour>.<species> = 敏捷程度.颜色.种类<br />Q1的绑定键 = `*.orange.*`，表示 Q1 对所有橘黄色的动物都感兴趣；<br />Q2 的绑定键 = `*.*.rabbit`和`lazy.#`，表示 Q2 对所有的兔子和所有懒惰的动物感兴趣；<br />举个栗子：

- Routing Key =`quick.orange.rabbit`或`lazy.orange.elephant`的消息将会被分别投递给 Q1 和 Q2；
- Routing Key =`quick.orange.fox`的消息只会投递给 Q1，一个携带路由键为`lazy.brown.fox`的消息只会投递给 Q2；
- Routing Key =`lazy.pink.rabbit`的消息只会投递给 Q2 一次，即使它同时匹配 Q2 的两个绑定；
- Routing Key = `quick.brown.fox`的消息不会投递给任何一个队列，消息将会被丢弃;
- Routing Key = `lazy.orange.male.rabbit`即使有四个单词，但是会匹配最后一个绑定，所以消息将会投递给 Q2；

主题交换机可以表现出跟其他交换机类似的行为，如下所示：

- 当一个队列的绑定键为`#`时，该队列将会无视消息中的路由键，接收所有的消息；
- 当`*`与`#`两个通配符都未在绑定键中出现时，此时 Topic Exchange = Direct Exchange；

> [!DEMO|label:用于演示 Topic Exchange 的案例]
>
> 在前面的 Direct Exchange 的案例中，咱们改进了日志系统，没有使用只能进行广播的扇形交换机（Fanout Exchange），而是使用的直连交换机（Direct Exchange），实现有选择性地接收日志信息。尽管使用直连交换机改进了日志系统，但是它任然有局限性 -- 不能根据多个标准进行路由。在咱们的日志系统中，咱们可能不仅想根据严重程度来订阅日志，还想根据发出日志的来源来订阅日志，这个概念来自 [syslog](http://en.wikipedia.org/wiki/Syslog) unix 工具，它基于严重程度（info/warn/crit...）和设施（auth/cron/kern...）来路由日志。这将给予咱们很大的灵活性，比如咱们可能指向监听来自 "cron" 的关键错误，但也想监听来自 "kern" 的所有日志。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311353516.jpeg" style="zoom:67%;" /><br />	生产者：
>
> ```java
> public class EmitLogTopic {
>     private static final String EXCHANGE_NAME = "topic_logs";
> 
>     public static void main(String[] argv) throws Exception {
>         ConnectionFactory connectionFactory = new ConnectionFactory();
>         connectionFactory.setHost("42.194.233.222");
>         try (Connection connection = connectionFactory.newConnection();
>              Channel channel = connection.createChannel()) {
>             // 声明一个名称为 "direct_logs" 的直连交换机
>             channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
>             Map<String, String> map = new HashMap<>(2);
>             map.put("kern.critical", "一个关键性的内核错误");
>             map.put("cron.warn", "一个警告级别的工作排程服务错误");
>             for (Map.Entry<String, String> entry: map.entrySet()) {
>                 String routingKey = entry.getKey();
>                 String message = entry.getValue();
>                 channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes(StandardCharsets.UTF_8));
>                 System.out.println(" [x] Sent '" + routingKey + "':'" + message + "'");
>             }
>         }
>     }
> }
> ```
>
> 消费者1：将收到的所有日志消息打印在控制台
>
> ```java
> public class ReceiveLogsTopic1 {
>     private static final String EXCHANGE_NAME = "topic_logs";
> 
>     public static void main(String[] argv) throws Exception {
>         ConnectionFactory factory = new ConnectionFactory();
>         factory.setHost("42.194.233.222");
>         Connection connection = factory.newConnection();
>         Channel channel = connection.createChannel();
> 
>         // 声明一个名称为 "topic_logs" 的主题交换机
>         channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
>         // 声明一个临时队列，队列的名称是随机的，当没有消费者监听该队列时，队列会自动删除
>         String queueName = channel.queueDeclare().getQueue();
>         // 将队列与交换机进行绑定，此时绑定用的路由键为 "#"
>         channel.queueBind(queueName, EXCHANGE_NAME, "#");
> 
>         System.out.println(" [*] Waiting for messages");
> 
>         DeliverCallback deliverCallback = (consumerTag, delivery) -> {
>             String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
>             System.out.println(" [x] Received '" +
>                     delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
>         };
>         channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
>     }
> }
> ```
>
> 消费者2：只会将收到的 "cron" 设施中的关键错误日志信息和 "kern" 设施中的所有日志信息存储到文件中
>
> ```java
> public class ReceiveLogsTopic2 {
>     private static final String EXCHANGE_NAME = "topic_logs";
> 
>     public static void main(String[] argv) throws Exception {
>         ConnectionFactory factory = new ConnectionFactory();
>         factory.setHost("42.194.233.222");
>         Connection connection = factory.newConnection();
>         Channel channel = connection.createChannel();
> 
>         // 声明一个名称为 "topic_logs" 的主题交换机
>         channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
>         // 声明一个临时队列，队列的名称是随机的，当没有消费者监听该队列时，队列会自动删除
>         String queueName = channel.queueDeclare().getQueue();
>         // 将队列与交换机进行绑定，此时绑定用的路由键分别为 "cron.critical"、"kern.*"
>         for (String bindingKey : new String[]{"cron.critical", "kern.*"}) {
>             channel.queueBind(queueName, EXCHANGE_NAME, bindingKey);
>         }
> 
>         System.out.println(" [*] Waiting for messages");
> 
>         DeliverCallback deliverCallback = (consumerTag, delivery) -> {
>             String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
>             System.out.println(" [x] Received '" +
>                     delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
>             try (BufferedWriter bufferedWriter = new BufferedWriter(new FileWriter("F:\\logs_from_rabbit.log"))) {
>                 bufferedWriter.write(message);
>             }
>             System.out.println(" [x] write to the file succeeded");
>         };
>         channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
>     }
> }
> ```
>
> 首先运行两个消费者程序，开始等待生产者发送消息，运行生产者程序时总共会发送两条日志消息。运行结果如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311355443.png)![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311355574.png)![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311355737.png)

### 头交换机
Headers Exchange：<span style="background-color: rgb(251, 228, 231);">TODO</span>
## 六种工作模式

> [!IMPORTANT]
>
> **默认交换机**：由 RabbitMQ 预先声明好名字为空字符串的直连交换机（direct exchange），**每个新建的队列（queue）都会自动绑定到默认交换机上**，**绑定时的绑定键（binding key）名称与队列名称相同**。

### 简单模式（Hello World）
一个生产者对应一个消费者。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311358426.jpeg" style="zoom:67%;" />

> [!DEMO|label:用于演示简单模式的案例]
>
> 生产者：
>
> ```java
> public class Send {
>  private final static String QUEUE_NAME = "hello";
> 
>  public static void main(String[] args) throws Exception {
>      // 创建连接工厂
>      ConnectionFactory connectionFactory = new ConnectionFactory();
>      // 主机地址，默认是 "localhost"
>      connectionFactory.setHost("42.194.233.222");
>      // 通过连接工厂获取一个连接 & 使用连接创建一个通道
>      try (Connection connection = connectionFactory.newConnection();
>           Channel channel = connection.createChannel()) {
>          // 声明一个队列，名称为 "hello"
>          channel.queueDeclare(QUEUE_NAME, false, false, false, null);
>          String message = "Hello World!";
>          // 使用默认交换机分发消息给名称为 "hello" 的队列
>          channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
>          System.out.println(" [x] Sent '" + message + "'");
>      }
>  }
> }
> ```
>
> 消费者：
>
> ```java
> public class Recv {
>  private final static String QUEUE_NAME = "hello";
> 
>  public static void main(String[] args) throws Exception {
>      ConnectionFactory connectionFactory = new ConnectionFactory();
>      connectionFactory.setHost("42.194.233.222");
>      Connection connection = connectionFactory.newConnection();
>      Channel channel = connection.createChannel();
> 
>      channel.queueDeclare(QUEUE_NAME, false, false, false, null);
>      System.out.println(" [*] Waiting for messages");
> 
>      DeliverCallback deliverCallback = (consumerTag, delivery) -> {
>          String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
>          System.out.println(" [x] Received '" + message + "'");
>      };
>      channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });
>  }
> }
> ```
>
> 首先启动消费者程序，开始等待生产者发送消息，运行生产者程序时会发送一条消息 "Hello World!"。运行结果如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311400955.png)![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311400026.png)
> 
> > 虽然多次使用`channel.queueDeclare(QUEUE_NAME)`命令来创建同一个队列，但是只有一个队列会被真正创建！为什么要<u>重复声明队列</u>呢？如果咱们确定队列已经存在的话，那么咱们可以不这么做，比如此前预先运行了生产者程序。可是咱们并不确定哪个程序会先运行。在这种情况下，在程序中重复声明一下是种值得推荐的做法！

### 工作队列（Work Queues）

- 工作队列，提高消息的处理速度，避免队列消息堆积；
- 一个生产者对应多个消费者；
- （一个）队列中的一条消息只能被其中某一个消费者进行消费，无法被多次消费！

<img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311404408.jpeg" style="zoom:67%;" /><br />在本小节中，咱们将创建一个工作队列（Work Queue），它会发送一些耗时的任务给多个工作者（Worker）。<br />工作队列（又称：任务队列 -- Task Queues）是为了避免等待一些占用大量资源、时间的操作。当咱们把任务（Task）当作消息发送到队列中，一个运行在后台的工作者（Worker）进程就会取出任务然后处理。当运行多个工作者（Workers)时，任务就会在它们之间共享。<br />这个概念在网络应用中是非常有用的，它可以在短暂的 HTTP 请求中处理一些复杂的任务。

> [!DEMO|label:用于演示工作队列模式的案例]
>
> 在简单模式的案例中，咱们发送了一个包含 "Hello World!" 的字符串消息。现在，咱们将发送一些字符串，把这些字符串当作复杂的任务。因为没有真实的例子，例如图片缩放、pdf 文件转换等，所以使用`Thread.sleep()`函数来模拟这种情况。咱们在字符串中加上点号`.`来表示任务的复杂程度，一个点`.`将会耗时 1 秒钟，比如 "Hello..." 就会耗时 3 秒钟。<br />	生产者：对之前简单模式案例中的生产者做些简单的调整，以便可以发送任意消息
>
> ```java
> public class NewTask {
>     private final static String TASK_QUEUE_NAME = "hello";
> 
>     public static void main(String[] args) throws Exception {
>         // 创建连接工厂
>         ConnectionFactory connectionFactory = new ConnectionFactory();
>         // 主机地址，默认是 "localhost"
>         connectionFactory.setHost("42.194.233.222");
>         // 通过连接工厂获取一个连接 & 使用连接创建一个通道
>         try (Connection connection = connectionFactory.newConnection();
>              Channel channel = connection.createChannel()) {
>             // 声明一个队列，名称为 "hello"
>             channel.queueDeclare(TASK_QUEUE_NAME, false, false, false, null);
>             Scanner scanner = new Scanner(System.in);
>             while (scanner.hasNext()) {
>                 // 按回车结束
>                 String message = scanner.nextLine();
>                 // 使用默认交换机分发消息给名称为 "hello" 的队列
>                 channel.basicPublish("", TASK_QUEUE_NAME, null, message.getBytes());
>                 System.out.println(" [x] Sent '" + message + "'");
>             }
>         }
>     }
> }
> ```
>
> 消费者：
>
> ```java
> public class Worker {
>     private final static String TASK_QUEUE_NAME = "hello";
> 
>     public static void main(String[] args) throws Exception {
>         ConnectionFactory connectionFactory = new ConnectionFactory();
>         connectionFactory.setHost("42.194.233.222");
>         Connection connection = connectionFactory.newConnection();
>         Channel channel = connection.createChannel();
> 
>         channel.queueDeclare(TASK_QUEUE_NAME, false, false, false, null);
>         System.out.println(" [*] Waiting for messages");
> 
>         DeliverCallback deliverCallback = (consumerTag, delivery) -> {
>             String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
> 
>             System.out.println(" [x] Received '" + message + "'");
>             try {
>                 doWork(message);
>             } finally {
>                 System.out.println(" [x] Done");
>             }
>         };
>         channel.basicConsume(TASK_QUEUE_NAME, true, deliverCallback, consumerTag -> {
>         });
>     }
> 
>     private static void doWork(String task) {
>         for (char ch : task.toCharArray()) {
>             if (ch == '.') {
>                 try {
>                     Thread.sleep(1000);
>                 } catch (InterruptedException e) {
>                     Thread.currentThread().interrupt();
>                 }
>             }
>         }
>     }
> }
> ```
>
> 在运行消费者程序之前，编辑运行配置信息，设置允许运行多个实例，如下图所示：<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311406939.png" alt="image.png" style="zoom: 80%;" />

#### 循环调度
使用工作队列的一个好处就是它能够并行的处理队列。如果堆积了很多任务，咱们只需要添加更多的工作者（workers）即可，扩展非常简单。<br />现在，咱们首先启动两个消费者程序，开始等待生产者发送消息，然后再启动生产者程序，发送一些消息给消费者（consumers），如下所示：
```
First message.
Second message..
Third message...
Fourth message....
Fifth message.....
```
![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311407440.png)<br />两个消费者程序运行结果分别如下所示：<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311407852.png" alt="image.png" style="zoom:67%;" />   <img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311407953.png" alt="image.png" style="zoom:67%;" /><br />默认情况下，RabbitMQ 会按顺序将每个消息发送给下一个消费者。平均而言，每个消费者都会收到同等数量的消息，这种分配消息的方式被称为**轮询**（round-robin)。
#### 消息确认
消费者完成一个任务可能需要一段时间，如果其中一个消费者在处理一个长时间的任务时还没完成就突然挂掉了。在咱们当前的代码中，一旦 RabbitMQ 将消息传递给消费者，它就会立即将其标记为删除。在这种情况下，如果突然有个消费者挂掉了的话，那么它刚刚正在处理的消息就会丢失，以及后续发送给该消费者但尚未处理的消息也都会丢失，因为它无法接收到。<br />但咱们不想丢失任何消息。如果一个消费者挂掉了，咱们希望消息会重新发送给其他的消费者。<br />为了确保消息不会丢失，RabbitMQ 支持消息确认，即<u>消息应答机制</u>。确认由消费者发回，以告诉 RabbitMQ 已经收到并处理了某一特定消息，然后 RabbitMQ 就可以自由删除该消息。<br />如果消费者挂掉了（其通道关闭、连接关闭或 TCP 连接丢失）而没有发送确认，RabbitMQ 就会认为消息尚未被完全处理，并将对其进行<u>重新入队</u>。如果同时有其他消费者在线的话，它就会迅速将其重新发送给其他的消费者。这样，即使消费者偶尔挂掉，也可以确保不会丢失任何消息。<br />在消费者交付确认时，会强制执行一个超时（默认为 30 分钟，可以根据实际情况进行调整）。这样有助于发现那些从不确保交付的有问题的（卡住的）消费者。<br />默认情况下，<u>消息手动确认</u>是打开的。只是在前面的案例中，咱们通过`autoAck = true`标志明确关闭了消息手动确认，使用自动应答。现在是时候将这个标志设置为假，一旦完成一个任务，就从消费者那里发送一个适当的确认。
```java
public class Worker2 {
    private final static String TASK_QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("42.194.233.222");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] Waiting for messages");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);

            System.out.println(" [x] Received '" + message + "'");
            try {
                doWork(message);
            } finally {
                System.out.println(" [x] Done");
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            }
        };
        channel.basicConsume(TASK_QUEUE_NAME, false, deliverCallback, consumerTag -> {
        });
    }

    private static void doWork(String task) {
        for (char ch : task.toCharArray()) {
            if (ch == '.') {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        }
    }
}
```
在启动消费者程序之前，同样需要编辑运行配置，设置允许运行多个实例。<br />开启两个消费者程序，开始等待生产者发送消息，生产者发送消息`Fifth message.....`，假设消费者1在收到消息之后，过一秒钟之后停掉消费者1程序（模拟消息尚未被完全处理，消费者1就挂掉了的情况），此时消费者1并没有发送确认就挂掉了，消息会重新入队，此时发现消费者2在线，就会迅速将消息发送给消费者2进行消费。如下所示：<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311409052.png" alt="image.png" style="zoom:67%;" /> <img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311409960.png" alt="image.png" style="zoom:67%;" /> <img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311409644.png" alt="image.png" style="zoom:67%;" />
#### 消息持久化
咱们已经可以在消费者挂掉的情况下也能确保消息不会丢失，但是当 RabbitMQ 服务器退出或者崩溃时，消息还是会丢失，那么此时该如何确保消息不会丢失呢？需要做两件事：将<u>队列</u>和<u>消息</u>都标记为<u>持久化</u>。<br />如何确保队列在 RabbitMQ 重启之后任然存在呢？需要在声明队列的时候将`durable`参数设置为`true`，将队列标记为持久化。
```java
boolean durable = true;
channel.queueDeclare("hello", durable, false, false, null);
```
尽管这行代码本身是正确的，但是仍然不会正确运行。因为咱们已经定义过一个名为 "hello" 的非持久化队列。RabbitMQ 不允许使用不同的参数重新定义一个队列，不然的话它会返回一个错误，如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311410625.png)<br />咱们可以把原先队列先删除或者重新创建一个名字不相同的持久化队列，例如`tast_queue`：
```java
boolean durable = true;
channel.queueDeclare("task_queue", durable, false, false, null);
```
❗注意：这个`queueDeclare`的更改需要同时应用于生产者和消费者代码。<br />这个时候即使 RabbitMQ 重新启动，task_queue 队列也依然存在。<br />现在，需要将消息标记为持久化，可以将 delivery_mode 属性设为2，即发送消息的时候使用 MessageProperties.PERSISTENT_TEXT_PLAIN
```java
channel.basicPublish("", "task_queue", MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
```
> [!ATTENTION]
> 将消息设为持久化<u>并不能完全保证不会丢失消息</u>。尽管它告诉 RabbitMQ 将消息保存到磁盘，但是从 RabbitMQ 收到消息到保存之间仍有一个短暂的时间间隔。因为 RabbitMQ 并没有为每条消息做 fsync(2) -- 它可能只是被保存到缓存中，而不是真正写入磁盘中，并不能保证真正的持久化，但已经足够应付咱们的简单工作队列。如果你一定要保证持久化的话，那么可以使用发布者确认。

#### 公平调度
你可能已经注意到，调度仍然不能完全按照咱们的要求工作。例如，有两个消费者，处理奇数消息的比较繁忙，处理偶数消息的很轻松，一个消费者累死累活，而另一个则几乎什么都不用做。但是 RabbitMQ 对此一无所知，并且仍然会均匀地派发消息。发生这种情况是因为 RabbitMQ 只管分发消息，而不会查看一个消费者的未确认消息的数量，盲目地将第 n-th 条消息分发给第 n-th 个消费者。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311411069.jpeg" style="zoom:67%;" /><br />prefetch：预取，默认值大小为 250，<span style="background-color: rgb(251, 228, 231);">消费者不管自己的处理能力如何，就轮流把消息从消息队列中提前取出来</span>。<br />为了解决这个问题，咱们可以使用带有`prefetchCount = 1`设置的`basicQos()`方法。这样就是告诉 RabbitMQ，一次不要发送多于一条消息给消费者 => 在它处理并确认上一条消息之前，不要将新的消息分派给该消费者,这样，RabbitMQ 就会把新的消息分发给下一个空闲的消费者 => 消费者每次只能从队列中预取一条消息，只有等消费者处理完并返回确认后才能继续取下一条消息。
```java
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```
> [!DEMO|label:用于演示工作队列模式的案例的最终代码实现]
>
> 生产者：
>
> ```java
> public class NewTask {
>     private final static String TASK_QUEUE_NAME = "task_queue";
> 
>     public static void main(String[] args) throws Exception {
>         // 创建连接工厂
>         ConnectionFactory connectionFactory = new ConnectionFactory();
>         // 主机地址，默认是 "localhost"
>         connectionFactory.setHost("42.194.233.222");
>         // 通过连接工厂获取一个连接 & 使用连接创建一个通道
>         try (Connection connection = connectionFactory.newConnection();
>              Channel channel = connection.createChannel()) {
>             // 声明一个队列，名称为 "hello"
>             channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
>             Scanner scanner = new Scanner(System.in);
>             while (scanner.hasNext()) {
>                 // 按回车结束
>                 String message = scanner.nextLine();
>                 // 使用默认交换机分发消息给名称为 "hello" 的队列
>                 channel.basicPublish("", TASK_QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
>                 System.out.println(" [x] Sent '" + message + "'");
>             }
>         }
>     }
> }
> ```
>
> 消费者：
>
> ```java
> public class Worker2 {
>     private final static String TASK_QUEUE_NAME = "task_queue";
> 
>     public static void main(String[] args) throws Exception {
>         ConnectionFactory connectionFactory = new ConnectionFactory();
>         connectionFactory.setHost("42.194.233.222");
>         Connection connection = connectionFactory.newConnection();
>         Channel channel = connection.createChannel();
> 
>         channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
>         System.out.println(" [*] Waiting for messages");
> 
>         channel.basicQos(1);
> 
>         DeliverCallback deliverCallback = (consumerTag, delivery) -> {
>             String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
> 
>             System.out.println(" [x] Received '" + message + "'");
>             try {
>                 doWork(message);
>             } finally {
>                 System.out.println(" [x] Done");
>                 channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
>             }
>         };
>         channel.basicConsume(TASK_QUEUE_NAME, false, deliverCallback, consumerTag -> {
>         });
>     }
> 
>     private static void doWork(String task) {
>         for (char ch : task.toCharArray()) {
>             if (ch == '.') {
>                 try {
>                     Thread.sleep(1000);
>                 } catch (InterruptedException e) {
>                     Thread.currentThread().interrupt();
>                 }
>             }
>         }
>     }
> }
> ```



### 发布订阅（Publish/Subscribe）
生产者先将消息发送到 [扇形交换机（Fanout Exchange）](#扇形交换机)，再将消息分发给每一个与其绑定的消息队列中，然后被监听这些队列的消费者消费。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311413099.jpeg" style="zoom:67%;" />
### 路由模式（Routing）
生产者先将消息发送到 [直连交换机（Direct Exchange）](#直连交换机)，当发送消息时的路由键（routing key）和交换机与消息队列绑定关系中的绑定键（binding key）<u>完全一致</u>时，消息就会路由到对应的队列，然后被监听这些队列的消费者消费，<u>让消费者有选择性的接收消息</u>。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311413907.jpeg" style="zoom:67%;" />
### 主题模式（Topics）
生产者先将消息发送到 [主题交换机（Topic Exchange）](#主题交换机)，当发送消息时的路由键（routing key）和交换机与消息队列绑定关系中的绑定键（binding key）满足<u>通配符</u>匹配时，消息就会路由到对应的队列，然后被监听这些队列的消费者消费。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307311413708.jpeg" style="zoom:67%;" />
### RPC
<span style="background-color: rgb(251, 228, 231);">TODO</span>

