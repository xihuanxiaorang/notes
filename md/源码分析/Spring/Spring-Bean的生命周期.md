# Spring-Bean的生命周期

## 环境搭建
1. 利用 [Spring-源码环境搭建](./Spring-源码环境搭建.md) 搭建的Spring源码环境，创建一个新的模块专门用来研究 Bean 的生命周期。选中项目右键新建一个模块，模块名为 spring-beanlifecycle-source-test，选择 Gradle，最后点击创建即可即可；![64313a2e-2b0b-4353-8d4c-0bea9750faba](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307251052342.png)

2. 引入相关依赖：在模块的 build.gradle 文件中引入以下依赖

   ```gr
   dependencies {
       testImplementation 'org.junit.jupiter:junit-jupiter-api:5.8.1'
       testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.8.1'
       implementation(project(':spring-context'))
       implementation(project(':spring-aspects'))
       implementation 'org.slf4j:slf4j-api:2.0.3'
       implementation 'ch.qos.logback:logback-classic:1.4.4'
       implementation 'javax.annotation:javax.annotation-api:1.3.2'
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

4. 创建一个 BeanLifecycle 类

   ```java
   public class BeanLifecycle implements BeanNameAware, BeanFactoryAware, BeanClassLoaderAware, ApplicationContextAware, InitializingBean, DisposableBean {
   	private static final Logger LOGGER = LoggerFactory.getLogger(BeanLifecycle.class);
   	private Integer id;
   
   	public BeanLifecycle() {
   		LOGGER.info("constructor");
   	}
   
   	public Integer getId() {
   		return id;
   	}
   
   	public void setId(Integer id) {
   		LOGGER.info("property setId() => {}", id);
   		this.id = id;
   	}
   
   	@Override
   	public void setBeanName(String name) {
   		LOGGER.info("BeanNameAware#setBeanName() => {}", name);
   	}
   
   	@Override
   	public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
   		LOGGER.info("BeanFactoryAware#setBeanFactory() => {}", beanFactory);
   	}
   
   	@Override
   	public void setBeanClassLoader(ClassLoader classLoader) {
   		LOGGER.info("BeanClassLoaderAware#setBeanClassLoader() => {}", classLoader);
   	}
   
   	@Override
   	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
   		LOGGER.info("ApplicationContextAware#setApplicationContext() => {}", applicationContext);
   	}
   
   	@PostConstruct
   	public void postConstruct() {
   		LOGGER.info("postConstruct()");
   	}
   
   	@Override
   	public void afterPropertiesSet() {
   		LOGGER.info("InitializingBean#afterPropertiesSet()");
   	}
   
   	public void initMethod() {
   		LOGGER.info("initMethod()");
   	}
   
   	@PreDestroy
   	public void preDestroy() {
   		LOGGER.info("preDestroy()");
   	}
   
   	@Override
   	public void destroy() {
   		LOGGER.info("DisposableBean#destroy()");
   	}
   
   	public void destroyMethod() {
   		LOGGER.info("destroyMethod()");
   	}
   
   	@Autowired
   	public void autowire(@Value("${spring.profiles.active}") String activeProfile) {
   		LOGGER.info("autowire: {}", activeProfile);
   	}
   
   	@Resource
   	public void resource(@Value("${JAVA_HOME}") String home) {
   		LOGGER.info("resource: {}", home);
   	}
   }
   ```

5. 创建一个 MyBeanPostProcessor 的后置处理器(BeanPostProcessor)，纯粹用于演示一个 Bean 实例对象的完整的生命周期

   ```java
   public class MyBeanPostProcessor implements InstantiationAwareBeanPostProcessor {
   	private static final Logger LOGGER = LoggerFactory.getLogger(MyBeanPostProcessor.class);
   
   	@Override
   	public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
   		if (beanClass == BeanLifecycle.class) {
   			LOGGER.info("postProcessBeforeInstantiation");
   		}
   		return null;
   	}
   
   	@Override
   	public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
   		if (bean instanceof BeanLifecycle) {
   			LOGGER.info("postProcessAfterInstantiation");
   		}
   		return true;
   	}
   
   	@Override
   	public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
   		if (bean instanceof BeanLifecycle) {
   			LOGGER.info("postProcessProperties");
   		}
   		return null;
   	}
   
   	@Override
   	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
   		if (bean instanceof BeanLifecycle) {
   			LOGGER.info("postProcessBeforeInitialization");
   		}
   		return bean;
   	}
   
   	@Override
   	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
   		if (bean instanceof BeanLifecycle) {
   			LOGGER.info("postProcessAfterInitialization");
   		}
   		return bean;
   	}
   }
   ```

6. 右键单击 resources 资源目录，创建 Spring 的核心配置文件 `applicationContext.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	   xmlns:context="http://www.springframework.org/schema/context"
   	   xmlns="http://www.springframework.org/schema/beans"
   	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
   	   http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
   	<bean class="fun.xiaorang.spring.beanlifecycle.BeanLifecycle" init-method="initMethod"
   		  destroy-method="destroyMethod">
   		<property name="id" value="1"/>
   	</bean>
   
   	<!--	注册自定义的 BeanPostProcessor 后置处理器-->
   	<bean class="fun.xiaorang.spring.beanlifecycle.MyBeanPostProcessor"/>
   
   	<!--	开启注解配置，其实在底层就是向容器中注册了 CommonAnnotationBeanPostProcessor、AutowiredAnnotationBeanPostProcessor 等后置处理器，用于处理 @Autowired、@Value、@Resource 等注解-->
   	<context:annotation-config/>
   </beans>
   ```

7. 创建一个测试类 ApiTest

   ```java
   public class ApiTest {
   	private static final Logger LOGGER = LoggerFactory.getLogger(ApiTest.class);
   
   	@Test
   	public void test_00() {
   		ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
   		BeanLifecycle bean = ctx.getBean(BeanLifecycle.class);
   		LOGGER.info("{} 使用中...", bean);
   		ctx.close();
   	}
   }
   ```

   测试结果如下所示：<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307251053744.png" alt="d5ae9a73-0e25-4baf-a513-a0e1e2eb8acc"  />

   > [!ATTENTION]
   >
   > 在执行测试方法前，需要编辑运行配置，增加环境变量 `spring.profiles.active=dev`; <img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307251054533.png" alt="image.png"  />

## 源码分析
一个 Bean 的生命周期主要分为四个阶段：实例化、属性填充(依赖注入)、初始化、销毁阶段。

1. 实例化阶段
   1. 实例化前：执行 InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation() 方法
   2. 实例化
      1. 执行 SmartInstantiationAwareBeanPostProcessor#determineCandidateConstructors() 方法推断使用哪个构造器方法创建 Bean 实例对象
      2. 兜底方案：通过反射调用无参构造方法创建 Bean 实例对象
2. 执行 MergedBeanDefinitionPostProcessor#postProcessMergedBeanDefinition() 方法处理合并的 BeanDefinition 信息（可以再次修改 BeanDefinition 信息），将 @Autowired、@Resource、@Value 等注解的元信息封装到 BeanDefinition 中；
3. 实例化后：执行 BeanPostProcessor#postProcessAfterInstantiation() 方法，如果该方法返回 false，则中断后续流程，即不再执行后续的属性填充(依赖注入)和初始化阶段！提供一个修改 Bean 状态的机会，默认未做任何操作；
4. 属性填充(依赖注入)阶段
   1. 执行 InstantiationAwareBeanPostProcessor#postProcessProperties() 方法
      1. 执行 CommonAnnotationBeanPostProcessor#postProcessProperties() 方法，用于对标注了 @Resource 注解的属性或方法完成依赖注入
      2. 执行 AutowiredAnnotationBeanPostProcessor#postProcessProperties() 方法，用于对标注了 @Autowired 和 @Value 注解的属性或方法完成依赖注入
   2. 执行 AbstractAutowireCapableBeanFactory#applyPropterValues() 方法，通过反射调用setter方法给相应的属性赋值，值来源于当前 beanDefinition 对象中的 propertyValues 属性。在解析并注册当前 beanDefinition 对象时，已经将 XML 配置文件中当前 bean 标签下的 property 子标签中的内容保存到 beanDefinition 对象的 propertyValues 属性中
5. 初始化阶段
   1. 如果当前 Bean 实例对象实现了 BeanNameAware 或 BeanClassLoaderAware 或 BeanFactoryAware 接口，则在执行 AbstractAutowireCapableBeanFactory#initializeBean() -> invokeAwareMethods() 方法时会调用当前 Bean 实例对象中对这些 Aware 接口的重写方法
   2. 如果当前 Bean 实例对象实现了诸如 ApplicationContextAware、EnvironmentAware、ResourceLoaderAware、MessageSourceAware 等 Aware 接口，则在执行 ApplicationContextAwareProcessor#postProcessBeforeInitialization() 方法时会调用当前 Bean 实例对象中对这些 Aware 接口的重写方法
   3. 执行 BeanPostProcessor#postProcessBeforeInitialization() 方法
   4. 执行 InitDestroyAnnotationBeanPostProcessor#postProcessBeforeInitialization() 方法，用于处理标注了 @PostConstruct 注解的初始化方法
   5. 如果当前 Bean 实例对象实现了 InitializingBean 接口，则在执行 AbstractAutowireCapableBeanFactory#initializeBean() -> invokeInitMethods() 方法时会调用当前 Bean 实例对象中对该接口重写的 afterPropertiesSet() 初始化方法 
   6. 如果当前 Bean 实例对象在 bean 标签配置了中 init-method 属性，则在执行 AbstractAutowireCapableBeanFactory#initializeBean() -> invokeInitMethods()  -> invokeCustomInitMethod() 方法时会通过反射的方式调用自定义的初始化方法 initMthod()
   7. 执行 BeanPostProcessor#postProcessAfterInitialization() 方法 
      1. 正常情况下，AOP 的执行时机发生在初始化阶段，会执行 AbstractAutoProxyCreator 后置处理器中的 postProcessAfterInitialization() 方法生成代理对象
      2. 如果存在**循环依赖**的话，则 AOP 的执行时机会提前到属性填充阶段，执行 AbstractAutoProxyCreator 后置处理器中的 getEarlyBeanReference() 方法生成代理对象；如果属性填充阶段已经生成代理对象，那么后续初始化阶段就不再生成代理对象，直接从缓存中取即可！
6. 销毁阶段：手动调用容器的 close() 方法触发所有单实例 Bean 对象的销毁操作，一般不怎么关心该阶段！
   1. 执行 InitDestroyAnnotationBeanPostProcessor#postProcessBeforeDestruction() 方法，用于处理标注了 @PreDestroy 注解的销毁方法
   2. 如果当前 Bean 实例对象实现了 DisposableBean 接口，则会调用当前 Bean 实例对象中对该接口重写的销毁方法 destroy()
   3. 如果当前 Bean 实例对象在 bean 标签配置了中 destroy-method 属性，则会通过反射的方式调用自定义的销毁方法 destroyMthod()

![img](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307251057195.jpeg)
