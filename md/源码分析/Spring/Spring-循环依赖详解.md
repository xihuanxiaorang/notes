# Spring-循环依赖详解

在分析 Spring 关于循环依赖的源码之前，先要了解何为循环依赖？主要分为以下三种情况：<br />
![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307312225083.jpeg)<br />无论循环依赖的数量有多少，循环依赖的本质是一样的：就是我的完整创建依赖于你，而你的完整创建也依赖于我，互相无法解耦，最终导致都创建失败！

> [!IMPORTANT|label:结论先行]
>
> Spring 通过<u>三级缓存提前暴露单例对象的引用</u>的方式只能解决<u><span style="background-color: rgb(193, 231, 126);">单例、setter 注入方式</span></u>的循环依赖。对于<u><span style="background-color: rgb(241, 162, 171);">多例、构造参数注入方式</span></u>的循环依赖是无法解决的；

<a name="doEpb"></a>

## 环境搭建

1. 利用 [Spring-源码环境搭建](./Spring-源码环境搭建) 搭建的Spring源码环境，创建一个新的模块专门用来研究 Spring 是如何通过三级缓存提前暴露对象的方式解决单例、setter 注入方式的循环依赖。选中项目右键新建一个模块，模块名为`spring- circularreference-source-test`，选择 Gradle，最后点击创建即可即可；<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010036015.png)

2. 引入相关依赖：在模块的 build.gradle 文件中引入以下依赖

   ```groovy
   dependencies {
       testImplementation 'org.junit.jupiter:junit-jupiter-api:5.8.1'
       testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.8.1'
       implementation(project(':spring-context'))
       implementation(project(':spring-aspects'))
       implementation 'org.slf4j:slf4j-api:2.0.3'
       implementation 'ch.qos.logback:logback-classic:1.4.4'
   }
   ```
