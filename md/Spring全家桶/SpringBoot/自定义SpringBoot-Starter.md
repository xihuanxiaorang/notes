# 自定义 SpringBoot Starter

## 何为 SpringBoot Starter？

SpringBoot Starter 是一个提供了 " 一站式服务（one-stop）" 的依赖 Jar 包。它主要有如下两个优点：

- 提供了自动配置的功能，遵从 " 约定大于配置 " 的原则，开发人员可以在少量配置甚至不配置的情况下 " 开箱即用 " 某个组件；
- 提供了良好的依赖管理，当需要某个组件时，只需要引入相关的 Starter 即可，不需要再手动引入各种依赖 Jar 包，从而避免了包遗漏、版本冲突等不必要的问题；

一个标准的 Starter，通常由两个模块构成，分别是 <u>starter</u> 模块和 <u>autoconfigure</u> 模块。

- starter 模块：
  - 通常是一个空的 Jar 包，它的主要作用是对依赖进行集中管理，构建了一个统一的依赖视图，方便用户使用
  - 它会依赖 autoconfigure 模块以及第三方组件的 Jar 包；
- autoconfigure 模块：
  - 自动配置的核心模块，其中包括了自动配置类、`META-INF/spring.factories` 配置文件以及所需的配置项，配置项通常是通过 `@ConfigurationProperties` 注解从 SpringBoot 应用的配置文件 `application.yml` 中读取对应的配置值，然后再通过 `@EnableConfigurationProperties` 注解将其注入到 IoC 容器中
  - 它会依赖必要的 SpringBoot 核心 Starter 和可选依赖于第三方组件的 Jar 包，从而使得该模块更容易扩展，增加更多的可选功能

Starter 结构图如下所示：<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261352849.jpeg" /><br />以官方的 spring-boot-starter-data-redis 举例：<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261352701.jpeg)<br />如上图所示：Redis 的自动配置类有 3 个，分别为 RedisAutoConfiguration、RedisReactiveAutoConfiguration 和 RedisRepositoriesAutoConfiguration，而且在配置文件 spring.factories 中也有对应的配置内容。最后，在 Redis 的自动配置模块中也定义了与 Redis 相关的配置项，咱们可以在 SpringBoot 应用程序中的配置文件 application.yml 配置文件中来配置 Redis 服务器的连接信息。<br />可选依赖（Optional) 在 Starter 中很重要，咱们花一点时间来了解一下，用以下两个例子来说明一下可选依赖的作用是什么。

- 案例一：假设有一个项目 A 可选依赖项目 B，那么在使用 Maven 编译项目 A 时，会将项目 B 添加到项目 A 的 classpath 中。此时，可选依赖和普通依赖表现是一致的。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261353623.jpeg" style="zoom: 67%;" />
- 案例二：假设有一个项目 X，它依赖项目 A， 而项目 A 又可选依赖项目 B，那么在使用 Maven 编译项目 X 时，项目 B 是不会被添加到项目 X 的 classpath 中的，除非，项目 X 直接声明依赖项目 B；<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261353210.jpeg" style="zoom: 50%;" /><br />总的来说，可选依赖的作用就是<strong style="font-size:19px;background-color: rgb(251, 228, 231);">阻断依赖传递</strong>。

## 如何自定义一个 SpringBoot Starter？

