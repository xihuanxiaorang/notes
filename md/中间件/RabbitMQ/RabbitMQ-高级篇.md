# RabbitMQ-高级篇

<a name="VPfcB"></a>

## Spring AMQP
🤔 何为 AMQP？<br />🤓 Advanced Message Queuing Protocol，是用于在应用程序之间传递业务消息的开放标准。该协议与语言和平台无关，更符合微服务中独立性的要求。<br />🤔 何为 Spring AMQP？[Spring AMQP](https://spring.io/projects/spring-amqp)<br />🤓 Spring AMQP 是基于 AMQP 协议定义的一套 API 规范，提供模板来发送和接收消息。包含两部分：spring-amqp 是基础抽象，spring-rabbit 是底层的默认实现。
<a name="brwqD"></a>
### 简单模式（Hello World）
案例：利用 Spring AMQP 实现 Hello World 案例，具体实现步骤如下所示：

1. 消费者
   1. 创建一个消费者（consumer）的 springboot 工程；

   2. 在 pom.xml 文件中添加如下依赖：

      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-amqp</artifactId>
          <version>2.7.12</version>
      </dependency>
      ```

   3. 在 application.yml 配置文件中增加 RabbitMQ 相关配置信息：

      ```yaml
      spring:
        rabbitmq:
          host: 42.194.233.222
          port: 5672
          virtual-host: /
          username: guest
          password: guest
      ```

   4. 创建一个 SpringAmqpConfiguration 配置类；

      1. 声明一个名为 "simple.queue" 的队列；
      2. 定义一个消费者来监听和消费 "simple.queue" 队列中的消息

      ```java
      @Configuration
      public class SpringAmqpConfiguration {
          private static final Logger LOGGER = LoggerFactory.getLogger(SpringAmqpConfiguration.class);
      
          @Bean
          public Queue simpleQueue() {
              return new Queue("simple.queue");
          }
      
          @RabbitListener(queues = {"simple.queue"})
          public void listenSimpleQueueMessage(String msg) {
              LOGGER.info("从 {} 队列中收到一条消息：{}", "simple.queue", msg);
          }
      }
      ```
2. 生产者
   
   1. 创建一个生产者（producer）的 springboot 工程；
   
   2. 在 pom.xml 文件中添加如下依赖：
   
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-amqp</artifactId>
          <version>2.7.12</version>
      </dependency>
      ```
   
   3. 在 application.yml 配置文件中增加 RabbitMQ 相关配置信息：
   
      ```yaml
      spring:
        rabbitmq:
          host: 42.194.233.222
          port: 5672
          virtual-host: /
          username: guest
          password: guest
      ```
   
   4. 创建 ApiTest 测试类，增加一个向 "simple.queue" 队列发送消息的测试方法
   
      ```java
      @SpringBootTest
      public class ApiTest {
      
          @Autowired
          private RabbitTemplate rabbitTemplate;
      
          @Test
          public void testSendMessage2SimpleQueue() {
              String queueName = "simple.queue";
              String message = "hello, spring amqp!";
              rabbitTemplate.convertAndSend(queueName, message);
          }
      }
      ```
3. 先启动消费者工程，然后再运行生产者工程中的测试方法，测试结果如下所示：在消费者工程的控制台中输出从 "simple.queue" 队列中收到一条消息：hello, spring amqp! <br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312023419.png)
   <a name="RIcEQ"></a>

### 工作队列（Work Queues）
案例：模拟 Work Queue，实现一个队列绑定多个消费者，具体实现步骤如下所示：

1. 在生产者（producer）服务的 ApiTest 测试类中增加新的测试方法，每秒发送 50 条消息到 work.queue 队列中；

   ```java
   @Test
   public void testSendMessage2WorkQueue() throws InterruptedException {
       String queueName = "work.queue";
       String message = "hello, message__";
       for (int i = 1; i <= 50; i++) {
           rabbitTemplate.convertAndSend(queueName, message + i);
           Thread.sleep(20);
       }
   }
   ```
2. 在消费者（consumer）服务中定义两个消费者，都监听 work.queue 队列，其中消费者 1 每秒可以处理 50 条消息，消费者 2 每秒可以处理 5 条消息；

   ```java
   @Configuration
   public class SpringAmqpConfiguration {
       private static final Logger LOGGER = LoggerFactory.getLogger(SpringAmqpConfiguration.class);
   
       @Bean
       public Queue simpleQueue() {
           return new Queue("simple.queue");
       }
   
       @RabbitListener(queues = {"simple.queue"})
       public void listenSimpleQueueMessage(String msg) {
           LOGGER.info("从 {} 队列中收到一条消息：{}", "simple.queue", msg);
       }
   
       @Bean
       public Queue workQueue() {
           return new Queue("work.queue");
       }
   
       @RabbitListener(queues = {"work.queue"})
       public void listenWorkQueueMessage1(String msg) throws InterruptedException {
           LOGGER.info("【消费者1】从 {} 队列中收到一条消息：{}", "work.queue", msg);
           Thread.sleep(20);
       }
   
       @RabbitListener(queues = {"work.queue"})
       public void listenWorkQueueMessage2(String msg) throws InterruptedException {
           LOGGER.error("【消费者2】从 {} 队列中收到一条消息：{}", "work.queue", msg);
           Thread.sleep(200);
       }
   }
   ```

