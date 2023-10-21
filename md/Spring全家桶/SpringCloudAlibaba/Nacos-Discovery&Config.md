# Nacos-Discovery&Config

Nacos 是 Dynamic Naming and Configuration Service 的首字母简称，是 Alibaba 开源的、易于构建云原生应用的<u>动态服务发现</u>、<u>配置管理</u>和<u>服务管理</u>平台。<br />Nacos 致力于帮助你发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助你快速实现动态服务发现、服务配置、服务元数据及流量管理。<br />Nacos 帮助你更敏捷和容易地构建、交付和管理微服务平台。Nacos 是构建以 "**服务**" 为中心地现代应用架构（例如微服务范式、云原生范式）的服务基础设施。

## 什么是 Nacos？

**服务（Service）**是 Nacos 世界的一等公民。Nacos 支持几乎所有主流类型的 "**服务**" 的发现、配置和管理：

- [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)
- [gRPC](https://grpc.io/docs/guides/concepts.html#service-definition) & [Dubbo RPC Service](https://dubbo.incubator.apache.org/)
- [Spring Cloud RESTful Service](https://spring.io/projects/spring-restdocs)

其关键特性包括:

- 服务发现和服务健康监测：支持基于 DNS 和基于 RPC 的服务发现，提供对服务的实时的健康检查，阻止向不健康的主机或服务实例发送请求；
- 动态配置服务：动态配置服务可以让你以中心化、外部化和动态化的方式管理所有环境的应用配置和服务配置；
- 动态 DNS 服务：动态 DNS 服务支持权重路由，让你更容易地实现中间层负载均衡、更灵活的路由策略、流量控制以及数据中心内网的简单 DNS 解析服务；
- 服务及其元数据管理：支持从微服务平台建设的视角管理数据中心的所有服务及元数据；

## 安装

### 预备环境准备

Nacos 依赖 Java 环境来运行，请确保是在以下版本环境中安装使用：

1. 64 bit OS，支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac；

2. 64 bit JDK 1.8+；
   1. 下载，请务必选对版本！[Download the Latest Java LTS Free](https://www.oracle.com/java/technologies/downloads/#java8)<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011511013.png)

   2. 解压

      ```bash
      tar -zxvf jdk-8u371-linux-x64.tar.gz
      ```

   3. 配置环境变量

      ```bash
      vim /etc/profile
      ```

   4. 输入`i`进入编辑模式，将以下内容粘贴到文件末尾，`esc` ➡️ `:wq`，保存退出

      ```bash
      export JAVA_HOME=/usr/local/java/jdk1.8.0_371
      export PATH=$JAVA_HOME/bin:$PATH
      export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
      ```

   5. 重新加载配置文件

      ```bash
      source /etc/profile 
      ```

   6. 使用`java -version`命令验证是否安装成功 <br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011513845.png)

### 版本选择
在安装前请先确认项目中使用的 SpringCloudAlibaba 版本 ➡️ Nacos 组件所对应的版本，[版本说明](https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E)

| SpringCloudAlibaba | Spring Cloud | SpringBoot | Nacos |
| --- | --- | --- | --- |
| 2.2.9.RELEASE | Spring Cloud Hoxton.SR12 | 2.3.12.RELEASE | 2.1.0 |

如上表所示，假如 SpringCloudAlibaba 选择 2.2.9.RELEASE 版本，则 Nacos 组件对应的版本为 2.1.0，因此需要安装 2.1.0 版本的 nacos-server 服务器 [Release 2.1.0 (Apr 29, 2022) · alibaba/nacos (github.com)](https://github.com/alibaba/nacos/releases/tag/2.1.0).<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011514024.png)
```bash
tar -zxvf nacos-server-2.1.0.tar.gz
```
### 启动服务器
Linux & 单机模式 `sh startup.sh -m standalone`<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011514320.png)<br />查看日志 `tail -f ../logs/nacos.log`<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011514458.png)<br />❗注意：服务器防火墙需要开放 8848 端口！访问 [http://42.194.233.222:8848/nacos](http://42.194.233.222:8848/nacos)<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011514543.png)<br />如上图所示，说明 nacos 服务器已经安装完成，可以正式开始使用！默认账号密码都是 nacos.

---

由于本人的 Linux 服务器比较差，所以换到 Windows 环境下使用，具体步骤如下所示：

1. 下载安装包并解压；<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011515323.png)

2. 进入解压文件夹目录，使用`.\startup.cmd -m standalone`命令启动 Nacos 服务器；<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011515748.png)

