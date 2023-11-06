# 杂记

## foreach 循环

`foreach` 循环只不过是个**语法糖**，让咱们在遍历集合时代码更简洁明了。其实`foreach`的背后是`Iterator`**迭代器**，为什么这么说呢？请看如下示例代码：

```java
List<String> names = new ArrayList();
names.add("marry");
names.add("jack");
names.add("tom");
for (String name : names) {
    System.out.println(name);
}
```

反编译之后的代码如下所示：

```java
List<String> names = new ArrayList();
names.add("marry");
names.add("jack");
names.add("tom");
Iterator var5 = names.iterator();
while(var5.hasNext()) {
    String name = (String)var5.next();
    System.out.println(name);
}
```

可以看到 `foreach` 循环的底层使用的就是 `Iterator` 迭代器。

## reduce()

> `reduce()`方法对数组中的每个元素按序执行一个提供的 **reducer** 函数，每一次运行 **reducer** 会将先前元素的计算结果作为参数传入，最后将其结果汇总为单个返回值。

语法：[Array.prototype.reduce() - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)

```javascript
reduce()
reduce(callbackFn, initialValue)
```

参数：

- `callbackFn`：为数组中每个元素执行的函数。其返回值将作为下一次调用`callbackFn`函数时的`accumulator` 参数。对于最后一次调用，返回值将作为整个`reduce()`方法的返回值。该函数被调用时将传入以下参数：

- - `accumulator`：累加器，上一次调用`callbackFn`函数的结果。在第一次调用`callbackFn()`函数时，如果指定了`initialValue`参数的话，则`accumulator`为指定的初始值，否则的话为数组第一个元素`array[0]`的值；
  - `currentValue`：当前元素的值。在第一次调用`callbackFn()`函数时，如果指定了`initialValue`参数的话，则`currentValue`为数组第一个元素`array[0]`的值，否则的话为数组第二个元素`array[1]`的值；
  - `currentIndex`：当前元素在数组中的索引位置。在第一次调用`callbackFn()`函数时，如果指定了`initialValue`参数的话，则`currentIndex`的值为 0，否则的话为 1；
  - `array`：调用了`reduce()`方法的数组本身；

- `initialValue`可选：第一次调用`callbackFn`回调函数时初始化`accumulator`的值。如果指定了`initialValue`参数的话，则`callbackFn`函数从数组中的第一个元素`array[0]`的值作为`currentValue`开始执行。如果没有指定 `initialValue`参数的话，则`accumulator`初始化为数组中的第一个元素`array[0]`的值，并且`callbackFn`函数从数组中第二个元素`array[1]`的值作为`currentValue`开始执行。在这种情况下，如果数组为空（没有第一个元素`array[0]`的值可以作为`accumulator`返回），则会抛出错误；

几种常见的用法如下所示：<span style="font-size: 12px;">可直接拷贝到控制台中运行查看执行结果！</span>

1. 数组求和

   ```javascript
   const array = [15, 16, 17, 18, 19];
   
   const sum = array.reduce((accumulator, currentValue, currentIndex) => {
     const returns = accumulator + currentValue;
     console.log(`accumulator: ${accumulator}, currentValue: ${currentValue}, index: ${currentIndex}, returns: ${returns}`);
     return returns;
   });
   
   console.log(sum);
   ```

2. 分组

   ```javascript
   const people = [
     { name: "Alice", age: 21 },
     { name: "Max", age: 20 },
     { name: "Jane", age: 20 },
   ];
   
   const grouped = people.reduce((obj, current) => {
     let array = obj[current.age] || [];
     array.push(current);
     obj[current.age] = array;
     return obj;
   }, {});
   
   console.log(grouped);
   ```

3. 去重

   ```javascript
   const array = ["a", "b", "a", "b", "c", "e", "e", "c", "d", "d", "d", "d"];
   
   const arrayWithNoDuplicates = array.reduce((accumulator, current) => {
     if (!accumulator.includes(current)) {
       return [...accumulator, current];
     }
     return accumulator;
   }, []);
   
   console.log(arrayWithNoDuplicates);
   ```

## 利用 IntelliJ IDEA 生成可运行的 JAR 包

具体步骤如下所示：

1. 配置项目输出路径：确保你的项目的输出路径正确配置为 `out` 或 `target` 目录。这将是生成 JAR 文件的位置。<br />![image-20230905180232767](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309051802867.png)
   - 在项目窗口中，右键单击项目的根文件夹，然后选择 "Open Module Settings"（或者使用快捷键 `F4`）；
   - 在弹出的窗口中，选择 "Project" 标签；
   - 在 "Project compiler output" 部分，确保输出路径设置正确；

1. 编译项目：在 IntelliJ IDEA 中，使用以下步骤编译项目：

   - 点击菜单栏中的 "Build" 或 "Build Project"；
   - 或者使用快捷键，通常是 `Ctrl + F9`（Windows/Linux）或 `Cmd + F9`（macOS）；

   这将触发项目的编译，将编译输出放置在配置的输出路径中。

2. **创建可运行 JAR 文件**：在 IntelliJ IDEA 中，使用以下步骤创建可运行的 JAR 文件：

   - 打开 "File" 菜单，选择 "Project Structure"；
   - 在左侧面板中，选择 "Artifacts"；
   - 点击右侧的 "+" 图标，然后选择 "JAR" > "From modules with dependencies..."；<br />![image-20230905180944860](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309051809932.png)
   - 在弹出窗口中，选择项目的主类（包含 `public static void main(String[] args)` 方法的类）；<br />![image-20230905181217770](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309051812849.png)
   - 在 "JAR files from libraries" 部分，确保所有依赖项都被包含在 JAR 文件中（默认情况下应该已经勾选了）；<br />![image-20230905181511455](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309051815517.png)
   - 点击 "OK"；