按道理来说，在1秒钟之内，消费者 1 和消费者 2 应该合力完成消息的处理，可是事实真是如此吗？测试结果如下所示：<br />从下面的测试结果中可以可以看到总共耗时 5 秒多一点时间，这不对呀，为什么会这样呢？其实是因为咱们以为消费者 1 一个人在 1 秒钟之内就能处理完这些消息，在配合消费者 2 的话，处理能力应该更强才对，其实并不然，仔细观察，可以发现两个消费者消费的消息数量居然是一样的，都是 25 个，完全没有考虑消费者 2 的处理能力，就分配和消费者 1 一样多的数量，所以导致耗时这么久！
```java
2023-07-08 10:06:49.785  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__1
2023-07-08 10:06:49.806 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__2
2023-07-08 10:06:49.837  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__3
2023-07-08 10:06:49.898  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__5
2023-07-08 10:06:49.962  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__7
2023-07-08 10:06:50.013 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__4
2023-07-08 10:06:50.020  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__9
2023-07-08 10:06:50.080  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__11
2023-07-08 10:06:50.141  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__13
2023-07-08 10:06:50.202  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__15
2023-07-08 10:06:50.226 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__6
2023-07-08 10:06:50.264  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__17
2023-07-08 10:06:50.326  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__19
2023-07-08 10:06:50.386  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__21
2023-07-08 10:06:50.440 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__8
2023-07-08 10:06:50.447  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__23
2023-07-08 10:06:50.509  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__25
2023-07-08 10:06:50.569  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__27
2023-07-08 10:06:50.630  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__29
2023-07-08 10:06:50.653 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__10
2023-07-08 10:06:50.691  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__31
2023-07-08 10:06:50.753  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__33
2023-07-08 10:06:50.812  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__35
2023-07-08 10:06:50.865 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__12
2023-07-08 10:06:50.872  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__37
2023-07-08 10:06:50.932  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__39
2023-07-08 10:06:50.993  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__41
2023-07-08 10:06:51.055  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__43
2023-07-08 10:06:51.079 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__14
2023-07-08 10:06:51.115  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__45
2023-07-08 10:06:51.177  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__47
2023-07-08 10:06:51.237  INFO 19572: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__49
2023-07-08 10:06:51.291 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__16
2023-07-08 10:06:51.502 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__18
2023-07-08 10:06:51.714 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__20
2023-07-08 10:06:51.928 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__22
2023-07-08 10:06:52.141 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__24
2023-07-08 10:06:52.354 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__26
2023-07-08 10:06:52.568 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__28
2023-07-08 10:06:52.783 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__30
2023-07-08 10:06:52.997 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__32
2023-07-08 10:06:53.210 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__34
2023-07-08 10:06:53.422 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__36
2023-07-08 10:06:53.635 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__38
2023-07-08 10:06:53.847 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__40
2023-07-08 10:06:54.060 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__42
2023-07-08 10:06:54.275 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__44
2023-07-08 10:06:54.488 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__46
2023-07-08 10:06:54.701 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__48
2023-07-08 10:06:54.914 ERROR 19572: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__50
```
那么有没有什么可以解决呢？当然是有的，不过首先得了解一个概念，prefetch：预取，默认值大小为 250，<u><span style="background-color: rgb(251, 228, 231);">消费者不管自己的处理能力如何，就轮流把消息从消息队列中提前取出来</span></u>。所以咱们只需要将该值改为 1，<u>让消费者每次只能从队列中预取一条消息，只有等处理完并返回确认后才能继续取下一条消息</u>，这样的话，可以保证消费能力强的消费者可以处理更多的消息，而消费能力弱的消费者就少处理一点消息。<br />修改 application.yml 配置文件：
```yaml
spring:
  rabbitmq:
    host: 42.194.233.222
    port: 5672
    virtual-host: /
    username: guest
    password: guest
    listener:
      simple:
        prefetch: 1 # 消费者每次只能从队列中预取一条消息，只有等处理完并返回确认后才能继续取下一条消息
```
再次测试，测试结果如下所示：可以发现消费者 1 处理了更多的消息，消费者 2 处理更少的消息，总共耗时 1 秒钟多一点。
```java
2023-07-08 10:03:06.167 ERROR 14568: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__1
2023-07-08 10:03:06.188  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__2
2023-07-08 10:03:06.220  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__3
2023-07-08 10:03:06.247  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__4
2023-07-08 10:03:06.279  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__5
2023-07-08 10:03:06.310  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__6
2023-07-08 10:03:06.339  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__7
2023-07-08 10:03:06.370  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__8
2023-07-08 10:03:06.401  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__9
2023-07-08 10:03:06.431 ERROR 14568: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__10
2023-07-08 10:03:06.460  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__11
2023-07-08 10:03:06.491  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__12
2023-07-08 10:03:06.521  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__13
2023-07-08 10:03:06.550  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__14
2023-07-08 10:03:06.582  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__15
2023-07-08 10:03:06.610  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__16
2023-07-08 10:03:06.640  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__17
2023-07-08 10:03:06.672 ERROR 14568: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__18
2023-07-08 10:03:06.703  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__19
2023-07-08 10:03:06.734  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__20
2023-07-08 10:03:06.765  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__21
2023-07-08 10:03:06.796  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__22
2023-07-08 10:03:06.825  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__23
2023-07-08 10:03:06.854  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__24
2023-07-08 10:03:06.885  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__25
2023-07-08 10:03:06.916 ERROR 14568: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__26
2023-07-08 10:03:06.946  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__27
2023-07-08 10:03:06.976  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__28
2023-07-08 10:03:07.006  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__29
2023-07-08 10:03:07.040  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__30
2023-07-08 10:03:07.080  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__31
2023-07-08 10:03:07.114  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__32
2023-07-08 10:03:07.127 ERROR 14568: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__33
2023-07-08 10:03:07.157  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__34
2023-07-08 10:03:07.187  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__35
2023-07-08 10:03:07.218  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__36
2023-07-08 10:03:07.249  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__37
2023-07-08 10:03:07.279  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__38
2023-07-08 10:03:07.309  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__39
2023-07-08 10:03:07.340  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__40
2023-07-08 10:03:07.370 ERROR 14568: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__41
2023-07-08 10:03:07.400  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__42
2023-07-08 10:03:07.430  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__43
2023-07-08 10:03:07.460  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__44
2023-07-08 10:03:07.490  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__45
2023-07-08 10:03:07.522  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__46
2023-07-08 10:03:07.552  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__47
2023-07-08 10:03:07.582  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__48
2023-07-08 10:03:07.612 ERROR 14568: 【消费者2】从 work.queue 队列中收到一条消息：hello, message__49
2023-07-08 10:03:07.642  INFO 14568: 【消费者1】从 work.queue 队列中收到一条消息：hello, message__50
```
<a name="xIwBf"></a>
### 发布订阅（Publish/Subscribe）
使用扇形交换机（Fanout Exchange）实现，Fanout Exchange 会将接收到的消息路由每一个与其绑定的队列（queue）中。<br />案例：利用 Spring AMQP 演示 Fanout Exchange 的使用，具体实现步骤如下所示：

1. 在消费者（consumer）服务中，

   1. 声明一个名称为 "itcast.fanout" 的扇形交换机（Fanout Exchange）；
   1. 声明两个名称分别为 "fanout.queue1" 和 "fanout.queue2" 的队列与该交换机进行绑定；
   1. 定义两个消费者分别监听这两个队列；

   ```java
   @Configuration
   public class FanoutConfig {
       private static final Logger LOGGER = LoggerFactory.getLogger(FanoutConfig.class);
   
       @Bean
       public FanoutExchange fanoutExchange() {
           return new FanoutExchange("itcast.fanout");
       }
   
       @Bean
       public Queue fanoutQueue1() {
           return new Queue("fanout.queue1");
       }
   
       @Bean
       public Binding bindingFanoutQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange) {
           return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
       }
   
       @RabbitListener(queues = {"fanout.queue1"})
       public void listenFanoutQueue1Message(String msg) {
           LOGGER.info("【消费者1】从 {} 队列中收到一条消息：{}", "fanout.queue1", msg);
       }
   
       @Bean
       public Queue fanoutQueue2() {
           return new Queue("fanout.queue2");
       }
   
       @Bean
       public Binding bindingFanoutQueue2(Queue fanoutQueue2, FanoutExchange fanoutExchange) {
           return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
       }
   
       @RabbitListener(queues = {"fanout.queue2"})
       public void listenFanoutQueue2Message(String msg) {
           LOGGER.info("【消费者2】从 {} 队列中收到一条消息：{}", "fanout.queue2", msg);
       }
   }
   ```

   

2. 在生产者（producer）服务的 ApiTest 测试类中增加新的测试方法，如下所示：

   ```java
   @Test
   public void testSendMessage2FanoutExchange() {
       String exchangeName = "itcast.fanout";
       String message = "hello, every one!";
       rabbitTemplate.convertAndSend(exchangeName, "", message);
   }
   ```
3. 测试结果如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312032513.png)
   <a name="glgVn"></a>

### 路由模式（Routing）
使用直连交换机（Direct Exchange）实现，Direct Exchange 会将接收到的消息根据路由规则路由指定的队列（queue）中，因此称为路由模式。

- 每一个 Queue 在与 Exchange 绑定时设置一个 Binding Key；
- 生产者发送消息时，需要指定消息的 Routing Key；
-  Direct Exchange 会将消息路由到 Binding Key 与消息 RoutingKey 一致的队列中；

案例：利用 Spring AMQP 演示 Direct Exchange 的使用，具体实现步骤如下所示：

