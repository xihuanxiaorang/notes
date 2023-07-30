# HashMap 源码分析

## 基本介绍

1. HashMap 是一个散列表，它存储的内容是键值对（K-V）映射。允许存储 KEY 和 VALUE 为 NULL 的元素，但是只允许存在一个 KEY 为 NULL 的元素，如果再次存入 KEY 为 NULL 的元素时，会用新的 VALUE 值替换旧的 VALUE 值；

2. HashMap 继承自 AbstractMap 抽象类，实现了 Map、Serializable 和 Cloneable 接口，其继承关系体系图如下所示：

   ```plantuml
   @startuml
   
   class HashMap<K, V> {
   
   }
   
   class AbstractMap<K, V> {
   
   }
   
   interface Map<K, V> {
   
   }
   
   interface Serializable {
   
   }
   
   interface Cloneable {
   
   }
   
   Map <|.. HashMap
   Map <|.. AbstractMap
   Serializable <|.. HashMap
   Cloneable <|.. HashMap
   AbstractMap <|-- HashMap
   
   @enduml
   ```

3. HashMap 是无序的。因为该容器在扩容时会对元素重新放置，所以是不保证元素顺序的。

4. 非线程安全的。

## 数据结构

底层数据结构由 **数组+链表+红黑树** 组成。<br />在添加元素时，当 **<span style="background-color: rgb(251, 228, 231);">数组长度 >=64 & 链表长度 > 8</span>** 的时候，**链表转为红黑树**！如果数组长度<64（即使链表长度>8）时，则只会通过**扩容**操作将链表拆分成低位链表和高位链表来解决链表过长而影响查询效率变低的问题。<br />其底层数据结构类似下图所示：<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307251416567.jpeg)<br />其中 Node 和 TreeNode 节点定义如下所示：