3. 增加日志配置文件：由于引入了 logback，所以需要在资源目录 resources 下创建一个 logback.xml 文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <configuration>
     <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
       <encoder>
         <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5p %c{1}:%L - %m%n</pattern>
       </encoder>
     </appender>
   
     <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
       <encoder>
         <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5p %c{1}:%L - %m%n</pattern>
         <charset>utf-8</charset>
       </encoder>
       <file>log/output.log</file>
       <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
         <fileNamePattern>log/output.log.%i</fileNamePattern>
       </rollingPolicy>
       <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
         <MaxFileSize>1MB</MaxFileSize>
       </triggeringPolicy>
     </appender>
   
     <root level="DEBUG">
       <appender-ref ref="CONSOLE"/>
       <appender-ref ref="FILE"/>
     </root>
   </configuration>
   ```
4. 创建两个互相依赖的类 A 和 B

   ```java
   public class A {
       private B b;
   
       public B getB() {
           return b;
       }
   
       public void setB(B b) {
           this.b = b;
       }
   }
   ```

   ```java
   public class B {
   	private A a;
   
   	public A getA() {
   		return a;
   	}
   
   	public void setA(A a) {
   		this.a = a;
   	}
   }
   ```
5. 右键单击 resources 资源目录，创建 Spring 的核心配置文件 `applicationContext.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	   xmlns="http://www.springframework.org/schema/beans"
   	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   	<bean id="a" class="fun.xiaorang.spring.circularreference.A">
   		<property name="b" ref="b"/>
   	</bean>
   
   	<bean id="b" class="fun.xiaorang.spring.circularreference.B">
   		<property name="a" ref="a"/>
   	</bean>
   </beans>
   ```
6. 创建一个测试类 ApiTest

   ```java
   public class ApiTest {
   	private static final Logger LOGGER = LoggerFactory.getLogger(ApiTest.class);
   
   	@Test
   	public void test_00() {
   		ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
   		A a = ctx.getBean(A.class);
   		B b = ctx.getBean(B.class);
   		LOGGER.info("A = {}", a);
   		LOGGER.info("B = {}", b);
   		LOGGER.info("A.getB() = {}", a.getB());
   		LOGGER.info("B.getA() = {}", b.getA());
   	}
   }
   ```

   测试结果如下所示：<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010034516.png)<br />由上图可知，从A（B）中获取到的属性B（A）与直接从容器中获取到的B（A)单例对象相等，说明 Spring 是可以解决单例、setter 注入方式的循环依赖。

   > [!DEMO|label:演示多例、构造参数注入方式的循环依赖 Spring 是无法解决的案例]
   >
   > 1. 多例循环依赖测试：修改 applicationContext.xml 配置文件，将 A 和 B 的 scope 属性修改为 "prototype"，如下所示：
   >
   >    ```xml
   >    <?xml version="1.0" encoding="UTF-8"?>
   >    <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   >    	   xmlns="http://www.springframework.org/schema/beans"
   >    	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   >    	<bean id="a" class="fun.xiaorang.spring.circularreference.A" scope="prototype">
   >    		<property name="b" ref="b"/>
   >    	</bean>
   >    
   >    	<bean id="b" class="fun.xiaorang.spring.circularreference.B" scope="prototype">
   >    		<property name="a" ref="a"/>
   >    	</bean>
   >    </beans>
   >    ```
   >
   >    点击测试之后，运行报错，抛出如下异常信息！<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010033771.png)
   >
   > 2. 构造参数注入方式的循环依赖测试：修改A 和 B 两个类，增加有参构造函数，并且修改 applicationContext.xml 配置文件，使用 consturct-arg 参数给属性赋值。
   >
   >    ```java
   >    public class A {
   >        private B b;
   >       
   >        public A(B b) {
   >            this.b = b;
   >        }
   >       
   >        public B getB() {
   >            return b;
   >        }
   >       
   >        public void setB(B b) {
   >            this.b = b;
   >        }
   >    }
   >    ```
   >
   >    ```java
   >    public class B {
   >    	private A a;
   >       
   >    	public B(A a) {
   >    		this.a = a;
   >    	}
   >       
   >    	public A getA() {
   >    		return a;
   >    	}
   >       
   >    	public void setA(A a) {
   >    		this.a = a;
   >    	}
   >    }
   >    ```
   >
   >    ```xml
   >    <?xml version="1.0" encoding="UTF-8"?>
   >    <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   >    	   xmlns="http://www.springframework.org/schema/beans"
   >    	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   >    	<bean id="a" class="fun.xiaorang.spring.circularreference.A">
   >    		<constructor-arg name="b" ref="b"/>
   >    	</bean>
   >       
   >    	<bean id="b" class="fun.xiaorang.spring.circularreference.B">
   >    		<constructor-arg name="a" ref="a"/>
   >    	</bean>
   >    </beans>
   >    ```
   >
   >    点击测试之后，运行报错，抛出如下异常信息！与多例循环依赖测试中抛出的异常信息一致。<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010030023.png)
   
   <a name="ruFGg"></a>

## 源码分析
已经知道 <span style="background-color: rgb(193, 231, 126);">Spring 可以通过三级缓存提前暴露单例对象的引用的方式解决单例、setter 注入方式的循环依赖</span>，那么 Spring 底层到底是如何实现的呢？<br />先来了解一下三级缓存！

- 一级缓存：**singletonObjects**，单例缓存池，成品，用于存储已完成实例化、属性填充和初始化的单例 Bean，可直接对外使用；
- 二级缓存：**earlySingletonObjects**，半成品，用于存储已实例化但尚未完成属性填充和初始化的单例 Bean，提前暴露，供内部其他 Bean 引用，解决循环依赖问题；
- 三级缓存：**singletonFactories**，value 为 <u>ObjectFactory 函数式接口</u>，如果允许提前暴露的话，时机：<u>在 Bean 实例化之后，属性填充之前</u>，将当前 Bean 所对应的单例工厂存入该三级缓存中；
   - 当要用到三级缓存的时候，说明已经出现循环依赖的情况，此时从三级缓存中取出 ObjectFactory 实现（一个 Lambda 表达式，`() -> getEarlyBeanReference()`），执行 `ObjectFactory#getObject()` 方法时，实际上会执行 getEarlyBeanReference() 方法，在 `getEarlyBeanReference()` 方法中可能会执行 <u>AbstractAutoProxyCreator 后置处理器</u>中的 `getEarlyBeanReference()` 方法返回<u>代理对象</u>（AOP 产生代理对象时机提前）。