1. 在消费者（consumer）服务中，可以直接使用 @RabbitListener 声明 Exchange、Queue、Binding 和定义消费者，
   1. 使用 @RabbitListener 声明一个名称为 "itcast.direct" 的直连交换机（Fanout Exchange）与一个名称为 "direct.queue1" 的队列进行绑定，绑定键（Binding Key）为 "red" 和 "blue"；
   2. 使用 @RabbitListener 声明一个名称为 "itcast.direct" 的直连交换机（Fanout Exchange）与一个名称为 "direct.queue2" 的队列进行绑定，绑定键（Binding Key）为 "red" 和 "yellow"；
   3. 定义两个消费者分别监听这两个队列；
   
   ```java
   @Configuration
   public class SpringAmqpConfiguration {
       private static final Logger LOGGER = LoggerFactory.getLogger(SpringAmqpConfiguration.class);
   
       @Bean
       public Queue simpleQueue() {
           return new Queue("simple.queue");
       }
   
       @RabbitListener(queues = {"simple.queue"})
       public void listenSimpleQueueMessage(String msg) {
           LOGGER.info("从 {} 队列中收到一条消息：{}", "simple.queue", msg);
       }
   
       @Bean
       public Queue workQueue() {
           return new Queue("work.queue");
       }
   
       @RabbitListener(queues = {"work.queue"})
       public void listenWorkQueueMessage1(String msg) throws InterruptedException {
           LOGGER.info("【消费者1】从 {} 队列中收到一条消息：{}", "work.queue", msg);
           Thread.sleep(20);
       }
   
       @RabbitListener(queues = {"work.queue"})
       public void listenWorkQueueMessage2(String msg) throws InterruptedException {
           LOGGER.error("【消费者2】从 {} 队列中收到一条消息：{}", "work.queue", msg);
           Thread.sleep(200);
       }
   
       @RabbitListener(bindings = {
               @QueueBinding(value = @org.springframework.amqp.rabbit.annotation.Queue("direct.queue1"),
                       exchange = @Exchange(name = "itcast.direct"), key = {"red", "blue"})
       })
       public void listenDirectQueueMessage1(String msg) {
           LOGGER.error("【消费者1】从 {} 队列中收到一条消息：{}", "direct.queue1", msg);
       }
   
       @RabbitListener(bindings = {
               @QueueBinding(value = @org.springframework.amqp.rabbit.annotation.Queue("direct.queue2"),
                       exchange = @Exchange(name = "itcast.direct"), key = {"red", "yellow"})
       })
       public void listenDirectQueueMessage2(String msg) {
           LOGGER.error("【消费者2】从 {} 队列中收到一条消息：{}", "direct.queue2", msg);
       }
   }
   ```
2. 在生产者（producer）服务的 ApiTest 测试类中增加新的测试方法，如下所示：

   ```java
   @Test
   public void testSendMessage2DirectExchange() {
       String exchangeName = "itcast.direct";
       String message = "hello, ";
       Stream.of("red", "blue", "yellow").forEach(color ->
               rabbitTemplate.convertAndSend(exchangeName, color, message + color));
   }
   ```
3. 测试结果如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312033154.png)
   <a name="I0yLU"></a>

### 主题模式（Topics）
使用主题交换机（Topic Exchange）实现，Topic Exchange 与 Direct Exchange 类似，区别在于 Routing key 必须是多个单词的列表，并且以`.`分割；<br />队列（Queue）与 交换机（Exchange）进行绑定时绑定键（Binding Key）可以使用通配符：

- `#`：代指 0 个或多个单词；
- `*`：代指一个单词 ；

案例：利用 Spring AMQP 演示 Topic Exchange 的使用，具体实现步骤如下所示：

1. 在消费者（consumer）服务中，
   1. 使用 @RabbitListener 声明一个名称为 "itcast.topic" 的主题交换机（Topic Exchange）与一个名称为 "topic.queue1" 的队列进行绑定，绑定键（Binding Key）为 "china.#" ；
   2. 使用 @RabbitListener 声明一个名称为 "itcast.topic" 的主题交换机（Topic Exchange）与一个名称为 "topic.queue2" 的队列进行绑定，绑定键（Binding Key）为 "#.news" ；
   3. 定义两个消费者分别监听这两个队列；
   
   ```java
   @Configuration
   public class SpringAmqpConfiguration {
       private static final Logger LOGGER = LoggerFactory.getLogger(SpringAmqpConfiguration.class);
   
       @Bean
       public Queue simpleQueue() {
           return new Queue("simple.queue");
       }
   
       @RabbitListener(queues = {"simple.queue"})
       public void listenSimpleQueueMessage(String msg) {
           LOGGER.info("从 {} 队列中收到一条消息：{}", "simple.queue", msg);
       }
   
       @Bean
       public Queue workQueue() {
           return new Queue("work.queue");
       }
   
       @RabbitListener(queues = {"work.queue"})
       public void listenWorkQueueMessage1(String msg) throws InterruptedException {
           LOGGER.info("【消费者1】从 {} 队列中收到一条消息：{}", "work.queue", msg);
           Thread.sleep(20);
       }
   
       @RabbitListener(queues = {"work.queue"})
       public void listenWorkQueueMessage2(String msg) throws InterruptedException {
           LOGGER.error("【消费者2】从 {} 队列中收到一条消息：{}", "work.queue", msg);
           Thread.sleep(200);
       }
   
       @RabbitListener(bindings = {
               @QueueBinding(value = @org.springframework.amqp.rabbit.annotation.Queue("direct.queue1"),
                       exchange = @Exchange(name = "itcast.direct"), key = {"red", "blue"})
       })
       public void listenDirectQueueMessage1(String msg) {
           LOGGER.info("【消费者1】从 {} 队列中收到一条消息：{}", "direct.queue1", msg);
       }
   
       @RabbitListener(bindings = {
               @QueueBinding(value = @org.springframework.amqp.rabbit.annotation.Queue("direct.queue2"),
                       exchange = @Exchange(name = "itcast.direct"), key = {"red", "yellow"})
       })
       public void listenDirectQueueMessage2(String msg) {
           LOGGER.info("【消费者2】从 {} 队列中收到一条消息：{}", "direct.queue2", msg);
       }
   
       @RabbitListener(bindings = {
               @QueueBinding(value = @org.springframework.amqp.rabbit.annotation.Queue("topic.queue1"),
                       exchange = @Exchange(name = "itcast.topic", type = ExchangeTypes.TOPIC), key = {"china.#"})
       })
       public void listenTopicQueueMessage1(String msg) {
           LOGGER.info("【消费者1】从 {} 队列中收到一条消息：{}", "topic.queue1", msg);
       }
   
       @RabbitListener(bindings = {
               @QueueBinding(value = @org.springframework.amqp.rabbit.annotation.Queue("topic.queue2"),
                       exchange = @Exchange(name = "itcast.topic", type = ExchangeTypes.TOPIC), key = {"#.news"})
       })
       public void listenTopicQueueMessage2(String msg) {
           LOGGER.info("【消费者2】从 {} 队列中收到一条消息：{}", "topic.queue2", msg);
       }
   }
   ```
2. 在生产者（producer）服务的 ApiTest 测试类中增加新的测试方法，如下所示：

   ```java
   @Test
   public void testSendMessage2TopicExchange() {
       String exchangeName = "itcast.topic";
       Map<String, String> map = new HashMap<>(2);
       map.put("china.news", "中华人民共和国成立啦！");
       map.put("china.weather", "今天阳光明媚");
       map.forEach((routingKey, message) -> rabbitTemplate.convertAndSend(exchangeName, routingKey, message));
   }
   ```
3. 测试结果如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312035414.png)
   <a name="y4IvS"></a>

### 消息转换器
案例：测试发送 Object 类型的消息，具体实现步骤如下：

