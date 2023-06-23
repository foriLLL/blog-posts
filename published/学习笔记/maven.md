最近由于考虑毕设可能会用到Java，决定大概复习一下，首先考虑复习一下Maven再去了解一下Gradle，本文是Maven的复习笔记。  
## Maven简介
### 简介
Maven主要服务于基于java平台的**项目构建**，**依赖管理**和**项目信息管理**。  

### 项目构建
除了编写源代码，我们每天有相当一部分时间花在了编译，运行单元测试，生成文档，打包和部署等繁琐的工作上，这就是构建。  
如果我们现在还手工这样做，成本太高，于是有人用软件的方法让这一系列工作完全自动化，使得软件的构建可以像全自动流水线一样，只需要一条简单的命令，所有繁琐的步骤都能够自动完成，很快就能得到最终结果。

### 项目构建工具
* Ant构建  
最早的构建工具，基于IDE, 大概是2000年有的，当时是最流行java构建工具，不过它的XML脚本编写格式让XML文件特别大。对工程构建过程中的过程控制特别好。

* Maven【JAVA】  
项目对象模型，通过其描述信息来管理项目的构建，报告和文档的软件项目管理工具。它填补了Ant缺点，Maven第一次支持了从网络上下载的功能，仍然采用xml作为配置文件格式。Maven专注的是依赖管理，使用Java编写。

* Gradle  
属于结合以上两个的优点，它继承了Ant的灵活和Maven的生命周期管理，它最后被google作为了Android御用管理工具。它最大的区别是不用XML作为配置文件格式，采用了DSL格式，使得脚本更加简洁。  


目前市面上Ant比较老, 所以一般是一些比较传统的软件企业公司使用, Maven使用Java编写, 是当下大多数互联网公司会使用的一个构建工具, 中文文档也比较齐全, gradle是用groovy编写, 是目前比较新型的构建工具，一些初创互联网公司会使用, 以后会有很大的使用空间。

### Maven的四大特性
#### 1. 依赖管理系统  
Maven为Java世界引入了一个新的依赖管理系统。jar 升级时修改配置文件即可。在Java世界中，可以用groupId、artifactId、version组成的Coordination（**坐标**）唯一标识一个依赖。  
任何基于Maven构建的项目自身也必须定义这三项属性，生成的包可以是jar包也可以是war包。一个典型的依赖引用如下所示：
```xml
<dependency>
 <groupId>javax.servlet</groupId>
 <artifactId>javax.servlet-api</artifactId>
 <version>3.1.0</version>
</dependency>
```
**groupId**  
定义当前Maven项目隶属的实际项目-公司名称。（jar包所在仓库路径）由于Maven中模块的概念，因此一个实际项目往往会被划分为很多模块。比如spring是一个实际项目，其对应的Maven模块会有很多，如spring-core,spring-webmvc等。

**artifactId**  
该元素定义实际项目中的一个Maven模块-项目名，推荐的做法是使用实际项目名称作为artifactId的前缀。 比如： spring-bean, spring-webmvc等。

**version**  
该元素定义Maven项目当前所处的版本。

#### 2. 多模块构建
在Maven中需要定义一个parent POM作为一组module的聚合POM。在该POM中可以使用标签来定义一组子模块。parent POM不会有什么实际构建产出。而parent POM中的build配置以及依赖配置都会自动继承给子module。

#### 3. 一致的项目结构
之前，Eclipse或IDEA创建的项目目录结构不一致，Maven制定了一套项目目录结构作为标准的Java项目结构，**解决不同IDE带来的文件目录不一致问题**。

#### 4. 一致的构建模型和插件机制
依赖和插件配置方法相同。（Maven插件可满足一些特定的运行方式，比如在服务器上运行web项目可用tomcat或jetty插件运行）
```xml
<plugin>
 <groupId>org.mortbay.jetty</groupId>
 <artifactId>maven-jetty-plugin</artifactId>
 <version>6.1.25</version>
 <configuration>
 <scanIntervalSeconds>10</scanIntervalSeconds>
 <contextPath>/test</contextPath>
 </configuration>
</plugin>
```