根据官方文档 [SpringBoot-Creating Your Own Starter](https://docs.spring.io/spring-boot/docs/3.0.6/reference/htmlsingle/#features.developing-auto-configuration.custom-starter) 所述，实现步骤如下图所示：<br />命名规范：官方提供的 Starter 以 spring-boot-starter-xxx 的形式命名，为了与官方提供的 Starter 进行区分，官方建议第三方开发者或技术厂商自定义的 Starter 以 xxx-spring-boot-starter 的形式命名，如 [mybatis-spring-boot-starter](https://github.com/mybatis/spring-boot-starter/tree/master/mybatis-spring-boot-starter)、[druid-spring-boot-starter](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)、[dynamic-datasource-spring-boot-starter](https://github.com/baomidou/dynamic-datasource-spring-boot-starter) 等等；<br />模块规范：官方建议在自定义 Starter 时，需要创建两个模块：autoconfigure 和 starter 模块，其中，starter 模块会依赖 autoconfigure 模块。当然，这只是官方的建议，并不是硬性规定，如果自动配置相对简单并且没有可选功能的话，可以将这两个模块合并为一个模块 xxx-spring-boot-starter。

```plantuml
@startmindmap
<style>
mindmapDiagram {
  .green {
    BackgroundColor lightgreen
  }
  .rose {
    BackgroundColor #FFBBCC
  }
  .blue {
    BackgroundColor lightblue
  }
}
</style>
* 自定义 SpringBoot Starter <<green>>
	* 1. 首先在 IDEA 中创建一个空项目，名称随意，如 my-spring-boot-starters <<rose>>
	* 2. 创建父模块，名称为 xxx-spring-boot，该模块主要用来进行依赖管理 <<rose>>
	* 3. 创建 autoconfigure 模块，名称为 xxx-spring-boot-autoconfigure <<rose>>
		* 1. 在 pom.xml 文件中添加 spring-boot-autoconfigure 基础依赖和其他第三方组件依赖（可选） <<blue>>
		* 2. 编写属性配置类，xxxProperties，在类上标注 @ConfigurationProperties(prefix = "xxx") <<blue>>
		* 3. 编写自动配置类，xxxAutoConfiguration，在类上标注 @Configuration 和 @EnableConfigurationProperties(xxxProperties.class) <<blue>>
		* 4. 在 resources 资源目录下增加 META-INF/spring.factories 配置文件，文件内容为 org.springframework.boot.autoconfigure.EnableAutoConfiguration = xxxAutoConfiguration 自动配置类的全限定名 <<blue>>
		* 5. mvn install，安装到本地仓库 <<blue>>
	* 4. 创建 starter 模块，名称为 xxx-spring-boot-starter <<rose>>
		* 1. 一个空的项目，无任何代码实现， 在 pom.xml 文件中 添加 spring-boot-starter、xxx-spring-boot-autoconfigure 和 第三方组件依赖即可 <<blue>>
		* 2. mvn install，安装到本地仓库，在其他项目的 pom.xml 中添加该 xxx-spring-boot-starter 依赖即可使用 <<blue>>
@endmindmap
```

现在从 0 开始，手撸一个基于 Amazon S3 协议可适配市面上大部分对象存储服务（OSS - Object Storage Service）的 SpringBoot Starter。

## 实现一个基于 Amazon S3 协议的对象存储服务的 SpringBoot Starter

- 什么是 Amazon S3 协议？

  Amazon Simple Storage Service（Amazon S3）是一种对象存储服务，提供行业领先的可扩展性、数据可用性、安全性和性能。各种规模和行业的客户都可以使用 Amazon S3 存储和保护任意数量的数据，用于数据湖、网站、移动应用程序、备份和恢复、归档、企业应用程序、IoT 设备和大数据分析。Amazon S3 提供了管理功能，使您可以优化、组织和配置对数据的访问，以满足您的特定业务、组织和合规性要求。

- 为什么要基于 Amazon S3 协议进行开发？

  由于目前市面上大部分 OSS 对象存储服务都兼容 Amazon S3 协议，如阿里云 OSS，腾讯 COS、七牛云 OSS、Minio 等等；正因如此，为了使自己封装的 Starter 可适配、可扩展、易于迁移，所以才使用 AmazonS3 协议进行开发，即使是将来更换产品，也不需要调整代码，仅修改配置文件即可！

### oss-spring-boot 父模块

创建一个空项目，名称随意，如 `my-spring-boot-starters` <br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261356517.png" alt="image.png" />

创建父模块，名称为 `oss-spring-boot`，该模块主要用来进行<u>**依赖管理**</u> <br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261356985.png)<br />其 pom.xml 配置文件如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>fun.xiaorang</groupId>
    <artifactId>oss-spring-boot</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>oss-spring-boot-autoconfigure</module>
        <module>oss-spring-boot-starter</module>
        <module>oss-spring-boot-starter-test</module>
    </modules>

    <properties>
        <maven.compiler.source>15</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <spring-boot.version>2.6.7</spring-boot.version>
        <aws.s3.version>1.12.466</aws.s3.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.amazonaws</groupId>
                <artifactId>aws-java-sdk-s3</artifactId>
                <version>${aws.s3.version}</version>
            </dependency>
            <dependency>
                <groupId>fun.xiaorang</groupId>
                <artifactId>oss-spring-boot-autoconfigure</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>fun.xiaorang</groupId>
                <artifactId>oss-spring-boot-starter</artifactId>
                <version>${project.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### oss-spring-boot-autoconfigure 自动配置模块

创建 autoconfigure 模块，名称为 `oss-spring-boot-autoconfigure` <br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261357282.png)

