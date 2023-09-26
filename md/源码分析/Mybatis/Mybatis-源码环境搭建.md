# Mybatis-源码环境搭建

## 源码下载 & 编译

在 Github 中找到 [mybatis](https://github.com/mybatis/mybatis-3) 项目，找到最新版本 [mybatis-3.5.13](https://github.com/mybatis/mybatis-3/releases/tag/mybatis-3.5.13) ，下载对应的源码压缩包并进行解压！<br />![image-20230920115510735](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201155857.png)

查看 mybatis 项目中的 `pom.xml` 配置文件，发现其父模块为 `mybatis-parent` 项目，如下所示：<br />![image-20230920120201235](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201202281.png)

所以还需要下载与其同源的 [mybatis-parent](https://github.com/mybatis/parent) 项目，找到 [mybatis-parent-37](https://github.com/mybatis/parent/releases/tag/mybatis-parent-37) 版本，同样下载对应的源码压缩包并进行解压！<br />![image-20230920120702903](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201207970.png)

进入 mybatis-parent 项目根目录，打开终端，执行 `mvn clean install "-Dmaven.test.skip=true"` 命令，用于编译 mybatis-parent 项目。<br />![image-20230920121447644](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201214411.png)

再进入 mybatis 项目根目录，先修改 mybatis 项目的版本号为 `3.5.13-MY`（与官方的区分开来），如下所示：<br />![image-20230920122130741](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201221787.png)

然后打开终端，执行与上面相同的 `mvn clean install "-Dmaven.test.skip=true"` 命令，用于编译 mybatis 项目。<br />![image-20230920122803967](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201228735.png)

编译成功之后可以在 target 目录下发现一个 `mybatis-3.5.13-MY.jar` 文件，如下所示：<br />![image-20230920123151105](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201232802.png)

## 创建父模块

打开 IDEA，创建一个 `mybatis-study` 的父模块项目，<br />![image-20230920130736551](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201310098.png)

删除其 `src` 目录并将其打包方式为修改为 `pom` 方式，并将以上两个模块 `parent-mybatis-parent-37` 和 `mybatis-3-mybatis-3.5.13` 作为 `mybatis-study` 模块的子模块，即在 `mybatis-study` 模块的 `pom.xml` 配置文件中的`modules` 标签中添加对应的 `module` 子标签，如下所示：<br />![image-20230920132532607](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201325708.png)

然后点击右上角的小图标，可以用于加载 Maven 更改。

## 测试

创建一个 `mybatis-source-test` 测试子模块，专门用于测试 mybatis 源码环境是否搭建成功。<br />![image-20230920153835837](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201539968.png)

### 引入依赖

首先在其父模块 `mybatis-study` 的 `pom.xml` 配置文件中声明项目依赖以及引入部分依赖。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>fun.xiaorang</groupId>
    <artifactId>mybatis-study</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>pom</packaging>

    <modules>
        <module>parent-mybatis-parent-37</module>
        <module>mybatis-3-mybatis-3.5.13</module>
        <module>mybatis-source-test</module>
    </modules>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <junit.version>5.9.1</junit.version>
        <slf4j.version>2.0.3</slf4j.version>
        <logback.version>1.4.4</logback.version>
        <lombok.version>1.18.24</lombok.version>
        <mysql.version>8.0.29</mysql.version>
        <druid.version>1.2.8</druid.version>
        <mybatis.version>3.5.13-MY</mybatis.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!-- junit5 -->
            <dependency>
                <groupId>org.junit.jupiter</groupId>
                <artifactId>junit-jupiter-api</artifactId>
                <version>${junit.version}</version>
                <scope>test</scope>
            </dependency>
            <dependency>
                <groupId>org.junit.jupiter</groupId>
                <artifactId>junit-jupiter-engine</artifactId>
                <version>${junit.version}</version>
                <scope>test</scope>
            </dependency>
            <!-- slf4j日志门面 -->
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-api</artifactId>
                <version>${slf4j.version}</version>
            </dependency>
            <!-- logback日志 -->
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-classic</artifactId>
                <version>${logback.version}</version>
            </dependency>
            <!-- lombok -->
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
            </dependency>
            <!-- MySQL驱动 -->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <!-- Druid数据库连接池 -->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>${druid.version}</version>
            </dependency>
            <!-- Mybatis持久层框架 -->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>${mybatis.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
</project>
```

然后在 `mybatis-source-test` 测试子模块的`pom.xml` 配置文件中引入所需的 MySQL 驱动、druid 连接池以及 `mybatis-3.5.13-MY` 版本的 mybatis 依赖。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>fun.xiaorang</groupId>
        <artifactId>mybatis-study</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>mybatis-source-test</artifactId>

    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
        </dependency>
    </dependencies>
</project>
```

### 编写测试案例

接下来的所有操作全部位于 `mybatis-source-test` 测试子模块中。

#### SQL脚本

执行如下所示 SQL，用于创建测试案例所需的 `mybatis-source-test` 数据库以及 `author` 和 `article` 表，为了方便测试，向表中添加少量测试数据。

```sql
CREATE TABLE IF NOT EXISTS `author`
(
    `id`    INT(50)      NOT NULL COMMENT '作者id' AUTO_INCREMENT,
    `name`  VARCHAR(30)  NOT NULL COMMENT '名字',
    `age`   INT(30)      NOT NULL COMMENT '年龄',
    `sex`   INT(30)      NOT NULL COMMENT '性别',
    `email` VARCHAR(255) NOT NULL COMMENT '邮箱',
    PRIMARY KEY (`id`)
) ENGINE = INNODB
  DEFAULT CHARSET = utf8;

CREATE TABLE IF NOT EXISTS `article`
(
    `id`          INT(50)      NOT NULL COMMENT '文章id' AUTO_INCREMENT,
    `author_id`   INT(50)      NOT NULL COMMENT '作者id',
    `title`       VARCHAR(100) NOT NULL COMMENT '标题',
    `type`        INT(30)      NOT NULL COMMENT '类型（1：JAVA，2：DUBBO，4：SPRING，8：MYBATIS）',
    `content`     VARCHAR(255) NOT NULL COMMENT '内容',
    `create_time` DATETIME     NOT NULL COMMENT '创建时间',
    PRIMARY KEY (`id`),
    FOREIGN KEY (`author_id`) REFERENCES `author` (`id`)
) ENGINE = INNODB
  DEFAULT CHARSET = utf8;

INSERT INTO `author`(`name`, `age`, `sex`, `email`)
VALUES ('coolblog.xyz', 28, 0, 'coolblog.xyz@outlook.com'),
       ('nullllun', 29, 1, 'coolblog.xyz@outlook.com');

INSERT INTO `article`(`author_id`, `title`, `type`, `content`, `create_time`)
VALUES (1, 'Mybatis源码分析系列文章导读', 8, 'Mybatis源码分析系列文章导读', '2018-07-15 15:30:09'),
       (2, 'HashMap源码详细分析（JDK1.8）', 1, 'HashMap源码详细分析（JDK1.8）', '2018-01-18 15:29:13'),
       (1, 'Java CAS 原理分析', 1, 'Java CAS 原理分析', '2018-05-15 15:28:33'),
       (1, 'Spring IOC 容器源码分析-获取单例Bean', 4, 'Spring IOC 容器源码分析-获取单例Bean', '2018-06-01 00:00:00'),
       (1, 'Spring IOC 容器源码分析-循环依赖的解决办法', 4, 'Spring IOC 容器源码分析-循环依赖的解决办法',
        '2018-06-08 00:00:00'),
       (2, 'Spring AOP 源码分析系列文章导读', 4, 'Spring AOP 源码分析系列文章导读', '2018-06-17 00:00:00'),
       (2, 'Spring AOP 源码分析-创建代理对象', 4, 'Spring AOP 源码分析-创建代理对象', '2018-06-20 00:00:00'),
       (1, 'Spring MVC 原理揭秘 - 一个请求的旅行过程', 4, 'Spring MVC 原理揭秘 - 一个请求的旅行过程',
        '2018-06-29 00:00:00'),
       (2, 'Spring MVC 原理揭秘 - 容器的创建过程', 4, 'Spring MVC 原理揭秘 - 容器的创建过程', '2018-06-30 00:00:00'),
       (2, 'Spring IOC 容器源码分析系列文章导读', 4, 'Spring IOC 容器源码分析系列文章导读', '2018-05-30 00:00:00');
```

#### 枚举类

```java
public enum SexEnum {
    /**
     * 男
     */
    MAN,
    /**
     * 女
     */
    FEMALE,
    /**
     * 未知
     */
    UNKNOWN
}
```

```java
@Getter
public enum ArticleTypeEnum {
    JAVA(1),
    DUBBO(2),
    SPRING(4),
    MYBATIS(8);

    private final int code;

    ArticleTypeEnum(int code) {
        this.code = code;
    }
    
    /**
     * 根据code获取枚举
     *
     * @param code code
     * @return 枚举
     */
    public static ArticleTypeEnum getEnumByCode(int code) {
        return Arrays.stream(ArticleTypeEnum.values())
                .filter(articleTypeEnum -> articleTypeEnum.code == code)
                .findFirst()
                .orElse(null);
    }
}
```

#### 实体类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Article {
    private Integer id;
    private String title;
    private ArticleTypeEnum type;
    private Author author;
    private String content;
    private Date createTime;
}
```

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Author {
    private Integer id;
    private String name;
    private Integer age;
    private SexEnum sex;
    private String email;
    private List<Article> articles;
}
```

#### 持久层接口

```java
public interface ArticleMapper {
    Article findOne(@Param("id") int id);
}
```

```java
public interface AuthorMapper {
    Author findOne(@Param("id") int id);
}
```

#### 配置类

为了使用`druid`作为数据源连接池，自定义数据源工厂类，继承自 `PooledDataSourceFactory`。

```java
public class MyDruidDataSourceFactory extends PooledDataSourceFactory {
    private DataSource dataSource;

    @Override
    public void setProperties(Properties properties) {
        try {
            this.dataSource = DruidDataSourceFactory.createDataSource(properties);
        } catch (Exception e) {
            throw new RuntimeException("init datasource error", e);
        }
    }

    @Override
    public DataSource getDataSource() {
        return this.dataSource;
    }
}
```

#### 类型处理器

自定义类型处理器用于处理 `ArticleTypeEnum` 枚举类型。注意：需要在 mybatis 核心配置文件中注册该类型处理器，否则不会生效！！！

```java
public class ArticleTypeHandler extends BaseTypeHandler<ArticleTypeEnum> {
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, ArticleTypeEnum parameter, JdbcType jdbcType) throws SQLException {
        ps.setInt(i, parameter.getCode());
    }

    @Override
    public ArticleTypeEnum getNullableResult(ResultSet rs, String columnName) throws SQLException {
        int code = rs.getInt(columnName);
        return ArticleTypeEnum.getEnumByCode(code);
    }

    @Override
    public ArticleTypeEnum getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        int code = rs.getInt(columnIndex);
        return ArticleTypeEnum.getEnumByCode(code);
    }

    @Override
    public ArticleTypeEnum getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        int code = cs.getInt(columnIndex);
        return ArticleTypeEnum.getEnumByCode(code);
    }
}
```

#### 配置文件

1. 数据库连接信息配置文件 `db.properties`

   ```properties
   url=jdbc:mysql://localhost:3306/mybatis-source-test?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai&useServerPrepStmts=true&cachePrepStmts=true&allowPublicKeyRetrieval=true&rewriteBatchedStatements=true  
   username=root
   password=root
   initialSize=10
   minIdle=20
   maxActive=50
   maxWait=500
   ```

2. mybatis 核心配置文件 `mybatis-config.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <properties resource="db.properties"/>
   
       <settings>
           <setting name="mapUnderscoreToCamelCase" value="true"/>
       </settings>
   
       <typeAliases>
           <typeAlias alias="ArticleTypeHandler" type="fun.xiaorang.mybatis.handler.ArticleTypeHandler"/>
           <package name="fun.xiaorang.mybatis.entity"/>
       </typeAliases>
   
       <typeHandlers>
           <typeHandler handler="fun.xiaorang.mybatis.handler.ArticleTypeHandler"/>
       </typeHandlers>
   
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <dataSource type="fun.xiaorang.mybatis.config.MyDruidDataSourceFactory">
                   <property name="url" value="${url}"/>
                   <property name="username" value="${username}"/>
                   <property name="password" value="${password}"/>
                   <property name="initialSize" value="${initialSize}"/>
                   <property name="minIdle" value="${minIdle}"/>
                   <property name="maxActive" value="${maxActive}"/>
                   <property name="maxWait" value="${maxWait}"/>
               </dataSource>
           </environment>
       </environments>
   
       <mappers>
           <package name="fun.xiaorang.mybatis.mapper"/>
       </mappers>
   </configuration>
   ```

3. 映射配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="fun.xiaorang.mybatis.mapper.ArticleMapper">
       <resultMap id="authorResult" type="fun.xiaorang.mybatis.entity.Author" autoMapping="true">
           <id property="id" column="author_id"/>
           <result property="sex" column="sex" typeHandler="org.apache.ibatis.type.EnumOrdinalTypeHandler"/>
       </resultMap>
   
       <resultMap id="articleResult" type="Article" autoMapping="true">
           <id property="id" column="id"/>
           <result property="type" column="type" typeHandler="ArticleTypeHandler"/>
           <association property="author" javaType="fun.xiaorang.mybatis.entity.Author" resultMap="authorResult"/>
       </resultMap>
   
       <select id="findOne" resultMap="articleResult">
           select ar.id,
                  ar.author_id,
                  ar.title,
                  ar.type,
                  ar.content,
                  ar.create_time,
                  au.name,
                  au.age,
                  au.sex,
                  au.email
           from `article` ar,
                `author` au
           where ar.author_id = au.id
             and ar.id = #{id}
       </select>
   </mapper>
   ```

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="fun.xiaorang.mybatis.mapper.AuthorMapper">
       <resultMap id="articleResult" type="Article" autoMapping="true">
           <id property="id" column="article_id"/>
           <result property="type" column="type" typeHandler="ArticleTypeHandler"/>
       </resultMap>
   
       <resultMap id="authorResult" type="fun.xiaorang.mybatis.entity.Author" autoMapping="true">
           <id property="id" column="id"/>
           <result property="sex" column="sex" typeHandler="org.apache.ibatis.type.EnumOrdinalTypeHandler"/>
           <collection property="articles" ofType="Article" resultMap="articleResult"/>
       </resultMap>
   
       <select id="findOne" resultMap="authorResult">
           select au.id,
                  au.name,
                  au.age,
                  au.sex,
                  au.email,
                  ar.id as article_id,
                  ar.title,
                  ar.type,
                  ar.content,
                  ar.create_time
           from `author` au,
                `article` ar
           where au.id = ar.author_id
             and au.id = #{id}
       </select>
   </mapper>
   ```

4. 日志配置文件 `logback.xml`

   ```xml
   <!--开启调试功能，配置文件将10秒钟扫描一次更改，当检测到配置文件发生改变时，该配置文件将会被重新加载-->
   <configuration debug="true" scan="true" scanPeriod="10 seconds">
       <!-- 定义变量 -->
       <!-- 定义root日志级别，优先级：TRACE < DEBUG < INFO < WARN < ERROR -->
       <property name="LOG_ROOT_LEVEL" value="DEBUG"/>
       <!-- 定义日志输出路径 -->
       <property name="LOG_HOME" value="logs"/>
       <!-- 定义日志输出格式，其中 %d：表示日期，%thread：表示线程名，%-5level：表示日志级别，从左开始显示5个字符宽度，
           %logger{36}：输出日志事件的位置，即类名，%msg：表示日志信息，%n：表示换行符 -->
       <property name="LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n"/>
       <!-- 定义控制台日志输出格式，其中 %d：表示日期，%thread：表示线程名，%-5level：表示日志级别，从左开始显示5个字符宽度，
               %logger{36}：输出日志事件的位置，即类名，%msg：表示日志信息，%n：表示换行符，%highlight()&%cyan()：着色-->
       <property name="LOG_CONSOLE_PATTERN"
                 value="%d{HH:mm:ss.SSS} [%thread] %highlight(%-5level) %cyan(%logger{50}) - %msg%n"/>
       <!-- 定义单个文件大小上限，超过之后会生成新文件  -->
       <property name="LOG_MAX_FILE_SIZE" value="512MB"/>
       <!-- 定义日志文件最大保留天数 -->
       <property name="LOG_MAX_HISTORY" value="30"/>
       <!-- 定义日志总大小，超过将删除最旧存档  -->
       <property name="LOG_TOTAL_SIZE_CAP" value="3GB"/>
   
       <!-- 控制台日志输出 -->
       <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
           <encoder>
               <pattern>${LOG_CONSOLE_PATTERN}</pattern>
           </encoder>
       </appender>
   
       <!-- INFO 级别的文件日志 -->
       <appender name="FILE_INFO_LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
           <!-- 定义输出时的级别过滤器，对于 ERROR 级别的日志全部 ACCEPT 接收 -->
           <filter class="ch.qos.logback.classic.filter.LevelFilter">
               <level>ERROR</level>
               <onMatch>ACCEPT</onMatch>
           </filter>
           <!-- 定义滚动策略 -->
           <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
               <!-- daily rollover -->
               <fileNamePattern>${LOG_HOME}/%d{yyyy-MM-dd}.%i.log</fileNamePattern>
               <!-- 单个文件大小上限 -->
               <maxFileSize>${LOG_MAX_FILE_SIZE}</maxFileSize>
               <!-- keep 30 days' worth of history capped at 3GB total size -->
               <maxHistory>${LOG_MAX_HISTORY}</maxHistory>
               <totalSizeCap>${LOG_TOTAL_SIZE_CAP}</totalSizeCap>
           </rollingPolicy>
           <!-- 输出格式 -->
           <encoder>
               <pattern>${LOG_PATTERN}</pattern>
               <charset>utf8</charset>
           </encoder>
       </appender>
   
       <!-- ERROR 级别的文件日志 -->
       <appender name="FILE_ERROR_LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
           <!-- 定义输出时的级别过滤器，对于低于 ERROR 级别的日志全部拒绝 -->
           <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
               <level>ERROR</level>
           </filter>
           <!-- 定义滚动策略 -->
           <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
               <!-- daily rollover -->
               <fileNamePattern>${LOG_HOME}/error.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
               <!-- 单个文件大小上限 -->
               <maxFileSize>${LOG_MAX_FILE_SIZE}</maxFileSize>
               <!-- keep 30 days' worth of history capped at 3GB total size -->
               <maxHistory>${LOG_MAX_HISTORY}</maxHistory>
               <totalSizeCap>${LOG_TOTAL_SIZE_CAP}</totalSizeCap>
           </rollingPolicy>
           <!-- 输出格式 -->
           <encoder>
               <pattern>${LOG_PATTERN}</pattern>
               <charset>utf8</charset>
           </encoder>
       </appender>
   
       <root level="${LOG_ROOT_LEVEL}">
           <appender-ref ref="STDOUT"/>
           <appender-ref ref="FILE_INFO_LOG"/>
           <appender-ref ref="FILE_ERROR_LOG"/>
       </root>
   </configuration>
   ```

#### 测试类

创建 `ArticleMapperTest` 测试类，用于测试 `ArticleMapper` 持久层接口中的方法。

```java
class ArticleMapperTest {
    private static final Logger logger = LoggerFactory.getLogger(ArticleMapperTest.class);

    @Test
    void findOne() {
        try (InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml")) {
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
            try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
                ArticleMapper articleMapper = sqlSession.getMapper(ArticleMapper.class);
                Article article = articleMapper.findOne(1);
                Author author = article.getAuthor();
                article.setAuthor(null);
                logger.info("article：{}", article);
                logger.info("author：{}", author);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

运行测试方法，发现居然报错！报错信息如下所示：<br />![image-20230920170957724](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201709914.png)

提示需要添加 `javassist` 依赖，存在如下两种解决方案：

1. 在 `mybatis-study` 父模块的 `pom.xml` 配置文件中声明该依赖，然后在当前模块中引入依赖即可！

   ```xml
   <dependency>
     <groupId>org.javassist</groupId>
     <artifactId>javassist</artifactId>
     <version>3.29.2-GA</version>
     <scope>compile</scope>
     <optional>true</optional>
   </dependency>
   ```

2. 修改 mybatis 源码模块中的 `pom.xml` 配置文件，将关于 `javassist` 依赖中的 `<optional>true</optional>` 改为 `false` 或者注释掉，如下所示：<br />![image-20230921204821783](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309212048015.png)

再次运行测试方法，发现还是报错！报错信息如下所示：<br />![image-20230920172040445](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201720566.png)

提示找不到 `ognl/PropertyAccessor` 类，查阅资料发现需要添加 `ognl` 依赖，依旧存在如下两种解决方案：

1. 在 `mybatis-study` 父模块的 `pom.xml` 配置文件中声明该依赖，然后在当前模块中引入依赖即可！

   ```xml
   <dependency>
       <groupId>ognl</groupId>
       <artifactId>ognl</artifactId>
       <version>3.3.4</version>
       <scope>compile</scope>
       <optional>true</optional>
   </dependency>
   ```

2. 修改 mybatis 源码模块中的 `pom.xml` 配置文件，将关于 `ognl` 依赖中的 `<optional>true</optional>` 改为 `false` 或者注释掉，如下所示：<br />![image-20230921205223117](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309212052226.png)

再次运行测试方法，发现终于成功啦！测试结果如下所示：<br />![image-20230920173521792](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201735888.png)

创建 `AuthorMapperTest` 测试类，用于测试 `AuthorMapper` 持久层接口中的方法。

```java
class AuthorMapperTest {
    private static final Logger logger = LoggerFactory.getLogger(AuthorMapperTest.class);

    @Test
    public void findOne() {
        try (InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml")) {
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
            try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
                AuthorMapper authorMapper = sqlSession.getMapper(AuthorMapper.class);
                Author author = authorMapper.findOne(1);
                List<Article> articles = author.getArticles();
                author.setArticles(null);
                logger.info("author：{}", author);
                logger.info("articles：{}", articles);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

运行测试方法，测试结果如下所示：<br />![image-20230920174140336](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309201741437.png)

至此，Mybatis 源码环境就搭建成功啦~🥳🥳🥳

