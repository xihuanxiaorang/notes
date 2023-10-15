# Mybatis-日志模块

> [!TIP]
>
> 在分析 Mybatis 中的日志模块之前，对于适配器模式和代理模式还不太清楚的小伙伴，建议先阅读 [适配器模式](../../../研磨设计模式/适配器模式.md) 和 [代理模式](../../../研磨设计模式/代理模式.md) 这两篇文章！

Java 开发中常用的几款日志框架：Apache Commons Logging、Log4j、Log4j2、java.util.logging，这些日志框架来源于不同的开源组织，给用户暴露的接口也有很多不同之处，所以很多开源框架会自定义一套统一的日志接口，兼容上述的第三方日志框架，供上层使用。一般的实现方式是使用**适配器模式**，**将各个第三方日志框架接口转换为框架内部自定义的日志接口**。

在 Mybatis 中也提供了类似的实现。Mybatis 使用的日志接口是自定义的 `Log` 接口，但是 Apache Commons Logging、Log4j、Log4j2、java.util.logging 等开源日志框架提供给用户的都是自定义的 `Logger` 接口，因此为了统一这些第三方的日志框架，**Mybatis 使用适配器模式添加了针对不同日志框架的 Adapter 实现**，使得第三方日志框架的 `Logger` 接口转换成 Mybatis 中的 `Log` 接口，从而实现集成第三方日志框架打印日志的功能。

Mybatis 中的日志模块位于 `org.apache.ibatis.logging` 包中，该模块的具体实现如下所示：

- 日志接口：`Log`，定义了自己的日志级别，该接口实现如下所示：

  ```java
  public interface Log {
  
    boolean isDebugEnabled();
  
    boolean isTraceEnabled();
  
    void error(String s, Throwable e);
  
    void error(String s);
  
    void debug(String s);
  
    void trace(String s);
  
    void warn(String s);
  
  }
  ```

  其类图如下所示：

  ```plantuml
  @startuml
  interface Log <<interface>> {}
  
  class Slf4jImpl {}
  
  class JakartaCommonsLoggingImpl {}
  
  class Log4j2Impl {}
  
  class Log4j2LoggerImpl {}
  
  class Log4jImpl{}
  
  class Jdk14LoggingImpl {}
  
  class NoLoggingImpl {}
  
  class StdOutImpl {}
  
  Log <|.. Slf4jImpl
  Log <|.. JakartaCommonsLoggingImpl
  Log <|.. Log4j2Impl
  Log <|.. Log4j2LoggerImpl
  Log <|.. Log4jImpl
  Log <|.. Jdk14LoggingImpl
  Log <|.. NoLoggingImpl
  Log <|.. StdOutImpl
  
  @enduml
  ```

- 日志工厂：`LogFactory`，它主要负责创建 `Log` 对象。在该类中存在一段**静态代码块**，其中会依次加载各个第三方日志框架的适配器。在静态代码块执行的 `tryImplementation()` 方法中，首先会检测 `logConstructor` 属性是否为空，如果不为空的话，则表示已经成功确定当前使用的日志框架，直接返回；如果为空的话，则在当前线程中执行传入的 `Runnable` 中的 `run()` 方法，尝试确定当前使用的日志框架。静态代码块实现如下所示：

  ```java
  static {
      // 依次尝试使用不同的日志框架 SLF4J > LOG4J2 > LOG4J > JDK > NO LOGGING
      // 在使用日志框架时，可以通过在类路径下放置相应的日志框架的实现类，来切换日志框架
      tryImplementation(LogFactory::useSlf4jLogging);
      tryImplementation(LogFactory::useCommonsLogging);
      tryImplementation(LogFactory::useLog4J2Logging);
      tryImplementation(LogFactory::useLog4JLogging);
      tryImplementation(LogFactory::useJdkLogging);
      tryImplementation(LogFactory::useNoLogging);
  }
  ```

  以 JDK Logging 的加载流程（`useJdkLogging()` 方法）为例，其具体代码实现和注释如下所示：

  ```java
  public static synchronized void useJdkLogging() {
  	setImplementation(org.apache.ibatis.logging.jdk14.Jdk14LoggingImpl.class);	
  }
  
  private static void setImplementation(Class<? extends Log> implClass) {
      try {
        // 获取 Log 接口的实现类（适配器）的构造方法
        Constructor<? extends Log> candidate = implClass.getConstructor(String.class);
        // 尝试加载当前使用的日志框架的适配器类，如果加载失败，则抛出异常
        Log log = candidate.newInstance(LogFactory.class.getName());
        if (log.isDebugEnabled()) {
          // 如果开启 DEBUG 日志，则打印当前使用的日志框架的名称
          log.debug("Logging initialized using '" + implClass + "' adapter.");
        }
        // 加载成功，则更新 logConstructor 属性，记录适配器的构造方法
        logConstructor = candidate;
      } catch (Throwable t) {
        throw new LogException("Error setting Log implementation.  Cause: " + t, t);
      }
  }
  ```