Spring 循环依赖处理流程如下所示：<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010039495.jpeg)<br />场景：组件 A 依赖于组件 B，组件 B 反过来依赖于组件 A；口述以下 DEBUG 流程，可用作面试。<br />创建 A 组件：

1. 实例化 A 组件，将 A 组件放入三级缓存中；
2. A 组件进行属性填充，发现依赖于组件 B，此时就会去容器中获取组件 B，发现没有，则开始创建 B 组件；
3. 创建 B 组件， 
   1. 实例化 B 组件，将 B 组件放入三级缓存中，
   2. B 组件进行属性填充，发现依赖于组件 A，此时就会去容器中获取组件 A，发现存在于三级缓存中，则从三级缓存中取，会执行 getEarlyRefrence() 方法，在执行该方法的时候会判断 A 组件是否需要增强，即是否有切面切到该组件，如果有的话，则会创建出 A 组件的代理对象放入二级缓存中，如果没有的话，则会将原来的 A 组件放入二级缓存中，二级缓存中的 A 组件此时还是一个半成品的 Bean，尚未完成属性填充和初始化操作。在放入二级缓存中的时候会将 A 组件从三级缓存中删除；
   3. B 组件执行初始化操作之后，也会判断是否需要进行增强，即是否有切面切到当前组件，如果有的话，则会创建出 B 组件的代理对象放入一级缓存中，如果没有的话，则会将原来的 B 组件放入一级缓存中。放入一级缓存的时候会从二级和三级缓存中删除（此时只有三级缓存中存在）。至此， B 组件的创建流程就完成了！
4. 从容器中获取到 B 组件之后，继续完成 A 组件的属性填充和初始化操作！
5. A 组件执行初始化操作之后，也会判断是否需要进行增强，如果需要进行增强的话，还会判断是否已经增强过，如果增强过就不再进行增强了，将 A 组件的代理对象放入一级缓存中，如果不需要进行增强的话，则将原生的 A 组件放入一级缓存中。放入一级缓存中的时候会从二级缓存中删除。至此，A 组件的创建流程也就完成了！

DEBUG 源码调试过程如下所示：记住！多在关键代码处打上断点，为了防止被干扰，给断点加上条件 "a".equals(beanName) || "b".equals(beanName) 

1. 在 AbstractBeanFactory#doGetBean() 方法中打上断点，加上条件："a".equals(beanName) || "b".equals(beanName)，如下所示：<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010041972.png)