1. 在消费者（consumer）服务中，声明一个名称为 "object.queue" 的队列

   ```java
   @Configuration
   public class SpringAmqpConfiguration {
       private static final Logger LOGGER = LoggerFactory.getLogger(SpringAmqpConfiguration.class);
   
       @Bean
       public Queue simpleQueue() {
           return new Queue("simple.queue");
       }
   
       @RabbitListener(queues = {"simple.queue"})
       public void listenSimpleQueueMessage(String msg) {
           LOGGER.info("从 {} 队列中收到一条消息：{}", "simple.queue", msg);
       }
   
       @Bean
       public Queue workQueue() {
           return new Queue("work.queue");
       }
   
       @RabbitListener(queues = {"work.queue"})
       public void listenWorkQueueMessage1(String msg) throws InterruptedException {
           LOGGER.info("【消费者1】从 {} 队列中收到一条消息：{}", "work.queue", msg);
           Thread.sleep(20);
       }
   
       @RabbitListener(queues = {"work.queue"})
       public void listenWorkQueueMessage2(String msg) throws InterruptedException {
           LOGGER.error("【消费者2】从 {} 队列中收到一条消息：{}", "work.queue", msg);
           Thread.sleep(200);
       }
   
       @RabbitListener(bindings = {
               @QueueBinding(value = @org.springframework.amqp.rabbit.annotation.Queue("direct.queue1"),
                       exchange = @Exchange(name = "itcast.direct"), key = {"red", "blue"})
       })
       public void listenDirectQueueMessage1(String msg) {
           LOGGER.info("【消费者1】从 {} 队列中收到一条消息：{}", "direct.queue1", msg);
       }
   
       @RabbitListener(bindings = {
               @QueueBinding(value = @org.springframework.amqp.rabbit.annotation.Queue("direct.queue2"),
                       exchange = @Exchange(name = "itcast.direct"), key = {"red", "yellow"})
       })
       public void listenDirectQueueMessage2(String msg) {
           LOGGER.info("【消费者2】从 {} 队列中收到一条消息：{}", "direct.queue2", msg);
       }
   
       @RabbitListener(bindings = {
               @QueueBinding(value = @org.springframework.amqp.rabbit.annotation.Queue("topic.queue1"),
                       exchange = @Exchange(name = "itcast.topic", type = ExchangeTypes.TOPIC), key = {"china.#"})
       })
       public void listenTopicQueueMessage1(String msg) {
           LOGGER.info("【消费者1】从 {} 队列中收到一条消息：{}", "topic.queue1", msg);
       }
   
       @RabbitListener(bindings = {
               @QueueBinding(value = @org.springframework.amqp.rabbit.annotation.Queue("topic.queue2"),
                       exchange = @Exchange(name = "itcast.topic", type = ExchangeTypes.TOPIC), key = {"#.news"})
       })
       public void listenTopicQueueMessage2(String msg) {
           LOGGER.info("【消费者2】从 {} 队列中收到一条消息：{}", "topic.queue2", msg);
       }
   
       @RabbitListener(queuesToDeclare = {@org.springframework.amqp.rabbit.annotation.Queue("object.queue")})
       public void listenObjectQueueMessage(Map<String, Object> msg) {
           LOGGER.info("【消费者】从 {} 队列中收到一条消息：{}", "object.queue", msg);
       }
   }
   ```
2. 在生产者（producer）服务的 ApiTest 测试类中增加新的测试方法，如下所示：

   ```java
   @Test
   public void testSendMessage2ObjectQueue() {
       String queueName = "object.queue";
       Map<String, Object> message = new HashMap<>(2);
       message.put("name", "xiaorang");
       message.put("age", 18);
       rabbitTemplate.convertAndSend(queueName, message);
   }
   ```
3. 测试结果如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312037374.png)<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312037458.png)<br />