3. 访问 [http://localhost:8848/nacos/](http://localhost:8848/nacos/)，默认账号密码都是 nacos.<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011515080.png)

## Spring Cloud Alibaba Nacos Discovery

该项目通过自动配置以及其他 Spring 编程模型的习惯用法为 SpringBoot 应用程序在服务注册与发现方面提供和 Nacos 的无缝集成。通过一些简单的注解，可以快速注册一个服务，并使用经过双十一考验的 Nacos 组件来作为大规模分布式系统的服务注册中心。
### 服务注册/发现: Nacos Discovery Starter
服务发现是微服务架构体系中最关键的组件之一。如果尝试着用手动的方式给每一个客户端来配置所有服务提供者的服务列表是一件非常困难的事，而且不利于服务的动态扩缩容。Nacos Discovery Starter 可以帮助你**将服务<u>自动注册</u>到 Nacos 服务端**并且**能够动态感知和刷新某个服务实例的服务列表**。除此之外，Nacos Discovery Starter 也将服务实例自身的一些元数据信息（如 host、port、健康检查 URL、主页等）注册到 Nacos。

### 如何引入 Nacos Discovery Starter 进行服务注册/发现

如果你想要在项目中使用 Nacos 来实现服务注册/发现，可以使用 groupId 为`com.alibaba.cloud`和 artifactId 为`spring-cloud-starter-alibaba-nacos-discovery`的 Starter。
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```
### 一个使用 Nacos Discovery 进行服务注册/发现并调用的例子
<img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011517289.jpeg" style="zoom: 67%;" />

#### 父模块
首先创建父模块 `spring-cloud-study`，其 pom.xml 配置文件如下所示：确定使用 的 SpringCloud 版本为 `Hoxton.SR12`。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>fun.xiaorang</groupId>
    <artifactId>spring-cloud-study</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <spring.cloud.version>Hoxton.SR12</spring.cloud.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```
再创建 `spring-cloud-alibaba-study` 模块，它的的父模块为 `spring-cloud-study` 模块，其 pom.xml 配置文件如下所示：确定使用的 SpringCloudAlibaba 版本为 `2.2.9.RELEASE`，SpringBoot 版本为 `2.3.12.RELEASE`。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>fun.xiaorang</groupId>
        <artifactId>spring-cloud-study</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <artifactId>spring-cloud-alibaba-study</artifactId>
    <packaging>pom</packaging>

    <properties>
        <spring.cloud.alibaba.version>2.2.9.RELEASE</spring.cloud.alibaba.version>
        <spring.boot.version>2.3.12.RELEASE</spring.boot.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```
以上两个模块主要用于管理依赖的版本；<br />再创建 `spring-cloud-alibaba-nacos-study` 模块，它的父模块为 `spring-cloud-alibaba-study` 模块，其 pom.xml 配置文件如下所示：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>fun.xiaorang</groupId>
        <artifactId>spring-cloud-alibaba-study</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>spring-cloud-alibaba-nacos-study</artifactId>
    <packaging>pom</packaging>
</project>
```
最后创建 `nacos-discovery-study` 模块，它的父模块为 `spring-cloud-alibaba-nacos-study` 模块，其 pom.xml 配置文件如下所示：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>fun.xiaorang</groupId>
        <artifactId>spring-cloud-alibaba-nacos-study</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>nacos-discovery-study</artifactId>
    <packaging>pom</packaging>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
      <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
#### 启动一个 Provider 应用
创建一个 `nacos-discovery-provider` 模块，它的父模块为 `nacos-discovery-study` 模块，其 pom.xml 配置文件如下所示：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>fun.xiaorang</groupId>
        <artifactId>nacos-discovery-study</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>nacos-discovery-provider</artifactId>
    <description>服务提供者</description>

</project>
```
application.yml 配置文件如下所示：配置当前服务的名称为 `nacos-provider`，端口为 `8081`、nacos 服务器地址为 `127.0.0.1:8848`；
```yaml
server:
  port: 8081
spring:
  application:
    name: nacos-provider
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
management:
  endpoints:
    web:
      exposure:
        include: '*'
```
对于以上 Nacos Starter 配置不清楚的小伙伴可以参考 [关于 Nacos Starter 更多的配置项信息](https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/zh-cn/index.html#_关于_nacos_discovery_starter_更多的配置项信息)，如下列举出常用配置项：

| 配置项     | Key                                      | 默认值                     | 说明                                                         |
| ---------- | ---------------------------------------- | -------------------------- | ------------------------------------------------------------ |
| 服务端地址 | spring.cloud.nacos.discovery.server-addr |                            | Nacos Server 启动监听的 ip 地址和端口                        |
| 服务名     | spring.cloud.nacos.discovery.service     | ${spring.application.name} | 给当前的服务命名                                             |
| 服务分组   | spring.cloud.nacos.discovery.group       | DEFAULT_GROUP              | 设置服务所处的分组                                           |
| 权重       | spring.cloud.nacos.discovery.weight      | 1                          | 取值范围 1 到 100，数值越大，权重越大                        |
| 命名空间   | spring.cloud.nacos.discovery.namespace   |                            | 常用场景之一是不同环境的注册的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。 |

启动 Provider 示例，如下所示：

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(NacosProviderApplication.class, args);
    }

    @RestController
    public static class EchoController {
        @GetMapping(value = "/echo/{string}")
        public String echo(@PathVariable String string) {
            return "Hello Nacos Discovery " + string;
        }
    }
}
```
> [!NOTE]
>
> 在启动 Provider 应用之前请先确保 Nacos 服务已经启动！

项目启动成功之后，查看 Nacos 控制台，可以看到 nacos-provider 服务已经注册成功！<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011518660.png)

#### 启动一个 Consumer 应用

创建一个 `nacos-discovery-consumer` 模块，它的父模块为 `nacos-discovery-study` 模块，其 pom.xml 配置文件如下所示：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>fun.xiaorang</groupId>
        <artifactId>nacos-discovery-study</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>nacos-discovery-consumer</artifactId>
    <description>服务消费者</description>
    
</project>
```
application.yml 配置如下所示：配置当前服务的名称为 `nacos-consumer`，端口为 `8082`、nacos 服务器地址为 `127.0.0.1:8848`；
```yaml
server:
  port: 8082
spring:
  application:
    name: nacos-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
management:
  endpoints:
    web:
      exposure:
        include: '*'
