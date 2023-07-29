# 入门示例

> Hello Docsify!

## tabs选项卡示例
<!-- tabs:start -->

#### **English**

Hello!

```java
public class LazyDoubleCheckSingleton {
    private static volatile LazyDoubleCheckSingleton INSTANCE;

    private LazyDoubleCheckSingleton() {
        // 私有构造函数
    }

    public static LazyDoubleCheckSingleton getInstance() {
        if (INSTANCE == null) {
            synchronized (LazyDoubleCheckSingleton.class) {
                if (INSTANCE == null) {
                    INSTANCE = new LazyDoubleCheckSingleton();
                }
            }
        }
        return INSTANCE;
    }
}
```

#### **French**

Bonjour!

#### **Italian**

Ciao!

<!-- tabs:end -->

## alerts示例

> [!NOTE]
> An alert of type 'note' using global style 'callout'.

> [!TIP]
> An alert of type 'tip' using global style 'callout'.

> [!WARNING]
> An alert of type 'warning' using global style 'callout'.

> [!ATTENTION|style:flat]
> An alert of type 'attention' using special style 'flat'.

> [!important]
> An alert of type 'important' using global style 'callout'.


## PlantUML示例

```plantuml
@startuml

interface Invocation << interface >> {
  + getArguments(): Object[]
}

interface Joinpoint << interface >> {
  + getThis(): Object?
  + proceed(): Object?
  + getStaticPart(): AccessibleObject
}

interface MethodInvocation << interface >> {
  + getMethod(): Method
}

interface ProxyMethodInvocation << interface >> {
  + getProxy(): Object
  + setArguments(Object[]): void
  + setUserAttribute(String, Object?): void
  + invocableClone(): MethodInvocation
  + invocableClone(Object[]): MethodInvocation
  + getUserAttribute(String): Object?
}

class ReflectiveMethodInvocation {
	# proxy: Object
	# target: Object
	# method: Method
	# arguments: Object[]
	- targetClass: Class<?>
	# interceptorsAndDynamicMethodMatchers: List<?>
	- currentInterceptorIndex: int = -1
  + getArguments(): Object[]
  + proceed(): Object?
  + setUserAttribute(String, Object?): void
  + invocableClone(): MethodInvocation
  + getStaticPart(): AccessibleObject
  + getThis(): Object?
  # invokeJoinpoint(): Object?
  + getUserAttribute(String): Object?
  + setArguments(Object[]): void
  + getUserAttributes(): Map<String, Object>
  + getMethod(): Method
  + invocableClone(Object[]): MethodInvocation
  + toString(): String
  + getProxy(): Object
}

class CglibMethodInvocation {
	- methodProxy: MethodProxy
	+ proceed(): Object
	+ invokeJoinpoint(): Object
}

Joinpoint <|-- Invocation
Invocation <|-- MethodInvocation
MethodInvocation <|-- ProxyMethodInvocation
ProxyMethodInvocation <|.. ReflectiveMethodInvocation
ReflectiveMethodInvocation <|-- CglibMethodInvocation

@enduml
```

```plantuml
@startuml

interface Advice {
}

interface Interceptor {

}

interface MethodInterceptor { 
+ Object invoke(MethodInvocation invocation)
}

abstract class AbstractAspectJAdvice {
}

class AspectJMethodBeforeAdvice { 
+ Object invoke(MethodInvocation mi)
}

class AspectJAroundAdvice { 
+ Object invoke(MethodInvocation mi)
}

class AspectJAfterReturningAdvice { 
+ void afterReturning(@Nullable Object returnValue, Method method, Object[] args, @Nullable Object target)
}

class AspectJAfterThrowingAdvice { 
+ Object invoke(MethodInvocation mi)
}

class AspectJAfterAdvice { 
+ Object invoke(MethodInvocation mi)
}

interface AdvisorAdapter << interface >> { 
+ boolean supportsAdvice(Advice advice)
+ MethodInterceptor getInterceptor(Advisor advisor)
}

class MethodBeforeAdviceAdapter { 
+ boolean supportsAdvice(Advice advice) 
+ MethodInterceptor getInterceptor(Advisor advisor)
}

class AfterReturningAdviceAdapter { 
+ boolean supportsAdvice(Advice advice)
+ MethodInterceptor getInterceptor(Advisor advisor)
}

class MethodBeforeAdviceInterceptor { 
- MethodBeforeAdvice  advice
+ Object invoke(MethodInvocation mi)
}

class AfterReturningAdviceInterceptor { 
- AfterReturningAdvice  advice
+ Object invoke(MethodInvocation mi)
}

Advice <|-- Interceptor
Interceptor <|-- MethodInterceptor
Advice <|.. AbstractAspectJAdvice
AbstractAspectJAdvice <|-- AspectJMethodBeforeAdvice
AbstractAspectJAdvice <|-- AspectJAroundAdvice
MethodInterceptor <|.. AspectJAroundAdvice
AbstractAspectJAdvice <|-- AspectJAfterReturningAdvice
AbstractAspectJAdvice <|-- AspectJAfterThrowingAdvice
MethodInterceptor <|.. AspectJAfterThrowingAdvice
AbstractAspectJAdvice <|-- AspectJAfterAdvice
MethodInterceptor <|.. AspectJAfterAdvice
AdvisorAdapter <|.. MethodBeforeAdviceAdapter
AdvisorAdapter <|.. AfterReturningAdviceAdapter
MethodInterceptor <|.. MethodBeforeAdviceInterceptor
MethodInterceptor <|.. AfterReturningAdviceInterceptor

@enduml
```

## 任务列表

* [ ] 每天记得喝八杯水
* [x] ~每天运动一小时~
* [x] 每天看书两小时

## FAQ问答示例

+ 问题1? +

  答案1

+ 问题2? +

  答案2