# Spring-源码环境搭建

> [!TIP|label:开发环境]
> IntelliJ IDEA 2023.1 + JDK15 + spring-framework-5.3.x

## 下载 Spring 源码

下载 Spring 5.3.x 版本的源码压缩包 [GitHub - spring-projects/spring-framework at 5.3.x](https://github.com/spring-projects/spring-framework/tree/5.3.x)。
由于某些原因，可能有的小伙伴下载起来会非常的慢！所以，可以先将项目从 Github 中导入到 Gitee 中，然后直接从 Gitee 中下载，此时会发现下载速度非常快！🚀🚀🚀

## 配置阿里云镜像源

Spring 5.x 版本之后使用 Gradle 来构建编译，在编译过程中需要下载一堆的插件和 jar 包，众所周知，下载的资源都是从中央仓库下载，如果不使用国内镜像源来下载的话，速度会非常的慢！所以得先将镜像源切换到国内的阿里云镜像源 [阿里云仓库服务](https://developer.aliyun.com/mvn/guide)。

```groovy
maven { url "https://maven.aliyun.com/repository/public/" }
maven { url "https://maven.aliyun.com/repository/gradle-plugin/" }
maven { url 'https://maven.aliyun.com/repository/spring/' }
```

解压之后，打开项目根目录下的 build.gradle 和 settings.gradle 文件，使用 Ctrl + F 快捷键找到文件中 repositories 关键字所在的位置，将上面的内容复制粘贴到此处，如下所示：

```groovy
repositories {
    maven { url "https://maven.aliyun.com/repository/public/" }
    maven { url 'https://maven.aliyun.com/repository/spring/' }
    maven { url "https://maven.aliyun.com/repository/gradle-plugin/" }
    mavenCentral()
    maven { url "https://repo.spring.io/libs-spring-framework-build" }
}
```

```groovy
pluginManagement {
    repositories {
        maven { url "https://maven.aliyun.com/repository/public/" }
        maven { url 'https://maven.aliyun.com/repository/spring/' }
        maven { url "https://maven.aliyun.com/repository/gradle-plugin/" }
        mavenCentral()
        gradlePluginPortal()
        maven { url "https://repo.spring.io/release" }
    }
}
```

## 配置 Gradle 以及项目结构

<img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307232233212.png" alt="0e46deef-72da-4152-a2fa-9ce56c309eb2.png"  /> <br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307232233939.png" alt="image.png"  />

## 下载依赖并编译

导入 IDEA，开始下载依赖并编译，此过程可能需要几分钟，耐心等待...，等出现 BUILD SUCCESSFUL 字样就表示已经编译成功。
![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307232233315.png)<br />
编译成功之后，使用 gradle 测试一下。<br />
<img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307232233761.png" alt="d8d2155d-2b3a-4523-ae84-977eac479bda.png" style="zoom:80%;" /><br />
双击点击执行，在执行过程中发现报错，其实是因为 `isAccessible()` 方法被弃用了，需要把这个方法改成 `canAccess(null)` 方法。
![a895869a-a475-4c93-8ce6-dd27ac00bef2.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307232234632.png)
再测试一下，发现执行成功！最后会提示 'git' 相关错误，但是不影响使用。
![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307232234074.png)
上面关于 git 的错误的意思是当前不是一个 git 仓库。这个好办，咱们直接使用 `git init` 命令建一个 git 仓库就好，然后再使用 `git add .` 命令将文件添加到暂存区，最后使用 `git commit -m "fix: git command error"` 提交到本地仓库，有需要的小伙伴还可以在 Github 或者 Gitee 建立一个远程仓库，然后将代码推送到远程仓库中。

至此，Spring 源码环境搭建就圆满完成啦！🎉🎉🎉
