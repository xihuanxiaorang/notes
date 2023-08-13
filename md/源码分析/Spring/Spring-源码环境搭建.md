# Spring-源码环境搭建

> [!TIP|label:开发环境]
> IntelliJ IDEA 2023.1 + Gradle 7.5.1 + JDK11 + spring-framework-5.3.x

## 下载 Spring 源码

下载 Spring 5.3.x 版本的源码压缩包 [GitHub - spring-projects/spring-framework at 5.3.x](https://github.com/spring-projects/spring-framework/tree/5.3.x)。
由于某些原因，可能有的小伙伴下载起来会非常的慢！所以，可以先将项目从 Github 中导入到 Gitee 中，然后直接从 Gitee 中下载，此时会发现下载速度非常快！🚀🚀🚀

## 下载安装 Gradle

查看 spring 源码根目录下的`gradle`->`wrapper`->`gradle-wrapper.properties`配置文件，其中配置文件内容如下所示：

```properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-7.5.1-bin.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

可知，需要下载 Gradle 7.5.1 版本，下载地址为：[Gradle | Releases](https://gradle.org/releases/)<br />![image-20230813021242291](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308130212358.png)

压缩包下载完成之后，可以解压到任意目录下，本人解压到`D:\devsoft`，因此在配置环境变量时，Gradle 安装目录`GRADLE_HOME`为`D:\devsoft\gradle-7.5.1`，对于 Gradle 用户目录`GRADLE_USER_HOME`本人配置到与 maven 本地仓库地址相同的文件夹下`D:\devsoft\apache-maven-repository`，如下所示：<br /><img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308130222020.png" alt="image-20230813022205966"  />           <img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308130223909.png" alt="image-20230813022317855"  />

测试是否配置成功，打开终端，输入`gradle -v`命令，查看是否输出 Gradle 版本号信息，如下所示：<br />![image-20230813022708760](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308130227806.png)

## Gradle 配置阿里云镜像源

由于 Spring 5.x 版本之后使用 Gradle 来构建编译，在编译过程中需要下载一堆的插件和 jar 包，众所周知，下载的资源都是从中央仓库下载，如果不使用国内镜像源来下载的话，速度会非常的慢！所以需要先将镜像源切换到国内的阿里云镜像源 [阿里云仓库服务](https://developer.aliyun.com/mvn/guide)。

参考自官方教程 [Gradle Initialization Scripts](https://docs.gradle.org/current/userguide/init_scripts.html)，为了对**所有项目**生效，可以在如下任意一个位置创建`init.gradle`配置文件：

* [ ] Gradle 用户目录，即环境变量`GRADLE_USER_HOME`目录：`GRADLE_USER_HOME/`
* [ ] Gradle 用户目录，即环境变量`GRADLE_USER_HOME`目录下的`init.d`目录：`GRADLE_USER_HOME/init.d/`
* [x] Gradle 安装目录下的 `init.d` 目录：`GRADLE_HOME/init.d/`

其中`init.gradle`配置文件内容如下所示：

```groovy
allprojects{
    repositories {
		mavenLocal()
        maven { url "https://maven.aliyun.com/repository/public/" }
		maven { url 'https://maven.aliyun.com/repository/spring/' }
		maven { url "https://maven.aliyun.com/repository/gradle-plugin/" }
		maven { url "https://maven.aliyun.com/repository/spring-plugin/" }
		mavenCentral()
    }
	
	println "${it.name}: Aliyun maven mirror injected"
}
```

---

```groovy
maven { url "https://maven.aliyun.com/repository/public/" }
maven { url 'https://maven.aliyun.com/repository/spring/' }
maven { url "https://maven.aliyun.com/repository/gradle-plugin/" }
```

打开 spring 源码根目录下的 build.gradle 和 settings.gradle 文件，使用 Ctrl + F 快捷键找到文件中 repositories 关键字所在的位置，将以上内容分别复制粘贴到此处，如下所示：

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

## IDEA 配置

![image-20230813024919309](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308130249385.png)

![image-20230813025109279](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308130251347.png)

![image-20230813025207295](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308130252361.png)

## 下载依赖并编译

导入 IDEA，开始下载依赖并编译，此过程可能需要几分钟，耐心等待...，等出现 BUILD SUCCESSFUL 字样就表示已经编译成功。
![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307232233315.png)
编译成功之后，使用 gradle 测试一下。<br />
<img src="https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307232233761.png" alt="d8d2155d-2b3a-4523-ae84-977eac479bda.png"  /> <br />
双击点击执行，在执行过程中发现报错，其实是因为 `isAccessible()` 方法被弃用了，需要把这个方法改成 `canAccess(null)` 方法。
![a895869a-a475-4c93-8ce6-dd27ac00bef2.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307232234632.png)
再测试一下，发现执行成功！最后会提示 'git' 相关错误，但是不影响使用。
![image.png](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202307232234074.png)

## 提交和推送远程仓库

上面关于 git 的错误的意思是当前不是一个 git 仓库。这个好办，咱们直接使用 `git init` 命令建一个 git 仓库就好，然后再使用 `git add .` 命令将文件添加到暂存区，最后使用 `git commit -m "fix: git command error"` 提交到本地仓库，有需要的小伙伴还可以在 Github 或者 Gitee 建立一个远程仓库，然后将代码推送到远程仓库中。

> [!ATTENTION]
>
> 在提交的时候请先查看`.gitignore`文件中是否配置了忽略`target`文件夹，如果配置了的话，**请注释掉或者删除掉**！如下图所示：<br />![image-20230813142412696](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308131424793.png)
>
> 为什么呢？因为不这么做的话，`org.springframework.aop.framework.autoproxy.target`和`org.springframework.aop.target`这两个包下的类就不会被 git 管理，假如有一天你一不小心删除了本地的 spring 源码，想从远程仓库上再克隆下来，此时就会发现有多坑！！！在运行测试方法`compileTestJava`时报错：说找不到这两个包下的类！

至此，Spring 源码环境搭建就圆满完成啦！🎉🎉🎉