Spring 使用 <u>MessageConverter</u> 来处理消息对象，默认实现为 SimpleMessageConverter，基于 JDK 的 ObjectOutputStream 完成序列化。显然，JDK 序列化的方式并不合适，咱们希望消息体的体积更小、可读性更高，因此可以使用 JSON 的方式来做序列化和反序列化。那么该如何实现呢？很简单，只需要自定义一个的 MessageConverter 类型的组件即可。具体实现步骤如下：<br />首先在 pom.xml 配置文件中增加如下依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.5</version>
</dependency>
```
❗注意：需要在消费者和生产者服务中都定义一个类型相同的 MessageConverter 组件，如下所示：
```java
@Bean
public MessageConverter messageConverter() {
    return new Jackson2JsonMessageConverter();
}
```
再次运行测试方式，测试结果如下所示：可以看到消息体相比较于默认情况体积更小、可读性更高；<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312039478.png)
<a name="ceZ5t"></a>

## RabbitMQ 高级特性
消息队列在使用的过程中，面临着很多实际的问题需要考虑，如下所示：

- 消息可靠性问题：如果确保消息至少被消费一次；解决方案：[生产者确认机制](#i5jok) + [消息持久化](#LS6fr) + [消费者确认机制](#y1PRt) + [失败重试机制](#HVnSZ)
- 延迟消息问题：如何实现消息的延迟投递；解决方案： [TTL + 死信交换机 = 延迟队列](#xzX50)
- 消息堆积问题：如何解决数百万消息堆积，无法及时消费的问题；解决方案：[惰性队列](#Xr6aQ)
- 高可用问题：如何避免单点的 MQ 故障而导致的不可用问题；
<a name="gPVYK"></a>

### 消息可靠性

消息从发送直至消费者接收，会经历多个过程，在这几个过程中都有可能导致消息的丢失。如下所示：

- 发送时丢失

  - 生产者发送的消息未达到交换机；

  - 消息到达交换机后未到达队列；

  - MQ 宕机，消息在队列中丢失

- 消费者接收到消息之后还没来得及消费就宕机

<img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312040678.jpeg" style="zoom: 50%;" /><br />针对这些问题，RabbitMQ 分别给出相应的解决方案，如下所示：

- 生产者确认机制；
- 消息持久化；
- 消费者确认机制；
- 失败重试机制；
<a name="i5jok"></a>

#### 生产者确认机制

RabbitMQ 提供了 publisher confirm 机制来避免消息发送到 MQ 过程中丢失，只有当消息成功到了 queue 中，消息才算发送成功！当消息发送到 MQ 之后，会返回一个结果给发送者，表示消息是否处理成功。<br />MQ 返回的结果存在如下两种情况：

- **publisher-confirm，发送者确认**：producer 发送消息时设置一个 confirmCallback 回调，不管消息是否到达交换机都会执行 confirmCallback 回调方法，告诉 producer 消息投递的结果。
   - 消息成功投递到交换机，返回 ack（只能说明到了 exchange，成功了一半）；
   - 消息未投递到交换机，返回 nack；
- **publisher-return，发送者回执**：当交换机将消息路由到消息队列失败时，则会返回一个 returnCallback，告诉 exchange 是否接收到了消息，queue -> exchange。
   - 消息投递到交换机了，但是没有路由到队列，则会调用 returnCallback 回调方法，返回 ACK 以及路由失败原因。
   - 一般情况下，消息能到交换机，那么就可以到达队列，如果到不了队列，则说明是代码写错了，没匹配到正确的队列！

![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312044721.jpeg)
> [!ATTENTION]
> 生产者确认机制在发送消息时，需要给每个消息设置一个全局唯一 id，以区分不同的消息，避免 ack 冲突！

在生产者（producer）服务中如何使用生产者确认机制，具体实现步骤如下所示：

1. 在 application.yml 配置文件中增加如下配置信息：

   ```yaml
   spring:
     rabbitmq:
       host: 42.194.233.222
       port: 5672
       virtual-host: /
       username: guest
       password: guest
       publisher-confirm-type: correlated
       publisher-returns: true
       template:
         mandatory: true
   ```

      1. publisher-confirm-type：开启 publisher confirm，支持如下两种类型：
         1. simple：同步等待 confirm 结果直至超时；
         2. correlated：异步回调，定义 ConfirmCallback 回调，MQ 返回结果时会执行该 ConfirmCallback 回调方法；
   2. publisher-returns：开启 publisher-return 功能，同样是基于 callback 机制，不过是定义 ReturnCallback；

   3. template.mandatory：定义消息路由失败时的策略。true：调用 ReturnCallback，将消息退回给生产者；false：直接丢弃消息；

2. 每个 RabbitTemplate 只能配置一个 ReturnCallback，因此需要在项目加载时配置。添加如下配置类：

   ```java
   @Configuration
   public class CommonConfig implements ApplicationContextAware {
       private static final Logger LOGGER = LoggerFactory.getLogger(CommonConfig.class);
   
       @Override
       public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
           RabbitTemplate rabbitTemplate = applicationContext.getBean(RabbitTemplate.class);
           rabbitTemplate.setReturnsCallback(returnedMessage -> {
               Message message = returnedMessage.getMessage();
               String exchange = returnedMessage.getExchange();
               int replyCode = returnedMessage.getReplyCode();
               String replyText = returnedMessage.getReplyText();
               String routingKey = returnedMessage.getRoutingKey();
               // 投递失败，记录日志
               LOGGER.error("消息路由失败，应答码：{}，原因：{}， 交换机：{}，路由键：{}，消息：{}", replyCode, replyText, exchange, routingKey, message);
               // 如果数据可靠性要求高，可以进行失败重试，使用 spring-retry 重发消息；
           });
       }
   }
   ```

3. ConfirmCallback 回调可以在发送消息时指定，因为每个业务处理 comfirm 成功或失败的逻辑不一定相同。

   ```java
   @Test
   public void testSendMessage2SimpleQueue() {
       // 消息体
       String message = "hello, spring amqp!";
       // 全局唯一的消息 ID，需要封装到 CorrelationData 中
       CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
       // 添加 callback
       correlationData.getFuture().addCallback(result -> {
           assert result != null;
           if (result.isAck()) {
               // ack，消息成功投递到交换机
               LOGGER.info("消息成功投递到交换机，ID：{}", correlationData.getId());
           } else {
               // nack，消息投递到交换机失败
               LOGGER.error("消息投递到交换机失败，ID：{}，原因：{}", correlationData.getId(), result.getReason());
               // 重发消息
           }
       }, ex -> {
           // 记录日志
           LOGGER.error("消息发送异常，ID：{}，原因：{}", correlationData.getId(), ex.getMessage());
           // 重发消息
       });
       rabbitTemplate.convertAndSend("", "simple.queue", message, correlationData);
   }
   ```

<a name="LS6fr"></a>
#### 消息持久化

生产者确认机制可以确保消息投递到 RabbitMQ 的消息队列中，但是消息发送到 RabbitMQ 之后，RabbitMQ 突然宕机了，也有可能会导致消息丢失。要想确保消息在 RabbitMQ 中安全保存，必须开启消息持久化机制。

- 交换机持久化
- 队列持久化
- 消息持久化

在 Spring AMQP 中，交换机、队列和消息默认都是持久化的，不需要刻意配置。但还是在这里学习如何配置持久化。
<a name="UH71p"></a>
##### 交换机持久化
在 RabbitMQ 中交换机默认是非持久化的，MQ 重启之后就会丢失。在 Spring AMQP 中可以通过代码指定交换机持久化：

```java
@Bean
public DirectExchange simpleExchange() {
    // 三个参数：交换机名称，是否持久化，当没有队列与其绑定时是否自动删除
    return new DirectExchange("simple.direct", true, false);
}
```

事实上没必要，因为由 Spring AMQP 声明的交换机默认就是持久化的。可以在控制台看到持久化的交换机都会带上`D`的标识，如下所示：<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312048904.png" alt="image.png" style="zoom:67%;" />
<a name="DnNeR"></a>

##### 队列持久化
在 RabbitMQ 中队列默认是非持久化的，MQ 重启之后就会丢失。在 Spring AMQP 中可以通过代码指定队列持久化：
```java
@Bean
public Queue simpleQueue() {
    // 使用 QueueBuilder 构建队列，使用 durable 指定持久化
    return QueueBuilder.durable("simple.queue").build();
}
```
事实上没必要，因为由 Spring AMQP 声明的队列默认就是持久化的。可以在控制台看到持久化的队列都会带上`D`的标识，如下所示：<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312048882.png" alt="image.png" style="zoom:67%;" />
<a name="uABM9"></a>

##### 消息持久化
利用 Spring AMQP 发送消息时，可以设置消息的属性（MessageProperties），指定 delivery-mode：1（非持久化）、2（持久化），存在对应的枚举`MessageDeliveryMode`。
```java
@Test
public void testSendDurableMessage() {
    Message message = MessageBuilder
            .withBody("hello, spring amqp".getBytes(StandardCharsets.UTF_8))
            .setDeliveryMode(MessageDeliveryMode.PERSISTENT)
            .build();
    rabbitTemplate.convertAndSend("simple.queue", message);
}
```
测试结果如下所示：发现居然报错，消息转换失败！**这里有点坑，请注意！**<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312048103.png)<br />在上面的异常信息中说需要一个 JSON 类型的字符串，所以在传递消息时需要先将内容转换成 JSON 字符串，如下所示：
```java
@Test
public void testSendDurableMessage() throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    String content = objectMapper.writeValueAsString("hello, spring amqp");
    Message message = MessageBuilder
            .withBody(content.getBytes(StandardCharsets.UTF_8))
            .setDeliveryMode(MessageDeliveryMode.PERSISTENT)
            .build();
    rabbitTemplate.convertAndSend("simple.queue", message);
}
```
咱们 DEBUG 调试一下，看一下转换后的 JSON 字符串长啥样？多了一对双引号！<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312051811.png)<br />测试结果如下所示，消费者服务的控制台输出：从 simple.queue 队列中收到一条消息：hello，spring amqp；<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312051436.png" alt="image.png"  />
<a name="y1PRt"></a>

#### 消费者确认机制
RabbitMQ 是通过消费者回执来确认消费者是否成功处理消息的，消费者获取消息后，应该向 RabbitMQ 发送 ACK（Ackonwledge，确认）回执，表明自己已经处理消息。<br />Spring AMQP 允许配置三种确认模式：

- manual：手动 ack，需要在业务代码结束后，调用 api 发送 ack；
- auto：自动 ack，由 spring 监测 listener 代码是否出现异常，没有异常返回 ack；抛出异常则返回 nack；
- none：关闭 ack，MQ 假定消费者获取到消息之后会成功处理，因此消息投递后立即被删除；

由此可知，

- none：消息投递是不可靠的，可能会丢失；
- auto：当 listener 代码出现异常时返回 nack，没有异常则返回 ack，底层应该是使用 AOP 进行增强；
- manual：根据业务情况自己判断什么该返回 ack 或 nack；
<a name="oZU6Z"></a>

##### 演示 none 模式

1. 修改消费者（consumer）服务中的 application.yml 配置文件，增加如下配置信息：

   ```yaml
   spring:
     rabbitmq:
       host: 42.194.233.222
       port: 5672
       virtual-host: /
       username: guest
       password: guest
       listener:
         simple:
           prefetch: 1 # 消费者每次只能从队列中预取一条消息，只有等消费者处理完并返回确认后才能继续取下一条消息
           acknowledge-mode: none # 关闭 ack，消息投递到消费者后立即从队列中删除
   ```

2. 修改监听方法，模拟一个消息处理异常：

   ```java
   @RabbitListener(queues = {"simple.queue"})
   public void listenSimpleQueueMessage(String msg) {
       LOGGER.info("从 {} 队列中收到一条消息：{}", "simple.queue", msg);
       // 模拟异常
       int i = 1 / 0;
       LOGGER.info("消息处理成功！");
   }
   ```

3. 测试结果如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312053631.png)<br />经过测试发现，当方法抛出异常时，消息依然从队列中删除了！
   <a name="mJIP3"></a>

##### 演示 manual 模式
如果设置了手动确认（manual）模式，

- 在业务处理成功后通过`basicAck()`方法进行手动签收；
- 出现异常的话，则可以通过`basicReject()`或`basicNack()`方法否定应答消息，使消息重新入队，从而使得其他消费者可以消费消息；
   - 使用`basicReject()`方法实现消费者否定应答单条消息并使其重新入队；方法参数说明如下：
      - deliveryTag：channel消息投递的唯一标识符；
      - requeue：被否定应答的消息是否重新入队？如果设置为 true，则消息重新入队；如果设置为 false，则消息被丢弃或发送到[死信交换机](#xzX50)；
   - 使用`basicNack()`方法实现消费者否定应答多条消息并使其重新入队；方法参数说明如下：
      - deliveryTag：channel消息投递的唯一标识符；
      - mulitiple：是否否定多条消息？如果设置为 true，则否定应答该 deliveryTag 的消息以及该 deliveryTag 之前的多条消息；如果设置为 false，仅否定应答该 deliveryTag 的单条消息；
      - requeue：被否定应答的消息是否重新入队？如果设置为 true，则消息重新入队；如果设置为 false，则消息被丢弃或发送到[死信交换机](#xzX50)；
1. 修改确认模式为 manual

   ```yaml
   spring:
     rabbitmq:
       host: 42.194.233.222
       port: 5672
       virtual-host: /
       username: guest
       password: guest
       listener:
         simple:
           prefetch: 1 # 消费者每次只能从队列中预取一条消息，只有等消费者处理完并返回确认后才能继续取下一条消息
           acknowledge-mode: manual # 手动签收
   ```

2. 修改监听方法，使用 try catch 捕捉异常，当没有异常时，使用`basicAck()`方法确认签收消息；当出现异常时，则在 catch 块中使用`basicReject()`或`basicNack()`方法否定应答消息，使消息重新入队。

   ```java
   @RabbitListener(queues = {"simple.queue"})
   public void listenSimpleQueueMessage(Message message, Channel channel) throws IOException {
       // 消息投递的唯一标识符
       long deliveryTag = message.getMessageProperties().getDeliveryTag();
       try {
           LOGGER.info("从 {} 队列中收到一条消息：{}", "simple.queue", new String(message.getBody()));
           // 模拟异常
           //            int i = 1 / 0;
           LOGGER.info("消息处理成功！");
           channel.basicAck(deliveryTag, true);
       } catch (Exception e) {
           LOGGER.info("消费失败，消息重新入队！");
           channel.basicReject(deliveryTag, true);
       }
   }
   ```

3. 经过测试发现，当没有异常时，消息被正常消费，从队列中删除；当出现异常时消息会不断重新入队，不断重发，无限循环，导致 MQ 的压力剧增，这样非常不好，应该通过 [失败重试机制](#HVnSZ) 来限定消费的最大次数。
<a name="UQE1V"></a>

##### 演示 auto 模式

1. 修改确认模式为 auto 或者不配置（因为默认情况下就是 auto）

   ```yaml
   spring:
     rabbitmq:
       host: 42.194.233.222
       port: 5672
       virtual-host: /
       username: guest
       password: guest
       listener:
         simple:
           prefetch: 1 # 消费者每次只能从队列中预取一条消息，只有等消费者处理完并返回确认后才能继续取下一条消息
           acknowledge-mode: auto # 自动签收，默认情况下就是该模式
   ```

2. 修改监听方法，❗注意：不能使用 try catch 捕获异常，否则 spring 无法感知到异常，就会自动返回 ack

   ```java
   @RabbitListener(queues = {"simple.queue"})
   public void listenSimpleQueueMessage(String message) {
       LOGGER.info("从 {} 队列中收到一条消息：{}", "simple.queue", message);
       // 模拟异常
       int i = 1 / 0;
       LOGGER.info("消息处理成功！");
   }
   ```

3. 使用 DEBUG 方式启动消费者服务，
   1. 在异常位置打断点，再次发送消息，程序卡在断点时，可以发现此时消息为 Unacked（未应答）状态 <br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312056003.png)
   
   2. 抛出异常后，自动返回 nack，消息会重新入队，恢复至 Ready（就绪）状态  <br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312056606.png)

4. 与 manual 模式测试效果一致！当出现异常时消息会不断重新入队，不断重发，无限循环。
<a name="HVnSZ"></a>

#### 失败重试机制

前面在演示消费者确认机制中的 auto 和 manual 模式时，当监听方法出现异常时，消息会不断重新入队，不断重发，无限循环，导致 MQ 的压力剧增！<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312058614.png" alt="image.png" style="zoom:67%;" /><br />🤔 那么该怎么办呢？<br />🤓可以利用 spring 的 retry 机制，在消费者出现异常时使用本地重试，而不是无限制的重新入队。<br />在消费者（consumer)服务的 application.yml 配置文件中，增加如下配置信息：
```yaml
spring:
  rabbitmq:
    host: 42.194.233.222
    port: 5672
    virtual-host: /
    username: guest
    password: guest
    listener:
      simple:
        prefetch: 1 # 消费者每次只能从队列中预取一条消息，只有等消费者处理完并返回确认后才能继续取下一条消息
        acknowledge-mode: auto # 自动签收，默认情况下就是该模式
        retry:
          enabled: true # 开启消费者失败重试
          initial-interval: 1000 # 重新投递消息的间隔时间（单位毫秒）
          max-attempts: 3 # 最大重试次数
          stateless: true # true：无状态，false：有状态；如果业务中包含事务，这里改为 false