```
在 Consumer 端需要去调用 Provider 端提供的 REST 服务。在以下例子中使用最原始的一种方式，即显示的使用 `LoadBalancerClient` 和 `RestTemplate` 结合的方式来访问。

> [!ATTENTION]
>
> 以下这种服务调用方式的 `RestTemplate` 不能标注 `@LoadBalanced` 注解，否则的话会报错！

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(NacosConsumerApplication.class, args);
    }

    /**
     * 实例化 RestTemplate 实例
     */
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @RestController
    public static class NacosController {
        private final LoadBalancerClient loadBalancerClient;
        private final RestTemplate restTemplate;

        @Value("${spring.application.name}")
        private String appName;

        public NacosController(LoadBalancerClient loadBalancerClient, RestTemplate restTemplate) {
            this.loadBalancerClient = loadBalancerClient;
            this.restTemplate = restTemplate;
        }

        @GetMapping("/echo/app-name")
        public String echoAppName() {
            //使用 LoadBalanceClient 和 RestTemplate 结合的方式来访问
            ServiceInstance serviceInstance = loadBalancerClient.choose("nacos-provider");
            String url = String.format("http://%s:%s/echo/%s", serviceInstance.getHost(), serviceInstance.getPort(), appName);
            System.out.println("request url:" + url);
            return restTemplate.getForObject(url, String.class);
        }
    }
}
```
> [!CODE|label:另一种服务调用方式：通过带有负载均衡的 RestTemplate 也是可以访问的！至于 FeignClient 这种访问方式后续再学习！]
>
> ```java
> @SpringBootApplication
> @EnableDiscoveryClient
> public class NacosConsumerApplication {
>        public static void main(String[] args) {
>            SpringApplication.run(NacosConsumerApplication.class, args);
>        }
>     
>        /**
>      * 实例化 RestTemplate 实例
>      * 标注 @LoadBalanced 注解即可在 RestTemplate 上开启 LoadBalanced 负载均衡的功能
>      */
>     @Bean
>     @LoadBalanced
>     public RestTemplate restTemplate() {
>         return new RestTemplate();
>     }
> 
>     @RestController
>     public static class NacosController {
>         private final RestTemplate restTemplate;
> 
>         @Value("${spring.application.name}")
>         private String appName;
> 
>         public NacosController(RestTemplate restTemplate) {
>             this.restTemplate = restTemplate;
>         }
> 
>         @GetMapping("/echo/app-name")
>         public String echoAppName() {
>             String url = String.format("http://%s/echo/%s", "nacos-provider", appName);
>             System.out.println("request url:" + url);
>             return restTemplate.getForObject(url, String.class);
>         }
>     }
> }
> ```

在这个例子中咱们注入了一个 `LoadBalancerClient` 的实例（因为 `spring-cloud-starter-alibaba-nacos-discovery` 内置了 <u>Ribbon</u>，所以可以直接注入 `LoadBalancerClient`），并且手动的实例化一个 `RestTemplate`，同时将 `spring.application.name`的配置值注入到应用中来，目的是调用 Provider 提供的服务时，将当前配置的应用名给显示出来。

项目启动成功之后，查看 Nacos 控制台，可以看到 nacos-consumer 服务已经注册成功！<br />
![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011525191.png)<br />
访问 [http://localhost:8082/echo/app-name](http://localhost:8082/echo/app-name)，浏览器输出 "Hello Nacos Discovery nacos-consumer" 字样。

### Nacos Discovery 对外暴露的 Endpoint

`spring-cloud-starter-alibaba-nacos-discovery` 在实现的时候提供了一个 EndPoint，EndPoint 的访问地址为 `http://ip:port/actuator/nacos-discovery`。EndPoint 的信息主要提供了两类：

1. subscribe：显示了当前服务有哪些订阅者
2. NacosDiscoveryProperties：当前应用 Nacos 的基础配置信息

> [!ATTENTION]
>
> 此处有一个坑，官方文档说 EndPoint 的访问地址为 `http://ip:port/actuator/nacos-discovery`，其实这是错的，本人尝试许久都不行，访问一直报 404！如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011533022.png) <br />后面直接访问一下  `http://ip:port/actuator`，发现 nacos discovery 的 EndPoint 访问地址为 `http://ip:port/actuator/nacosdiscovery`，nacos 与 discovery 之间并没有什么短横线！ <br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011529608.png)