2. 调用 DefaultSingletonBeanRegistry#getSingleton(beanName) 方法，

   ```java
   public Object getSingleton(String beanName) {
       // 获取 beanName 的单例对象（默认允许早期引用，即 allowEarlyReference == true）
       return getSingleton(beanName, true);
   }
   
   protected Object getSingleton(String beanName, boolean allowEarlyReference) {
       // Quick check for existing instance without full singleton lock
       // 从一级缓存中获取当前 Bean 实例对象
       Object singletonObject = this.singletonObjects.get(beanName);
       // 如果从一级缓存中获取不到并且当前 Bean 实例对象正在创建中
       if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
           // 则从二级缓存中获取当前 Bean 实例对象
           singletonObject = this.earlySingletonObjects.get(beanName);
           // 如果从二级缓存中也获取不到并且是允许早期引用的
           if (singletonObject == null && allowEarlyReference) {
               synchronized (this.singletonObjects) {
                   // Consistent creation of early reference within full singleton lock
                   // Double Lock Check 双重检查锁机制！
                   singletonObject = this.singletonObjects.get(beanName);
                   if (singletonObject == null) {
                       singletonObject = this.earlySingletonObjects.get(beanName);
                       if (singletonObject == null) {
                           // 最后从三级缓存中取出对应的单例工厂对象，是一个 Lambda 表达式，() -> getEarlyBeanReference(beanName, mbd, bean)
                           ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                           if (singletonFactory != null) {
                               // 通过工厂创建出早期单例对象，即调用 getEarlyBeanReference(beanName, mbd, bean) 方法创建对象，在该方法中可能会执行 AbstractAutoProxyCreator 后置处理器中的 getEarlyBeanReference() 方法返回代理对象
                               singletonObject = singletonFactory.getObject();
                               // 将创建出来的早期单例对象（可能是目标对象或代理对象）放入二级缓存中
                               this.earlySingletonObjects.put(beanName, singletonObject);
                               // 从三级缓存中移除该 beanName 所对应的单例工厂
                               this.singletonFactories.remove(beanName);
                           }
                       }
                   }
               }
           }
       }
       return singletonObject;
   }
   ```

   1. 在该方法中会先从一级缓存中获取，如果能获取到，则直接返回成品（已完成实例化、属性填充和初始化）的 Bean 实例对象；
   2. 如果从一级缓存中获取不到并且当前 Bean 实例对象正在创建中，则会从二级缓存中获取，如果能获取到，则返回半成品（只完成实例化但尚未完成属性填充和初始化） 的 Bean 实例对象；
   3. 如果从二级缓存中也获取不到并且允许早期引用的话，则会从三级缓存中获取，先从三级缓存中取出 ObjectFactory 实现，是一个 Lambda 表达式，() -> getEarlyBeanReference(beanName, mbd, bean)，当执行 ObjectFactory#getObject() 方法时，实际上会执行 getEarlyBeanReference() 方法，在 getEarlyBeanReference() 方法中可能会执行 AbstractAutoProxyCreator 后置处理器中的 getEarlyBeanReference() 方法返回代理对象。

   根据 beanName = "a" 从三级缓存中获取当前 A 实例对象，显然一级缓存中肯定不存在，并且当前 A 实例对象也没有被标记为正在创建中（即添加到正在创建中的缓存中），所以此时会直接退出该方法返回 NULL！<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010043102.png)