在 `pom.xml` 文件中添加如下依赖，其中包括必要的编译依赖、亚马逊对象存储服务 SDK 和 SpringBoot 注解处理器等依赖；

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>fun.xiaorang</groupId>
        <artifactId>oss-spring-boot</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>oss-spring-boot-autoconfigure</artifactId>

    <dependencies>
        <!-- 编译依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <!-- 可选的、但是强烈推荐的注解处理器 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- 日志 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- 亚马逊对象存储服务SDK -->
        <dependency>
            <groupId>com.amazonaws</groupId>
            <artifactId>aws-java-sdk-s3</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

#### 两个注解处理器

强烈推荐在 pom.xml 文件中添加以下这两个注解处理器，所谓的注解处理器是 SpringBoot 提供的功能，它们的作用是在编译阶段通过分析特定的注解从而生成相关的元数据。

`spring-boot-configuration-processor`：它的作用是分析 `@ConfigurationProperties` 注解生成配置项的描述信息并存放在配置文件 `META-INF/spring-configuration-metadata.json` 中，从而使得 IDEA 可以进行自动提示，方便开发人员的使用，如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261358661.png)<br />配置文件 `META-INF/spring-configuration-metadata.json` 中的内容如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261358632.png)

`spring-boot-autoconfigure-processor`：则是在编译阶段分析注解 `@EnableAutoConfiguration` 生成所有自动配置类的过滤条件的元数据，然后存放在配置文件 `META-INF/spring-autoconfigure-metadata.properties` 中，从而使得 SpringBoot 在启动时能快速的过滤掉不符合条件的自动配置类，加快 SpringBoot 应用的启动速度

配置文件 `META-INF/spring-autoconfigure-metadata.properties` 中的内容如下所示：<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261359126.png" alt="bbe8074d-9946-4989-bf68-1bd48ad13f62" style="zoom:80%;" />

#### 操作模板类