## Maven的安装配置和目录结构
### 安装配置
* 检查JDK的版本 JDK版本1.7及以上版本
* 下载Maven
* 配置Maven环境变量  
把Maven的根目录配置到系统环境变量`MAVEN_HOME`，将bin目录配置到path变量中。注：maven解压后存放的目录不要包含中文和空格  
* 检查Maven是否安装成功  
执行 mvn -v

（以下可选）
* 修改默认仓库位置  
打开maven目录 -> conf -> settings.xml 添加仓库位置配置
```xml
<localRepository>F:/m2/repository</localRepository>
<!--注：仓库位置改为自己本机的指定目录，"/"不要写反-->
```
* 更换阿里镜像,加快依赖下载
```xml
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

### 认识Maven目录结构
* src/main/java 项目的源代码所在的目录
* src/main/resources 项目的资源文件所在的目录（比如property文件）
* src/test/java 测试代码所在的目录（比如JUnit代码）
* src/test/resources 测试相关的资源文件所在的目录

#### 认识POM
```xml
<?xml version="1.0" encoding="utf-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.xxxx</groupId>
    <artifactId>maven01</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>maven01</name>
    <url>http://maven.apache.org</url>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```
#### cmd 下编译并运行
cmd下面，进入项目的根目录  
1. 编译java文件  
```
mvn compile
```
2. 执行main 方法  
```
mvn exec:java -Dexec.mainClass="com.xxxx.demo.Hello"
```
 
 ***

## Maven命令
Maven命令格式
```
mvn [plugin-name]:[goal-name]
```
### 常用命令
|命令 | 描述|
|:--:|:--|
|mvn –version | 显示版本信息|
|mvn clean | 清理项目生产的临时文件,一般是模块下的target目录|
|mvn compile | 编译源代码，一般编译模块下的src/main/java目录|
|mvn package | 项目打包工具,会在模块下的target目录生成jar或war等文件|
|mvn test | 测试命令,或执行src/test/java/下junit的测试用例.|
|mvn install | 将打包的jar/war文件复制到你的本地仓库中,供其他模块使用|
|mvn deploy | 将打包的文件发布到远程参考，提供其他人员进行下载依赖|
|mvn site | 生成项目相关信息的网站|
|mvn eclipse:eclipse | 将项目转化为Eclipse项目|
|mvn dependency:tree | 打印出项目的整个依赖树|
|mvn archetype:generate | 创建Maven的普通java项目|
|mvn tomcat7:run | 在tomcat容器中运行web应用|
|mvn jetty:run | 调用 Jetty 插件的 Run 目标在 Jetty Servlet 容器中|启动 web 应用

### -D传入属性参数
例如：
```sh
mvn package -Dmaven.test.skip=true
```
以 -D 开头，将 maven.test.skip 的值设为 true ,就是告诉 maven 打包的时候跳过单元测试。同理，mvn deploy-Dmaven.test.skip=true 代表部署项目并跳过单元测试。

### -P 使用指定的Profile配置
比如项目开发需要有多个环境，一般为开发，测试，预发，正式4个环境，在pom.xml中的配置如下：
```xml
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <env>dev</env>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <id>qa</id>
        <properties>
            <env>qa</env>
        </properties>
    </profile>
    <profile>
        <id>pre</id>
        <properties>
            <env>pre</env>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <env>prod</env>
        </properties>
    </profile>
</profiles>
......
<build>
    <filters>
        <filter>config/${env}.properties</filter>
    </filters>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    ......
</build>
```
profiles 定义了各个环境的变量 id ， filters 中定义了变量配置文件的地址，其中地址中的环境变量就是上面 profile 中定义的值， resources 中是定义哪些目录下的文件会被配置文件中定义的变量替换。

