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

## 开发过程中碰到的问题的解决方案🚀

### Git

#### fatal detected dubious ownership in repository at 'xxx'

问题描述：<br />![Snipaste_2023-08-13_19-58-37](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308132015079.png)

原因：由于 git 的新安全策略会导致使用 git 操作无所有权的仓库目录时报此错误，即当前 git 仓库的所有者与当前登陆用户不一致！

解决方案：更改当前 git 仓库文件夹的所有者！具体步骤如下所示：

1. 鼠标右键文件夹➡️属性➡️安全➡️高级 <br />![Snipaste_2023-08-13_19-59-28](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308132016267.png)
2. 更改所有者 <br />![Snipaste_2023-08-13_20-00-38](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308132017219.png)
3. 选择用户或组➡️高级 <br />![Snipaste_2023-08-13_20-01-17](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308132017292.png)
4. 立即查找➡️选择当前登陆用户为所有者 <br />![Snipaste_2023-08-13_20-05-28](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308132018349.png)
5. 应用于当前文件夹下的子文件夹和文件 <br />![Snipaste_2023-08-13_20-08-01](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308132021398.png) <br />![Snipaste_2023-08-13_20-09-02](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308132021006.png)
6. 查看是否已解决 <br />![Snipaste_2023-08-13_20-09-43](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308132023173.png)