3. **构建 JAR 文件**：在 IntelliJ IDEA 中，使用以下步骤构建 JAR 文件：

   - 点击菜单栏中的 "Build" > "Build Artifacts"；<br />![image-20230905181848955](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309051818002.png)
   - 选择你刚刚创建的 JAR 配置（通常是项目的名称）；<br />![image-20230905182006422](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309051820461.png)
   - 这将触发 JAR 文件的构建，并将生成的 JAR 文件放置在输出路径中；

4. 查找可执行 JAR 文件：生成的可执行 JAR 文件将被放置在输出路径中，可以在 `out` 或 `target` 目录下找到该 JAR 文件。<br />![image-20230905182143766](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202309051821822.png)

5. 运行可执行 JAR 文件：在终端中使用以下命令来运行生成的可执行 JAR 文件：

   ```bash
   java -jar your-project-name.jar
   ```

   替换 `your-project-name` 为实际的 JAR 文件名即可！

如此，即可在 IntelliJ IDEA 中直接生成和运行可执行的 JAR 文件，而无需使用其他 Maven 插件。

## mapper 文件不生效的原因以及解决方案

原因：默认情况下，在 `src/main/java` 目录下的 `xxxMapper.xml` 文件在编译时并没有被正确复制到类路径中。

解决方案：配置 pom.xml 文件中的 `<build>` 部分的 `resources` 配置，用于定义该如何处理项目中的资源文件，如下所示：

```xml
<build>
  <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <filtering>true</filtering>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

- 当设置 `<filtering>true</filtering>` 时，表示要启用资源过滤。这意味着 Maven 会在将资源复制到目标目录之前，解析并替换资源文件中的占位符或属性。这对于将配置信息动态注入到资源文件中非常有用，例如将版本号、数据库连接信息等替换到配置文件中。
- 当设置 `<filtering>false</filtering>` 时，表示禁用资源过滤。资源文件将以原样复制到目标目录，不会执行任何替换操作。

在 Maven 项目中，你可以在 `<build>` 部分的 `<resources>` 配置中设置 `<filtering>` 为 `true`，并在资源文件中使用 `${property}` 或 `@property@` 这样的占位符，然后在 Maven 的构建过程中，这些占位符会被替换为实际的属性值或配置信息。

举个栗子，在 `src/main/resources` 目录中有一个配置文件 `application.properties`，其中包含如下内容：

```properties
database.url=@db.url@
database.username=@db.username@
```

如果你在 `pom.xml` 中设置了 `<filtering>true>`，并在 Maven 的 `<properties>` 中定义了 `db.url` 和 `db.username` 的值，那么 Maven 会在构建过程中替换 `@db.url@` 和 `@db.username@` 为实际的值，然后再将这个文件复制到目标目录。这允许你根据不同环境的需要自定义配置文件。

## try-with-resources 自动关闭资源

`try-with-resources` 是 Java 7 引入的一个语言特性，用于自动关闭实现 `java.lang.AutoCloseable` 接口的资源，例如文件流、数据库连接、网络连接等。这个特性的目的是简化资源管理和减少资源泄漏的风险。使用 `try-with-resources` 可以更加优雅地处理资源的关闭，无需手动编写 `finally` 块来确保资源的释放。它有助于避免资源泄漏，提高代码的可读性和稳定性。

`try-with-resources` 还支持多个资源的同时管理，你可以在 `try` 后面的括号中列出多个资源，它们会**按照声明的顺序从后往前依次关闭资源**，确保资源的正确关闭和释放。举个栗子，如下所示：

```java
try (ResourceType1 resource1 = new ResourceType1();
     ResourceType2 resource2 = new ResourceType2()) {
    // 在这个块中使用资源1和资源2
    // 当块结束时，资源2会首先关闭，然后是资源1
} catch (Exception e) {
    // 异常处理
}
```

如果 `try-with-resources` 块中关闭资源的 `close()` 方法抛出异常，会发生如下两种情况：

- **关闭资源的异常（Suppressed Exception）**：当资源的 `close()` 方法抛出异常时，这个异常不会中断程序的执行。相反，它会被捕获，然后以一个"被抑制的异常"的形式存储在主异常中，主异常继续传播。
- **主异常**：主异常是在 `try-with-resources` 块内部的代码中抛出的异常，它是最初引发 `close()` 方法异常的原因。主异常会继续传播，可以由上层代码捕获和处理。

为了处理资源的 `close()` 方法抛出的异常，你可以使用以下方式：

```java
try (ResourceType resource = new ResourceType()) {
    // 使用资源
} catch (Exception e) {
    // 主异常处理
    Throwable[] suppressed = e.getSuppressed();
    for (Throwable t : suppressed) {
        // 处理被抑制的异常
    }
}
```

在上面的示例中，当主异常被捕获时，你可以使用 `getSuppressed()` 方法来获取被抑制的异常，然后处理它们。被抑制的异常通常代表了资源关闭时的问题，例如文件无法正常关闭或数据库连接出现问题。

需要注意的是，资源的 `close()` 方法抛出的异常通常不应该中断程序的执行，而应该被记录并处理，以确保资源得到适当的关闭和释放。处理被抑制的异常是一种优雅的方式来管理资源的关闭异常，同时保持程序的稳定性。