- 日志实现类（**适配器**）：以 `Jdk14LoggingImpl` 为例介绍一下 `Log` 接口的实现， `Jdk14LoggingImpl` 作为 Java Logging 的适配器，在实现 `Log` 接口的同时，在其内部持有一个 `java.util.logging.Logger` 类型的引用，会将日志输出操作委托给持有的 `java.util.logging.Logger` 对象的相应方法，实现与典型的类适配器模式完全一致。 `Jdk14LoggingImpl` 中的核心实现以及代码注释如下所示：

  ```java
  public class Jdk14LoggingImpl implements Log {
  
    /**
     * JDK 的 Logger 对象
     */
    private final Logger log;
  
    public Jdk14LoggingImpl(String clazz) {
      // 实例化 JDK 的 Logger 对象
      log = Logger.getLogger(clazz);
    }
  
    @Override
    public boolean isDebugEnabled() {
      return log.isLoggable(Level.FINE);
    }
  
    @Override
    public void error(String s, Throwable e) {
      // 委托给 JDK 的 Logger 对象进行日志记录
      log.log(Level.SEVERE, s, e);
    }
  
    // 省略其他级别的日志输出方法
  
  }
  ```

在 Mybatis 的 `org.apache.ibatis.logging` 包中，除了集成第三方日志框架的适配器实现之外，还有一个 `jdbc` 包，该包的功能不是将日志写入数据库中，而是将 JDBC 操作涉及的信息通过指定的 `Log` 进行输出到控制台（或保存到日志文件中），咱们可以通过这个包，将执行的 SQL 语句、SQL 绑定的参数、SQL 执行之后影响的行数等信息，统统输出到控制台，该功能主要是在测试环境进行调试时使用，很少会在线上开启，因为会产生非常多的日志，拖慢系统性能。咱们接下来就好好分析一下 `org.apache.ibatis.logging.jdbc` 包中的内容，

- `BaseJdbcLogger`：最基础的抽象类，是 `org.apache.ibatis.logging.jdbc` 包下其他 `Logger` 的父类，其继承关系如下图所示：

  ```plantuml
  @startuml
  abstract class BaseJdbcLogger {
  	# statementLog: Log
  	# queryStack: int
  }
  
  class ConnectionLogger {
  	- connection: Connection
  	+ newInstance(Connection conn, Log statementLog, int queryStack): Connection
  	+ invoke(Object proxy, Method method, Object[] params): Object
  }
  
  class PreparedStatementLogger {
  	- statement: PreparedStatement
  	+ newInstance(PreparedStatement stmt, Log statementLog, int queryStack): PreparedStatement
  	+ invoke(Object proxy, Method method, Object[] params): Object
  }
  
  class ResultSetLogger {
  	- rs: ResultSet
  	+ newInstance(ResultSet rs, Log statementLog, int queryStack): ResultSet
  	+ invoke(Object proxy, Method method, Object[] params): Object
  }
  
  interface InvocationHandler <<interface>> {
  	+ invoke(Object proxy, Method method, Object[] args): Object
  }
  
  BaseJdbcLogger <|-- ConnectionLogger
  BaseJdbcLogger <|-- PreparedStatementLogger
  BaseJdbcLogger <|-- ResultSetLogger
  InvocationHandler <|.. ConnectionLogger
  InvocationHandler <|.. PreparedStatementLogger
  InvocationHandler <|.. ResultSetLogger
  
  @enduml
  ```

  在 `BaseJdbcLogger` 抽象父类中，定义了如下两个静态常量：

  - SET_METHODS：该集合用于存储 `PreparedStatement` 中所有 `set*()` 方法的名称，例如：setString、setInt、setLong 等；
  - EXECUTE_METHODS：该集合用于存储 `PreparedStatement` 中所有 `execute*()` 方法的名称，例如：execute、executeUpdate、executeQuery、addBatch 等；

