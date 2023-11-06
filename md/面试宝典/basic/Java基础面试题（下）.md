# Java 基础面试题（下）

## Integer a1 = 100，Integer a2 = 100，Integer a3 = 128，Integer a4 = 128，int a5 = 128，描述 a1 == a2 、a3 == a4 以及 a3 == a5 的运行结果以及原因

🤓：运行结果为 **true**、**false**、**true**，原因见如下分析过程。

测试代码如下：

```java
public class Test1 {  
    private static final Logger LOGGER = LoggerFactory.getLogger(Test1.class);  
  
    public static void main(String[] args) {  
        Integer a1 = 100;  
        Integer a2 = 100;  
        Integer a3 = 128;  
        Integer a4 = 128;  
        int a5 = 128;  
        LOGGER.debug("a1 == a2？{}", a1 == a2);  
        LOGGER.debug("a3 == a4？{}", a3 == a4);  
        LOGGER.debug("a3 == a5？{}", a3 == a5);  
    }  
}
```

运行结果如下所示：

```
2022-10-14 15:55:56.375 DEBUG Test1:22 - a1 == a2？true
2022-10-14 15:55:56.376 DEBUG Test1:23 - a3 == a4？false
2022-10-14 15:55:56.376 DEBUG Test1:24 - a3 == a5？true
```

### 原理

#### == 比较的是两个对象的引用地址

用 == 比较的是两个对象的引用地址，如果两个对象的引用地址相同，则 == 比较后返回的结果为 true；如果不同的话，则比较后的结果为 false。  

#### Integer 对象的自动拆装箱操作

自动装箱：就是将基本数据类型自动转换成对应的包装类；自动拆箱：就是将包装类自动转换成对应的基本类型数据。

```java
Integer i = 10; // 自动装箱
int b = i; // 自动拆箱
```

对以上代码进行反编译后可以得到如下代码：

```java
Integer i = Integer.valueOf(10);  
int b = i.intValue();
```

从上面反编译后的代码可以看出，`int` 的自动装箱是通过 `Integer` 类中的 `valueOf()` 方法来实现的，而 `Integer` 自动拆箱都是通过 `intValue()` 方法来实现的。感兴趣的小伙伴，可以试着将八种基本数据类型都照这样反编译一遍，你就会发现一个规律：**自动装箱都是通过包装类的 `valueOf()` 方法来实现的，自动拆箱都是通过包装类的 `xxxValue()` 方法来实现的**。

#### Integer 对象的 valueOf 方法

>`Integer` 对象中的 `valueOf()` 方法用到了 **享元设计模式**，如何判断是否使用享元模式呢？它会先返回缓存对象而不是创建新的对象。对于 **享元设计模式** 不清楚的小伙伴可以查看 [享元模式](../../研磨设计模式/享元模式.md) 这一篇文章，文章中详细地介绍了使用享元模式的目的以及如何实现享元模式。  

```java
public static Integer valueOf(int i) {  
    if (i >= IntegerCache.low && i <= IntegerCache.high)  
        return IntegerCache.cache[i + (-IntegerCache.low)];  
    return new Integer(i);  
}
```

在 `Integer` 包装类中有一个私有的静态内部类 `IntegerCache`，在这个类中缓存了从 [-128, 127] 之间的所有的 `Integer` 对象，并将其存放在 `cache` 数组中。

```java
private static class IntegerCache {  
    static final int low = -128;  
    static final int high;  
    static final Integer[] cache;  
    static Integer[] archivedCache;  
  
    static {  
        // high value may be configured by property  
        int h = 127;  
        String integerCacheHighPropValue =  
            VM.getSavedProperty("java.lang.Integer.IntegerCache.high");  
        if (integerCacheHighPropValue != null) {  
            try {  
                h = Math.max(parseInt(integerCacheHighPropValue), 127);  
                // Maximum array size is Integer.MAX_VALUE  
                h = Math.min(h, Integer.MAX_VALUE - (-low) -1);  
            } catch( NumberFormatException nfe) {  
                // If the property cannot be parsed into an int, ignore it.  
            }  
        }  
        high = h;  
  
        // Load IntegerCache.archivedCache from archive, if possible  
        CDS.initializeFromArchive(IntegerCache.class);  
        int size = (high - low) + 1;  
  
        // Use the archived cache if it exists and is large enough  
        if (archivedCache == null || size > archivedCache.length) {  
            Integer[] c = new Integer[size];  
            int j = low;  
            for(int i = 0; i < c.length; i++) {  
                c[i] = new Integer(j++);  
            }  
            archivedCache = c;  
        }  
        cache = archivedCache;  
        // range [-128, 127] must be interned (JLS7 5.1.7)  
        assert IntegerCache.high >= 127;  
    }  
  
    private IntegerCache() {}  
}
```

![|502](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202311061428623.png)  
通过上述源码不难发现，在 `Integer` 的 `valueOf()` 方法中，如果数值在 [-128, 127] 范围的时候，就会去 `IntegerCache` 类中的 `cache` 数组中查找对应 `Integer` 对象的引用并返回；如果数值超过了规定的范围，则会新建一个 `Integer` 对象返回。  

### 分析