```
重新启动消费者（consumer）服务，发送一条消息，可以发现：

- 开启失败重试之后，监听方法出现异常时，消息不会重新入队，而是直接在本地进行重试；
- 在达到最大重试次数之后，如果还是失败的话，Spring AMQP 会抛出 AmqpRejectAndDontRequeueException 异常，并且输出 Retry Policy Exhausted（重试次数已用完）信息；
- 查看 RabbitMQ 控制台，发现消息已从队列中删除，说明在达到最大重试次数之后如果还是失败的话则消息将会被丢弃；

在开启失败重试机制之后，当达到最大重试次数消息依然消费失败，则需要有 MessageRecovery 接口来进行处理，它包含三种不同的实现：

- RejectAndDontRequeueRecoverer：默认情况，重试次数耗尽后，返回 nack 并且消息不会重新入队，即消息将会被丢弃；
- ImmediateRequeueMessageRecoverer：重试次数耗尽后，返回 nack ，消息会重新入队；
- RepublishMessageRecoverer：值得推荐！是比较优雅的一种处理方案，重试次数耗尽后，将消费失败的消息投递给指定的专门用来处理异常消息的交换机，后续交由人工集中处理（因为重试这么多次还是失败，继续重试也没有什么意义，所以交由人工集中处理）；

<img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312100241.jpeg" alt="image.png" style="zoom:50%;" />

在消费者（consumer)服务中增加如下配置类：

- 向 IoC 容器中注册一个 RepublishMessageRecoverer 组件，指定重新投递的交换机和路由键；
- 声明用于处理异常消息的交换机（error.direct）和队列（error.queue）并将它俩进行绑定，绑定键 = error；

```java
@Configuration
public class RepublishMessageConfig {
    private static final Logger LOGGER = LoggerFactory.getLogger(RepublishMessageConfig.class);