访问 [http://localhost:8081/actuator/nacosdiscovery](http://localhost:8081/actuator/nacosdiscovery)，Provider 服务实例访问 EndPoint 的信息如下所示：

```json
{
    "subscribe":[
        {
            "name":"nacos-provider",
            "groupName":"DEFAULT_GROUP",
            "clusters":"DEFAULT",
            "cacheMillis":1000,
            "hosts":[
                {
                    "ip":"192.168.1.3",
                    "port":8081,
                    "weight":1,
                    "healthy":true,
                    "enabled":true,
                    "ephemeral":true,
                    "clusterName":"DEFAULT",
                    "serviceName":"DEFAULT_GROUP@@nacos-provider",
                    "metadata":{
                        "preserved.register.source":"SPRING_CLOUD"
                    },
                    "ipDeleteTimeout":30000,
                    "instanceHeartBeatInterval":5000,
                    "instanceHeartBeatTimeOut":15000
                }
            ],
            "lastRefTime":0,
            "checksum":"",
            "allIPs":false,
            "reachProtectionThreshold":false,
            "valid":true
        }
    ],
    "NacosDiscoveryProperties":{
        "serverAddr":"127.0.0.1:8848",
        "username":"",
        "password":"",
        "endpoint":"",
        "namespace":"",
        "watchDelay":30000,
        "logName":"",
        "service":"nacos-provider",
        "weight":1,
        "clusterName":"DEFAULT",
        "group":"DEFAULT_GROUP",
        "namingLoadCacheAtStart":"false",
        "metadata":{
            "preserved.register.source":"SPRING_CLOUD"
        },
        "registerEnabled":true,
        "ip":"192.168.1.3",
        "networkInterface":"",
        "ipType":"IPv4",
        "port":8081,
        "secure":false,
        "accessKey":"",
        "secretKey":"",
        "heartBeatInterval":null,
        "heartBeatTimeout":null,
        "ipDeleteTimeout":null,
        "instanceEnabled":true,
        "ephemeral":true,
        "failFast":true,
        "nacosProperties":{
            "password":"",
            "endpoint":"",
            "secretKey":"",
            "serverAddr":"127.0.0.1:8848",
            "accessKey":"",
            "clusterName":"DEFAULT",
            "namespace":"",
            "com.alibaba.nacos.naming.log.filename":"",
            "namingLoadCacheAtStart":"false",
            "username":""
        }
    }
}
```

## Spring Cloud Alibaba Nacos Config

Nacos 提供用于存储配置和其他元数据的 key/value 存储，为分布式系统中的外部化配置提供服务器端和客户端支持。使用 Spring Cloud Alibaba Nacos Config，你可以在 Nacos Server 集中管理你的 Spring Cloud 应用的外部属性配置。

Spring Cloud Alibaba Nacos Config 是 Config Server 和 Client 的替代方案，客户端和服务器上的概念与 Spring <u>Enviroment</u> 和 <u>PropertySource</u> 有着一致的抽象，在特殊的 <u><span style="background-color: rgb(232, 247, 207);">bootstrap</span></u> 阶段，配置被加载到 Spring 环境中。当应用程序从开发 ➡️ 测试 ➡️ 生产时，你可以管理这些环境之间的配置，并确保应用程序具有迁移时需要运行的所有内容。

### 如何引入 Nacos Config Starter 进行配置管理

如果你想要在项目中使用 Nacos 来实现配置管理，可以使用 groupId 为`com.alibaba.cloud`和 artifactId 为`spring-cloud-starter-alibaba-nacos-config`的 Starter。
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```
### 快速开始
#### Nacos 服务端初始化

在启动 Nacos Server 之后，先添加一个配置，如下所示：
```
Data ID:    nacos-config.properties

Group  :    DEFAULT_GROUP

配置格式:    Properties

配置内容：   user.name=nacos-config-properties
            user.age=90
```
> [!NOTE]
>
> dataId 是以 **properties（默认的文件扩展名方式）**为扩展名。

![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011534793.png)<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011534294.png)

#### 客户端使用方式
创建一个 `nacos-config-study` 模块，它的父模块为 `spring-cloud-alibaba-nacos-study` 模块，其 pom.xml 配置文件如下所示：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>fun.xiaorang</groupId>
        <artifactId>spring-cloud-alibaba-nacos-study</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>nacos-config-study</artifactId>
    <description>nacos 作为配置中心</description>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
    </dependencies>

</project>
```
 nacos config 示例，如下所示：
```java
@SpringBootApplication
public class NacosConfigApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(NacosConfigApplication.class, args);
        String userName = applicationContext.getEnvironment().getProperty("user.name");
        String userAge = applicationContext.getEnvironment().getProperty("user.age");
        System.err.println("user name :" + userName + "; age: " + userAge);
    }
}
```
> [!ATTENTION]
>
> 在运行此示例程序之前，**必须使用 bootstrap.properties 或者 <u>bootstrap.yml</u> 配置文件来配置 Nacos Server 地址！**

bootstrap.yml 配置文件如下所示：

```yaml
server:
  port: 8083
spring:
  application:
    name: nacos-config
  cloud:
    nacos:
      config:
        server-addr: localhost:8848
management:
  endpoints:
    web:
      exposure:
        include: '*'
```
> [!NOTE]
>
> 当你使用使用域名的方式来访问 Nacos 时，`spring.cloud.nacos.config.server-addr`配置的方式为`域名:port`。例如 Nacos 的域名为 abc.com.nacos，监听的端口为 80，则`spring.cloud.nacos.config.server-addr=abc.com.nacos:80`。❗注意：80 端口不能省略！

对于以上 Nacos Config Starter 配置不清楚的小伙伴可以参考 [关于 Nacos Config Starter 更多的配置项信息](https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/zh-cn/index.html#_关于_nacos_config_starter_更多的配置项信息)，如下列举出常用配置项：

| 配置项            | Key                                         | 默认值                     | 说明                                                         |
| ----------------- | ------------------------------------------- | -------------------------- | ------------------------------------------------------------ |
| 服务端地址        | spring.cloud.nacos.config.server-addr       |                            | Nacos Server 启动监听的 ip 地址和端口                        |
| 配置对应的 DataId | spring.cloud.nacos.config.name              |                            | `dataId = ${prefix}-${spring.profiles.active}.${file-extension}`，而其中 prefix 的可选值有如下三种，优先级从高到底依次为：`spring.cloud.nacos.config.prefix` ➡️ `spring.cloud.nacos.config.name` ➡️ 默认值`${spring.application.name}` |
| 配置对应的 DataId | spring.cloud.nacos.config.prefix            | ${spring.application.name} | 同上                                                         |
| GROUP             | spring.cloud.nacos.config.group             | DEFAULT_GROUP              | 配置对应的组                                                 |
| 文件扩展名        | spring.cloud.nacos.config.fileExtension     | properties                 | 配置项对应的文件扩展名，目前支持 properties 和 yaml(yml)     |
| 命名空间          | spring.cloud.nacos.config.namespace         |                            | 常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等 |
| 共享配置          | spring.cloud.nacos.config.shared-configs    |                            | 属性是个集合，内部由 `Config` POJO 组成。`Config` 有 4 个属性，分别是 `dataId`, `group`，`fileExtension`以及 `refresh` |
| 扩展配置          | spring.cloud.nacos.config.extension-configs |                            | 属性是个集合，内部由 `Config` POJO 组成。`Config` 有 4 个属性，分别是 `dataId`, `group`，`fileExtension`以及 `refresh` |

现在启动该示例程序，可以看到如下输出结果：

```
2023-07-13 15:14:06.359  INFO 11824 --- [           main] f.x.s.c.a.nacos.NacosConfigApplication   : Started NacosConfigApplication in 2.483 seconds (JVM running for 2.873)
user name :nacos-config-properties; age: 90
2023-07-13 15:14:06.650  INFO 11824 --- [(3)-192.168.1.3] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
```
### 基于 DataId 为 yaml 的文件扩展名配置方式
`spring-cloud-alibaba-nacos-config` 对于 yaml 格式也是完美支持的。此时只需要完成以下两步：

1. 在应用的 bootstrap.properties 或者 bootstrap.yml 配置文件中显示的声明 dataId 文件扩展名，即 `file-extension: yaml`。如下所示：

   ```yaml
   server:
     port: 8083
   spring:
     application:
       name: nacos-config
     cloud:
       nacos:
         config:
           server-addr: localhost:8848
           file-extension: yaml
   management:
     endpoints:
       web:
         exposure:
           include: '*'
   ```
2. 在 Nacos 的控制台新增一个 dataId 为 yaml 为扩展名的配置，如下所示：

   ```
   Data ID:        nacos-config.yaml
   
   Group  :        DEFAULT_GROUP
   
   配置格式:        YAML
   
   配置内容:        user.name: nacos-config-yaml
                   user.age: 68
   ```

   ![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011538005.png)

这两步完成之后，重新启动示例程序，可以看到如下输出结果：

```
2023-07-13 15:05:16.886  INFO 18916 --- [           main] f.x.s.c.a.nacos.NacosConfigApplication   : Started NacosConfigApplication in 2.385 seconds (JVM running for 2.871)
user name :nacos-config-yaml; age: 68
2023-07-13 15:05:17.185  INFO 18916 --- [(3)-192.168.1.3] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
```
### 支持配置的动态更新
`spring-cloud-alibaba-nacos-config` <u>默认</u>支持<u>**配置的动态更新**</u>，不过可以通过配置 `spring.cloud.nacos.config.refresh.enabled=false` 来关闭动态刷新。

启动 SpringBoot 应用测试的代码如下所示：

```java
@SpringBootApplication
public class NacosConfigApplication {
    public static void main(String[] args) throws InterruptedException {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(NacosConfigApplication.class, args);
        while (true) {
            //当动态配置刷新时，会更新到 Enviroment 中，因此这里每隔一秒中从 Enviroment 中获取配置
            String userName = applicationContext.getEnvironment().getProperty("user.name");
            String userAge = applicationContext.getEnvironment().getProperty("user.age");
            System.err.println("user name :" + userName + "; age: " + userAge);
            TimeUnit.SECONDS.sleep(1);
        }
    }
}
```
如下所示，当变更 `user.name` 时，应用程序中能够获取到最新的值：
```
user name :nacos-config-yaml; age: 68
user name :nacos-config-yaml; age: 68
user name :nacos-config-yaml; age: 68
user name :nacos-config-yaml; age: 68
2023-07-13 15:15:58.902  WARN 8864 --- [ternal.notifier] c.a.c.n.c.NacosPropertySourceBuilder     : Ignore the empty nacos configuration and get it based on dataId[nacos-config] & group[DEFAULT_GROUP]
2023-07-13 15:15:58.906  INFO 8864 --- [ternal.notifier] b.c.PropertySourceBootstrapConfiguration : Located property source: [BootstrapPropertySource {name='bootstrapProperties-nacos-config.yaml,DEFAULT_GROUP'}, BootstrapPropertySource {name='bootstrapProperties-nacos-config,DEFAULT_GROUP'}]
2023-07-13 15:15:58.907  INFO 8864 --- [ternal.notifier] o.s.boot.SpringApplication               : No active profile set, falling back to default profiles: default
2023-07-13 15:15:58.913  INFO 8864 --- [ternal.notifier] o.s.boot.SpringApplication               : Started application in 0.111 seconds (JVM running for 28.015)
2023-07-13 15:15:58.994  INFO 8864 --- [ternal.notifier] o.s.c.e.event.RefreshEventListener       : Refresh keys changed: [user.name]
// 从 Enviroment 中 读取到更改后的值
user name :nacos-config-yaml-update; age: 68
user name :nacos-config-yaml-update; age: 68
```
### 支持 profile 粒度的配置

`spring-cloud-starter-alibaba-nacos-config` 在加载配置的时候，不仅仅加载 dataId 为`${spring.application.name}.${file-extension:properties}`为前缀的基础配置，还加载了 dataId 为`${spring.application.name}-${profile}.${file-extension:properties}`的配置。在日常开发中如果遇到多套环境下的不同配置，可以通过 Spring 提供的`spring.profiles.active`这个配置项来配置。

```yaml
server:
  port: 8083
spring:
  application:
    name: nacos-config
  cloud:
    nacos:
      config:
        server-addr: localhost:8848
        file-extension: yaml
  profiles:
    active: develop
management:
  endpoints:
    web:
      exposure:
        include: '*'
```
> [!ATTENTION]
>
> ${spring.profiles.active} 当通过配置文件来指定时必须放在 bootstrap.properties 或者 **<u>bootstrap.yml</u>** 配置文件中。

Nacos 上新增一个 dataId 为 nacos-config-develop.yaml 的基础配置，如下所示：

```
Data ID:        nacos-config-develop.yaml

Group  :        DEFAULT_GROUP

配置格式:        YAML

配置内容:        current.env: develop-env
```
![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011539821.png)<br />启动 SpringBoot 应用测试的代码如下所示：
```java
@SpringBootApplication
public class NacosConfigApplication {
    public static void main(String[] args) throws InterruptedException {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(NacosConfigApplication.class, args);
        while (true) {
            //当动态配置刷新时，会更新到 Enviroment 中，因此这里每隔一秒中从 Enviroment 中获取配置
            String userName = applicationContext.getEnvironment().getProperty("user.name");
            String userAge = applicationContext.getEnvironment().getProperty("user.age");
            //获取当前部署的环境
            String currentEnv = applicationContext.getEnvironment().getProperty("current.env");
            System.err.println("in " + currentEnv + " enviroment; " + "user name :" + userName + "; age: " + userAge);
            TimeUnit.SECONDS.sleep(1);
        }
    }
}
```
启动后，可见控制台的输出结果：
```
2023-07-13 15:37:35.100  INFO 17260 --- [           main] f.x.s.c.a.nacos.NacosConfigApplication   : Started NacosConfigApplication in 2.476 seconds (JVM running for 2.948)
in develop-env enviroment; user name :nacos-config-yaml-update; age: 68
2023-07-13 15:37:35.297  INFO 17260 --- [(3)-192.168.1.3] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
```
如果需要切换到生产环境，只需要更改`${spring.profiles.active}`参数配置即可。如下所示：
```yaml
server:
  port: 8083
spring:
  application:
    name: nacos-config
  cloud:
    nacos:
      config:
        server-addr: localhost:8848
        file-extension: yaml
  profiles:
    active: product
management:
  endpoints:
    web:
      exposure:
        include: '*'
```
同时生产环境上的 Nacos 添加对应 dataId 的基础配置即可。例如，在生产环境下的 Nacos 添加了 dataId 为 nacos-config-product.yaml 的配置：
```
Data ID:        nacos-config-product.yaml

Group  :        DEFAULT_GROUP

配置格式:        YAML

配置内容:        current.env: product-env
```
![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011540363.png)<br />启动测试程序，输出结果如下所示：
```
2023-07-13 15:57:08.931  INFO 15096 --- [           main] f.x.s.c.a.nacos.NacosConfigApplication   : Started NacosConfigApplication in 2.349 seconds (JVM running for 2.792)
in product-env enviroment; user name :nacos-config-yaml-update; age: 68
2023-07-13 15:57:09.295  INFO 15096 --- [(1)-192.168.1.3] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
```
在此案例中咱们通过`spring.profiles.active=<profilename>`的方式写死在配置文件中，而在真正的项目实施过程中这个变量的值是需要根据不同环境而有不同的值。

> [!IMPORTANT]
>
> 通常的做法是通过<u>`-Dspring.profiles.active=<profile>`</u>参数指定其配置来达到环境间灵活的切换；如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308011541299.png)

### 支持自定义 namespace 的配置
namespace：命名空间，用于进行租户粒度的配置隔离；对于 Nacos 中某些概念不懂的小伙伴可以查看 [Nacos 概念](https://nacos.io/zh-cn/docs/concepts.html)

- 在不同的命名空间下，可以存在相同的 group 或 dataId 的配置；

- namespace 的常用场景之一是**对不同环境的配置进行区分隔离**，例如开发环境（dev）、测试环境（test）和生产环境（prod）的资源（如配置、服务）隔离等；

- 在没有明确指定`${spring.cloud.nacos.config.namespace}`配置的情况下，<u>默认</u>使用的是 Nacos 上 **<u>public</u>** 这个 namespace。如果需要使用自定义的命名空间，可以通过以下配置来实现：

  ```properties
  spring.cloud.nacos.config.namespace=1330e6ed-dcc3-4fe4-9b36-b3d1b2f3c16b
  ```

  > [!ATTENTION]
  >
  > 该配置必须放在 bootstrap.properties 或者 **bootstrap.yml**  配置文件中。
  
  此时`spring.cloud.nacos.config.namespace`的值是 namespace 对应的 id，该 id 值可以在 Nacos 的控制台获取。并且在添加配置时注意不要选择其他的 namespace，否则将会导致读取不到正确的配置。

### 支持自定义 Group 的配置
group：配置分组，Nacos 中的一组配置集，是组织配置的维度之一；

> [!NOTE|label:何为配置项和配置集？]
>
> - 配置项：一个具体的可配置的参数与其值域，通常以 param-key=param-value 的形式存在。例如咱们常配置系统的日志输出级别（logLevel=INFO|WARN|ERROR）就是一个配置项。
> - 配置集：一组相关或者不相关的配置项合称为配置集。在系统中，<u>一个配置文件通常就是一个配置集</u>，包含了系统各方面的配置。例如，一个配置集可能包含了数据源、线程池、日志级别等配置项。

- 通过一个有意义的字符串（如 Buy 或 Trade）对配置集进行分组，从而区分 dataId 相同的配置集；

- group 的常见场景：不同的应用或组件使用了相同的配置类型，如 database_url 配置和 MQ_topic 配置；

- 在没有明确指定`${spring.cloud.nacos.config.group}`配置的情况下，默认使用的是 **DEFAULT_GROUP**。如果需要使用自定义的 group，可以通过以下配置来实现：

  ```properties
  spring.cloud.nacos.config.group=DEVELOP_GROUP
  ```

  > [!ATTENTION]
  >
  > 该配置必须放在 bootstrap.properties 或者 **bootstrap.yml**  配置文件中。并且在添加配置时 group 的值一定要和配置文件中`spring.cloud.nacos.config.group`的配置值一致！

### 支持自定义扩展的 Data Id 配置

`spring-cloud-starter-alibaba-nacos-config` 从 0.2.1 版本之后，可支持自定义 dataId 的配置。关于这部分详细的设计可参考<span style="background-color: rgb(251, 228, 231);">**<u>共享配置</u>**</span>[[discuss]: nacos config support sharing configuration with multi Applications · Issue #141 · alibaba/spring-cloud-alibaba](https://github.com/alibaba/spring-cloud-alibaba/issues/141)

一个完整的配置案例如下所示：

```properties
spring.application.name=opensource-service-provider
spring.cloud.nacos.config.server-addr=127.0.0.1:8848

# config external configuration
# 1、Data Id 在默认的组 DEFAULT_GROUP,不支持配置的动态刷新
spring.cloud.nacos.config.extension-configs[0].data-id=ext-config-common01.properties

# 2、Data Id 不在默认的组，不支持动态刷新
spring.cloud.nacos.config.extension-configs[1].data-id=ext-config-common02.properties
spring.cloud.nacos.config.extension-configs[1].group=GLOBALE_GROUP

# 3、Data Id 既不在默认的组，但支持动态刷新
spring.cloud.nacos.config.extension-configs[2].data-id=ext-config-common03.properties
spring.cloud.nacos.config.extension-configs[2].group=REFRESH_GROUP
spring.cloud.nacos.config.extension-configs[2].refresh=true
```
可以看到：

- 通过`spring.cloud.nacos.config.extension-configs[n].data-id`的配置方式来支持多个 dataId 的配置；
- 通过`spring.cloud.nacos.config.extension-configs[n].group`的配置方式自定义 dataId 所在的组，不明确配置的话，默认是 DEFAULT_GROUP；
- 通过`spring.cloud.nacos.config.extension-configs[n].refresh`的配置方式来控制该 dataId 在配置变更时，是否支持应用中可动态刷新，感知到最新的配置值。**默认是不支持的**，即不会动态更新；

🎨当多个 dataId 同时配置时，它的优先级是`spring.cloud.nacos.config.extension-configs[n].data-id`其中 **<u>n 的值越大，优先级越高</u>**

> [!ATTENTION]
>
> `spring.cloud.nacos.config.extension-configs[n].data-id`的值**必须带文件扩展名**，文件扩展名即可支持 properties，又可以支持 yaml/yml。此时`spring.cloud.nacos.config.file-extension`的配置对自定义扩展配置的 dataId 文件扩展名没有影响。

🎯优点：通过自定义扩展的 dataId 配置，<span style="background-color: rgb(251, 228, 231);"><u>既可以解决多个应用间配置共享的问题</u></span>，<span style="background-color: rgb(251, 228, 231);"><u>又可以支持一个应用有多个配置文件</u></span>。

为了更加清晰的在多个应用间配置共享的 dataId，可以通过以下方式来配置：

```properties
# 配置支持共享的 Data Id
spring.cloud.nacos.config.shared-configs[0].data-id=common.yaml

# 配置 Data Id 所在分组，缺省默认 DEFAULT_GROUP
spring.cloud.nacos.config.shared-configs[0].group=GROUP_APP1

# 配置Data Id 在配置变更时，是否动态刷新，缺省默认 false
spring.cloud.nacos.config.shared-configs[0].refresh=true
```
可以看到：

- 通过`spring.cloud.nacos.config.shared-configs[n].data-id`来支持多个共享 dataId 的配置；
- 通过`spring.cloud.nacos.config.shared-configs[n].group`来配置自定义 dataId 所在的组，不明确配置的话，默认是 DEFAULT_GROUP；
- 通过`spring.cloud.nacos.config.shared-configs[n].refresh`来控制该 dataId 在配置变更时，是否支持应用中可动态刷新，默认 `false`；

### 配置的优先级

`spring-cloud-starter-alibaba-nacos-config` 目前提供了三种配置能力从 Nacos 拉取相关的配置。

- A：通过`spring.cloud.nacos.config.shared-configs[n].data-id`来支持多个共享 dataId 的配置；
- B：通过`spring.cloud.nacos.config.extension-configs[n].data-id`的方式支持多个扩展 dataId 的配置；
- C：通过内部相关规则（应用名、应用名 + profile）自动生成相关的 dataId 配置；

当三种方式共同使用时，它们的优先级关系是：A < B < C。<br />![nacos配置优先级](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310210247585.svg)

### Nacos Config 对外暴露的 Endpoint

Nacos Config 内部提供了一个 Endpoint，EndPoint 的访问地址为 [http://localhost:8083/actuator/nacosconfig](http://localhost:8083/actuator/nacosconfig)<br />Endpoint 暴露的 json 中包含了三种属性：

1. Sources：当前应用配置的数据信息
2. RefreshHistory：配置刷新的历史记录
3. NacosDiscoveryProperties：当前应用 Nacos 的基础配置信息

以下为 Endpoint 暴露的 json 示例：
```json
{
    "NacosConfigProperties": {
        "serverAddr": "localhost:8848",
        "username": "",
        "password": "",
        "encode": null,
        "group": "DEFAULT_GROUP",
        "prefix": null,
        "fileExtension": "yaml",
        "timeout": 3000,
        "maxRetry": null,
        "configLongPollTimeout": null,
        "configRetryTime": null,
        "enableRemoteSyncConfig": false,
        "endpoint": null,
        "namespace": null,
        "accessKey": null,
        "secretKey": null,
        "ramRoleName": null,
        "contextPath": null,
        "clusterName": null,
        "name": null,
        "sharedConfigs": null,
        "extensionConfigs": null,
        "refreshEnabled": true,
        "refreshableDataids": null,
        "configServiceProperties": {
            "encode": "",
            "secretKey": "",
            "configLongPollTimeout": "",
            "maxRetry": "",
            "password": "",
            "configRetryTime": "",
            "endpoint": "",
            "serverAddr": "localhost:8848",
            "accessKey": "",
            "enableRemoteSyncConfig": "false",
            "fileExtension": "yaml",
            "clusterName": "",
            "namespace": "",
            "ramRoleName": "",
            "username": ""
        },
        "sharedDataids": null,
        "extConfig": null
    },
    "RefreshHistory": [],
    "Sources": [
        {
            "lastSynced": "2023-07-13 22:37:44",
            "dataId": "nacos-config"
        },
        {
            "lastSynced": "2023-07-13 22:37:44",
            "dataId": "nacos-config.yaml"
        },
        {
            "lastSynced": "2023-07-13 22:37:44",
            "dataId": "nacos-config-product.yaml"
        }
    ]
}
```
### 完全关闭 Nacos Config 的自动化配置

可以通过设置`spring.cloud.nacos.config.enabled=false`来完全关闭 Spring Cloud Nacos Config.
### 总结
>[!IMPORTANT]
>
>Nacos Config **通过领域模型（ namespace + group + dataId） 来<u>唯一</u>确定一条配置！**

- namespace：命名空间，用于进行租户粒度的配置隔离；
   - **在不同的命名空间 （namespace）下，可以存在相同的 group 或 dataId 的配置**；①
   
   - namespace 的常用场景之一是不同环境的配置的区分隔离，例如开发环境（dev）、测试环境（test）和生产环境（prod）的资源（如配置、服务）隔离等；
   
   - 在没有明确指定`${spring.cloud.nacos.config.namespace}`配置的情况下，默认使用的是 Nacos 上 public 这个 namespace。如果需要使用自定义的命名空间，可以通过以下配置来实现：
   
     ```properties
     spring.cloud.nacos.config.namespace=1330e6ed-dcc3-4fe4-9b36-b3d1b2f3c16b
     ```
   
     >[!ATTENTION]
     >
     >该配置必须放在 bootstrap.properties 或者 **bootstrap.yml** 配置文件中。此时`spring.cloud.nacos.config.namespace`的值是 namespace 对应的 id，该 id 值可以在 Nacos 的控制台获取。并且在添加配置时注意不要选择其他的 namespace，否则将会导致读取不到正确的配置。
   
- group：配置分组，Nacos 中的一组配置集，是组织配置的维度之一；

  - **通过一个有意义的字符串（如 Buy 或 Trade）对配置集进行分组，从而区分 dataId 相同的配置集**；②

  - group 的常见场景：不同的应用或组件使用了相同的配置类型，如 database_url 配置和 MQ_topic 配置；

  - 在没有明确指定`${spring.cloud.nacos.config.group}`配置的情况下，默认使用的是 DEFAULT_GROUP。如果需要使用自定义的 group，可以通过以下配置来实现：

     ```properties
     spring.cloud.nacos.config.group=DEVELOP_GROUP
     ```

     >[!ATTENTION]
     >
     >该配置必须放在 bootstrap.properties 或者 **bootstrap.yml** 配置文件中。并且在添加配置时 group 的值一定要和`spring.cloud.nacos.config.group`的配置值一致。

- dataId：配置集 ID，Nacos 中某个配置集的 ID，是组织划分配置的维度之一；一个系统或者应用可以包含多个配置集，每个配置集都可以被一个有意义的名称标识。其完整的拼接格式如下所示：

  ```
  ${prefix}-${spring.profiles.active}.${file-extension}
  ```

  - prefix：默认为`${spring.application.name}`的值，不过可以通过配置项`spring.cloud.nacos.config.prefix`来配置；
  - spring.profiles.active：当前激活的环境；
    - ❗注意：当该配置值为空时，dataId 变为`${prefix}.${file-extension}`的拼接格式；
    - ❗注意：`spring.profiles.active`当通过配置文件来指定时必须放在 bootstrap.properties 或者 <u>bootstrap.yml</u> 配置文件中；在真正的项目实施过程中这个变量的值是需要根据不同环境而有不同的值，这个时候通常的做法是通过`-Dspring.profiles.active=<profile>`参数指定其配置来达到环境间灵活的切换；
    - Nacos Config 在加载配置时，不仅仅加载 dataId 为`${prefix}.${file-extension}`为前缀的基础配置，还会加载 dataId 为`${prefix}-${spring.profiles.active}.${file-extension}`的配置；

  - file-extension：配置内容的数据格式；

    - 目前只支持 properties 和 yaml 两种格式；

    - 默认为 properties 格式，不过可以通过`spring.cloud.nacos.config.file-extension`来配置；
## 参考资料🎁

- [Spring Cloud Alibaba 参考文档 (spring-cloud-alibaba-group.github.io)](https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/zh-cn/index.html)
- [Nacos](https://nacos.io/zh-cn/)
- [Nacos discovery](https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-discovery) & [Nacos config](https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config)

Nacos 配置管理，其中一个立身之本就是为敏感配置保驾护航。它提供上述场景所需的功能，通过命名空间区分不同环境（开发、测试、预发、生产），通过“版本控制”保证变更可追溯，通过“快速回滚”保证错误变更时影响最小，通过的“灰度发布”功能保障配置安全平稳地变更，还有更多更全面功能（权限管控、变更审计等）即将支持。