a1 和 a2 会先各自调用 `Integer` 中的 `valueOf()` 进行装箱操作，因为 a1 和 a2 的数值都在 [-128, 127] 之间，所以会从事先已经初始化好的 `cache` 数组中直接获取，由于变量 a1 和 a2 的数值一样，所以从数组中取出的是同一个 `Integer` 对象，所以它俩用 == 比较返回的结果是 true。  

---

a2 和 a3 也会先调用 `Integer` 中的 `valueOf()` 进行装箱操作，但是值却不在 [-128, 127] 之间，因此不是从数组中获取，而是各自实例化一个新的 `Integer` 对象返回，此时这两个对象的引用地址肯定不同，所以它俩用 == 比较返回的结果是 false。  
a3 和 a5 比较的时候

---

执行反编译之后可以发现，执行 a3 == a5 的时候，会先执行 a3 的自动拆箱操作，调用 `Integer` 的 `intValue()`，由于比较的是两个基本数据类型并且数值又相等，所以它俩用 == 比较返回的结果是 true。

```java
public class Test1 {  
  private static final Logger LOGGER = LoggerFactory.getLogger(Test1.class);  
    
  public static void main(String[] args) {  
    Integer a1 = Integer.valueOf(100);  
    Integer a2 = Integer.valueOf(100);  
    Integer a3 = Integer.valueOf(128);  
    Integer a4 = Integer.valueOf(128);  
    int a5 = 128;  
    LOGGER.debug("a1 == a2？{}", Boolean.valueOf((a1 == a2)));  
    LOGGER.debug("a3 == a4？{}", Boolean.valueOf((a3 == a4)));  
    LOGGER.debug("a3 == a5？{}", Boolean.valueOf((a3.intValue() == a5)));  
  }  
}
```

### 扩展

如果将上面的包装类型换成 `Double` 类型，

```java
public class Test1 {  
    private static final Logger LOGGER = LoggerFactory.getLogger(Test1.class);  
  
    public static void main(String[] args) {  
        Double a1 = 100.0;  
        Double a2 = 100.0;  
        Double a3 = 128.0;  
        Double a4 = 128.0;  
        int a5 = 128;  
        LOGGER.debug("a1 == a2？{}", a1 == a2);  
        LOGGER.debug("a3 == a4？{}", a3 == a4);  
        LOGGER.debug("a3 == a5？{}", a3 == a5);  
    }  
}
```

运行结果会是怎样呢？如下所示：

```
2022-10-14 16:48:56.684 DEBUG Test1:22 - a1 == a2？false
2022-10-14 16:48:56.685 DEBUG Test1:23 - a3 == a4？false
2022-10-14 16:48:56.685 DEBUG Test1:24 - a3 == a5？true
```

为什么会这样呢？咱们看下 `Double` 类中的 `valueOf()` 方法的源码就不难发现原因。

```java
public static Double valueOf(double d) {  
    return new Double(d);  
}
```

可以看到，`Double` 并没有 `Integer` 的缓存机制，而是直接返回了一个新的 `Double` 对象，所以如果用 == 比较的话，两个对象的引用地址不一样，自然返回 false。  
为什么 `Double` 类中的 `valueOf()` 方法会采用与 `Integer` 类中的 `valueOf()` 方法不同的实现呢？很简单，在某个范围内的整数型数值的个数是有限的，而浮点数却不是。  
查看了一下其他几种数据类型的 `valueOf()` 源码，其中，`Double` 和 `Float` 没有缓存机制，都是直接返回新的对象；`Integer`、`Short`、`Byte`、`Character` 都有缓存机制。  至于 `Boolean` 类型，它的 `valueOf()` 方法如下：

```java
public static Boolean valueOf(boolean b) {  
    return (b ? TRUE : FALSE);  
}
```

其中的 `TRUE` 和 `FALSE`，代表两个静态成员属性。

```java
/**
 * The {@code Boolean} object corresponding to the primitive
 * value {@code true}.
 */
public static final Boolean TRUE = new Boolean(true);

/**
 * The {@code Boolean} object corresponding to the primitive
 * value {@code false}.
 */
public static final Boolean FALSE = new Boolean(false);
```

聪明的小伙伴肯定已经知道，用 == 比较两个 `Boolean` 类型的对象时，只要值相等，那么返回的结果就是 true；值如果不相等的话，返回的结果就是 false。

## 重写 equals() 方法为什么必须重写 hashCode() 方法？

### equals() 方法

`Object` 类中的 `equals()` 方法的源码如下所示：

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

从以上源码中可以得知，如果一个类未重写 `equals()` 方法，那么本质上与 `==` 比较的效果一样，比较的是两个对象的内存地址是否相同。如果重写了 `equals()` 方法的话，则按照其需要进行比较，比如在 `Integer` 中重写了 `equals()` 方法，比较的就不再是两个对象的内存地址是否相等，而是比较的两个 `Integer` 对象的值是否相等。

Java 语言规范要求 `equals()` 方法具有以下特性：