编写 `OssTemplate` 操作模板类，如果以后想对文件操作加入更多的功能，只需修改该类即可！以下内容参考自官方文档 [Amazon Simple Storage Service](https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide) [使用 AWS SDK for Java 的 Amazon S3 示例 - AWS SDK for Java1.x](https://docs.aws.amazon.com/zh_cn/sdk-for-java/v1/developer-guide/examples-s3.html)

```java
/**
 * @author liulei
 * @description <p style = " font-weight:bold ; ">OSS操作接口，参考自官方文档：<a href="https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide">用户指南</a><p/>
 * @github <a href="https://github.com/xihuanxiaorang/my-spring-boot-starters">my-spring-boot-starters</a>
 * @Copyright 博客：<a href="https://blog.xiaorang.fun">小让的糖果屋</a>  - show me the code
 * @date 2023/5/24 3:28
 */
public interface OssOperations {
    /**
     * 根据指定的桶名称创建一个桶
     *
     * @param bucketName 桶名称
     * @return 桶
     */
    Bucket createBucket(String bucketName);

    /**
     * 根据指定的桶名称获取桶
     *
     * @param bucketName 桶名称
     * @return 桶
     */
    Bucket getBucket(String bucketName);

    /**
     * 获取所有的桶
     *
     * @return 已创建的桶集合
     */
    List<Bucket> listBuckets();

    /**
     * 根据指定的桶名称删除桶
     * 参考自 <a href="https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/delete-bucket.html">删除桶</a>
     *
     * @param bucketName 桶名称
     */
    void deleteBucket(String bucketName);

    /**
     * 上传文件到指定的桶中
     *
     * @param bucketName 桶名称
     * @param file       要上传的文件
     */
    void putObject(String bucketName, File file);

    /**
     * 上传文件到指定的桶中
     * 参考自 <a href="https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/upload-objects.html">上传对象</a>
     *
     * @param bucketName  桶名称
     * @param fileName    文件名称
     * @param inputStream 要上传文件的输入流
     * @param contextType 文件类型
     */
    void putObject(String bucketName, String fileName, InputStream inputStream, String contextType);

    /**
     * 上传文件到指定的桶中
     *
     * @param bucketName  桶名称
     * @param fileName    文件名称
     * @param inputStream 要上传文件的输入流
     */
    default void putObject(String bucketName, String fileName, InputStream inputStream) {
        putObject(bucketName, fileName, inputStream, "application/octet-stream");
    }

    /**
     * 从指定的桶中删除文件
     *
     * @param bucketName 桶名称
     * @param fileName   文件名称
     */
    void deleteFile(String bucketName, String fileName);

    /**
     * 生成文件预览地址
     * 参考自 <a href="https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html">使用预签名URL共享对象</a>
     *
     * @param bucketName    桶名称
     * @param fileName      文件名称
     * @param expTimeMillis 过期时间（单位：毫秒）
     * @return 文件预览地址URL
     */
    String generatePresignedUrl(String bucketName, String fileName, long expTimeMillis);

    /**
     * 生成生成文件预览地址，默认一小时后失效
     *
     * @param bucketName 桶名称
     * @param fileName   文件名称
     * @return 文件预览地址URL
     */
    default String generatePresignedUrl(String bucketName, String fileName) {
        long expTimeMillis = Instant.now().toEpochMilli();
        expTimeMillis += 1000 * 60 * 60;
        return generatePresignedUrl(bucketName, fileName, expTimeMillis);
    }


    /**
     * 得到对象
     *
     * @param bucketName 桶名称
     * @param fileName   文件名称
     * @return {@link S3Object}
     */
    S3Object getObject(String bucketName, String fileName);
}
```

```java
@RequiredArgsConstructor
public class OssTemplate implements OssOperations {
    private static final Logger LOGGER = LoggerFactory.getLogger(OssTemplate.class);
    private final AmazonS3 ossClient;

    @Override
    public Bucket createBucket(String bucketName) {
        if (ossClient.doesBucketExistV2(bucketName)) {
            LOGGER.info("Bucket {} already exists.", bucketName);
            return getBucket(bucketName);
        }
        return ossClient.createBucket(bucketName);
    }

    @Override
    public Bucket getBucket(String bucketName) {
        return listBuckets().stream()
        .filter(bucket -> bucket.getName().equals(bucketName))
        .findFirst()
        .orElse(null);
    }

    @Override
    public List<Bucket> listBuckets() {
        return ossClient.listBuckets();
    }

    @Override
    public void deleteBucket(String bucketName) {
        // Delete all objects from the bucket. This is sufficient
        // for unversioned buckets. For versioned buckets, when you attempt to delete objects, Amazon S3 inserts
        // delete markers for all objects, but doesn't delete the object versions.
        // To delete objects from versioned buckets, delete all of the object versions before deleting
        // the bucket (see below for an example).
        ObjectListing objectListing = ossClient.listObjects(bucketName);
        while (true) {
            for (S3ObjectSummary s3ObjectSummary : objectListing.getObjectSummaries()) {
                ossClient.deleteObject(bucketName, s3ObjectSummary.getKey());
            }

            // If the bucket contains many objects, the listObjects() call
            // might not return all of the objects in the first listing. Check to
            // see whether the listing was truncated. If so, retrieve the next page of objects
            // and delete them.
            if (objectListing.isTruncated()) {
                objectListing = ossClient.listNextBatchOfObjects(objectListing);
            } else {
                break;
            }
        }

        // Delete all object versions (required for versioned buckets).
        VersionListing versionList = ossClient.listVersions(new ListVersionsRequest().withBucketName(bucketName));
        while (true) {
            for (S3VersionSummary vs : versionList.getVersionSummaries()) {
                ossClient.deleteVersion(bucketName, vs.getKey(), vs.getVersionId());
            }

            if (versionList.isTruncated()) {
                versionList = ossClient.listNextBatchOfVersions(versionList);
            } else {
                break;
            }
        }

        // After all objects and object versions are deleted, delete the bucket.
        ossClient.deleteBucket(bucketName);
    }

    @Override
    public void putObject(String bucketName, File file) {
        String fileName = file.getName();
        LOGGER.info("uploading {} file to {} bucket...", fileName, bucketName);
        ossClient.putObject(bucketName, fileName, file);
        LOGGER.info("Done!");
    }

    @Override
    public void putObject(String bucketName, String fileName, InputStream inputStream, String contextType) {
        LOGGER.info("uploading {} file to {} bucket...", fileName, bucketName);
        ObjectMetadata objectMetadata = new ObjectMetadata();
        objectMetadata.setContentType(contextType);
        PutObjectRequest putObjectRequest = new PutObjectRequest(bucketName, fileName, inputStream, objectMetadata);
        ossClient.putObject(putObjectRequest);
        LOGGER.info("Done!");
    }

    @Override
    public void deleteFile(String bucketName, String fileName) {
        LOGGER.info("deleting {} file from {} bucket...", fileName, bucketName);
        ossClient.deleteObject(bucketName, fileName);
        LOGGER.info("Done!");
    }

    @Override
    public String generatePresignedUrl(String bucketName, String fileName, long expTimeMillis) {
        Date expiration = new Date(expTimeMillis);
        GeneratePresignedUrlRequest generatePresignedUrlRequest =
        new GeneratePresignedUrlRequest(bucketName, fileName)
        .withMethod(HttpMethod.GET)
        .withExpiration(expiration);
        return ossClient.generatePresignedUrl(generatePresignedUrlRequest).toString();
    }

    @Override
    public S3Object getObject(String bucketName, String fileName) {
        return ossClient.getObject(bucketName, fileName);
    }
}
```

#### 属性配置类

```java
@Data
@ConfigurationProperties(prefix = "oss")
public class OssProperties {
    private String accessKey;
    private String accessSecret;
    /**
     * endpoint 配置格式为
     * 通过外网访问OSS服务时，以URL的形式表示访问的OSS资源，详情请参见OSS访问域名使用规则。OSS的URL结构为[$Schema]://[$Bucket].[$Endpoint]/[$Object]
     * 。例如，您的Region为华东1（杭州），Bucket名称为 examplebucket，Object访问路径为destfolder/example.txt，
     * 则外网访问地址为https://examplebucket.oss-cn-hangzhou.aliyuncs.com/destfolder/example.txt
     * <a href="https://help.aliyun.com/document_detail/375241.html">...</a>
     */
    private String endpoint;
    /**
     * refer com.amazonaws.regions.Regions;
     * 阿里云 region 对应表
     * <a href="https://help.aliyun.com/document_detail/31837.htm?spm=a2c4g.11186623.0.0.695178eb0nD6jp">...</a>
     */
    private String region;
    private boolean pathStyleAccess = true;
}
```

#### 自动配置类

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(OssProperties.class)
@ConditionalOnClass(AmazonS3.class)
public class OssAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean
    public AmazonS3 ossClient(OssProperties ossProperties) {
        AWSCredentialsProvider awsCredentialsProvider = new AWSStaticCredentialsProvider(new BasicAWSCredentials(ossProperties.getAccessKey(),
                ossProperties.getAccessSecret()));
        return AmazonS3Client.builder()
                .withEndpointConfiguration(new AwsClientBuilder.EndpointConfiguration(ossProperties.getEndpoint(), ossProperties.getRegion()))
                .withCredentials(awsCredentialsProvider)
                .disableChunkedEncoding()
                .withPathStyleAccessEnabled(ossProperties.isPathStyleAccess())
                .build();
    }

    @Bean
    @ConditionalOnBean(AmazonS3.class)
    public OssTemplate ossTemplate(AmazonS3 amazonS3) {
        return new OssTemplate(amazonS3);
    }
}
```

#### 配置文件 (spring.factories)

在 `resources` 资源目录下增加 `META-INF/spring.factories` 配置文件

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
fun.xiaorang.oss.OssAutoConfiguration
```

