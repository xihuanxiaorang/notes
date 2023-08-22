# ArrayList 源码分析

## 基本设计

```java
public class DynamicArray<E> {
    /**
     * 默认的初始容量
     */
    private static final int DEFAULT_CAPACITY = 10;
    /**
     * 用于空实例的共享空数组实例
     */
    private static final Object[] EMPTY_ELEMENT_DATA = {};

    /**
     * 用于默认大小的空实例的共享空数组实例。
     * 将它与 EMPTY_ELEMENT_DATA 区别开来，以了解添加第一个元素时要膨胀多少
     */
    private static final Object[] DEFAULT_CAPACITY_EMPTY_ELEMENT_DATA = {};
    /**
     * 数组缓冲区，数组列表的元素被存储在其中。数组列表的容量就是这个数组缓冲区的长度。
     * 当添加第一个元素时，如果 data == DEFAULT_CAPACITY_EMPTY_ELEMENT_DATA，则进行第一次扩容，容量大小变为 DEFAULT_CAPACITY。
     */
    private Object[] elementData;
    /**
     * 数组中元素的数量
     */
    private int size;

    /**
     * 无参构造函数
     */
    public DynamicArray() {
        elementData = DEFAULT_CAPACITY_EMPTY_ELEMENT_DATA;
    }

    /**
     * 构造具有指定初始容量的空数组
     *
     * @param initialCapacity 初始容量大小
     * @throws IllegalArgumentException 如果指定的初始容量为负数
     */
    public DynamicArray(int initialCapacity) {
        if (initialCapacity > 0) {
            elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            elementData = EMPTY_ELEMENT_DATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
        }
    }

    /**
     * 返回数组中的元素个数
     *
     * @return 数组中的元素个数
     */
    public int size() {
        return size;
    }

    /**
     * 判断数组是否为空
     *
     * @return {@code true} 如果数组为空，即没有任何元素
     */
    public boolean isEmpty() {
        return size == 0;
    }
}
```

1. 无参构造函数：elementData 被赋值为静态常量 DEFAULT_CAPACITY_EMPTY_ELEMENT_DATA。此时数组容量 capacity=0，元素数量 size=0；
2. 有参构造函数：构造具有指定初始容量的空数组
   1. 当指定的初始容量 initialCapacity>0 时，创建一个大小为 initialCapacity 的空数组，并将引用赋给 elementData；。此时数组容量 capacity=initialCapacity，元素数量 size=0;
   2. 当指定的初始容量 initialCapacity=0 时，elementData 被赋值为静态常量 EMPTY_ELEMENT_DATA。此时数组容量 capacity=0，元素数量 size=0；
   3. 当指定的初始容量 initialCapacity<0 时，抛出异常！

> [!ATTENTION|style:flat]
> DEFAULT_CAPACITY_EMPTY_ELEMENT_DATA 与 EMPTY_ELEMENT_DATA 虽然都是空数组，但是有所不同，主要体现在添加第一个元素后数组容量大小不一样。
> - 在无参构造函数中，elementData = DEFAULT_CAPACITY_EMPTY_ELEMENT_DATA，在添加第一个元素后，数组容量变为 DEFAULT_CAPACITY = 10；
> - 在有参构造参数中，如果指定的初始容量 initialCapacity=0，则 elementData = EMPTY_ELEMENT_DATA，在添加第一个元素后，数组容量变为 1。

## 添加元素

### 在数组尾部添加元素

```java
/**
 * 在数组的尾部添加元素
 *
 * @param e 待添加的元素
 */
public void add(E e) {
    if (size == elementData.length)
        elementData = grow();
    }
    elementData[size++] = e;
}
```

1. 先判断当前数组已满无法再添加元素，如果已满，则说明此时数组容量不足需要进行扩容操作
2. 在数组的最后位置（即 index=size 的位置）添加元素
3. 元素数量加一 size++。

### 在指定位置插入元素

```java
/**
 * 在数组中的指定位置添加元素
 *
 * @param index 索引位置
 * @param e     待添加的元素
 */
public void add(int index, E e) {
    rangeCheckForAdd(index);
    if (size == elementData.length) {
        elementData = grow();
    }
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    elementData[index] = e;
    size++;
}

/**
 * 检查添加元素时索引是否越界
 *
 * @param index 索引位置
 * @throws IndexOutOfBoundsException 如果索引越界的话
 */
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0) {
        throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);
    }
}
```

1. 先判断索引是否符合要求，如果 index < 0 || index > size，则抛出 **索引越界异常**！

2. 判断当前数组已满无法再添加元素，如果已满，则说明此时数组容量不足需要进行扩容操作