    @Bean
    public MessageRecoverer messageRecoverer(RabbitTemplate rabbitTemplate) {
        return new RepublishMessageRecoverer(rabbitTemplate, "error.direct", "error");
    }

    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue(name = "error.queue"),
                    exchange = @Exchange(name = "error.direct"),
                    key = {"error"}
            )
    })
    public void listenErrorMessage(String message) {
        LOGGER.error("从 error.queue 队列中收到一条异常信息：{}，请及时处理！", message);
    }
}
```

再次发送一条消息到 "simple.queue" 队列中，测试结果如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312101143.png)
<a name="xzX50"></a>

### 死信交换机

<a name="KYgUw"></a>
#### 死信 & 死信交换机
🤔 什么是死信？<br />🤓当一个队列中的消息满足下列情况之一时，可以成为死信（dead letter）：

- 消费者使用 basic.reject 或 basic.nack 声明消费失败，并且消息的 requeue 参数设置为 false；
- 消息是一个过期消息，超时无人消费；
- 要投递的队列消息堆积满了，最早的消息可能成为死信；

如果该队列配置了`dead-letter-exchange`属性，指定了一个交换机，那么队列中的死信就会投递到这个交换机中，而这个交换机称为死信交换机（Dead Letter Exchange，简称 DLX）。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312105269.jpeg" style="zoom:50%;" />
<a name="fr60V"></a>

#### TTL
全称 Time-To-Live（存活时间/过期时间）。如果一个队列中的消息在存活时间结束后仍然未被消费，则会变成死信，ttl 超时分为两种情况：

- 消息所在的队列设置了存活时间，使用`x-message-ttl`参数，单位为毫秒（ms）；
- 消息本身设置了存活时间，使用`expiration`参数，单位为毫秒（ms）；

> [!IMPORTANT]
>
> TTL + 死信交换机 = 延迟队列

<img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312113833.jpeg" style="zoom:50%;" />

1.  使用 @RabbitListener 注解声明一组死信交换机和队列；

    ```java
    @RabbitListener(bindings = {
        @QueueBinding(
            value = @Queue(name = "dl.queue"),
            exchange = @Exchange(name = "dl.direct"),
            key = {"dl"}
        )
    })
    public void listenDeadMessage(String message) {
    	LOGGER.info("从 dl.queue 队列中收到一条延迟消息：{}", message);
    }
    ```
2. 声明一个超时队列 "ttl.queue"，指定 TTL = 10000，并且指定死信交换机 dead-letter-exchange = "dl.direct"，死信路由键 dead-letter-routing-key = "dl"，再声明一个交换机 "ttl.direct" 与该队列进行绑定，绑定键 bindingKey = "ttl"；

   ```java
   public org.springframework.amqp.core.Queue ttlQueue() {
       return QueueBuilder
       // 指定队列名称以及持久化
       .durable("ttl.queue")
       // 设置队列的超时时间为 10 秒，即配置 x-message-ttl 属性
       .ttl(10000)
       // 指定死信交换机
       .deadLetterExchange("dl.direct")
       // 指定死信路由键
       .deadLetterRoutingKey("dl")
       .build();
   }
   
   public DirectExchange ttlExchange() {
       return new DirectExchange("ttl.direct");
   }
   
   public Binding ttlBinding() {
       return BindingBuilder.bind(ttlQueue()).to(ttlExchange()).with("ttl");
   }
   ```
   ![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312114719.png)<br />
   ![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312114178.png)

3. 测试一：向超时队列 "ttl.queue" 发送一条消息，消息不设置过期时间；

   ```java
   @Test
   public void testTTLQueue() {
       // 发送消息
       rabbitTemplate.convertAndSend("ttl.direct", "ttl", "hello, ttl queue");
       // 记录日志
       LOGGER.info("发送消息成功");
   }
   ```

   发送消息的日志：

   ```
   2023-07-11 16:16:28.562  INFO 13208 --- [           main] f.xiaorang.rabbitmq.spring.amqp.ApiTest  : 发送消息成功
   ```

   接收消息的日志：

   ```
   2023-07-11 16:16:38.580  INFO 20136 --- [tContainer#11-1] f.x.r.s.amqp.config.TtlMessageConfig     : 从 dl.queue 队列中收到一条延迟消息：hello, ttl queue
   ```

   由于队列的 TTL = 10000 ms = 10 s，可以看到消息发送与接收之间刚好相差 10 秒.

4. 测试二：向超时队列 "ttl.queue" 发送一条消息，消息设置过期时间 expiration = 5000；

   ```java
   @Test
   public void testTTLMessage() throws JsonProcessingException {
       ObjectMapper objectMapper = new ObjectMapper();
       String content = objectMapper.writeValueAsString("hello, ttl message");
       Message message = MessageBuilder
               .withBody(content.getBytes(StandardCharsets.UTF_8))
               .setDeliveryMode(MessageDeliveryMode.PERSISTENT)
           	// 设置过期时间
               .setExpiration("5000")
               .build();
       // 发送消息
       rabbitTemplate.convertAndSend("ttl.direct", "ttl", message);
       // 记录日志
       LOGGER.info("发送消息成功");
   }
   ```

   发送消息的日志：

   ```
   2023-07-11 16:19:30.125  INFO 18876 --- [           main] f.xiaorang.rabbitmq.spring.amqp.ApiTest  : 发送消息成功
   ```

   接收消息的日志：

   ```
   2023-07-11 16:19:35.145  INFO 17076 --- [tContainer#11-1] f.x.r.s.amqp.config.TtlMessageConfig     : 从 dl.queue 队列中收到一条延迟消息：hello, ttl message
   ```

   由于消息的 Expiration = 5000 ms = 5 s，可以看到消息发送与接收之间刚好相差 5 秒. => 如果对两者都进行了设置，则以时间短的为准！
   <a name="abswp"></a>

#### 延迟队列
延迟队列可以解决如下实际的业务需求：

- 下单后，30分钟未支付，则取消订单，回滚库存；
- 新用户注册成功 7 天之后，发送短信问候；

较早版本的 RabbitMQ 没有提供延迟队列的功能，但是可以用 **TTL + 死信队列 **组合的方式实现延迟队列的效果。由于延迟队列的需求非常多，所以 RabbitMQ 官方推出了一个延迟队列的插件。该插件就是 DelayExchange，具体详情请参考 RabbitMQ 插件列表页面 [Community Plugins — RabbitMQ](https://www.rabbitmq.com/community-plugins.html)<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312121173.png)
<a name="IGucS"></a>

##### 安装

1. 下载插件，下载前请先确认 RabbitMQ 版本，下载与之对应版本的插件；<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312121283.png)

2. 上传插件；<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312122848.png)

3. 进入容器内部，将刚上传的插件拷贝到 plugins 目录下；其中的 9e6ae6f9f0e3 需要替换为自己的容器 id

   ```bash
   docker ps -a
   docker cp rabbitmq_delayed_message_exchange-3.12.0.ez 9e6ae6f9f0e3:/plugins
   docker exec -it 9e6ae6f9f0e3 /bin/bash
   ls
   cd plugins
   ls
   ```

   ![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312122281.png)

4. 使用命令`rabbitmq-plugins enable rabbitmq_delayed_message_exchange`启动插件；<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312122975.png)

5. 使用命令`docker restart`重新启动容器；<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312123243.png)
   <a name="K88j2"></a>

##### 使用
该插件使用起来非常简单，如下所示：关于具体如何使用该插件参考自官方博客 [Scheduling Messages with RabbitMQ | RabbitMQ - Blog](https://blog.rabbitmq.com/posts/2015/04/scheduling-messages-with-rabbitmq)

1. DelayExchange 的本质还是官方的三种交换机，只是添加了延迟功能。因此使用时只需声明一个任意类型的交换机，然后设置 delayed 属性为 true 即可，接着再声明一个队列与其进行绑定；

   ```java
   @Configuration
   public class DelayMessageConfig {
       private static final Logger LOGGER = LoggerFactory.getLogger(DelayMessageConfig.class);
   
       @RabbitListener(bindings = {
           @QueueBinding(
               value = @Queue(name = "delay.queue"),
               exchange = @Exchange(name = "delay.direct", delayed = "true"),
               key = {"delay"}
           )
       })
       public void listenDelayMessage(String message) {
           LOGGER.info("从 delay.queue 队列中收到一条延迟消息：{}", message);
       }
   }
   ```

   ![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312123608.png)	<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312123725.png)

2. 发送消息时，消息必须携带 "x-delay" 属性，用于指定消息的延迟时间；

   ```java
   @Test
   public void testDelayMessage() throws JsonProcessingException {
       ObjectMapper objectMapper = new ObjectMapper();
       String content = objectMapper.writeValueAsString("hello, delay message");
       Message message = MessageBuilder
               .withBody(content.getBytes(StandardCharsets.UTF_8))
               .setDeliveryMode(MessageDeliveryMode.PERSISTENT)
               // 设置过期时间
               .setHeader("x-delay", 5000)
               .build();
       // 发送消息
       rabbitTemplate.convertAndSend("delay.direct", "delay", message);
       // 记录日志
       LOGGER.info("发送消息成功");
   }
   ```

   发送消息的日志：

   ```
   2023-07-11 18:10:04.862  INFO 2972 --- [           main] f.xiaorang.rabbitmq.spring.amqp.ApiTest  : 发送消息成功
   2023-07-11 18:10:04.867 ERROR 2972 --- [nectionFactory1] f.x.r.spring.amqp.config.CommonConfig    : 消息路由失败，应答码：312，原因：NO_ROUTE， 交换机：delay.direct，路由键：delay，消息：(Body:'[B@7247177e(byte[22])' MessageProperties [headers={}, contentType=application/octet-stream, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, receivedDelay=5000, deliveryTag=0])
   ```

   接收消息的日志：

   ```
   2023-07-11 18:10:09.868  INFO 2172 --- [ntContainer#0-1] f.x.r.s.amqp.config.DelayMessageConfig   : 从 delay.queue 队列中收到一条延迟消息：hello, delay message
   ```

   可以看到消息发送与接收之间刚好相差 5 秒.<br />❗注意：在 [GitHub - rabbitmq/rabbitmq-delayed-message-exchange: Delayed Messaging for RabbitMQ](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange#limitations) 指出 DelayExchange 插件不支持 mandatory = true 参数，如果启用的话会同时收到延迟消息和路由失败消息。mandatory：如果 exchange 根据自身类型和消息 routingKey 无法找到一个合适的 queue 存储消息，那么 broker 会调用 basic.return 方法将消息返还给生产者，为 false 则会直接丢弃消息。建议在路由失败回调方法中加入特殊判断，如果是延迟队列的话就不作处理直接返回。

   ```java
   @Configuration
   public class CommonConfig implements ApplicationContextAware {
       private static final Logger LOGGER = LoggerFactory.getLogger(CommonConfig.class);
   
       @Override
       public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
           RabbitTemplate rabbitTemplate = applicationContext.getBean(RabbitTemplate.class);
           rabbitTemplate.setReturnsCallback(returnedMessage -> {
               Message message = returnedMessage.getMessage();
               String exchange = returnedMessage.getExchange();
               int replyCode = returnedMessage.getReplyCode();
               String replyText = returnedMessage.getReplyText();
               String routingKey = returnedMessage.getRoutingKey();
               // 判断是否为延迟消息
               Integer delay = Optional.ofNullable(message.getMessageProperties().getReceivedDelay()).orElse(0);
               if (delay > 0) {
                   return;
               }
               // 投递失败，记录日志
               LOGGER.error("消息路由失败，应答码：{}，原因：{}， 交换机：{}，路由键：{}，消息：{}", replyCode, replyText, exchange, routingKey, message);
               // 如果数据可靠性要求高，可以进行失败重试，使用 spring-retry 重发消息；
           });
       }
   }
   ```

<a name="Xr6aQ"></a>
### 惰性队列
当生产者发送消息的速度超过了消费者处理消息的速度，就会导致队列中的消息堆积，直至队列存储消息达到上限为止。最早接收到的消息可能会成为死信，会被丢弃，这就是消息堆积问题。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312125172.jpeg" style="zoom:50%;" /><br />解决消息堆积问题有如下三种思路：

- 增加更多消费者，提高消费速度；
- 在消费者内开启线程池加快消息处理速度；
- 扩大队列容积，提高堆积上限；

从 RabbitMQ 3.6.0 版本开始，就增加了 Lazy Queues 的概念，也就是惰性队列。惰性队列的特征如下：

- 接收到消息后直接存入磁盘而非内存；
- 消费者要消费消息时才会从磁盘中读取并加载到内存；
- 支持数百万条消息的存储；

要设置一个队列为惰性队列，只需在声明队列时，指定`x-queue-mode`属性为 lazy 即可。可以通过命令行将一个正在运行中的队列修改为惰性队列：
```shell
rabbitmqctl set_policy Lazy "^lazy-queue$" '{"queue-mode":"lazy"}' --apply-to queues  
```

- `rabbitmqctl`：RabbitMQ 的命令行工具
- `set_policy`：添加一个策略
- `Lazy`：策略名称，可以自定义
- `"^lazy-queue$"`：用正则表达式匹配队列的名字
- `'{"queue-mode":"lazy"}'`：设置队列模式为 lazy 模式
- `--apply-to queues` ：策略的作用对象，是所有的队列

基于 @Bean 注解声明惰性队列
```java
@Bean
public Queue lazyQueue() {
    return QueueBuilder.durable("lazy.queue").lazy().build();
}
```
基于 @RabbitListener 声明惰性队列
```java
@Configuration
public class LazyQueueConfig {
    private static final Logger LOGGER = LoggerFactory.getLogger(LazyQueueConfig.class);

    @RabbitListener(queuesToDeclare = {
            @Queue(
                    name = "lazy.queue",
                    durable = "true",
                    arguments = {@Argument(name = "x-queue-mode", value = "lazy")}
            )
    })
    public void listenLazyMessage(String message) {
        LOGGER.info("从 lazy.queue 队列中收到一条消息：{}", message);
    }
}
```
```java
@Test
public void testLazyQueue() {
    rabbitTemplate.convertAndSend("lazy.queue", "hello, lazy queue");
}
```
测试结果如下所示：
```
2023-07-11 21:52:45.299  INFO 10420 --- [ntContainer#3-1] f.x.r.s.amqp.config.LazyQueueConfig      : 从 lazy.queue 队列中收到一条消息：hello, lazy queue
```
<a name="MWQI2"></a>
### 幂等性保障
<span style="background-color: rgb(251, 228, 231);">TODO</span>
<a name="Q6ZkM"></a>

### 消息顺序输出
<span style="background-color: rgb(251, 228, 231);">TODO</span>