- `ConnectionLogger`：负责创建 `Connection` 数据库连接对象的<u>**代理对象**</u>。

  - 其中的 `newInstance()` 方法用于创建并返回 `Connection` 数据库连接对象的<u>**代理对象**</u>，这样在调用 `Connection` 代理对象中的方法时，就会进行日志记录；

    ```java
    /**
    * 创建并返回 Connection 数据库连接对象的代理对象
    *
    * @param conn         原始的 Connection 数据库连接对象
    * @param statementLog StatementLog 日志对象
    * @param queryStack   查询堆栈
    * @return Connection 数据库连接对象的代理对象，带有日志记录功能
    */
    public static Connection newInstance(Connection conn, Log statementLog, int queryStack) {
        InvocationHandler handler = new ConnectionLogger(conn, statementLog, queryStack);
        ClassLoader cl = Connection.class.getClassLoader();
        // 创建 Connection 数据库连接对象的代理对象
        return (Connection) Proxy.newProxyInstance(cl, new Class[]{Connection.class}, handler);
    }
    ```

  - 重写 `InvocationHandler` 接口中的 `invoke()` 方法，在该方法中，会判断当前执行的是 `Connection` 中的哪一个方法？存在如下几种情况：⬇️

    1. 如果当前方法是 `Object` 类中的方法的话，则直接调用，不做其他任何处理；
    2. 如果当前方法是 `prepareStatement()` 方法的话，则会打印出 SQL 语句并使用 `PreparedStatementLogger` 中的 `newInstance()` 方法创建并返回 `PreparedStatement` 数据库操作对象的<u>**代理对象**</u>；
    3. 如果当前方法是 `createStatement()` 方法的话，则使用 `StatementLogger` 中的 `newInstance()` 方法创建并返回 `Statement` 数据库操作对象的<u>**代理对象**</u>，这样在调用 `Statement` 对象中的方法时，同样会进行日志记录；
    4. 如果当前方法是 `Connection` 中的其他方法，则直接调用原始 `Connection` 对象中对应的方法。

    ```java
    public Object invoke(Object proxy, Method method, Object[] params) throws Throwable {
        try {
          // 如果当前方法是 Object 类中的方法，则直接调用，不做其他任何处理
          if (Object.class.equals(method.getDeclaringClass())) {
            return method.invoke(this, params);
          }
          // 如果当前方法是 Connection 中的 prepareStatement 或 prepareCall 方法，则进行日志记录
          if ("prepareStatement".equals(method.getName()) || "prepareCall".equals(method.getName())) {
            if (isDebugEnabled()) {
              // 如果开启 DEBUG 日志，则打印 SQL 语句，格式类似于：==>  Preparing: select * from user where id = ?
              debug(" Preparing: " + removeExtraWhitespace((String) params[0]), true);
            }
            // 使用 Connection 数据库连接对象创建 PreparedStatement 对象
            PreparedStatement stmt = (PreparedStatement) method.invoke(connection, params);
            // 创建并返回 PreparedStatement 数据库操作对象的代理对象，这样在调用 PreparedStatement 对象中的方法时，就会进行日志记录
            return PreparedStatementLogger.newInstance(stmt, statementLog, queryStack);
          }
          // 如果当前方法是 Connection 中的 createStatement 方法
          if ("createStatement".equals(method.getName())) {
            // 使用 Connection 数据库连接对象创建 Statement 对象
            Statement stmt = (Statement) method.invoke(connection, params);
            // 创建并返回 Statement 数据库操作对象的代理对象，这样在调用 Statement 对象中的方法时，就会进行日志记录
            return StatementLogger.newInstance(stmt, statementLog, queryStack);
          } else {
            // 如果当前方法是 Connection 中的其他方法，则直接调用
            return method.invoke(connection, params);
          }
        } catch (Throwable t) {
          throw ExceptionUtil.unwrapThrowable(t);
        }
    }
    ```

