# Mybatis 整体架构

MyBatis 分为三层架构，分别是**基础支撑层**、**核心处理层**和**接口层**，如下图所示：<br />![mybatis三层架构](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309210154545.png)

![image-20231010155757544](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310101557621.png)

## 基础支撑层

- **日志模块**。日志是咱们生产实践中排查问题、定位 Bug、锁定性能瓶颈的主要线索来源，在任何一个成熟的系统中都会有级别合理、信息翔实的日志模块，Mybatis 也不例外。Mybatis 提供了日志模块来集成 Java 生态中的第三方日志框架，该模块目前可以集成 slf4j、Log4j2、Log4j 等优秀的日志框架。
- **数据源模块**。持久层框架核心组件之一就是**数据源**，一款性能出众的数据源可以成倍提升系统的性能。MyBatis 自身提供了一套不错的数据源实现，也是 MyBatis 的默认实现。另外，在 Java 生态中，就有很多优异开源的数据源（如 [druid](https://github.com/alibaba/druid)、[HikariCP](https://github.com/brettwooldridge/HikariCP)）可供选择，MyBatis 的数据源模块中也提供了与第三方数据源集成的相关接口，这也为用户提供了更多的选择空间，提升了数据源切换的灵活性。
- **事务管理模块**。持久层框架一般都会提供一套事务管理机制实现数据库的事务控制，MyBatis 对数据库中的事务进行了一层简单的抽象，提供了简单易用的事务接口和实现。一般情况下，Java 项目都会集成 Spring，并由 Spring 框架管理事务。在后面的源码分析文章中，会深入分析 MyBatis 与 Spring 集成的原理，其中就包括事务管理相关的集成。



<span style="background-color: rgb(251, 228, 231);">TODO</span>

## 核心处理层

<span style="background-color: rgb(251, 228, 231);">TODO</span>

## 接口层

<span style="background-color: rgb(251, 228, 231);">TODO</span>