### oss-spring-boot-starter 模块

创建 starter 模块，名称为 `oss-spring-boot-starter`；<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261401220.png)

在 `pom.xml` 文件中添加如下依赖，其中包括 `spring-boot-starter` 基础依赖、`oss-spring-boot-autoconfigure` 自动配置模块以及第三方组件 jar 包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>fun.xiaorang</groupId>
        <artifactId>oss-spring-boot</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>oss-spring-boot-starter</artifactId>

    <dependencies>
        <!-- 必要的依赖，SpringBoot核心Starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- 自动配置模块 -->
        <dependency>
            <groupId>fun.xiaorang</groupId>
            <artifactId>oss-spring-boot-autoconfigure</artifactId>
        </dependency>
        <!-- 亚马逊对象存储服务SDK -->
        <dependency>
            <groupId>com.amazonaws</groupId>
            <artifactId>aws-java-sdk-s3</artifactId>
        </dependency>
    </dependencies>

</project>
```

执行 `mvn install` 命令，将 `oss-spring-boot-starter` 模块安装到本地仓库，或者使用 IDEA 中的操作按钮进行安装，如下所示：<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261402669.png)<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261402770.png" alt="image.png" style="zoom:80%;" />

### oss-spring-boot-starter-test 测试模块

创建一个测试模式，名称为 `oss-spring-boot-starter-test` <br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261402142.png)

#### 依赖

在 `pom.xml` 文件中添加如下依赖，其中包括 `oss-spring-boot-starter` 和 `spring-boot-starter-web` 依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>fun.xiaorang</groupId>
        <artifactId>oss-spring-boot</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>oss-spring-boot-starter-test</artifactId>

    <dependencies>
        <dependency>
            <groupId>fun.xiaorang</groupId>
            <artifactId>oss-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>

</project>
```