- `PreparedStatementLogger`：负责创建 `PreparedStatement` 数据库操作对象的<u>**代理对象**</u>。

  - 其中的 `newInstance()` 方法用于创建并返回 `PreparedStatement` 数据库操作对象的<u>**代理对象**</u>，这样在调用 `PreparedStatement` 代理对象中的方法时，就会进行日志记录；

    ```java
    public static PreparedStatement newInstance(PreparedStatement stmt, Log statementLog, int queryStack) {
        InvocationHandler handler = new PreparedStatementLogger(stmt, statementLog, queryStack);
        ClassLoader cl = PreparedStatement.class.getClassLoader();
        // 创建 PreparedStatement 数据库操作对象的代理对象
        return (PreparedStatement) Proxy.newProxyInstance(cl, new Class[]{PreparedStatement.class, CallableStatement.class}, handler);
    }
    ```

  - 重写 `InvocationHandler` 接口中的 `invoke()` 方法，在该方法中，会判断当前执行的是 `PreparedStatement` 中的哪一个方法？存在如下几种情况：⬇️

    1. 如果当前方法是 `Object` 类中的方法的话，则直接调用，不做其他任何处理；
    2. 如果当前方法在 `EXECUTE_METHODS` 集合中，则会打印出参数信息，并且进一步判断当前方法是不是 `executeQuery()` 方法？
       - 如果当前方法是 `executeQuery()` 方法的话，则会调用原始 `PreparedStatement` 对象中的 `executeQuery()` 方法，得到 `ResultSet` 结果集对象，如果 `ResultSet` 结果集对象不为空的话，则会使用 `ResultSetLogger` 中的 `newInstance()` 方法创建并返回 ResultSet 结果集对象的<u>**代理对象**</u>；
       - 如果当前方法是其他三个方法 `execute()`、`executeUpdate()`、`addBatch()` 中的一个，则直接调用原始 `PreparedStatement` 对象中对应的方法。
    3. 如果当前方法在 `SET_METHODS` 集合中，则会进行参数记录，然后再直接调用原始 `PreparedStatement` 对象中对应的 `set*()` 方法；
    4. 如果当前方法是 `getResultSet()` 方法的话，则会调用原始 `PreparedStatement` 对象中的 `getResultSet()` 方法，得到 `ResultSet` 结果集对象，如果 `ResultSet` 结果集对象不为空的话，则会使用 `ResultSetLogger` 中的 `newInstance()` 方法创建并返回 ResultSet 结果集对象的<u>**代理对象**</u>；
    5. 如果当前方法是 `getUpdateCount()` 方法的话，则会调用原始 `PreparedStatement` 对象中的 `getUpdateCount()` 方法，得到更新的行数，如果更新的行数不为 -1 的话，则打印更新的行数，最后返回更新的行数；
    6. 如果当前方法是 `PreparedStatement` 中的其他方法，则直接调用原始 `PreparedStatement` 对象中对应的方法。

    ```java
    public Object invoke(Object proxy, Method method, Object[] params) throws Throwable {
        try {
          // 如果当前方法是 Object 类中的方法，则直接调用，不做其他任何处理
          if (Object.class.equals(method.getDeclaringClass())) {
            return method.invoke(this, params);
          }
          // 如果当前方法是 PreparedStatement 接口中的 execute、executeUpdate、executeQuery、addBatch 方法，则进行日志记录
          if (EXECUTE_METHODS.contains(method.getName())) {
            if (isDebugEnabled()) {
              // 如果开启 DEBUG 日志，则打印参数信息，格式类似于：==>  Parameters: 1(String)
              debug("Parameters: " + getParameterValueString(), true);
            }
            // 清空 column* 集合
            clearColumnInfo();
            if ("executeQuery".equals(method.getName())) {
              // 如果当前方法是 executeQuery 方法，则会调用原始 PreparedStatement 对象中的 executeQuery 方法，得到 ResultSet 结果集对象
              ResultSet rs = (ResultSet) method.invoke(statement, params);
              // 如果 ResultSet 结果集对象不为空，则创建并返回 ResultSet 结果集对象的代理对象，这样在调用 ResultSet 对象中的方法时，就会进行日志记录
              return rs == null ? null : ResultSetLogger.newInstance(rs, statementLog, queryStack);
            } else {
              // 如果当前方法是其他三个方法 execute、executeUpdate、addBatch 中的一个，则直接调用原始 PreparedStatement 对象中对应的方法
              return method.invoke(statement, params);
            }
          }
          // 如果当前方法是 PreparedStatement 接口中的 set 方法，则进行参数记录
          if (SET_METHODS.contains(method.getName())) {
            if ("setNull".equals(method.getName())) {
              setColumn(params[0], null);
            } else {
              // 记录参数信息，第一个参数是 key = 参数的索引位置，第二个参数是 value = 参数的值
              setColumn(params[0], params[1]);
            }
            // 直接调用原始 PreparedStatement 对象的 set 方法
            return method.invoke(statement, params);
          } else if ("getResultSet".equals(method.getName())) {
            // 如果当前方法是 getResultSet 方法，则调用原始 PreparedStatement 对象的 getResultSet 方法，得到 ResultSet 结果集对象
            ResultSet rs = (ResultSet) method.invoke(statement, params);
            // 如果 ResultSet 结果集对象不为空，则创建并返回 ResultSet 结果集对象的代理对象，这样在调用 ResultSet 对象中的方法时，就会进行日志记录
            return rs == null ? null : ResultSetLogger.newInstance(rs, statementLog, queryStack);
          } else if ("getUpdateCount".equals(method.getName())) {
            // 如果当前方法是 getUpdateCount 方法，则调用原始 PreparedStatement 对象的 getUpdateCount 方法，得到更新的行数
            int updateCount = (Integer) method.invoke(statement, params);
            if (updateCount != -1) {
              // 如果更新的行数不为 -1，则打印更新的行数，格式类似于：   Updates: 1
              debug("   Updates: " + updateCount, false);
            }
            // 返回更新的行数
            return updateCount;
          } else {
            // 如果当前方法是 PreparedStatement 接口中的其他方法，则直接调用原始 PreparedStatement 对象中对应的方法
            return method.invoke(statement, params);
          }
        } catch (Throwable t) {
          throw ExceptionUtil.unwrapThrowable(t);
        }
    }
    ```