3. 从三级缓存中获取不到，说明当前 Bean 实例对象还没有开始创建，那么开始创建当前 Bean 实例对象

   ```java
   if (mbd.isSingleton()) {
       /*
           getSingleton() 方法，其中第二个参数的类型是一个 ObjectFactory<?>（函数式接口），不会立即执行，
           1. 先标记当前 bean 正在创建中，即将当前 Bean 实例对象添加到正在创建中的缓存中；
           2. 在该方法内部调用 ObjectFactory#getObject() 方法时，才会执行 Lambda 表达式中的 createBean() 方法开始创建 Bean 实例对象
           3. 创建完成之后，将正在创建中的标记移除
           4. 放入一级缓存中的同时从二级、三级缓存中移除，这三个缓存是互斥的，一个 Bean 只会存在于这三个缓存中的一个
        */
       sharedInstance = getSingleton(beanName, () -> {
           try {
               // 创建 bean 实例对象，开启完整的 bean 生命周期流程（实例化、属性填充、初始化）
               return createBean(beanName, mbd, args);
           } catch (BeansException ex) {
               // Explicitly remove instance from singleton cache: It might have been put there
               // eagerly by the creation process, to allow for circular reference resolution.
               // Also remove any beans that received a temporary reference to the bean.
               destroySingleton(beanName);
               throw ex;
           }
       });
       beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
   }
   ```

   getSingleton() 方法定义如下所示：仅列出比较关键的代码.

   ```java
   public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
       beforeSingletonCreation(beanName);
       singletonObject = singletonFactory.getObject();
       afterSingletonCreation(beanName);
       addSingleton(beanName, singletonObject);
   }
   
   protected void beforeSingletonCreation(String beanName) {
   	// 标记当前 Bean 正在创建中
       if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
           throw new BeanCurrentlyInCreationException(beanName);
       }
   }
   
   protected void afterSingletonCreation(String beanName) {
       // 移除当前 Bean 的正在创建中标记
       if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.remove(beanName)) {
           throw new IllegalStateException("Singleton '" + beanName + "' isn't currently in creation");
       }
   }
   
   protected void addSingleton(String beanName, Object singletonObject) {
       synchronized (this.singletonObjects) {
           // 放入一级缓存
           this.singletonObjects.put(beanName, singletonObject);
           // 从三级缓存中移除
           this.singletonFactories.remove(beanName);
           // 从二级缓存中移除
           this.earlySingletonObjects.remove(beanName);
           this.registeredSingletons.add(beanName);
       }
   }
   ```

   调用 DefaultSingletonBeanRegistry#getSingleton(beanName, ObjectFactory<?> singletonFactory) 方法，该方法的第二个参数是一个 ObjectFactory<?>（函数式接口），传递的参数实现是一个 Lambda 表达式，() -> createBean()。在该方法中，

      1. 先标记当前 Bean 正在创建中，即将当前 beanName 添加到正在创建中的缓存中；
      1. 调用 ObjectFactory#getObject() 方法，会执行 Lambda 表达式中的 createBean() 方法开始创建当前 Bean 实例对象，开启完整的 bean 生命周期流程（实例化、属性填充、初始化）；
      1. 创建完成之后，将正在创建中的标记移除；
      1. 将创建完成的 Bean 实例对象放入一级缓存，并从二级缓存和三级缓存中移除，这三个缓存是互斥的，一个 Bean 实例对象只会存在于这三个缓存中的一个；

   将 beanName = "a" 标记为正在创建中，即放入正在创建中的缓存中，<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010046215.png)<br />调用 ObjectFactory#getObject() 方法，即调用 createBean() 方法开始创建 A 实例对象。<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010046780.png)<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010047642.png)

