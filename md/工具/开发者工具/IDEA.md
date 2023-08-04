# IntelliJ IDEA

## 快捷键

1. 显示快速修复（可用于生成变量、重构代码）：Alt + Enter
2. 将当前行移动到上一行或者下一行：Shift + Alt + ⬆ / Shift + Alt + ⬇
3. 复制当前行：Ctrl + D
4. 删除当前行：Ctrl + Y
5. 继承体系结构：Ctrl + H
6. 查看类中的方法：Ctrl + F12
7. 全局搜索：Shift + Shift
8. 全局替换：Ctrl + Shift + R
9. 生成构造方法、getter\setter方法、重写方法：Alt + Insert
10. 重命名：Shift + F6
11. 代码格式化：Ctrl + Alt + L
12. 调试时计算变量/方法返回值：Alt + F8
13. 上一步：Ctrl + Alt + ⬅
14. 下一步：Ctrl + Alt + ➡
15. 转到方法/接口具体实现，可跳过接口：Ctrl + Alt + B
16. 前往方法/接口/变量的定义处：Ctrl + B，等价于 Ctrl + 鼠标左键
17. 最近的文件：Ctrl + E
18. 包围方式：Ctrl + Alt + T

## 配置

### 如何隐藏不想看到的文件夹或者文件？

File -> settings -> Editor -> File Types -> Ignore Files and Folders，配置不想看到的文件夹或者文件。<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308041718338.png)

如果配置无误还能看到文件夹或者文件的话，请确保没有勾选显示排除的文件选项，如果勾选了的话取消勾选即可！<br />![](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308041723979.png)

### 以 TODO 为例添加自定义的实时模板

设置 -> 编辑器 -> 实时模板，先创建一个属于自己的分组，组名可以根据自己的喜欢取；<br />![img](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308041706690.png)在创建的分组下新建一个模板，以 <span style="background-color: rgb(251, 228, 231);">TODO</span> 为例，步骤如下所示：

1. 先右键选中新创建的分组，然后点击 + 号选择实时模板；<br />![img](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308041707518.png)

2. 填写模板信息，缩写，描述信息，模板文本，以及编辑模板文本中涉及到的变量，最后选择该模板应用于 Java 环境；<br />![img](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308041707094.png)

3. 设置 -> 编辑器 -> TODO，先创建一个模式，按照如下方式进行填写；<br />![img](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308041707324.png)

4. 添加一个筛选器，名称任意，模式选择上面新建的，最后点击确定退出设置；<br />![img](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308041708909.png)

5. 打开 TODO 窗口，选中咱们新建的筛选器，这样就只会显示出咱们添加的 TODO，把其他人或其他引入的第三方项目中的 TODO 给过滤掉！<br />![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308041708006.png)