通过maven可以实现按不同环境进行打包部署，例如：
```
mvn package -Pdev -Dmaven.test.skip=true
```
表示打包本地环境，并跳过单元测试
## IDEA编辑器集成Maven环境
注意修改IDEA设置的Maven，选择要使用的Maven，配置对应的配置文件和仓库位置即可。
## Maven项目的创建
创建Maven根据选择的模板（或不选择）会自动创建Mavcen目录结构，建好Maven项目后，可在IDEA右上角运行配置中加入 `compile` 以及 `package` 等需要的运行方式，可随时切换运行。也可以在右侧Maven工具栏中从生命周期随时运行需要的命令。
## Maven仓库的基本概念
当第一次运行Maven命令的时候，你需要Internet链接，因为它需要从网上下载一些文件。那么它从哪里下载呢？它是从Maven默认的远程库下载的。这个远程仓库有Maven的核心插件和可供下载的 jar 文件。  

对于Maven来说， 仓库只分为两类： **本地仓库**和**远程仓库**。  
当Maven根据坐标寻找构件的时候，它首先会查看本地仓库，如果本地仓库存在，则直接使用；如果本地没有，Maven就会去远程仓库查找，发现需要的构件之后，下载到本地仓库再使用。如果本地仓库和远程仓库都没有，Maven就会报错。  
远程仓库分为三种： **中央仓库**，**私服**，**其他公共库**。  
中央仓库是默认配置下，Maven下载jar包的地方。私服是另一种特殊的远程仓库，为了节省带宽和时间，应该在局域网内架设一个私有的仓库服务器，用其代理所有外部的远程仓库。**内部的项目还能部署到私服上供其他项目使用。**  

***一般来说，在 Maven 项目目录下，没有诸如 lib/ 这样用来存放依赖文件的目录***。 当Maven在执行编译
或测试时，如果需要使用依赖文件，它总是基于坐标使用本地仓库的依赖文件。  
默认情况下，每个用户在自己的用户目录下都有一个路径名为.m2/repository/的仓库目录。 有时候，因为某些原因（比如c盘空间不足）,需要修改本地仓库目录地址。
对于仓库路径的修改，可以通过maven配置文件conf目录下settings.xml来指定仓库路径。  
其他公共库例如阿里云仓库。

## Maven环境下构建多模块项目
一个项目下有多个模块（module）时，若A模块依赖B模块，使用 `mvn install` 将模块B复制到本地仓库中，在模块A的pom.xml中加入模块B，便可调用模块B的内容。

## Maven的打包操作
对于企业级项目，无论是进行本地测试，还是测试环境测试以及最终的项目上线，都会涉及项目的打包操作，对于每个环境下项目打包时，对应的项目所有要的配置资源就会有所区别，实现打包的方式有
很多种，可以通过ant或者通过idea自带的打包功能实现项目打包，但当项目很大并且需要的外界配置很多时，此时打包的配置就会异常复杂，对于maven项目，我们可以用过pom.xml 配置的方式来实现打包时的环境选择，相比较其他形式打包工具，通过maven 只需要通过简单的配置，就可以轻松完成不同环境先项目的整体打包。

假如一个项目在不同的环境下需要不同的配置文件。