3. 在指定的位置添加元素，则需要**将指定位置到最后位置的元素都往后挪一格，也就是用前一个元素的值覆盖后一个元素的值，最后再覆盖指定位置上的值即可**
   > [!ATTENTION|style:flat]
   > 数组中的某个区段整体往后挪的时候，一定是后面的先动！否则，会出现数据覆盖的情况。

   对应到代码中则体现为**倒序遍历**，用前一个元素的值覆盖后一个元素的值。如下所示：
   
   ```java
   for (int i = size; i > index; i--) {
       elementData[i] = elementData[i - 1];
   }
   ```
   
   等价于如下代码，使用 System.arraycopy() 方法从原数组 index 位置开始，拷贝到原数组 index + 1 位置开始，拷贝的数量 = size - index。
   
   ```java
   System.arraycopy(data, index, data, index + 1, size - index);
   ```
   
   ![8b5f97f9-74c3-4fab-932d-53a40f67b26d](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307262304836.png)
   
4. 元素数量加一 size++

### 扩容机制

```java
/**
 * 扩容
 *
 * @return 扩容后的新数组
 */
private Object[] grow() {
    return grow(size + 1);
}

/**
 * 扩容
 *
 * @param minCapacity 需要的最小容量
 * @return 扩容后的新数组
 */
private Object[] grow(int minCapacity) {
    int oldCapacity = elementData.length;
    if (oldCapacity > 0 || elementData != DEFAULT_CAPACITY_EMPTY_ELEMENT_DATA) {
        int newCapacity = oldCapacity + Math.max(minCapacity - oldCapacity, oldCapacity >> 1);
        System.out.printf("grow: oldCapacity = %d => newCapacity = %d%n", oldCapacity, newCapacity);
        return elementData = Arrays.copyOf(elementData, newCapacity);
    } else {
        int newCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        System.out.printf("grow: oldCapacity = %d => newCapacity = %d%n", oldCapacity, newCapacity);
        return elementData = new Object[newCapacity];
    }
}
```

1. 在扩容函数中，参数 minCapacity 的意思为数组需要的最小容量 = size + 1，即当前容量（旧容量）+ 1；
2. 首先记录一下当前容量（旧容量）oldCapacity；
3. 如果当前容量>0 或者 elementData != DEFAULT_CAPACITY_EMPTY_ELEMENT_DATA，

   1. 前面说过如果使用无参构造函数进行初始化的话，则 elementData = DEFAULT_CAPACITY_EMPTY_ELEMENT_DATA，根据条件判断的话，当使用无参构造函数进行初始化的时，则会走 else 语句，创建一个容量大小为 DEFAULT_CAPACITY=10 的空数组赋值给 elementData。
   2. 其余的情况都归 if 语句处理，if 语句处理逻辑如下：

      1. **确定 newCapacity 新容量大小**。咱们知道 minCapacity = size + 1 = oldCapacity + 1，所以 minCapacity - oldCapacity 始终等于 1。oldCapacity >> 1 进行位运算，右移一位，即为 oldCapacity 的一半。

         1. 当 oldCapacity=0 的时候，即使用有参构造函数并指定初始容量为 0 进行初始化的情况，oldCapacity >> 1 = 0，则 newCapacity = 0 + Math.max(1, 0) = 1。
         2. 当 oldCapacity>0 的时候，newCapacity = oldCapacity + oldCapacity >> 1，即**新容量为旧容量的 1.5 倍**。

      2. **数据迁移**。新的容量大小确定之后，则需要进行数据迁移操作。Arrays.copyOf() 方法实际上就是创建一个新的数组，然后在方法的内部调用 System.arraycopy() 方法将原来数组中的数据迁移到新创建的数组上。其中 Arrays.copyOf() 方法中的大概逻辑如下所示：

         ```java
         Object[] newElementData = new Object[newCapacity];
         if (size >= 0) System.arraycopy(elementData, 0, newElementData, 0, size);
         elementData = newElementData;
         ```

         其中 System.arraycopy() 方法，它是一个本地方法，可以让你从原始数组 src 的某个位置 srcPos 开始，拷贝到目标数组 dest 从某个位置 desPos 开始往后的 size 个元素。<br />等价于：

         ```java
         for(int i = 0; i < size; i++) {
             newElementData[i] = elementData[i];
         }
         ```

> [!IMPORTANT|label:总结]
> - 如果当前容量等于 0，则分为两种情况。
>   - 当使用无参构造函数进行初始化时，当前容量=0，在添加第一个元素之后，新容量=10。
>   - 当使用有参构造函数并指定初始容量为 0 进行初始化时，当前容量=0，在添加第一个元素之后，新容量=1。
> - 如果当前容量大于 0，则新容量=旧容量 + 旧容量 >> 1，即新容量=旧容量的 1.5 倍。

## 移除元素

<span style="background-color: rgb(251, 228, 231);">TODO</span>
