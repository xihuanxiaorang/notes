# JavaFX

## 环境搭建

以下内容参考自 [Getting Started with JavaFX (openjfx.cn)](https://openjfx.cn/openjfx-docs/#IDE-Intellij)，步骤如下所示：

1. 访问 [JavaFX - Gluon (gluonhq.com)](https://gluonhq.com/products/javafx/)，下载 JavaFX SDK，如下所示：<br />![image-20231008160645825](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310081606970.png)

   下载完成之后，将压缩包放到一个合适的位置然后解压缩，后面创建项目时会用到。

2. 如下图所示，JavaFX 21 版本要求最低的 JDK 版本为 17，因此咱们在创建项目时需要使用 JDK17；<br />![image-20231008161103461](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310081611526.png)

3. 使用 IDEA 创建一个 JavaFX 项目，项目名随意，JDK 则必须选择 17+ 版本；<br />![image-20231008161542945](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310081615003.png)

4. 创建启动类 `Main`，根据 JavaFx 规范，该类需要继承自 `javafx.application.Application`，此时发现根本找不到这个类；<br />![image-20231008161912874](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310081619929.png)

   此时就需要用到咱们上面下载解压好的 JavaFX SDK， 首先跳转到 `File -> Project Structure -> Libraries`，然后将 JavaFX SDK 作为库添加到项目中，选择 JavaFX SDK 中的 `lib` 文件夹；<br />![image-20231008163407369](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310081634432.png)

   添加完 JavaFX 类库之后，发现可以正常导入 `javafx.application.Application` 类啦！

5. 完善程序代码，如下所示：

   ```java
   public class Main extends Application {
   
       public static void main(String[] args) {
           launch(args);
       }
   
       @Override
       public void start(Stage stage) {
           Label welcomeText = new Label();
           Button button = new Button("Hello!");
           button.setOnAction((event) -> welcomeText.setText("Welcome to JavaFX Application!"));
           VBox root = new VBox(welcomeText, button);
           root.setAlignment(Pos.CENTER);
           Scene scene = new Scene(root, 320, 240);
           stage.setTitle("Hello!");
           stage.setScene(scene);
           stage.show();
       }
   }
   ```

6. 开始运行程序，发现居然报如下错误：<br />![image-20231008164633175](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310081646238.png)

   该如何解决呢？需要添加 VM 选项，点击 `Run -> Edit Configurations...` 添加 VM 选项： `--module-path "\path\to\javafx-sdk-21\lib" --add-modules javafx.controls,javafx.fxml`；其中的 `\path\to` 替换为 JavaFX SDK 解压缩后的位置，比如本人将 SDK 放在 `D:\devsoft\java` 目录下，那么需要添加的 VM 选项为 `--module-path "D:\devsoft\java\javafx-sdk-21\lib" --add-modules javafx.controls,javafx.fxml`，如下所示：<br />![image-20231008165730178](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310081657238.png)

   再次运行程序，出现如下窗口表示环境已经成功搭建！<br />![image-20231008170757606](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310081707640.png)

## 参考资料🎁

- 文档

  - [JavaFX中文官方网站 (openjfx.cn)](https://openjfx.cn/)

  - [JavaFX Documentation Project (fxdocs.github.io)](https://fxdocs.github.io/docs/html5/)

  - [javadoc Overview (JavaFX 21) (openjfx.io)](https://openjfx.io/javadoc/21/)

  - [JavaFX CSS Reference Guide (openjfx.io)](https://openjfx.io/javadoc/21/javafx.graphics/javafx/scene/doc-files/cssref.html)

  - [Scene Builder | JavaFX中文官方网站 (openjfx.cn)](https://openjfx.cn/scene-builder/)

  - [JavaFX Tutorial - javatpoint](https://www.javatpoint.com/javafx-tutorial)

- 视频教程
  - [Aimls的个人空间-Aimls个人主页-哔哩哔哩视频 (bilibili.com)](https://space.bilibili.com/5096022/channel/seriesdetail?sid=394169)
  - [LeeWyatt视频专辑-LeeWyatt视频合集-哔哩哔哩视频 (bilibili.com)](https://space.bilibili.com/397562730/channel/series)

