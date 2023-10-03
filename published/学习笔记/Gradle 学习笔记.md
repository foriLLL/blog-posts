---
description: "在学习 Gradle 的过程中遇到了很多不理解的地方，以下是在初步学习中的学习笔记，会在之后逐渐完善勘误。"
time: 2021-11-13 13:26:19+08:00
---

在学习 Gradle 的过程中遇到了很多不理解的地方，以下是在初步学习中的学习笔记，许多内容基于 Gradle 8.2.1 官方文档，会在之后逐渐完善勘误。

## 什么是 Gradle

Gradle 是一种开源构建自动化工具，其设计足够灵活，几乎可以构建任何类型的软件。使用基于 Groovy 的特定领域语言（DSL）来声明项目设置，抛弃了基于 XML 的各种繁琐配置。面向 Java 应用为主，当前其支持的语言限于 Java，Scala 和 Groovy，计划未来支持更多语言。 

相比 Ant 的不规范，Maven 的配置复杂、生命周期限制严重，Gradle 既规范也更灵活，可以使用DSL（领域特定语言，如 Groovy 或 Kotlin）编写构建脚本，脚本更短小精悍。  

> Ant、Maven 有的 Gradle 也有，Gradle 有的它们不一定有；  
> Ant、Maven 能干的，Gradle 都能干，而且干得更好

如果你想要做的只是运行一个已经存在的 Gradle 构建，那么如果构建有一个 Gradle wrapper（可以通过构建根目录下的 `gradlew` 和/或 `gradlew.bat` 文件来识别），你就不需要安装 Gradle。您只需要确保您的系统满足 Gradle 的先决条件（8 及以上的 JDK）。

## 设计理念

### 高性能

Gradle 通过只运行那些因为输入或输出改变而需要运行的任务来避免不必要的工作。您还可以使用构建缓存来重用来自先前运行或甚至来自不同机器（具有共享构建缓存）的任务输出。  
Gradle 实施了许多其他优化，开发团队不断努力提高 Gradle 的性能。

### JVM 基础

Gradle 在 JVM 上运行，您必须安装 Java 开发工具包 (JDK) 才能使用它。这对于熟悉 Java 平台的用户来说是一个好处，因为您可以在构建逻辑中使用标准 Java API，例如自定义任务类型和插件。它还可以轻松地在不同平台上运行 Gradle。

### 约定

Gradle 借鉴了 Maven 的经验，通过实现约定，使通用类型的项目(如 Java 项目)易于构建。应用适当的插件，你就可以很容易地为许多项目创建一个轻量级的构建脚本。但这些约定并不会限制你：Gradle 允许你重写它们，添加你自己的任务，并对基于约定的构建进行许多其他定制。

### 可扩展性

你可以很容易地扩展 Gradle 来提供你自己的任务类型甚至构建模型。Android 的构建支持就是一个例子：它添加了许多新的构建概念，比如 flavors 和构建类型。

### IDE 支持

几个主要的 IDE 提供了与 Gradle 构建的交互，包括 Android Studio、IntelliJ IDEA、Eclipse、VSCode 和 NetBeans。Gradle 还可以生成将项目加载到 Visual Studio 所需的解决方案文件。

### Insight

构建扫描提供有关构建运行的广泛信息，您可以使用这些信息来识别构建问题。它们特别擅长帮助您识别与构建性能有关的问题。您还可以与他人共享 Build Scan，这在您需要咨询修复构建问题的建议时特别有用。


## Terminology

在深入了解 Gradle 的细节之前，我们需要了解以下术语。

### Projects

Project 是 Gradle 构建的东西。项目包含一个构建脚本，该脚本是位于项目根目录中的文件，通常名为 `build.gradle` 或 `build.gradle.kts`。构建脚本为该项目定义任务、依赖项、插件和其他配置。单个构建可以包含一个或多个项目，每个项目可以包含它们自己的子项目。

### Tasks

Task 包含执行某些工作(编译代码、运行测试或部署软件)的逻辑。在大多数用例中，您将使用现有的任务。Gradle 提供了实现许多常见构建系统需求的任务，比如可以运行测试的内置 Java Test 任务。插件提供了更多类型的任务。

Task 本身包括:
1. Action：做某事的工作片段，如复制文件或编译源代码
2. Input：操作使用或操作的值、文件和目录
3. Output：操作修改或生成的文件和目录

### Plugins