![](https://img.foril.fun/maven_switch.jpg?imageView2/1/w/400)


可在pom.xml中配置Profiles，应对不同的环境。
```xml
<!-- 打包环境配置 开发环境 测试环境 正式环境 -->
<profiles>
    <profile>
        <id>dev</id>
        <!-- properties可作为变量被外部标签获取 -->
        <properties>
            <env>dev</env>
        </properties>
        <!-- 未指定环境时，默认打包dev环境 -->
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>

    <profile>
        <id>test</id>
        <properties>
            <env>test</env>
        </properties>
    </profile>

    <profile>
        <id>product</id>
        <properties>
            <env>product</env>
        </properties>
    </profile>
</profiles>
 ```
然后在pom.xml中对资源文件进行配置。
```xml
<!-- 对于项目资源文件的配置放在build中 -->
<resources>
    <resource>
        <!-- ${}可以获取properties变量 -->
        <directory>src/main/resources/${env}</directory>
    </resource>
    <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*.xml</include>
            <include>**/*.properties</include>
            <include>**/*.tld</include>
        </includes>
        <filtering>false</filtering>
    </resource>
</resources>
```
然后就可以通过Maven命令进行打包，`-P`参数选择指定的Profile。  
如：
```sh
mvn package -Pdev -Dmaven.test.skip=true
```

## Maven依赖的基本概念
### 依赖范围
首先需要知道，Maven在编译项目主代码的时候需要使用一套classpath。  
比如：编译项目代码的时候需要用到spring-core，该文件以依赖的方式被引入到classpath中。  
其次，Maven在执行测试的时候会使用另外一套classpath。如：junit。  
最后在实际运行项目时，⼜会使用一套classpath，spring-core需要在该classpath中，而junit不需要。  

那么依赖范围就是用来控制依赖与这三种classpath(编译classpath，测试classpath，运行时classpath)的关系，具体内容见下文。

### 依赖的基本配置
根元素project下的dependencies可以包含多个dependence元素，以声明多个依赖。每个依赖都应该包含以下元素：
1. groupId, artifactId, version：**依赖的基本坐标**， 对于任何一个依赖来说，基本坐标是最重要的，Maven根据坐标才能找到需要的依赖。  
2. Type：依赖的类型，大部分情况下不需要声明。默认值为jar。  
3. Scope：依赖范围（compile,test,provided,runtime,system）  
**compile**：**编译依赖范围**。  
如果没有指定，就会**默认**使用该依赖范围。使用此依赖范围的Maven依赖，对于 *编译、测试、运行* 三种classpath都有效。  
**test**: **测试依赖范围**。  
使用此依赖范围的Maven依赖，只对于 *测试* classpath有效，在编译主代码或者运行项目的使用时将无法使用此类依赖。**典型的例子就是JUnit**，它只有在编译测试代码及运行测试的时候才需要。  
**provided**: **已提供依赖范围**。  
使用此依赖范围的Maven依赖，对于 *编译和测试* classpath有效，但在运行时无效。**典型的例子是servlet-api**，编译和测试项目的时候需要该依赖，但在运行项目的时候，由于容器已经提供，就不需要Maven重复地引入一遍(如：servlet-api)。  
**runtime**: **运行时依赖范围**。  
使用此依赖范围的Maven依赖，对于 *测试和运行* classpath有效，但在编译主代码时无效。**典型的例子是JDBC驱动实现**，项目主代码的编译只需要JDK提供的JDBC接⼝，只有在执行测试或者运行项目的时候才需要实现上述接⼝的具体JDBC驱动。  
**system**: **系统依赖范围**。
该依赖与三种classpath的关系，和provided依赖范围完全一致。但是，使用system范围依赖时必须通过systemPath元素显式地指定依赖文件的路径。由于此类依赖不是通过Maven仓库解析的，而且往往与本机系统绑定，可能造成构建的不可移植，所以一般不使用。 

4. Optional：标记依赖是否可选  
5. Exclusions： 用来排除传递性依赖。

### 传递性依赖
传递依赖机制，让我们在使用某个jar的时候就不用去考虑它依赖了什么。也不用担心引入多余的依赖。Maven会解析各个直接依赖的POM，将那些必要的间接依赖，以传递性依赖的形式引入到当前项目中。  

***注意： 传递依赖有可能产生冲突！！***  
冲突场景：  
```
A-->B--->C (2.0)
A-->E--->C (1.0)
```
如果A下同时存在两个不同version的C，冲突！！（自动选取同时适合A、B的版本）
```xml
<dependencies>
    <dependency>
        <groupId>A</groupId>
        <artifactId>A</artifactId>
        <version>xxx</version>
        <exclusions>
            <exclusion>
                <groupId>C</groupId>
                <artifactId>C</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>B</groupId>
        <artifactId>B</artifactId>
    </dependency>
</dependencies>
```

## 参考
https://www.bilibili.com/video/BV1Fz4y167p5?p=13