4. 进入 createBean() -> doCreateBean() 方法，真正开始创建 Bean 实例对象，需要经历 Bean 的实例化、属性填充（依赖注入）、初始化三个阶段。

   ```java
   protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
       // 实例化阶段：创建 bean 实例，默认使用无参构造创建对象
   	instanceWrapper = createBeanInstance(beanName, mbd, args);
   
       /*
           提前暴露单实例 Bean，专门解决循环依赖.
           如果当前 bean 是单例的 && 允许循环引用 && 当前 bean 正在创建中，那么此时允许暴露早期单例 bean
           在此处将当前 Bean 实例对象所对应的对象工厂加到三级缓存中。
        */
       boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
               isSingletonCurrentlyInCreation(beanName));
       if (earlySingletonExposure) {
           if (logger.isTraceEnabled()) {
               logger.trace("Eagerly caching bean '" + beanName +
                       "' to allow for resolving potential circular references");
           }
           // 暴露早期单例 Bean 的引用存入三级缓存中，用于解决循环依赖
           // 在调用 getEarlyBeanReference() 方法时可能会执行 AbstractAutoProxyCreator 后置处理器中的 getEarlyBeanReference() 方法返回代理对象
           addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
       }
   
       // 属性填充（依赖注入）
       populateBean(beanName, mbd, instanceWrapper);
       // 初始化
       exposedObject = initializeBean(beanName, exposedObject, mbd);
   
       return exposedObject;
   }
   
   protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
       Object exposedObject = bean;
       if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
           for (SmartInstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().smartInstantiationAware) {
               // AbstractAutoProxyCreator 后置处理器中的 getEarlyBeanReference() 方法可能会返回代理对象（AOP生成代理对象的时机提前到此处）
               exposedObject = bp.getEarlyBeanReference(exposedObject, beanName);
           }
       }
       return exposedObject;
   }
   ```

   ```java
   protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
       Assert.notNull(singletonFactory, "Singleton factory must not be null");
       synchronized (this.singletonObjects) {
           if (!this.singletonObjects.containsKey(beanName)) {
               // 将单例工厂放入三级缓存中
               this.singletonFactories.put(beanName, singletonFactory);
               // 从二级缓存中移除
               this.earlySingletonObjects.remove(beanName);
               this.registeredSingletons.add(beanName);
           }
       }
   }
   ```

   1. 实例化阶段：先创建 Bean 实例对象，默认使用无参构造创建对象，这一步已经可以拿到 Bean 的引用，只是尚未完成属性填充和初始化而已，属于半成品的 Bean；
   2. 如果当前要创建的 Bean 是单例的 && 允许循环引用 && 正在创建中，说明允许提前暴露单例 Bean 引用，将单例工厂 () -> getEarlyBeanReference(beanName, mbd, bean) 放入三级缓存中，在调用 getEarlyBeanReference() 方法时可能会执行 AbstractAutoProxyCreator 后置处理器中的 getEarlyBeanReference() 方法返回代理对象（ AOP 产生代理对象时机提前），这一步专门用来解决循环依赖的；
   3. 属性填充阶段：给当前正在创建的 Bean 实例对象中的属性赋值或注入，这一步有可能会依赖其他 Bean 实例对象，那么会从三级缓存中获取依赖的其他 Bean 实例对象，如果缓存中不存在的话，则会触发其他 Bean 实例对象的创建流程；
   4. 初始化阶段：当前正在创建的 Bean 实例对象执行 InitializingBean#afterPropertiesSet()、自定义的 initMethod() 等初始化方法，在执行完一系列初始化方法之后，会执行 AbstractAutoProxyCreator 后置处理器中的 postProcessAfterInitialization() 方法返回代理对象（正常的 AOP 产生代理对象时机）。

   开始创建 A 实例对象，先通过反射调用 A 的无参构造方法创建出 A 实例对象，此时仅完成实例化尚未属性填充和初始化。<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010049741.png)<br />由于 A 实例对象满足是单例的 && 允许循环引用 && 正在创建中，所以允许提前暴露 A 实例对象的引用，创建 A 实例对象所对应的对象工厂实现存入三级缓存中，<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010050391.png)<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010050996.png)<br />由上图可知，A 实例对象中的 b 属性还没有被赋值，此时进入 A 实例对象的属性填充阶段，调用 populateBean() 方法<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010051488.png)<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010051540.png)<br /> 由上图可知，PropertyValue 中的 value 类型是一个 RuntimeBeanReference 类型，说明 A 实例对象依赖的 b 属性是一个 Bean 实例对象，需要调用 AbstractBeanFactory#doGetBean() 方法从容器中获取 B 实例对象。

5. 由于 A 实例对象依赖 B 实例对象，所以需要调用 AbstractBeanFactory#doGetBean() 方法从容器中获取 B 实例对象。其实就是回到第一步，只不过是 beanName 换成了 "b" 而已！<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010053372.png)

6. 调用 DefaultSingletonBeanRegistry#getSingleton(beanName) 方法，根据 beanName = "b" 从三级缓存中获取当前 B 实例对象，显然一级缓存中肯定不存在，并且当前 B 实例对象也没有被标记为正在创建中（即添加到正在创建中的缓存中），所以此时会直接退出该方法返回 NULL！<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010054292.png)

7. 从三级缓存中获取不到 B 实例对象，说明 B 实例对象还没有开始创建，那么就开始创建 B 实例对象，先标记 B 实例对象正在创建中，即放入正在创建中的缓存中，<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010055132.png)<br />调用 ObjectFactory#getObject() 方法，即调用 createBean() 方法开始创建 B 实例对象。<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010055666.png)