Plugin 允许你在构建中引入任务、文件和依赖项配置之外的新概念。例如，大多数语言插件都会将 source sets 的概念添加到构建中。

插件提供了跨多个项目重用逻辑和配置的方法。使用插件，您可以编写一次任务，并在多个构建中使用它。**或者您可以将常见的配置(如日志记录、依赖项和版本管理)存储在一个地方。** 这减少了构建脚本中的重复。适当地用插件对构建过程建模可以极大地提高易用性和效率。

Java plugin - tasks
<img alt="Java_plugin_tasks" src="https://img.foril.fun/Java_plugin_tasks.png" width=600px style="displat: block; margin:10px auto"/>

### Build Phases

Gradle 在构建生命周期的三个构建阶段评估和执行构建脚本:

#### Initialization

为构建设置环境，并确定哪些项目将参与其中。

#### Configuration

为构建构造和配置任务图。根据用户想要运行的任务，确定需要以何种顺序运行哪些任务。

#### Execution

运行在配置阶段结束时选择的任务。

### Builds

Build 是在 Gradle 项目中执行一组 Tasks。通过指定任务选择器，您可以通过命令行界面（CLI）或 IDE 运行 build。Gradle 配置构建并选择要运行的任务。Gradle根据请求的 tasks 及其依赖项运行最小的完整 tasks 集。


## The Gradle Wrapper

Gradle Wrapper 的存在就是一个项目自带的 Gradle，有了 Wrapper，你就不需要在你的运行环境中再单独下载一个 Gradle，而是直接使用 Wrapper 提供的脚本。如果你想直接运行一个 Gradle 构建的项目，你完全不需要在你的电脑上单独安装 Gradle，只需要 Wrapper 存在，运行任意 `gradlew` 脚本 Gradle 就会自动在 wrapper 文件夹下下载和开发者版本相同的 Gradle。

Wrapper 是一个调用已声明版本的 Gradle 的脚本，如果需要，可以提前下载该版本。因此，开发人员可以快速启动并运行 Gradle 项目，而不必遵循手动安装过程。

好处： 
* 在 **给定的 Gradle 版本** 上 **标准化** 一个项目，导致更可靠和健壮的构建。  
* 只需要改变 Wrapper 的定义就能为不同的用户和执行环境(例如 IDE 或持续集成服务器)提供一个新的 Gradle 版本。

### 如何获取 Wrapper