- `ResultSetLogger`：负责创建 `ResultSet` 结果集对象的<u>**代理对象**</u>，其中会针对 `ResultSet` 中的 `next()` 方法进行后置处理，主要是打印结果集中的每一行数据以及统计结果集总行数等信息。

  - 其中的 `newInstance()` 方法用于创建并返回 `ResultSet` 结果集对象的<u>**代理对象**</u>，这样在调用 `ResultSet` 代理对象中的方法时，就会进行日志记录；

    ```java
    public static ResultSet newInstance(ResultSet rs, Log statementLog, int queryStack) {
        InvocationHandler handler = new ResultSetLogger(rs, statementLog, queryStack);
        ClassLoader cl = ResultSet.class.getClassLoader();
        // 创建 ResultSet 结果集对象的代理对象
        return (ResultSet) Proxy.newProxyInstance(cl, new Class[]{ResultSet.class}, handler);
    }
    ```

  - 重写 `InvocationHandler` 接口中的 `invoke()` 方法，在该方法中，会判断当前执行的是 `ResultSet` 中的哪一个方法？存在如下几种情况：⬇️

    1. 如果当前方法是 `Object` 类中的方法的话，则直接调用，不做其他任何处理；
    2. 如果当前方法属于 `ResultSet` 类中的方法，则直接调用原始 `ResultSet` 对象中对应的方法。不过会额外针对 `next()` 方法进行后置处理，判断 `next()` 方法的返回值是否为真？
       - 如果为真的话，则表示存在下一行数据，`rows++`，并且如果开启 TRACE 日志，则打印 `ResultSet` 结果集中的列名称和每一行数据；
       - 如果为假的话，则表示已完成结果集的遍历，打印总行数；

    ```java
    public Object invoke(Object proxy, Method method, Object[] params) throws Throwable {
        try {
          // 如果当前方法是 Object 类中的方法，则直接调用，不做其他任何处理
          if (Object.class.equals(method.getDeclaringClass())) {
            return method.invoke(this, params);
          }
          // 直接调用原始 ResultSet 对象中对应的方法
          Object o = method.invoke(rs, params);
          // 针对 next 方法进行后置处理
          if ("next".equals(method.getName())) {
            // 检测 next 方法的返回值，确定是否还有下一行数据，如果方法返回 true 的话，则打印 ResultSet 结果集中的列名称和每一行数据
            if ((Boolean) o) {
              // 记录结果集中的行数
              rows++;
              // 如果开启 TRACE 日志，则打印 ResultSet 结果集中的列名称和每一行数据
              if (isTraceEnabled()) {
                // 获取 ResultSet 结果集中的列元数据
                ResultSetMetaData rsmd = rs.getMetaData();
                // 获取 ResultSet 结果集中的列数量
                final int columnCount = rsmd.getColumnCount();
                if (first) {
                  first = false;
                  // 打印 ResultSet 结果集中的列名称，格式类似于：   Columns: id, name, age
                  printColumnHeaders(rsmd, columnCount);
                }
                // 打印 ResultSet 结果集中的每一行数据，格式类似于：       Row: 1, 张三, 18
                printColumnValues(columnCount);
              }
            } else {
              // 如果 next 方法返回 false，则表示没有下一行数据了，打印总共查询到的行数，格式类似于：     Total: 1
              debug("     Total: " + rows, false);
            }
          }
          // 清空 column* 集合
          clearColumnInfo();
          return o;
        } catch (Throwable t) {
          throw ExceptionUtil.unwrapThrowable(t);
        }
    }
    ```