```java
static class Node<K,V> implements Map.Entry<K,V> {
    // KEY的hash值
    final int hash;
    // KEY值
    final K key;
    // VALUE 值
    V value;
    // 当前节点的下一个节点
    Node<K,V> next;
}
```

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    // 当前节点的父节点
    TreeNode<K,V> parent;
    // 当前节点的左节点
    TreeNode<K,V> left;
    // 当前节点的右节点
    TreeNode<K,V> right;
    // 当前节点在双向链表中的上一个节点
    TreeNode<K,V> prev;
    // 当前节点的颜色(红/黑)
    boolean red;
}
```

## 重要属性

### 静态常量

1. DEFAULT_INITIAL_CAPACITY：默认初始化容量，如果使用构造器进行初始化时没有指定初始化容量参数时，则初始化容量等于该值 = 16；
2. DEFAULT_LOAD_FACTOR：默认负载因子，如果使用构造器进行初始化时没有指定负载因子参数时，则负载因子等于该默认值 = 0.75f；
   链表树化的两个条件，当 <span style="background-color: rgb(251, 228, 231);">数组长度>=64 & 链表长度>8</span> 时，链表 => 红黑树
3. TREEIFY_THRESHOLD：链表树化阈值 = 8，配合 MIN_TREEIFY_CAPACITY 一起使用；
4. MIN_TREEIFY_CAPACITY：最小树化容量 = 64；

### 成员变量

1. Node<K,V>[] table：Node<K, V> 类型的数组，用于存放 Node<K, V>（K-V 键值对） 元素。
2. size：元素个数。注意：不止数组长度大小的数量，因为数组某一个索引位置的元素可能已经形成链表或者红黑树。
3. modCount：记录结构修改次数。主要用于[快速失败(fail-fast)机制](https://www.yuque.com/xihuanxiaorang/java/uvgg4isy7ic8gph2?view=doc_embed) ，在集合的遍历过程中，一旦发现集合的结构被修改过了，就会立刻抛出一个 `CocurrentModificationException` 的异常，从而导致整个遍历过程是失败的。在`java.util` 包下的集合类都是属于快速失败机制的，如常用的 `ArrayList` 和 `HashMap` 等等，不能用于多线程环境下的并发修改。
4. threshold：扩容阈值。当元素个数大于该值时，则会触发扩容操作。计算公式：threshold = capacity _ loadFactor；扩容阈值 = 数组容量 _ 负载因子。
5. loadFactor：负载因子。负载因子决定了元素个数到了多少之后会进行扩容。在添加元素的时候，可能会出现即使你要添加的元素个数比数组容量大也不一定正正好的把数组占满，而是在某些下标位置出现大量的碰撞，只能在同一个位置用链表存放，那么这样就失去了哈希表的性能。所以，要选择在一个合理的大小下进行扩容，默认值 DEFAULT_LOAD_FACTOR = 0.75f 就是说当元素个数占了容量的 3/4 时赶紧扩容，减少 hash 碰撞。同时 0.75 是一个默认值，可以进行调整，如果你希望用更多的空间换时间，可以把负载因子调的小一些，减少碰撞。

那么，为什么会选择 0.75 呢？为什么不是其他数？背后有什么考虑？在源码中有这样一段描述：

```java
/**
As a general rule, the default load factor (.75) offers a good tradeoff between time and space costs.
Higher values decrease the space overhead but increase the lookup cost (reflected in most of the operations of the HashMap class, including get and put).
The expected number of entries in the map and its load factor should be taken into account when setting its initial capacity,
so as to minimize the number of rehash operations.
If the initial capacity is greater than the maximum number of entries divided by the load factor, no rehash operations will ever occur.
*/
```

翻译过来大致意思是：通常，默认负载因子（0.75）在时间和空间成本之间提供了一个很好的折中方案。较高的值虽然会减少空间开销，但会增加查找成本（在 HashMap 类的大多数操作中都得到体现，包括 get 和 put）。设置映射表的初始容量时，应考虑映射中的预期条目数及其负载因子，以最大程度地**减少重新哈希 rehash** 操作的次数。如果初始容量大于最大条目数除以负载因子，则将不会进行任何哈希操作。<br />通俗点来说就是，**当负载因子过大时，虽然会减少空间开销，但是会存在很高的哈希碰撞的概率，会大大降低查询和添加元素的速度**；如果**当负载因子过小时，那么将会频繁扩容，大大浪费内存空间**。所以，0.75 这个值是非常考究的。不推荐在使用构造器进行初始化的时候指定该参数，就用默认的负载因子（0.75f）即可。

## 主要操作

### 初始化

> [!IMPORTANT|label:延迟初始化]
>
> 在构造函数中并没有对 `table` 进行初始化，而是在第一次添加元素的时候才会对 `table` 进行初始化，这样设计主要是为了**延迟初始化，避免内存的浪费**，因为有可能初始化了之后并没有向其中添加元素，这样会造成不必要的浪费。

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

构造函数分析如下：

1. 对传入的两个参数：initialCapacity（初始化容量 ） 和 loadFactor（负载因子）进行校验，如果不满足要求就抛出异常！

2. tableSizeFor(initialCapacity)：为了保证数组的容量必须满足 2N 的要求，该方法的作用就是找到比传入的初始容量要大或者刚好相等的最小的 2N 数。假设当传入的初始容量 initialCapacity = 17，这个时候 `tableSizeFor()` 方法就应该返回一个接近 17 并比 17 要大的 2N 数，结果应该为 32，24=16 < 17，所以找到的是 25=32 > 17；

   ```java
   static final int tableSizeFor(int cap) {
       int n = cap - 1;
       n |= n >>> 1;
       n |= n >>> 2;
       n |= n >>> 4;
       n |= n >>> 8;
       n |= n >>> 16;
       return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
   }
   ```

   1.  `n = cap - 1`，为什么要执行 cap - 1 操作？因为当你传入的**初始容量刚好是 2 的幂次方数**，如果没有这行代码，执行完后面的几次无符号右移操作之后再执行 n + 1，最终返回的结果会是你传入的**初始容量的两倍**，即你传入的是 16，最后返回的结果为 32，所以需要先执行 cap - 1。

   2.  后面的步骤乍一看可能会有点晕 😵，怎么都是对 n 进行无符号右移 1、2、4、8、16 位后再与自身进行按位与操作，其实这样做的目的是为了把二进制的每一位都变为 1，当二进制的每一位都是 1 之后，就是一个标准的 2 的幂次方减 1，比如 15=0b1111 = 26 - 1，最后返回 n+1 即可获得一个比初始容量要大或者刚好相等的最小的 2N 数。如下图所示：

       <img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307251417551.jpeg" style="zoom: 50%;" />


3. 可以看到最后将通过 `tableSizeFor()` 方法确定出来的容量大小 capacity 赋值给了扩容阈值 `threshold`，这不对吧！正常的阈值计算公式不应该是 threshold = capacity \* loadFactor; 吗？为什么这里是直接将容量大小 capacity 赋值给扩容阈值 `threshold`？这样设计是为什么呢？如上面所说，为了实现**延迟初始化**，在构造函数中并没有对数组 table 进行初始化，那么计算出来的容量大小肯定要拿一个变量进行保存对吧，所以这里使用 `threshold`保存一下而已。在添加第一个元素时，在`put()`方法中会对数组 table 进行初始化，此时就会将保存在`threshold`中的容量大小取出当作数组的容量进行初始化，然后再利用正常的阈值计算公式去重新计算`threshold`的值即可。

### 添加元素 🎯

> [!CODE|label:添加元素 put 方法中用到的扰动函数 hash 和最重要的 putVal 方法]
>
> ```java
> static final int hash(Object key) {
>     int h;
> 	// 返回  key 的哈希值无符号右移 16 位后（高位全部补 0）与原哈希值本身进行按位异或运算 之后的值
>     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
> }
> 
> public V put(K key, V value) {
>     return putVal(hash(key), key, value, false, true);
> }
> 
> // 参数onlyIfAbsent表示是否覆盖原来的值，true表示不覆盖，false表示覆盖，默认为false
> final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
> 	// tab是存放节点的数组，n是数组长度,i是当前要插入的节点在数组中的下标，p是当前数组下标处的节点
>     Node<K,V>[] tab; Node<K,V> p; int n, i;
>     // tab指向全局的table数组，判断tab是否为null或者tab的长度为0
>     if ((tab = table) == null || (n = tab.length) == 0)
> 	    // 条件成立，表示table数组还没有被初始化，此时添加的元素为集合中的第一个元素，调用resize扩容方法初始化tab或者给tab扩容
>         n = (tab = resize()).length;
>     // hash & (n-1) => 当前要插入元素在数组中的数组下标index，p指向数组当前位置的元素，判断数组当前位置是否存在元素
>     if ((p = tab[i = (n - 1) & hash]) == null)
> 		// 条件成立，表明数组当前位置没有元素，则直接在当前位置插入元素
>         tab[i] = newNode(hash, key, value, null);
>     // 数组当前位置存在节点
>     else {
> 	    // e指向数组当前位置处已有的元素
>         Node<K,V> e; K k;
>         // 判断当前要插入元素的hash值和key值是否与当前位置处已有元素的hash值和key值相同
>         if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
>             // 条件成立，表示当前要插入的元素与当前位置处已有元素是同一个元素
>             e = p;
>         // 判断该元素是否为红黑树节点
>         else if (p instanceof TreeNode)
> 	        // 如果是的话，则使用红黑树的方法插入元素
>             e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
>         // 最后一种情况，在当前位置处的链表中插入元素
>         else {
>             // 遍历数组当前位置处的链表，此时p节点为数组当前位置处的第一个元素
>             for (int binCount = 0; ; ++binCount) {
> 	            // e指向p的下一个节点，在循环体的最后又将p指向e，如此这般可以用来遍历当前链表
>                 // 判断p节点的下一个节点是否为null
>                 if ((e = p.next) == null) {
> 	                // 条件成立，表明p节点没有下一个节点，即p节点为最后一个节点，则直接将要插入的节点挂在p节点后面
>                     p.next = newNode(hash, key, value, null);
>                     // 判断当前链表长度是否大于8个
>                     if (binCount >= TREEIFY_THRESHOLD - 1)
> 	                    // 在treeifyBin方法中还会继续判断数组长度是否小于64
> 	                    // 如果小于的话，则优先进行扩容而不是树化；如果数组长度大于等于64，才会将链表转化为红黑树结构
>                         treeifyBin(tab, hash);
>                     // 跳出循环，停止链表遍历
>                     break;
>                 }
>                 // 判断当前要插入元素的hash值和key值是否与正在遍历的链表中的e节点的hash值和key值相同
>                 if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
>                     // 条件成立，表明要插入的元素与e节点是同一个元素，则不执行插入操作，跳出循环，停止链表遍历
>                     break;
>                 p = e;
>             }
>         }
>         // 判断e是否为null，如果不为null，表明在当前链表中找到了与要插入元素的key相同的节点
>         if (e != null) {
> 	        // 取出e节点的value值赋值给oldValue
>             V oldValue = e.value;
>             // 判断onlyIfAbsent是否为false或者原来的值是否为null
>             if (!onlyIfAbsent || oldValue == null)
> 	            // 条件成立，表明onlyIfAbsent为false或者原来的值为null，则用要插入元素的value值覆盖掉原来的值
>                 e.value = value;
>             afterNodeAccess(e);
>             // 返回原来的值
>             return oldValue;
>         }
>     }
>     // modCount+1表示HashMap此时结构已经发生变化
>     ++modCount;
>     // 判断节点数量是否已经大于阈值，不管是在数组上添加一个元素还是在链表中添加一个元素，size都会加1
>     if (++size > threshold)
> 	    // 如果大于的话，则调用resize方法进行扩容操作
>         resize();
>     afterNodeInsertion(evict);
>     // 当前要插入的节点是一个新的节点，在原来的数组+链表+红黑树中不存在，当然原来的值也就不存在，返回null
>     return null;
> }
> ```

先上个流程图，对照着流程图一步一步进行分析：<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307251418779.jpeg)

#### 扰动函数

使用扰动函数 hash() 获取 KEY 的 hash 值；

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

在扰动函数中，哈希值等于将 KEY 的哈希值无符号右移 16 位后（高位全部补 0）与原哈希值本身进行按位异或运算。也就是让低 16 位与高 16 位进行异或，高 16 位保持不变 (与 0 异或都是自己本身)，让高位也得以参与散列运算。说白了，使用**扰动函数就是为了增加随机性，使得散列更加均匀，减少碰撞**。

#### 计算数组下标

数组下标 index = hash & (length - 1)。顺便说一下，这也正好解释了为什么数组长度必须是 2 的整数幂，因为这样（数组长度 - 1）正好相当于一个 “低位掩码”，使用 `&`操作之后的结果就是散列值的高位全部归零，只保留低位值，正好用来做数组下标访问。以初始长度 16 为例，16-1=15，其二进制表示就是`0000 0000 0000 0000 0000 0000 0000 1111`，和某个散列值 hash 做`&`操作如下所示，最后返回的结果就是截取了 hash 值的低四位。<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307251419590.png" alt="image.png" style="zoom: 50%;" />

#### 添加元素主流程

1. 获取数组在当前下标位置处的元素 p = tab[index]，判断元素 p 是否为 NULL？如果元素 p = NULL，则说明数组在当前下标位置处不存在元素，则直接将元素放在该位置即可，**元素个数 size + 1**；

2. 经过第一步，说明当前数组在当前下标位置处已经存在元素，此时会分为三种情况：

   1. 判断当前要插入的元素否与当前数组下标位置处的元素的 hash 值和 key 值相等？条件成立，表示当前要插入的元素与当前位置处已有元素是同一个元素，则让**节点 e 直接指向该元素**。

      > [!ATTENTION]
      >
      > 在使用自定义对象作为 KEY 时，需要重写对象的`hashCode()`和`equals()`方法。其中，`hashCode()`方法用于决定对象会被放到哪个 bucket 里，当多个对象的哈希值冲突时，`equals()`方法用于判断这些对象是否为同一个对象。

   2. 经过上一步，说明当前要插入的元素否与当前数组下标位置处的元素的 hash 值和 key 值并不相等，此时会将元素添加到链表的末尾或者红黑树的叶子节点。所以此时需要判断当前位置的元素是否为红黑树节点？条件成立的话，则走红黑树添加元素的逻辑，返回与要插入元素的 hash 值和 key 值都相等的树节点，让**节点 e 指向该树节点**。

   3. 经过上一步，说明当前数组下标位置处是链表结构，遍历该链表，寻找与当前要插入的元素的 hash 值和 key 值相等的节点，
      1. 如果找到了的话，则跳出循环，让**节点 e 指向在链表中找到的节点**；
      2. 如果找不到的话，就将元素添加到链表的末尾，然后判断是否进行树化，即判断链表的长度>8 & 数组的长度>=64？条件成立的话，则将链表转化为红黑树，最后无论是否树化，**元素个数 size + 1**；

3. 判断节点 e 是否不为 NULL？条件成立的话，会根据传入的 onlyIfAbsent 参数判断是否要覆盖原来的 VAULE 值？
   1. 如果 onlyIfAbsent 参数为 fasle 的话，则使用新的 VALUE 覆盖原来的 VALUE 值并返回原来的 VALUE 值；
   2. 如果 onlyIfAbsent 参数为 true 的话，则不会覆盖原来的值，而是直接将原来的 VALUE 值返回；
   
4. 元素个数加一之后，判断元素个数是否已经超过阈值？条件成立的话，说明需要调用 resize() 方法对数组进行扩容。

#### 数组初始化/扩容

> [!CODE|label:resize 方法]
>
> ```java
> final Node<K,V>[] resize() {
> 	// 将原来的table交由oldTab保存
>     Node<K,V>[] oldTab = table;
>     // 获取原来的数组长度赋值给oldCap
>     int oldCap = (oldTab == null) ? 0 : oldTab.length;
>     // 将原来的阈值大小threshold交由oldThr保存
>     int oldThr = threshold;
>     // 创建变量 newCap->新的数组大小，newThr->新的阈值大小
>     int newCap, newThr = 0;
>     // 如果原来的数组长度大于0
>     if (oldCap > 0) {
>         // 如果原来的数组长度已经大于等于最大的数组长度(1<<30)
>         if (oldCap >= MAXIMUM_CAPACITY) {
> 	        // 直接把阈值大小设置为最大整数2^31-1
>             threshold = Integer.MAX_VALUE;
>             // 返回原来的数组
>             return oldTab;
>         }
>         // oldCap<<1 => 将原来的数组大小*2
>         else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
>             // 将阈值也扩大1倍 <=> oldThr * 2
>             newThr = oldThr << 1;
>     }
>     // 当原来的阈值大于0但数组长度=0时，对应的情况就是使用带有指定数组长度和加载因子的构造器创建HashMap
>     else if (oldThr > 0)
> 	    // 新的数组长度等于原来的阈值大小
>         newCap = oldThr;
>     // 对应的情况是使用默认的构造器创建HashMap
>     else {
> 	    // 默认的数组大小为16
>         newCap = DEFAULT_INITIAL_CAPACITY;
>         // 默认的阈值大小 = 16 * 0.75(加载因子) = 12
>         newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
>     }
>     // 当新的阈值大小为0时，对应的情况就是使用带有指定数组长度和加载因子的构造器创建HashMap
>     if (newThr == 0) {
>         // 使用阀值公式计算新的阈值 = 新的容量 * 负载因子
>         float ft = (float)newCap * loadFactor;
>         newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
>                   (int)ft : Integer.MAX_VALUE);
>     }
>     // 将新的阈值大小赋值给threshold
>     threshold = newThr;
>     @SuppressWarnings({"rawtypes","unchecked"})
>     // 此时数组初始化或者扩容已经完成，接下来的工作就是进行数据拷贝，将原数组中的数据迁移到新数组中
>     // 创建新的数组，数组长度为上面计算出来的新的数组大小
>     Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
>     // table指向新数组
>     table = newTab;
>     if (oldTab != null) {
>         // 遍历原数组
>         for (int j = 0; j < oldCap; ++j) {
>             // 当前节点变量
>             Node<K,V> e;
>             // 判断数组中该位置的元素是否为null，即判断原数组中该位置是否存在元素
>             if ((e = oldTab[j]) != null) {
> 	            // 条件成立，说明原数组中该位置存在元素，则将原数组中该位置的元素清空
>                 oldTab[j] = null;
>                 // 判断当前元素是否有下一个节点，即判断该位置是否存在链表，如果为null，则表示当前位置只存在一个元素，所以只需要计算该元素位于新数组中的数组下标即可
>                 if (e.next == null)
> 	                // 那么再次使用 hash & (n - 1) 来计算当前元素位于新数组中的数组下标，在新数组的该位置存放当前节点
>                     newTab[e.hash & (newCap - 1)] = e;
>                 // 该节点存在下一个节点，所以有可能是链表或者红黑树结构
>                 else if (e instanceof TreeNode)
>                     // 判断当前节点是不是树节点，即判断当前位置是否已经转化为红黑树结构
>                     ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
>                 // 表明该节点是一个链表结构
>                 else {
> 	                // 新的位置只有两种可能：原位置 或者 原位置+原数组长度
> 	                // 把原链表拆成两个链表，然后再分别插入到新数组的两个位置上
> 	                // loXXX表示数组下标位置不变的链表，低位链表
>                     Node<K,V> loHead = null, loTail = null;
>                     // hiXXX表示原数组下标+原数组长度的位置处的链表，高位链表
>                     Node<K,V> hiHead = null, hiTail = null;
>                     // next节点，用于递归该位置的链表
>                     Node<K,V> next;
>                     do {
> 	                    // next节点指向当前节点的下一个节点
>                         next = e.next;
>                         // 使用hash & 原数组长度(如16 = 10000)，如果最高位为0，表示该节点位于原位置，即存在于低位链表中
>                         if ((e.hash & oldCap) == 0) {
>                             // 判断低位链表的尾节点是否为null，如果为null，表示当前低位链表还没有节点，则将低位链表的头节点指向当前节点
>                             if (loTail == null)
>                                 // 将低位链表的头节点指向当前节点，作为该链表的第一个节点
>                                 loHead = e;
>                             // 如果不为null，则让低位链表的尾节点的下一个节点指向当前节点，即将整个低位链表串起来
>                             else
>                                 loTail.next = e;
>                             // 最后让低位链表的尾节点指向当前节点，即移动低位链表的尾节点
>                             loTail = e;
>                         }
>                         else {
>                             if (hiTail == null)
>                                 hiHead = e;
>                             else
>                                 hiTail.next = e;
>                             hiTail = e;
>                         }
>                     } while ((e = next) != null); // 递归当前链表直至结束
>                     // 如果低位链表不为空
>                     if (loTail != null) {
> 	                    // 将低位链表的尾节点的下一个节点清空
>                         loTail.next = null;
>                         // 在新数组的原位置处放入低位链表
>                         newTab[j] = loHead;
>                     }
>                     // 如果高位链表不为空
>                     if (hiTail != null) {
> 	                    // 将高位链表的尾节点的下一个节点清空
>                         hiTail.next = null;
>                         // 在新数组的原位置+原来数组长度大小处放入高位链表
>                         newTab[j + oldCap] = hiHead;
>                     }
>                 }
>             }
>         }
>     }
>     // 返回新的数组
>     return newTab;
> }
> ```

resize() 方法有两个作用：初始化数组 table 和 对数组进行扩容（包括数据迁移操作）。

1. 在计算新数组的容量和扩容阈值之前，先用 oldTab、oldCap 和 oldThr 三个变量分别保存旧数组的引用、旧数组容量以及旧扩容阈值。

   1. 当旧数组容量大于 0 时，说明数组已经初始化过，此时走的是数组扩容流程，新数组容量 newCap = oldCap << 1 <=> newCap = oldCap _ 2，等于旧数组容量的两倍；新扩容阈值 newThr = oldThr << 1 <=> oldThr _ 2，等于旧扩容阈值的两倍；
   2. 当旧扩容阈值>0&旧数组容量=0 时，对应的情况就是使用指定数组初始容量和负载因子的构造器进行初始化。大家应该还记得在上面[初始化.构造函数分析.第 3 点](#初始化)的中曾提到过将使用`tableSizeFor()`方法获取到的容量大小保存在扩容阈值变量中，此时刚好从旧扩容阈值 oldThr 中取出数组应该初始化的容量大小赋值给 newCap；新扩容阈值 newThr = 新数组容量 newCap \* 指定的负载因子参数 loadFactor；
   3. 当旧扩容阈值=0&旧数组容量=0 时，对应的情况就是使用默认的无参构造进行初始化。新数组容量 newCap = DEFAULT_INITIAL_CAPACITY = 16，扩容阈值 newThr = DEFAULT_LOAD_FACTOR _ DEFAULT_INITIAL_CAPACITY = 0.75 _ 16 = 12；

2. 创建一个新的数组，数组长度为 newCap：Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];，让 table 指向新创建出来的数组，至此数组初始化或者扩容就已经完成，接下来的工作就是进行数据拷贝，将原数组中的数据迁移到新数组中。

3. 数据迁移：遍历原数组，将原数组中的数据拷贝到新数组中。当遍历到某一个索引位置时，判断原数组中该位置的元素是否为 NULL，即判断原数组中该位置是否存在元素？

   1. 条件成立的话，判断当前元素是否有下一个节点？有可能是链表或者红黑树结构；

      1. 条件成立的话，判断当前节点是不是树节点，即判断当前索引位置处的元素是否已经由链表转化为红黑树结构？

         1. 条件成立的话，则走红黑树的扩容流程；

         2. 条件不成立的话，说明当前索引位置处的元素是一个链表结构。在迁移数据时会把原链表拆成一个低位链表和一个高位链表，然后再分别插入到新数组的两个位置上，这两个位置分别为当前索引位置和当前索引位置+原数组长度。这个结论是怎么来的？举个栗子，如下图所示：图 a 和图 b 分别表示扩容前和扩容后 key1 和 key2 在数组中的索引位置。

            ![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307251424670.png)

            1. 扩容前 key1 和 key2 的 hash 值后四位相等，所以在旧数组中的索引位置相同，都等于 5，两个节点处在同一个链表上；

            2. 数组在扩容后，因为数组长度 n 变为原来的 2 倍，即 n - 1 就会比原来在高位处多 1 个 bit 位，因此新的数组下标就会发生这样的变化：

               ![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307251424202.png)<br />

               导致在计算节点位于新数组中的索引位置时，key1 的索引位置还是等于 5，而 key2 的索引位置 = 21 = 5 + 16，即原位置+原数组长度。所以在扩容时，只需要看 hash 值所对应二进制的最高位的位置是 0 还是 1，如果是 0 的话表示还是位于原位置，是 1 的话则表示当前节点所在新数组的位置=原位置 + 原来的数组长度。

      2. 条件不成立的话，则表示当前索引位置在原数组中只存在一个元素，所以只需要计算该元素位于新数组中的数组下标即可。

   2. 条件不成立的话，直接跳过进入下一次循环。

### 获取元素

<span style="background-color: rgb(251, 228, 231);">TODO</span>

### 删除元素

<span style="background-color: rgb(251, 228, 231);">TODO</span>
