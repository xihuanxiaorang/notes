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
> An alert of type 'attention' using global style 'callout'.