- 自反性：对于任何非空引用值 `x`，`x.equals(x)` 应返回 true。
- 对称性：对于任何非空引用值 `x` 和 `y`，当且仅当 `x.equals(y)` 是 true 时，`y.equals(x)` 也是 true。
- 传递性：对于任何非空引用值 `x` 和 `y` 和 `z`，如果 `x.equals(y)` 是 true，同时 `y.equals(z)` 也是 true，那么 `x.equals(z)` 也是 true。
- 一致性：对于任何非空的引用值 `x` 和 `y` ，如果用于 `equals()` 方法比较的对象信息没有被修改的话，多次调用 `x.equals(y)` 一致返回 true 或者 false。
- 对于任何非空引用值 `x` 的 `x.equals(null)` 应返回 false。

在 `equals()` 方法上有这样一段注释：

> It is generally necessary to override the hashCode method whenever this method is overridden, so as to maintain the general contract for the hashCode method, which states that equal objects must have equal hash codes.

大致意思就是，重写 `equals()` 方法的时候需要重写 `hashCode()` 方法，这样做才能不违背 `hashCode()` 中“**相同对象必须有相同哈希值**”的约定。

### hashCode() 方法

`Object` 类中的 `hashCode()` 方法的源码如下所示：

```java
public native int hashCode();
```

该方法是一个本地方法，它的作用是返回一个 `int` 类型的哈希码，也成为散列码。这个值为任意整数，可能是正数也可能是负数。

#### hashCode() 方法的约定

关于 `hashCode()` 方法的约定大致有以下三条：

1. 如果用于 `equals()` 方法比较的对象信息没有被修改的话，那么在同一对象上多次调用 `hashCode()` 方法返回的哈希值必须相同。
2. **如果两个对象通过 `equals()` 方法比较是相等的，那么要求这两个对象调用 `hashCode()` 方法返回的哈希值也必须相等**。因此可以推导出，**当两个对象调用 `hashCode()` 方法返回的哈希值不相等时，那么这两个对象通过 `equals()` 方法比较也是不相等的**。
3. 如果两个对象通过 `equals()` 比较不相等，**不要求** 这两个对象调用 `hashCode()` 方法返回的哈希值必须不相等。但是咱们应该知道，对于不同对象生成不同的哈希值可能会提高哈希表的性能。

#### hashCode() 方法的应用场景

`HashSet` 集合要求添加的元素不能重复，试想一下，如果使用迭代的方式，依次对每个元素使用 `equals()` 方法比对两个元素是否相等的话，数据量小的话还可以接受，但是在数据量比较大的时候效率显然是非常低的，那么该怎么解决这个问题呢？先使用 `HashCode()` 方法返回的哈希值进行判断，相等的话再使用 `equals()` 方法来做进一步判断，这样可以大大减少调用 `equals()` 方法的次数。

`HashSet` 底层使用的其实是 `HashMap`，维护了一个 `HashMap` 类型的属性，添加元素都是存放在该 `HashMap` 中。

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    private transient HashMap<E,Object> map;
    
    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();

    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
}
```

熟悉 `HashMap` 的小伙伴可能知道，在 `HashMap` 添加的元素的时候，首先会通过一个 `hash()` 扰动函数 `(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)` 获取一个 `hash` 值，其中利用的就是对象 `hashCode()` 方法返回的哈希值，再通过扰动函数返回的 `hash` 值去定位元素在数组中的索引下标位置 `i = (n - 1) & hash`。如果索引下标位置已经存在元素，则说明两个对象的哈希值相等，这时才会调用 `equals()` 方法来进一步判断两个元素是否真的相等，即是否是同一个对象，如果 `equals()` 方法比较相等，则说明两个元素是同一个对象，用新元素的值替换掉已有元素的值；如果 `euqals()` 方法比较不相等的话，则说明两个元素并不是同一个对象，将新元素挂在已有元素的后面形成链表结构。如果索引下标位置没有元素，则说明新元素的哈希值与集合中其他元素的哈希值都不相等，直接添加即可。

### 重写 equals() 方法为什么必须重写 hashCode() 方法？

举一个例子来说明为什么重写 equals() 方法时必须重写 hashCode() 方法。

#### Set 集合的“异常”

当只重写 `equals()` 方法时，有趣的事情就发生了，如以下代码所示：

```java
public class Test3 {
    private static final Logger LOGGER = LoggerFactory.getLogger(Test3.class);

    public static void main(String[] args) {
        Set<Person> s = new HashSet<>();
        s.add(new Person("xiaorang", 18));
        s.add(new Person("xiaorang", 18));
        s.forEach(System.out::println);
    }
}