8. 进入 createBean() -> doCreateBean() 方法，真正开始创建 B 实例对象，同样需要经历实例化、属性填充（依赖注入）、初始化三个阶段。

   先通过反射调用 B 的无参构造方法创建出 B 实例对象，此时仅完成实例化尚未属性填充和初始化。<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010057447.png)<br />由于 B 实例对象满足是单例的 && 允许循环引用 && 正在创建中，所以允许提前暴露 B 实例对象的引用，创建 B 实例对象所对应的对象工厂实现存入三级缓存中，<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010057539.png)<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010057648.png)<br />由上图可知，B 实例对象中的 a 属性还没有被赋值，此时进入 B 实例对象的属性填充阶段，调用 populateBean() 方法，<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010058962.png)<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010058432.png)<br />由上图可知，PropertyValue 中的 value 类型是一个 RuntimeBeanReference 类型，说明 B 实例对象依赖的 a 属性是一个 Bean 实例对象，需要调用 AbstractBeanFactory#doGetBean() 方法从容器中获取 A 实例对象。

9. 由于 B 实例对象依赖 A 实例对象，所以需要调用 AbstractBeanFactory#doGetBean() 方法从容器中获取 A 实例对象。其实就是回到第一步，只不过是 beanName 又换回了 "a" 而已！<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010059167.png)

10. 调用 DefaultSingletonBeanRegistry#getSingleton(beanName) 方法，根据 beanName = "a" 从三级缓存中获取当前 A 实例对象，此时这种情况就与前面不相同了，一级缓存中肯定不存在（因为一级缓存中存储的都是成品的单例 Bean，而 A 实例对象的创建流程还没执行完成，才走到属性填充阶段呢！），但是 A 实例对象已经处在正在创建中，所以可以试着从二级缓存中获取，发现从二级缓存中也获取不到，因为允许早期引用，所以可以再试着从三级缓存中获取 A 实例对象。<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010100601.png)<br />从三级缓存中获取 A 实例对象时，会先从三级缓存中取出 ObjectFactory 实现（一个 Lambda 表达式，() -> getEarlyBeanReference()），执行 ObjectFactory#getObject() 的方法，实际上会执行 getEarlyBeanReference() 方法，在 getEarlyBeanReference() 方法中可能会执行 AbstractAutoProxyCreator 后置处理器中的 getEarlyBeanReference() 方法返回 A 的代理对象（AOP 产生代理对象的时机提前到此处）。<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010100867.png)<br />然后将 A 实例对象（目标对象或代理对象）放入二级缓存中，并从三级缓存中移除！升级：三级缓存 -> 二级缓存<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010101536.png)<br />此时，能够从二级缓存中获取到 A 实例对象（目标对象或代理对象，由于本次测试案例很简单，并没有 AOP 介入，获取到的是目标对象），那么就赋值给 B 实例对象中的 a 属性，此时 B 实例对象的属性填充阶段就完成了！<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010101176.png)<br />至于 B 实例对象的初始化阶段不在本次循环依赖解决方案的讨论范围内，直接跳过，至此，B 实例对象完整的生命周期流程已经完成！取消 B 实例对象正在创建标记，即从正在创建中的缓存中移除。<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010102234.png)<br />将 B 实例对象添加到一级缓存中，并从二级缓存中移除！升级：二级缓存 -> 一级缓存<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010102266.png)

11. B 实例对象创建完成之后，将 B 实例对象赋值给 A 实例对象的 b 属性，让 A 实例对象完成属性填充阶段。<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010103916.png)<br />至于 A 实例对象的初始化阶段不在本次循环依赖解决方案的讨论过程中，直接跳过，至此，A 实例对象完整的生命周期流程也已经完成！同样取消 A 实例对象正在创建标记，即从正在创建中的缓存中移除。<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010103576.png)<br />将 A 实例对象添加到一级缓存中，并从二级缓存中移除！升级：二级缓存 -> 一级缓存<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308010104204.png)<br />互相依赖的 A 、B 两个组件都成功创建并放入一级缓存中。

至此，DEBUG 源码调试循环依赖解决方案的过程就圆满成功！🎉🎉🎉