#### 配置文件 (application.yml)

```yaml
oss:
  access-key: tackIrdbOzfpltFVHpCu
  access-secret: DCErOHD4EVUMucnqCnQQSg5hqAUwVpsk7vlZtHc9
  endpoint: http://120.78.177.161:9000

spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 100MB
```

#### 控制器

```java
@RequiredArgsConstructor
@RestController
@RequestMapping("/oss")
public class OssController {
    private final OssTemplate ossTemplate;

    /**
     * 根据指定的桶名称创建一个桶
     *
     * @param bucketName 桶名称
     */
    @PostMapping("/bucket/{bucketName}")
    public void createBucket(@PathVariable String bucketName) {
        ossTemplate.createBucket(bucketName);
    }

    /**
     * 根据指定的桶名称创建一个桶
     *
     * @param bucketName 桶名称
     */
    @PostMapping("/object/{bucketName}")
    public void upload(@PathVariable String bucketName, MultipartFile file) throws IOException {
        ossTemplate.putObject(bucketName, file.getOriginalFilename(), file.getInputStream());
    }
}
```

#### 启动类

```java
@SpringBootApplication
public class SampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(SampleApplication.class, args);
    }
}
```

#### 测试

启动 SpringBoot 应用，开始测试

1. 使用 ApiFox 发送 POST 请求 http://localhost:8080/oss/bucket/{bucketName} ，请求参数为 "test"，测试能否成功创建一个名称为 "test" 的文件夹目录 <br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261404943.png)<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261404427.png)
2. 使用 ApiFox 发送 POST 请求 http://localhost:8080/oss/object/{bucketName} ，请求参数为 "test" 和文件，测试能否成功将文件上传到刚创建的文件目录中 <br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261404849.png)<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307261404861.png)

至此，一个基于 Amazon S3 协议可适配市面上大部分对象存储服务（OSS - Object Storage Service）的 SpringBoot Starter 就圆满成功啦！🍺🍺🍺