@Data
@AllArgsConstructor
class Person {
    private String name;
    private Integer age;

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Person person = (Person) o;
        return Objects.equals(name, person.name) && Objects.equals(age, person.age);
    }
}
```

运行结果如下所示：

```java
Person(name=xiaorang, age=18)
Person(name=xiaorang, age=18)
```

从上述结果看到，即使两个对象是相等的，`Set` 集合并没有去重，集合中存在重复的元素。这也就是只重写 `equals()` 方法不重写 `hashCode()` 方法的问题所在。

#### 解决“异常”

为了解决上面的问题，在重写 `equals()` 方法的时候也重写 `hashCode()` 方法，实现代码如下：

```java
@Data
@AllArgsConstructor
class Person {
    private String name;
    private Integer age;

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Person person = (Person) o;
        return Objects.equals(name, person.name) && Objects.equals(age, person.age);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

运行结果如下所示：

```java
Person(name=xiaorang, age=18)
```

从上述结果看到，两个相等的对象添加到 `Set` 集合中时只会保留一条记录。

#### 原因分析

在 `HashSet` 添加元素时首先会判断新元素的哈希值是否与集合中已有元素的哈希值是否相等，哈希值相等的话才会调用 `euqals()` 方法进一步判断两个对象是否真的相等。如果对象只重写了 `equals()` 方法，没有重写 `hashCode()` 方法的话，那么会使用 `Object` 中的 `hashCode()` 方法，其默认行为是对堆上的对象产生独特值，相同类型的两个对象无论如何都不会相等，所以结果为 false，`equals()` 方法也就不会被调用，直接添加元素到集合中，那么此时 `HashSet` 集合中就会出现两个相等的元素，违背了 `HashSet` 集合不能存在相同元素的前提条件，那么就会产生一些不可预知的错误。但是如果在重写 `equals()` 方法的时候也重写了 `hashCode()` 方法，那么在哈希值相等的情况下会进一步调用 `equals()` 方法判断两个对象是否真的相等，也就保证 `HashSet` 集合中不会存在重复的元素。

### 总结

`hashCode()` 和 `equals()` 两个方法是用来协同判断两个对象是否相等的，采用这种方式的原因是可以提高插入和查询的速度。如果只重写 `equals()` 方法，不重写 `hashCode()` 方法，就会导致在某些场景下出现异常，如导致两个相等的对象存储在 `HashSet` 集合中，所以为了保证程序的正常运行，在重写 `euqals()` 方法的时候，必须重写 `hashCode()` 方法。

## String s = new String("abc") 创建了几个对象？

🤓：创建了 **一个或者两个对象**。

> 参考链接：[流程图详解 new String("abc") 创建了几个字符串对象](https://www.bilibili.com/video/BV1tL4y1F7UH/?spm_id_from=333.337.search-card.all.click&vd_source=bf3d4320498e90d36e1361cc18b45e48)

### 查看字节码

#### javap 命令

```shell
javap -verbose Test2
```

运行结果如下所示：  

![image-20221123101334919](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202311061428539.png)  

#### jclassib Bytecode Viewer 插件

IDEA 安装 `jclassib Bytecode Viewer` 插件，使用 `shift + shift` 快捷键搜索关键字 `jclassib`，打开插件界面。如下所示：  
![|1132](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202311061428765.png)  
在插件界面中找到你此时正在运行的类中对应的方法即可。  

![image-20221123101633333](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202311061428493.png)

### 分析

#### 两个概念

引用变量和对象，对象一般通过 `new` 关键字在堆中创建，`s` 只是一个引用变量。

#### 字符串常量池

`JVM` 为了提升性能和减少内存开销，避免字符的重复创建，其维护了一块特殊的内存空间，即字符串常量池。当需要使用字符串时，先去字符串常量池中查看该字符串是否已经存在，如果存在，则可以直接使用，如果不存在，则初始化之后将该字符串放入字符串常量池中。

字符串常量池的位置也是随着 `JDK` 版本的不同而位置不同。在 `JDK6` 中，常量池的位置在永久代（方法区）中，此时常量池中存储的是 **对象**。在 `JDK7` 中，常量池的位置在堆中，此时，常量池中存储的就是 **引用**。在 `JDK8` 中，永久代被元空间所取代。

#### intern() 方法

`intern()` 方法的作用是将对应的字符串常量进行特殊处理，在 `JDK6` 之前和 `JDK7` 之后有不同的处理：

- 在 `JDK6` 中，`intern()` 方法的处理是先判断字符串常量是否存在于字符串常量池中，如果存在，则直接返回该常量；如果不存在，则将该字符串常量加入到该字符串常量池，即在字符串常量池中建立该字符串常量。
- 在 `JDK7` 中，`intern()` 方法的处理是先判断字符串常量是否存在于字符串常量池中，如果存在则返回字符串常量在字符串常量池中的引用；如果不存在，说明该字符串常量只存于堆中，则处理是把堆中该对象的引用保存到字符串常量池中，以后别人拿到的就是该字符串常量的引用，实际上对象还是存在于堆中的。

#### 为什么说创建了一个或两个对象？

`String s = new String("abc")` 会在堆中创建一个存储 "abc" 的字符串对象 (StringObject)，然后将其引用赋值给变量 `s` 保存。在构造方法中传递了一个字符串常量 "abc"，在首次构建这个对象的时候，`JVM` 会先检索字符串常量池中是否存在 "abc"，如果存在，则返回其对应的对象引用；如果不存在，则在堆中创建一个 "abc" 对象，并将其引用保存到字符串常量池中，然后返回。所以答案为创建一个或者两个对象。

### 升级加码

以下实例咱们暂且不考虑字符串常量池中是否已经存在对应字符串的问题，假设都不存在对应的字符串。

请问以下代码会创建几个对象？

```java
String s1 = "abc" + "def";
```

当一个字符串由多个字符串常量拼接成一个字符串时，它自己也肯定是字符串常量。字符串常量的 "+" 号连接，`JVM` 会在程序编译期将其优化为连接后的值。就上面的示例而言，在编译时已经被合并成了 "abcdef" 字符串，因此，只会创建 **一个对象**。并没有创建临时字符串对象 "abc" 和 "def"，这样减轻了 `GC` 的压力。

通过字节码也可以看出，通过指令 `ldc` 从字符串常量池中获取 "abcdef" 字符串，如下所示：

![image-20221123115039204](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202311061428826.png)

针对上面的问题，咱们再次升级一下，下面的代码会创建几个对象？

```java
String s2 = new String("abc") + new String("def");
```

![image-20221123121358288](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202311061428744.png)

从上面的字节码不难发现，创建了一个 `StringBuilder` 对象，一个 `new String("abc")` 对象，然后用指令 `ldc` 从字符串常量池中获取 "abc" 字符串，接着创建了一个 `new String("def")` 对象，然后接着用指令 `ldc` 从字符串常量池中获取 "def" 字符串，最后调用 `StringBuilder` 中的 `toString()` 方法创建一个新的字符串对象返回。所以这一行代码总共创建了 **6 个对象**。

查看 `StringBuilder` 中的 `toString()` 方法：

```java
@Override
public String toString() {
    // Create a copy, don't share the array
    return new String(value, 0, count);
}
```

此处虽然使用 `new` 关键字创建了一个字符串对象，但是并不会将 "abcdef" 字符串对象的引用存放到字符串常量池中，变量 `s2` 其实指向的是 `new String("abcdef")` 在堆空间中的引用地址。不信咱们可以验证一下，将代码变成如下形式：

```java
String s2 = new String("abc") + new String("def");
String s3 = "abcdef";
LOGGER.debug("s2 == s3？{}", s2 == s3);
```

如果说输出结果为 `true`，则说明会将 "abcdef" 字符串对象的引用存放到字符串常量池中，在创建字符串 `s3` 的时候直接就会从字符串常量池中获取，而不用重新在堆中创建一个新的对象，即 `s2` 和 `s3` 指向的都是堆中的同一个对象。反之，就会创建一个新的对象，自然 `s2` 的引用地址也就不会等于 `s3` 的引用地址。运行一下代码，可知，输出结果为 `false`，说明并不会将 "abcdef" 字符串对象的引用存放到字符串常量池中。

那么有没有办法让 `s2` 和 `s3` 指向的是同一个对象呢？办法是有的，还记得咱们上面提到的 `intern()` 方法，可以调用 `s2.intern()` 方法将 `s2` 的引用地址保存到字符串常量池中，之后在创建字符串 `s3` 的时候，发现字符串常量池中已经存在 "abcdef"，则不会再创建一个新的对象，而是直接使用该引用，即 `s2` 和 `s3` 会指向同一个对象，`s2 == s3` 的输出结果也就会变成 `true`。

## 为什么 String 是不可变类？以及为什么设计成不可变类？

### 什么是不可变类？

一个类的对象在通过构造方法创建之后如果状态不会再被改变，那么它就是一个不可变（immutable）类。它的所有成员变量的赋值仅在构造方法中完成，不会提供任何 `setter()` 方法供外部类去修改。

自从有了多线程，生产力就被无限放大了，所有的程序员都爱它，因为强大的硬件能力被充分地利用了。但与此同时，所有的程序员都对它心生忌惮，因为一不小心，多线程就会把对象的状态变得混乱不堪。

为了保证状态的原子性、可见性、有序性，程序员可以说是竭尽所能。其中，`synchronized`（同步）关键字是最简单最入门的一种解决方案。

假如说类是不可变的，那么对象的状态也就是不可变的。这样的话，每次修改对象的状态，就会产生一个新的对象供不同的线程使用，程序员就不必再担心并发问题了。

```java
public void testFinal() {
    String str = "abc";
    str = "def";
}
```

![image-20221124060547175](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202311061428477.png)

### String 为什么不可变？

1. `String` 类中的 `value` 属性是一个 `char` 类型的数组，被 `final` 修饰，保证该数组一旦被初始化之后，就不能再改变其引用了。
2. 成员变量的访问权限为 `private`，同时没有提供方法将字段暴露出来。
3. `String` 类中的方法不会去改动 `value` 属性的值，需要的话都是直接创建一个新的 `String` 对象并返回。
4. `String` 类被 `final` 修饰，表示该类不可被继承。

### String 为什么被设计成不可变类？

> 参考链接：[Why String is immutable in Java? (programcreek.com)](https://www.programcreek.com/2013/04/why-string-is-immutable-in-java/)

1. 字符串常量池的需要：字符串常量池是 Java 堆内存中一个特殊的存储区域，当创建一个 `String` 对象时，如果该字符串已经存在于池中，将返回现有字符串的引用，而不是创建一个新对象，这样的话可以节省内存空间，提高效率。

   ![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202311061428093.jpeg)

2. hashCode 的需要：因为字符串是不可变的，所以在它第一次调用 `hashCode()` 方法的时候，其 `hash` 值就已经被缓存了，因此非常适合作为哈希值（比如说作为 `HashMap` 的键），多次调用返回的是同一个值，来提高效率。

   ```java
   public int hashCode() {
       int h = hash;
       if (h == 0 && !hashIsZero) {
           h = isLatin1() ? StringLatin1.hashCode(value)
               : StringUTF16.hashCode(value);
           if (h == 0) {
               hashIsZero = true;
           } else {
               hash = h;
           }
       }
       return h;
   }
   ```

3. `String` 经常作为参数，`String` 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 `String` 是可变的，那么在网络连接过程中，`String` 被改变，改变 `String` 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。

   ```java
   boolean connect(string s){
       if (!isSecure(s)) { 
       	throw new SecurityException(); 
   	}
       //here will cause problem, if s is changed before this by using other references.    
       causeProblem(s);
   }
   ```

4. 线程安全：因为不可变的对象不能被改变，所以它们可以在多个线程之间自由共享。

## 如何自定义一个不可变类？

要自定义个不可变类，需要遵循以下 4 个原则:

1. 确保类被 `final` 所修饰，不允许被其他类继承。
2. 确保类中的所有成员变量是 `final` 的。
3. 不要对外提供方法去修改成员变量（如 `setter()` 方法）
4. 不可变对象的状态在创建之后就不能在改变，任何对它的改变都应该返回一个新的对象。如果 **类中包含可变类对象时**，你只需谨慎一点，**不要共享可变对象的引用** 即可，如果需要变化时，就 **返回原对象的一个拷贝**，别是对象本身。

## String，StringBuffer，StringBuilder 的区别是什么？

- 可变性
  - `String` 是一个不可变类，内部的 `value` 属性被 final 修饰。因此，每次对 `String` 的操作都会产生一个新的对象。
  - `StringBuffer` 和 `StringBuiler` 是可变类，它们在字符串变更的时候，不会产生新的对象。
- 线程安全：
  - `String` 是不可变类，所以它是线程安全的。
  - `StringBuffer` 是线程安全的，因为它的每个操作方法都加了 `synchronzied` 同步关键字。
  - `StringBuilder` 不是线程安全的。
- 性能：
  - `String` 的性能是最低的，因为它是不可变类，意味着在做字符串拼接和修改的时候，需要反复地重新创建新的对象和分配内存。
  - 其次是 `StringBuffer`，它要比 `String` 性能高，因为它的可变性使得字符串可以直接被修改。
  - 性能最高的是 `StringBuilder`，因为 `StringBuffer` 加了同步锁，而 `StringBuilder` 是非阻塞的。

最后再补充一下，`StringBuilder` 和 `StringBuffer` 都继承自 `AbstractStringBuilder` 抽象类。

## 为什么说 Java 中只有值传递？

开始之前，咱们先来搞懂下面这两个概念：

- 形参&实参
- 值传递&引用传递

###  形参&实参

方法的定义可能会用到 **参数**（有参的方法），参数在程序语言中分为：

- **实参**（实际参数，Arguments）：在调用有参方法时传入的参数，必须有确定的值。
- **形参**（形式参数，Parameters）：用于定义方法，接收实参，不需要有确定的值。

```java
public static void main(String[] args) {
    String hello = "Hello!";
    // hello为实参
	sayHello(hello);
}

// str为形参
public sayHello(String str) {
    System.out.println(str);
}
```

### 值传递&引用传递

程序设计语言将实参传递给方法的方式分为两种：

- **值传递**：方法接收的是 **实参值的拷贝**，会创建副本。
- **引用传递**：方法接收的是 **实参所引用的对象在堆中的地址**，不会创建副本，**对形参的修改将影响到实参**。

很多设计语言（如 `C`、`C++`）提供了两种参数传递的方法，不过，**在 Java 中只有值传递**。

### 基本类型是值传递的

Java 中的数据类型可以分为两种，一种是基本类型，一种是引用类型。我相信小伙伴还没看到这之前，就能够达成这样一个共识：基本类型是值传递的。这一点毫无疑问。

```java
public static void main(String[] args) {
    int num1 = 10;
    int num2 = 20;
    swap(num1, num2);
    System.out.println("num1 = " + num1);
    System.out.println("num2 = " + num2);
}

public static void swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
    System.out.println("a = " + a);
    System.out.println("b = " + b);
}
```

输出结果为：

```text
a = 20
b = 10
num1 = 10
num2 = 20
```

在 `swap()` 方法中，将 `a`、`b` 的值进行交换，并不会影响 `num1` 和 `num2`。因为，`a` 和 `b` 的值只是从 `num1` 和 `num2` 拷贝过来的。也就是说，`a` 和 `b` 相当于 `num1` 和 `num2` 的副本，副本的内容无论怎么修改，都不会影响到原件的值。

### 引用类型是值传递吗？

小伙伴们之所以不确定 Java 是值传递还是引用传递的，原因就出在这个引用类型上面。单从字面的意思上就容易混淆：引用类型不是引用传递吗？难道还是值传递吗？

```java
public static void main(String[] args) {
    int[] arr = { 1, 2, 3, 4, 5 };
    System.out.println(arr[0]);
    change(arr);
    System.out.println(arr[0]);
}

public static void change(int[] array) {
    // 将数组的第一个元素变为0
    array[0] = 0;
}
```

输出结果为：

```text
1
0
```

很多看了这个案例的小伙伴肯定会觉得 Java 对引用类型的参数采用的引用传递。实际上，并不是的，这里传递的还是值，只不过这个值是**实参的地址**！也就是说 `change()` 方法的 `array` 形参拷贝的是实参 `arr` 的地址，因此，**它和 `arr` 指向的是同一个数组对象**，这也就说明了为什么方法内部对形参的修改会影响到实参。

为了更强有力地反驳 Java 对引用类型的参数采用的不是引用传递，咱们再来看下下面这个案例！

```java
public class Person {
    private String name;
   // 省略构造函数、Getter&Setter方法
}

public static void main(String[] args) {
    Person xiaoZhang = new Person("小张");
    Person xiaoLi = new Person("小李");
    swap(xiaoZhang, xiaoLi);
    System.out.println("xiaoZhang:" + xiaoZhang.getName());
    System.out.println("xiaoLi:" + xiaoLi.getName());
}

public static void swap(Person person1, Person person2) {
    Person temp = person1;
    person1 = person2;
    person2 = temp;
    System.out.println("person1:" + person1.getName());
    System.out.println("person2:" + person2.getName());
}
```

输出结果为：

```text
person1:小李
person2:小张
xiaoZhang:小张
xiaoLi:小李
```

`swap()` 方法的参数 `person1` 和 `person2` 拷贝的只是实参 `xiaoZhang` 和 `xiaoLi` 的地址。因此， `person1` 和 `person2` 的互换只是拷贝的两个地址进行互换罢了，并不会影响到实参 `xiaoZhang` 和 `xiaoLi` 。

咱们通过一张图就能非常清晰这个过程。

![image-20221125061322645](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202311061428812.png)

在进行交换前，`person1` 和 `xiaoZhang` 指向的是同一个对象（`小张`），`person2` 和 `xiaoLi` 指向的是同一个对象（`小李`），交换之后，只是让 `person1` 指向 `小李` 对象，`person2` 指向 `小张` 对象，而 `xiaoZhang` 和 `xiaoLi` 指向的还是原来的对象，并未改变，所以输出的结果如上所示。

## 浅拷贝和深拷贝有什么区别？它的实现方式有哪些？

### 浅拷贝

#### 定义

浅拷贝（`Shadow Clone`），把原对象中值类型的成员变量为和引用类型的成员变量的引用地址复制给克隆对象。也就是说，**原对象和克隆对象中引用类型的成员变量指向的是堆内存中的同一个对象**。

![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202311061429904.png)

#### 实现方式

需要满足以下两点要求：

1. 类需要 **实现 `Cloneable` 接口**。如果没有实现 `Cloneable` 接口就调用 `clone()` 方法则会抛出 `CloneNotSupportedException` 异常。
2. **重写 `Object` 类中的 `clone()` 方法**。`Object` 类中的 `clone()` 方法源码如下所示：

   ```java
   protected native Object clone() throws CloneNotSupportedException;
   ```

   方法被 `protected` 所修饰，也就是说，如果一个类不显式地去重写 `clone()` 方法，其他类是无法调用该类实例的 `clone()` 方法的。
   
   在该方法的注释上可以发现 `clone()` 方法有以下约定：
   
   - 对于所有对象来说，`x.clone() != x` 应当返回 true，因为 **克隆对象与原对象不是同一个对象**。
   - 对于所有对象来说，`x.clone().getClass() == x.getClass()` 应当返回 true，因为 **克隆对象与原对象的类型是一样的**。
   - 对于所有对象来说，`x.clone().equals(x)` 应当返回 true，因为使用 `equals()` 方法进行比较时，**克隆对象与原对象的的值都是相同的**。

举个例子演示一下：

```java
public class Test4 {
    private static final Logger LOGGER = LoggerFactory.getLogger(Test4.class);

    public static void main(String[] args) throws CloneNotSupportedException {
        Address address = new Address(1, "changsha");
        People p1 = new People(1, "xiaorang", address);
        People p2 = p1.clone();

        LOGGER.info("p1 == p2 ? {}", p1 == p2);
        p1.getAddress().setCity("shenzhen");
        LOGGER.info("p2.getAddress().getCity() = {}", p2.getAddress().getCity());
    }
}

@Data
@AllArgsConstructor
class People implements Cloneable {
    private Integer id;
    private String name;
    private Address address;

    @Override
    protected People clone() throws CloneNotSupportedException {
        return (People) super.clone();
    }
}

@Data
@AllArgsConstructor
class Address {
    private Integer id;
    private String city;
}
```

运行结果如下所示：

```java
2022-11-30 12:17:21.310 [main] INFO  t.x.i.j.Test4:23 - p1 == p2 ? false
2022-11-30 12:17:21.317 [main] INFO  t.x.i.j.Test4:25 - p2.getAddress().getCity() = shenzhen
```

分析：`p1 != p2`，说明 **原对象和克隆对象不是同一个对象**。`p1.getAddress().setCity("shenzhen");` 当修改原型对象中引用类型的成员变量中的属性时，克隆对象中引用类型的成员变量中的属性也会跟着一样改变 `p2.getAddress().getCity() = shenzhen`，说明 **原对象和克隆对象中引用类型的成员变量指向的是堆内存中的同一个对象**。

### 深拷贝

#### 定义

深拷贝（`Deep Clone`），将原对象中所有类型（无论是值类型还是引用类型）的成员变量都复制一份给克隆对象。也就是说，**原对象和克隆对象中引用类型的成员变量指向的是堆内存中两个不同的对象**。

![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202311061429411.png)

#### 实现方式

##### 克隆嵌套

```java
@Data
@AllArgsConstructor
class People implements Cloneable {
    private Integer id;
    private String name;
    private Address address;

    @Override
    protected People clone() throws CloneNotSupportedException {
        People people = (People) super.clone();
        people.setAddress(this.address.clone());
        return people;
    }
}

@Data
@AllArgsConstructor
class Address implements Cloneable {
    private Integer id;
    private String city;

    @Override
    protected Address clone() throws CloneNotSupportedException {
        return (Address) super.clone();
    }
}
```

运行结果如下所示：

```java
2022-11-30 13:02:28.377 [main] INFO  t.x.i.j.Test4:23 - p1 == p2 ? false
2022-11-30 13:02:28.381 [main] INFO  t.x.i.j.Test4:25 - p2.getAddress().getCity() = changsha
```

分析：`p1 != p2`，说明 **原对象和克隆对象不是同一个对象**。`p1.getAddress().setCity("shenzhen");` 当修改原型对象中引用类型的成员变量中的属性时，克隆对象中引用类型的成员变量中的属性并不会跟着改变 `p2.getAddress().getCity() = changsha`，说明 **原对象和克隆对象中引用类型的成员变量指向的是堆内存中两个不同的对象**。

##### 通过构造方法

Effective Java 中推荐的方式。

```java
public class Test4 {
    private static final Logger LOGGER = LoggerFactory.getLogger(Test4.class);

    public static void main(String[] args) throws CloneNotSupportedException {
        Address address = new Address(1, "changsha");
        People p1 = new People(1, "xiaorang", address);
        People p2 = new People(p1);

        LOGGER.info("p1 == p2 ? {}", p1 == p2);
        p1.getAddress().setCity("shenzhen");
        LOGGER.info("p2.getAddress().getCity() = {}", p2.getAddress().getCity());
    }
}

@Data
@AllArgsConstructor
class People {
    private Integer id;
    private String name;
    private Address address;

    public People(People original) {
        this.id = original.id;
        this.name = original.name;
        this.address = new Address(original.address);
    }
}

@Data
@AllArgsConstructor
class Address {
    private Integer id;
    private String city;

    public Address(Address original) {
        this.id = original.id;
        this.city = original.city;
    }
}
```

##### 通过 JDK 自带的字节流

```java
public class Test4 {
    private static final Logger LOGGER = LoggerFactory.getLogger(Test4.class);

    public static void main(String[] args) {
        Address address = new Address(1, "changsha");
        People p1 = new People(1, "xiaorang", address);
        People p2 = StreamClone.clone(p1);

        LOGGER.info("p1 == p2 ? {}", p1 == p2);
        p1.getAddress().setCity("shenzhen");
        LOGGER.info("p2.getAddress().getCity() = {}", p2.getAddress().getCity());
    }
}

@Data
@AllArgsConstructor
class People implements Serializable {
    private Integer id;
    private String name;
    private Address address;
}

@Data
@AllArgsConstructor
class Address implements Serializable {
    private Integer id;
    private String city;
}
```

其中，`StreamClone` 为自定义的一个使用 JDK 自带的字节流实现的深度克隆工具类，代码如下：

```java
public class StreamClone {
    public static <T extends Serializable> T clone(T obj) {
        T cloneObj;
        try (
                ByteArrayOutputStream bos = new ByteArrayOutputStream();
                ObjectOutputStream oos = new ObjectOutputStream(bos);
        ) {
            oos.writeObject(obj);
            try (
                    ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
                    ObjectInputStream ois = new ObjectInputStream(bis);
            ) {
                cloneObj = (T) ois.readObject();
            } catch (ClassNotFoundException e) {
                throw new RuntimeException(e);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        return cloneObj;
    }
}
```

##### 使用 JSON 工具类

使用 JSON 工具类实现深拷贝，如 Gson 或者 FastJson 等。

引入 Gson 依赖：

```xml
<dependency>
   <groupId>com.google.code.gson</groupId>
   <artifactId>gson</artifactId>
   <version>2.8.7</version>
</dependency>
```

```java
public class Test4 {
    private static final Logger LOGGER = LoggerFactory.getLogger(Test4.class);

    public static void main(String[] args) {
        Address address = new Address(1, "changsha");
        People p1 = new People(1, "xiaorang", address);
        Gson gson = new Gson();
        People p2 = gson.fromJson(gson.toJson(p1), People.class);

        LOGGER.info("p1 == p2 ? {}", p1 == p2);
        p1.getAddress().setCity("shenzhen");
        LOGGER.info("p2.getAddress().getCity() = {}", p2.getAddress().getCity());
    }
}

@Data
@AllArgsConstructor
class People implements Serializable {
    private Integer id;
    private String name;
    private Address address;
}

@Data
@AllArgsConstructor
class Address implements Serializable {
    private Integer id;
    private String city;
}
```