1. 建立了一个新的 Gradle 项目，并希望 [将 Wrapper 添加到项目](https://docs.gradle.org/current/userguide/gradle_wrapper.html#sec:adding_wrapper)。
2. 运行 [已提供 Wrapper 的项目](https://docs.gradle.org/current/userguide/gradle_wrapper.html#sec:using_wrapper)。
3. [升级项目的 Wrapper 版本](https://docs.gradle.org/current/userguide/gradle_wrapper.html#sec:upgrading_wrapper)


### Wrapper 目录

1. **gradle-wrapper.jar**  
Wrapper Jar 文件，包含下载 Gradle 发行版的代码。包括 Jar 在内的所有 Wrapper 文件都很小，可以加入 VCS。

2. **gradle-wrapper.properties**  
存储有关 Gradle 发行版的信息：
   * 托管 Gradle 发行版的服务器。
   * Gradle 发行版的类型。默认情况下，它是 `-bin` 发行版，只包含运行时，但不包含示例代码和文档。
   * 用于执行构建的 Gradle 版本。默认情况下，`wrapper` 任务选择用于生成 Wrapper 文件完全相同的 Gradle 版本。
   * 可选的，在下载 Gradle 发行版时使用的超时(以毫秒为单位)。
   * 可选的，一个用于设置分发 url 的验证布尔值。

<img alt="wrapper.properties" src="https://img.foril.fun/20230712133836.png" width=600px style="displat: block; margin:10px auto"/>

3. **gradlew, gradlew.bat**  
一个 shell 脚本和一个 Windows 批处理脚本，用于使用 Wrapper 执行构建。

### 使用 Wrapper

使用 Wrapper 几乎就像在安装了 Gradle 的情况下运行构建。根据操作系统的不同，你可以运行 `gradlew` 或 `gradlew.bat` 而不是 `gradle` 命令。


## 有关 build.gradle

### 调用 Groovy

在 `build.gradle` (*可以称为 build script，构建配置脚本*) 中可以调用 Groovy 的类库（也包含 Java 类库），下面是示例：  

```groovy
// build.gradle
task upper {
  String str = 'this is a simple test'
  println "原始值：" + str
  println "转大写后：" + str.toUpperCase()
}
```

执行命令 `gradle -q upper`

这个构建脚本定义了一个名为 upper 的 task，并向其添加了一个操作。当你运行 `gradle upper` 时，Gradle 会执行 upper task，而 upper task 又会执行你提供的动作。动作只是一个包含要执行的代码块。

> -q 的作用是静默输出，使输出更加清晰。  
> Groovy 兼容 Java 语法，我们可以通过在 task 中调用 Groovy 或 Java 的方法来完成想做的操作。  

与其直接操作脚本类路径，建议应用有自己的 classpath 的插件。对于自定义构建逻辑，建议使用自定义插件。

task 也可以依赖于其他的 task。

```groovy
tasks.register('hello') {
    doLast {
        println 'Hello world!'
    }
}
tasks.register('intro') {
    dependsOn tasks.hello
    doLast {
        println "I'm Gradle"
    }
}
```

> 许多自带插件预定义了很多你在构建项目中可能遇到的基本操作 task，比如 `java` 插件，自带了 `test` 、`classes` 等 task，可以直接调用完成测试或编译任务；  
> `application` 插件（应用 `application` 插件也会隐式地应用 `java` 插件）包含了 `run` 等 task 来运行指定的 main 类的 main 方法。  
> 具体见：  
> [The Java Plugin](https://docs.gradle.org/current/userguide/java_plugin.html#java_plugin)  
> [The Application Plugin](https://docs.gradle.org/current/userguide/application_plugin.html#header)


### 定义项目 version/group

在 Maven 中，可以明确定义项目版本，构建时会将这个版本包含在 war 或 jar 等制品的文件名称中，推送到 Maven 私服中也需要设置 `group` `artifactId` `version` 信息，那么 Gradle 中如何定义呢？

Gradle 中，对应 Maven 的三个参数，将 `artifactId` 变成了 `rootProject.name`，那么只需额外定义 `group` 与 `version`

```groovy
//build.gradle
group 'edu.ssdut'
version '0.0.1'
```

## 生成 Java Application

使用 Maven 时我们可以通过以下命令来创建一个简单的 Java CLI 项目。

```bash
mvn archetype:generate -DgroupId=xxx -DartifactId=yyy -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeCatalog=local -DinteractiveMode=false
```

使用 Gradle 创建一个遵循 Gradle 传统的 Java CLI 项目，我们可以使用 `gradle init` （built-in task，类似 npm init 作用）来初始化一个 Java 项目。

> Gradle comes with a built-in task, called init, that initializes a new Gradle project in an empty folder. The init task uses the (also built-in) wrapper task to create a **Gradle wrapper script**, gradlew.

下面我们了解一下生成的东西。

```
生成目录结构
├── gradle ①
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew ②
├── gradlew.bat ②
├── settings.gradle ③
└── app
    ├── build.gradle ④
    └── src
        ├── main
        │   └── java 
        │       └── demo
        │           └── App.java
        └── test
            └── java 
                └── demo
                    └── AppTest.java
```

① 为wrapper文件生成的文件夹  
② 启动脚本  
③ 定义 **build 名称** 和 **所包含子项目** 的设置文件
④ app 项目的 build script  

```groovy
//settings.gradle
rootProject.name = 'demo'
include('app')
```

`rootProject.name` 为构建分配一个名称，这将重写用其所在目录命名构建的默认行为。建议设置一个固定的名称，因为如果项目是共享的，文件夹可能会改变——例如作为 Git 仓库的根目录。  
`include("app")` 定义构建由一个名为 app 的子项目组成，该子项目包含实际的代码和构建逻辑。可以通过附加的 `include(…)` 语句添加更多子项目。

```groovy
// app/build.gradle
plugins {
    id 'application' ①
}

repositories {
    mavenCentral() ②
}

dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.7.2' ③

    implementation 'com.google.guava:guava:30.1.1-jre' ④
}

application {
    mainClass = 'demo.App' ⑤
}

tasks.named('test') {
    useJUnitPlatform()  ⑥
}
```  

① 应用 `application` 插件，增加对在 Java 中构建命令行应用程序的支持。  
② 使用 Maven 中央仓库来解析依赖关系。  
③ 使用 JUnit Jupiter 进行测试。   
④ 这个依赖被 `application` 使用。   
⑤ 定义应用程序的主类。  
⑥ 使用 JUnit 平台进行单元测试。

### 运行应用

由于 `application` 插件，你可以直接从命令行运行应用程序。`run` 任务告诉 Gradle 执行分配给 mainClass 属性的类中的 main 方法。  

```cmd
./gradlew run
或
gradle run（不使用Wrapper）
```

<img src="https://img.foril.fun/gradle_plugin_application.jpg" width='400px'/>

>当你第一次运行包装器脚本 `gradlew` 时，可能会有一个延迟，因为这个版本的 Gradle 被下载并存储在本地 `~/.gradle/wrapper/dists` 文件夹。

### 打包应用

`application` 插件也可以打包应用程序及其所有依赖项。该归档文件还将包含一个脚本，用于使用单个命令启动应用程序。
  
```bash
gradlew build
```

生成 `app/build/distributions/app.tar` 和 `app/build/distributions/app.zip`。

### 发布一个 Build Scan

Build Scan是用于开发和维护 Gradle 构建的重要工具。它为你提供了构建的详细信息，并为你识别构建环境、依赖或性能上存在的问题，同时可以帮你更全面地理解并提升 build过程，也便于与他人的合作

在 Gradle 构建运行时，Build Scans 插件会抓取数据，并将数据传送到 Build Scans 服务端。同时返回一个可被共享的链接，内部包含有用的构建信息。这些信息包含两大类：  
（1）环境信息，包括操作系统、Java 版本和时区；  
（2）构建相关信息，包含使用的插件、任务、测试以及依赖信息。  

要了解构建在幕后的更多信息，最好的方法就是发布构建扫描。要做到这一点，运行 Gradle 带 `--scan` 标志。

```bash 
gradle build --scan
```

---

## 依赖管理

Gradle 内置了对依赖项管理的支持，能够满足现代软件项目中遇到的典型场景。  
依赖关系通常以模块的形式出现。你需要告诉 Gradle 在哪里可以找到这些模块，以便在构建中使用它们。存储模块的位置称为存储库。通过为构建声明存储库，Gradle 将知道如何查找和检索模块。  
在运行时，如果需要操作特定的任务，Gradle 会定位声明的依赖项。依赖关系可能需要从远程存储库下载，从本地目录检索，或者需要在多项目设置中构建另一个项目。这个过程称为依赖关系解析。  

模块可以提供额外的元数据。元数据是更详细地 **描述模块** 的数据，例如在存储库中 **查找模块的坐标、关于项目或其作者的信息**。作为元数据的一部分，模块可以定义需要其他模块才能正常工作。例如，JUnit Jupiter 平台模块也需要平台公共模块。Gradle 会自动解析这些额外的模块，也就是所谓的 **传递依赖**。如果需要，您可以自定义行为来处理项目需求的传递依赖项。

### 声明存储库

Gradle 可以解析一个或多个基于 Maven、Ivy 或 flat directory 格式的存储库的依赖关系。  

Gradle 可以同时声明多个存储库按顺序检索，一般情况下只指定 `Maven Central` 仓库便可以解决大多数情况，具体信息可查看 [Gradle官方文档](https://docs.gradle.org/current/userguide/declaring_repositories.html)。

```groovy
repositories {
    mavenCentral()
    maven {
        url "https://repo.spring.io/release"
    }
    maven {
        url "https://maven.restlet.com"
    }
}
```

### Configuration

在学习声明依赖以前，我们需要了解依赖项 Configuration 的概念。  
Configuration 为 一个 Gradle 项目声明的每个依赖项都适用于一个特定的 **作用域**（对应 Maven 的 `scope`）。例如，一些依赖项应该用于编译源代码，而其他依赖项只需要在 runtime 可用。Gradle 在 Configuration 的帮助下表示依赖的范围。  

> 其实就是不同阶段会使用该阶段对应的 classpath

*一般插件都会提前添加好预声明的 Configuration。例如，Java 插件有多个 Configuration 来表示不同阶段的 classpath，如 源代码编译、test等。详见 [Java 插件](https://docs.gradle.org/current/userguide/java_plugin.html#sec:java_plugin_and_dependency_management) 中的示例。*  

#### Configuration 类型

在 dependency 前面可指明其依赖类型，可选。常用现成的类型：

|Configuration name|Role|Consumable?|Resolvable?|Description|
|--|---|--|---|--|
|api|Declaring API dependencies|no|no|This is where you declare dependencies which are transitively exported to consumers, for compile time and runtime.|
|implementation|Declaring implementation dependencies|no|no|This is where you declare dependencies which are purely internal and not meant to be exposed to consumers (they are still exposed to consumers at runtime).|
|compileOnly|Declaring compile only dependencies|no|no|This is where you declare dependencies which are required at compile time, but not at runtime. This typically includes dependencies which are shaded when found at runtime.|
|compileOnlyApi|Declaring compile only API dependencies|no|no|This is where you declare dependencies which are required at compile time by your module and consumers, but not at runtime. This typically includes dependencies which are shaded when found at runtime.|
|runtimeOnly|Declaring runtime dependencies|no|no|This is where you declare dependencies which are only required at runtime, and not at compile time.|
|testImplementation|Test dependencies|no|no|This is where you declare dependencies which are used to compile tests.|
|testCompileOnly|Declaring test compile only dependencies|no|no|This is where you declare dependencies which are only required at test compile time, but should not leak into the runtime. This typically includes dependencies which are shaded when found at runtime.|
|testRuntimeOnly|Declaring test runtime dependencies|no|no|This is where you declare dependencies which are only required at test runtime, and not at test compile time.|

#### Configuration 的继承和复合

Configuration 可以继承其他 Configuration 以形成继承层次结构。子配置继承任何其超配置声明的整个依赖集。

```groovy
configurations {
    smokeTest.extendsFrom testImplementation
}

dependencies {
    testImplementation 'junit:junit:4.13'
    smokeTest 'org.apache.httpcomponents:httpclient:4.5.5'
}
```

### 声明依赖

和 Maven 类似，依赖的坐标直接为：group、name、version。

此外，引入不同的插件会引入不同的 Configuration。

```groovy
dependencies {
    compile group: 'org.apache.commons ', name: 'commons-lang3', version: '3.8.1'
}
```

依赖坐标可以简写（用 : 分隔）：

```groovy
dependencies {
    // Use JUnit Jupiter for testing.
    testImplementation 'org.junit.jupiter:junit-jupiter:5.7.2'

    // This dependency is used by the application.
    implementation 'com.google.guava:guava:30.1.1-jre'
}
```

#### 项目的依赖类型

* 远程模块依赖：外部仓库  
* 项目依赖：一个项目依赖另一个项目。如：compile project(':projectB')  
* 文件依赖：依赖本地文件  

```groovy
dependencies {
    //将本地 module library 编译到项目中
    compile project(":mylibrary")
    //编译远程依赖
    compile 'com.android.support:appcompat-v7:23.4.0'
    //编译本地 jar 包
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```

## Source Set

假设您有一个遗留项目，该项目使用 `src` 目录作为生产代码（而不是 `src/main/java`）， `test` 目录作为测试代码（而不是 `src/test/java`）。这样非传统的目录结构不能工作，所以你需要告诉 Gradle 在哪里找到源文件。您可以通过 `source set` 配置来实现这一点。

每个 source set 定义其源代码所在的位置，以及资源和类文件的输出目录。您可以使用以下语法覆盖约定值:

```groovy
sourceSets {
    main {
         java {
            srcDirs = ['src']
         }
    }

    test {
        java {
            srcDirs = ['test']
        }
    }
}
```

现在 Gradle 只会直接在 `src` 中搜索源代码、`test` 搜索测试代码。如果您不想 **重写** 约定，而只是想 **添加** 一个额外的源目录（可能包含一些您希望单独保存的第三方源代码），该怎么办？语法类似:

```groovy
sourceSets {
    main {
        java {
            srcDir 'thirdParty/src/main/java'
        }
    }
}
```

一定需要注意的是，我们在这里使用了 `srcDir()` **方法** 来追加目录路径，而设置 `srcDirs` **属性** 将替换任何现有值。这是 Gradle 中的一个常见约定:设置一个属性替换值，而相应的方法附加值。

## 参考

https://docs.gradle.org/current/userguide/getting_started.html  
https://www.cnblogs.com/hellxz/p/helloworld-gradle.html
https://docs.gradle.org/current/samples/sample_building_java_applications.html
https://www.jianshu.com/p/646deb0010d1  
https://www.jianshu.com/p/642641dc7df3  
https://www.cnblogs.com/quanxi/p/10510142.html
