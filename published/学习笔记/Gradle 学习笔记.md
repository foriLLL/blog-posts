---
description: "在学习 Gradle 的过程中遇到了很多不理解的地方，以下是在初步学习中的学习笔记，会在之后逐渐完善勘误。"
time: 2021-11-13 13:26:19+08:00
---

在学习 Gradle 的过程中遇到了很多不理解的地方，以下是在初步学习中的学习笔记，会在之后逐渐完善勘误。

## 什么是 Gradle

Gradle 是一种开源构建自动化工具，其设计足够灵活，几乎可以构建任何类型的软件。使用基于Groovy的特定领域语言（DSL）来声明项目设置，抛弃了基于XML的各种繁琐配置。面向Java应用为主，当前其支持的语言限于Java，Scala和Groovy，计划未来支持更多语言。 

相比 Ant 的不规范，Maven 的配置复杂、生命周期限制严重，Gradle 既规范也更灵活，可以使用DSL (领域特定语言，如Groovy 或 Kotlin）编写构建脚本，脚本更短小精悍。  

> Ant、Maven 有的 Gradle也有，Gradle有的它们不一定有；  
> Ant、Maven能干的，Gradle 都能干，而且干得更好

如果你想要做的只是运行一个已经存在的Gradle构建，那么如果构建有一个Gradle wrapper(可以通过构建根目录下的gradlew 和/或 gradlew.bat文件来识别)，你就不需要安装Gradle。您只需要确保您的系统满足Gradle的先决条件（8及以上的JDK）。

## 特性
### 高性能
Gradle通过只运行那些因为输入或输出改变而需要运行的任务来避免不必要的工作。您还可以使用构建缓存来重用来自先前运行或甚至来自不同机器（具有共享构建缓存）的任务输出。  
Gradle 实施了许多其他优化，开发团队不断努力提高 Gradle 的性能。

### JVM基础
Gradle 在 JVM 上运行，您必须安装 Java 开发工具包 (JDK) 才能使用它。这对于熟悉 Java 平台的用户来说是一个好处，因为您可以在构建逻辑中使用标准 Java API，例如自定义任务类型和插件。它还可以轻松地在不同平台上运行 Gradle。

### 约定
Gradle借鉴了Maven的经验，通过实现约定，使通用类型的项目(如Java项目)易于构建。应用适当的插件，你就可以很容易地为许多项目创建一个轻量级的构建脚本。但这些约定并不会限制你:Gradle允许你重写它们，添加你自己的任务，并对基于约定的构建进行许多其他定制。

### 可扩展性
你可以很容易地扩展Gradle来提供你自己的任务类型甚至构建模型。Android的构建支持就是一个例子:它添加了许多新的构建概念，比如flavors和构建类型。

### Insight
构建扫描提供有关构建运行的广泛信息，您可以使用这些信息来识别构建问题。它们特别擅长帮助您识别与构建性能有关的问题。您还可以与他人共享Build Scan，这在您需要咨询修复构建问题的建议时特别有用。

> 推荐阅读：[5 things you need to know about Gradle](https://docs.gradle.org/current/userguide/what_is_gradle.html#five_things)

## The Gradle Wrapper
执行任何Gradle构建的推荐方法是在Gradle Wrapper的帮助下。

Wrapper是一个调用已声明版本的Gradle的脚本，如果需要，可以提前下载该版本。因此，开发人员可以快速启动并运行Gradle项目，而不必遵循手动安装过程。

好处： 
* 在给定的Gradle版本上标准化一个项目，导致更可靠和健壮的构建。  
* 只需要改变Wrapper的定义就能为不同的用户和执行环境(例如ide或持续集成服务器)提供一个新的Gradle版本。

一个Gradle项目通常为每个子项目提供一个settings.gradle(.kts)文件和一个build.gradle(.kts)文件。Wrapper文件就在gradle目录和项目的根目录中。下面的列表解释了它们的目的。  
和Wrappr有关的目录结构：
```
.
├── a-subproject
│   └── build.gradle
├── settings.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
└── gradlew.bat
```
**gradle-wrapper.jar**  
Wrapper JAR文件，包含下载Gradle发行版的代码。

**gradle-wrapper.properties**  
一个负责配置Wrapper运行时行为的属性文件，例如与此版本兼容的Gradle版本。请注意，更一般的设置，如配置Wrapper以使用代理，需要进入不同的文件。

**gradlew, gradlew.bat**  
一个shell脚本和一个Windows批处理脚本，用于使用Wrapper执行构建。

### 使用Wrapper
使用包装器看起来几乎就像在安装了Gradle的情况下运行构建。根据操作系统的不同，你可以运行 `gradlew` 或 `gradlew.bat` 而不是 `gradle` 命令。



## 使用Gradle构建项目
### projects 和 tasks
`projects` 和 `tasks` 是Gradle中最重要的两个概念。

任何一个**Gradle构建**都是由一个project或多个projects组成。每个project或许是一个 jar 包或者一个 web 应用，它也可以是一个由许多其他项目中产生的 jar 构成的 zip 压缩包。

每个project都由多个tasks组成。每个task都代表了构建执行过程中的一个原子性操作。如编译，打包，生成javadoc，发布到某个仓库等操作。

*简单来说，project 相当于一个构建工程的一个模块，而 task 是其中一个模块的一个操作。*

### 调用Groovy
在 `build.gradle` (*可以称为build script，构建配置脚本*) 中可以调用 Groovy 的类库（也包含 Java 类库），下面是示例：  
```gradle
// build.gradle
task upper {
  String str = 'this is a simple test'
  println "原始值：" + str
  println "转大写后：" + str.toUpperCase()
}
```
执行命令 gradle -q upper

这个构建脚本定义了一个名为hello的task，并向其添加了一个操作。当你运行gradle hello时，gradle会执行hello task，而hello task又会执行你提供的动作。动作只是一个包含要执行的代码块。

> -q 的作用是静默输出，使输出更加清晰。  
> Groovy 兼容 Java 语法，我们可以通过在 task 中调用 Groovy 或 Java 的方法来完成想做的操作。  

与其直接操作脚本类路径，建议应用有自己的classpath的插件。对于自定义构建逻辑，建议使用自定义插件。

task 也可以依赖于其他的 task。
```gradle
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
> 许多自带插件预定义了很多你在构建项目中可能遇到的基本操作task，比如 `java` 插件，自带了 `test` 、`classes` 等task，可以直接调用完成测试或编译任务；  
> `application` 插件（应用`application`插件也会隐式地应用`java`插件）包含了 `run` 等task来运行指定的main类的main方法。  
> 具体见：  
> [The Java Plugin](https://docs.gradle.org/current/userguide/java_plugin.html#java_plugin)  
> [The Application Plugin](https://docs.gradle.org/current/userguide/application_plugin.html#header)


### 定义项目version/group
在 Maven 中，可以明确定义项目版本，构建时会将这个版本包含在 war 或 jar 等制品的文件名称中，推送到 Maven 私服中也需要设置 `group` `artifactId` `version` 信息，那么 Gradle 中如何定义呢？

Gradle 中，对应 Maven 的三个参数，将 `artifactId` 变成了 `rootProject.name`，那么只需额外定义 `group` 与 `version`
```gradle
//build.gradle
group 'edu.ssdut'
version '0.0.1'
```

## Java 构建入门
### 生成Java项目
使用 Maven 时我们可以通过以下命令来创建一个简单的 Java 项目。
```bash
mvn archetype:generate -DgroupId=xxx -DartifactId=yyy -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeCatalog=local -DinteractiveMode=false
```
使用Gradle创建一个遵循Gradle传统的Java项目，我们可以使用 `gradle init` （built-in task，类似npm init作用）来初始化一个 Java 项目
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
③ 设置文件，以定义构建名称和子项目  
④ 构建app项目脚本  

```gradle
//settings.gradle
rootProject.name = 'demo'
include('app')
```
`rootProject.name` 为构建分配一个名称，这将重写用其所在目录命名构建的默认行为。建议设置一个固定的名称，因为如果项目是共享的，文件夹可能会改变——例如作为Git仓库的根目录。  
`include("app")` 定义构建由一个名为app的子项目组成，该子项目包含实际的代码和构建逻辑。可以通过附加的 `include(…)` 语句添加更多子项目。

```gradle
//app/build.gradle
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
    useJUnitPlatform() 
}
```  
① 应用`application`插件，增加对在Java中构建命令行应用程序的支持。  
② 使用Maven中央仓库来解析依赖关系。  
③ 使用JUnit Jupiter进行测试。   
④ 这个依赖被`application`使用。   
⑤ 定义应用程序的主类。

### 运行应用
由于`application`插件，你可以直接从命令行运行应用程序。`run` 任务告诉Gradle执行分配给mainClass属性的类中的main方法。  
```cmd
./gradlew run
或
gradle run（不使用Wrapper？）
```
<img src="https://img.foril.fun/gradle_plugin_application.jpg" width='400px'/>

>当你第一次运行包装器脚本`gradlew`时，可能会有一个延迟，因为这个版本的gradle被下载并存储在本地`~/.gradle/wrapper/dists`文件夹。

### 打包应用
`application` 插件也可以打包应用程序及其所有依赖项。该归档文件还将包含一个脚本，用于使用单个命令启动应用程序。  
```bash
gradlew build
```
生成`app/build/distributions/app.tar` 和 `app/build/distributions/app.zip`。

### 发布一个Build Scan
  构建扫描是用于开发和维护Gradle构建的重要工具。它为你提供了构建的详细信息，并为你识别构建环境、依赖或性能上存在的问题，同时可以帮你更全面地理解并提升构建过程，也便于与他人的合作。  
在Gradle构建运行时，Build Scans插件会抓取数据，并将数据传送到Build Scans服务端。同时返回一个可被共享的链接，内部包含有用的构建信息。这些信息包含两大类：  
（1）环境信息，包括操作系统、Java版本和时区；  
（2）构建相关信息，包含使用的插件、任务、测试以及依赖信息。  

要了解构建在幕后的更多信息，最好的方法就是发布构建扫描。要做到这一点，运行Gradle带`--scan`标志。
```bash 
gradle build --scan
```

---
## 依赖管理
Gradle内置了对依赖项管理的支持，能够满足现代软件项目中遇到的典型场景。  
依赖关系通常以模块的形式出现。你需要告诉Gradle在哪里可以找到这些模块，以便在构建中使用它们。存储模块的位置称为存储库。通过为构建声明存储库，Gradle将知道如何查找和检索模块。  
在运行时，如果需要操作特定的任务，Gradle会定位声明的依赖项。依赖关系可能需要从远程存储库下载，从本地目录检索，或者需要在多项目设置中构建另一个项目。这个过程称为依赖关系解析。  

模块可以提供额外的元数据。元数据是更详细地**描述模块**的数据，例如在存储库中**查找模块的坐标、关于项目或其作者的信息**。作为元数据的一部分，模块可以定义需要其他模块才能正常工作。例如，JUnit 5平台模块也需要平台公共模块。Gradle会自动解析这些额外的模块，也就是所谓的**传递依赖**。如果需要，您可以自定义行为来处理项目需求的传递依赖项。

### 声明存储库
Gradle可以解析一个或多个基于Maven、Ivy或flat directory格式的存储库的依赖关系。  

Gradle可以同时声明多个存储库按顺序检索，一般情况下只指定`Maven Central`仓库便可以解决大多数情况，具体信息可查看[Gradle官方文档](https://docs.gradle.org/current/userguide/declaring_repositories.html)。
```gradle
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
在学习声明依赖以前，我们需要了解依赖项Configuration的概念。  
Configuration为一个Gradle项目声明的每个依赖项都适用于一个特定的**作用域**（对应Maven的`scope`）。例如，一些依赖项应该用于编译源代码，而其他依赖项只需要在运行时可用。Gradle在Configuration的帮助下表示依赖的范围。  

*一般插件都会提前添加好预声明的Configuration。*  

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
Configuration可以继承其他Configuration以形成继承层次结构。子配置继承任何其超配置声明的整个依赖集。
```gradle
configurations {
    smokeTest.extendsFrom testImplementation
}

dependencies {
    testImplementation 'junit:junit:4.13'
    smokeTest 'org.apache.httpcomponents:httpclient:4.5.5'
}
```

### 声明依赖
和 Maven 类似，依赖的坐标直接为：group、name、version。此外，引入不同的插件会引入不同的依赖配置类别。
```gradle
dependencies {
    compile group: 'org.apache.commons ', name: 'commons-lang3', version: '3.8.1'
}
```
依赖坐标可以简写（用 : 分隔）：
```gradle 
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
```gradle
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
假设您有一个遗留项目，该项目使用`src`目录作为生产代码（而不是`src/main/java`），`test`目录作为测试代码（而不是`src/test/java`）。这样非传统的目录结构不能工作，所以你需要告诉Gradle在哪里找到源文件。您可以通过`source set`配置来实现这一点。

每个source set定义其源代码所在的位置，以及资源和类文件的输出目录。您可以使用以下语法覆盖约定值:
```gradle
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
现在Gradle只会直接在`src`中搜索源代码、`test`搜索测试代码。如果您不想**重写**约定，而只是想**添加**一个额外的源目录(可能包含一些您希望单独保存的第三方源代码)，该怎么办？语法类似:
```gradle
sourceSets {
    main {
        java {
            srcDir 'thirdParty/src/main/java'
        }
    }
}
```
一定需要注意的是，我们在这里使用了`srcDir()`**方法**来追加目录路径，而设置`srcDirs`**属性**将替换任何现有值。这是Gradle中的一个常见约定:设置一个属性替换值，而相应的方法附加值。




## 参考
https://docs.gradle.org/current/userguide/getting_started.html  
https://www.cnblogs.com/hellxz/p/helloworld-gradle.html
https://docs.gradle.org/current/samples/sample_building_java_applications.html
https://www.jianshu.com/p/646deb0010d1  
https://www.jianshu.com/p/642641dc7df3  
https://www.cnblogs.com/quanxi/p/10510142.html