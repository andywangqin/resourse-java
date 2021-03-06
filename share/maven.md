### 1、maven安装和配置
---
#### 1.1、maven的安装步骤
1. 在安装maven之前，先确保已经安装JDK1.6及以上版本，并且配置好环境变量。
2. 下载maven3，最新版本是Maven3.3.9 ，下载地址：[http://maven.apache.org/download.html](http://maven.apache.org/download.html), 下载apache-maven-3.3.9-bin.zip文件后，并解压到D:\maven\apache-maven-3.3.9
3. 配置maven3的环境变量：先配置M2_HOME的环境变量，新建一个系统变量：M2_HOME , 路径是：D:\maven\apache-maven-3.3.9,再配置path环境变量，在path值的末尾添加"%M2_HOME%\bin"
4. 点击确定之后，打开cmd窗口：输入 mvn -version,验证是否安装成功。

#### 1.2、配置文件settings.xml
maven的配置文件为settings.xml，在下面路径中可以找到这个文件，分别为：

- $M2_HOME/conf/settings.xml：全局设置，在maven的安装目录下
- ${user.home}/.m2/settings.xml：用户设置，需要用户手动添加，可以将安装目录下的settings.xml文件拷贝过来修改

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                        http://maven.apache.org/xsd/settings-1.0.0.xsd">
        <localRepository/>
        <interactiveMode/>
        <usePluginRegistry/>
        <offline/>
        <pluginGroups/>
        <servers/>
        <mirrors/>
        <proxies/>
        <profiles/>
        <activeProfiles/>
    </settings>
```

### 2、POM介绍
---
#### 2.1、Simple POM(坐标)
```xml
<groupId>com.mycompany.app</groupId>  
<artifactId>my-app</artifactId>  
<packaging>jar</packaging>  
<version>0.0.1-SNAPSHOT</version>  

<name>app for ios</name> 
<url>http://</url> 
<classifier>jdk15</classifier> 
```

|节点 |	描述 |
| ------------- |:-------------:|
|groupId	|这是工程组的标识。它在一个组织或者项目中通常是唯一的。例如，一个银行组织 com.company.bank 拥有所有的和银行相关的项目。|
|artifactId	|这是工程的标识。它通常是工程的名称。例如，消费者银行。groupId 和 artifactId 一起定义了 artifact 在仓库中的位置。|
|version	|这是工程的版本号。在 artifact 的仓库中，它用来区分不同的版本。例如：com.company.bank:consumer-banking:1.0 com.company.bank:consumer-banking:1.1.|
|packaging	|项目产生的构件类型，例如jar、war、ear、pom。插件可以创建他们自己的构件类型，所以前面列的不是全部构件类型|
|name	|项目的名称, Maven产生的文档用|
|url	|项目主页的URL, Maven产生的文档用|
|classifier	|它表示在相同版本下针对不同的环境或者jdk使用的jar|

#### 2.2、Super POM
所有的 POM 都继承自一个父 POM，无论是否显式定义了这个父 POM，默认继承lib/maven-model-builder-3.x.x.jar:org/apache/maven/model/pom-4.0.0.xml。父 POM 也被称作Super POM，它包含了一些可以被继承的默认设置。

查看effective pom(Super pom 加上工程自己的配置)命令：mvn help:effective-pom

#### 2.3、Full POM
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instCentral Repositoryance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <!-- The Basics -->
  <groupId>...</groupId>
  <artifactId>...</artifactId>
  <version>...</version>
  <packaging>...</packaging>
  <dependencies>...</dependencies>
  <parent>...</parent>
  <dependencyManagement>...</dependencyManagement>
  <modules>...</modules>
  <properties>...</properties>
 
  <!-- Build Settings -->
  <build>...</build>
  <reporting>...</reporting>
 
  <!-- More Project Information -->
  <name>...</name>
  <description>...</description>
  <url>...</url>
  <inceptionYear>...</inceptionYear>
  <licenses>...</licenses>
  <organization>...</organization>
  <developers>...</developers>
  <contributors>...</contributors>
 
  <!-- Environment Settings -->
  <issueManagement>...</issueManagement>
  <ciManagement>...</ciManagement>
  <mailingLists>...</mailingLists>
  <scm>...</scm>
  <prerequisites>...</prerequisites>
  <repositories>...</repositories>
  <pluginRepositories>...</pluginRepositories>
  <distributionManagement>...</distributionManagement>
  <profiles>...</profiles>
</project>
```

### 3、构建生命周期（Build Lifecycle）
---
#### 3.1、概念
概念区分：构建生命周期（Build Lifecycle）、阶段（Phase）、插件(Plugin)、目标（Goal）

解释：构建生命周期由多个有序的构建阶段（sequence of phases）组成，一个构建阶段（Phase）可以绑定一个或者多个的目标。构建生命周期和阶段只是抽象的概念，不涉及具体的功能。 具体的功能由插件（Plugin）实现,一个插件可以实现多个目标（Goal）。而maven默认将某些目标自动绑定到某些阶段，只要进行了绑定，执行阶段时，自动就会执行该目标。

例如：mvn package

这条命令执行的是default生命周期中的package阶段，而这个阶段默认绑定了maven-jar-plugin的jar目标（即jar:jar），所以执行这个阶段就会调用该插件jar目标所做的事情，也就是创建项目的jar包。当然也可以直接执行目标，比如：mvn dependency:copy-dependencies
这里dependency是插件的名称，copy-denpendencies是插件的目标。
如果需要将某个目标绑定到某个阶段，在POM文件中配置插件即可。

Maven 有以下三个标准的构建生命周期:

- clean: 主要目的是清理项目
- default(or build)：定义了真正构建时所需要执行的所有步骤，它是生命周期中最核心的部分
- site: 生成项目站点文档

#### 3.2、Clean 生命周期
- pre-clean
- clean
- post-clean

#### 3.3、Default (or Build) 生命周期
这是 Maven 的主要生命周期，被用于构建应用。

|阶段	|处理	|描述|
| ------------- |:-------------:|-------------:|
|prepare-resources|	资源拷贝	|本阶段可以自定义需要拷贝的资源|
|compile	|编译	|本阶段完成源代码编译|
|package	|打包	|本阶段根据 pom.xml 中描述的打包配置创建 JAR / WAR 包|
|install	|安装	|本阶段在本地 / 远程仓库中安装工程包|

#### 3.4、Site 生命周期
Maven Site 插件一般用来创建新的报告文档、部署站点等。

### 4、dependencies 
---
#### 4.1、仓库
Maven 仓库有以下几种类型：

- 本地（local）
- 中央（central）:http://search.maven.org/#browse --> https://repo.maven.apache.org/maven2
- 远程（remote）
- 私服 

#### 4.2、依赖管理
通过传递依赖，所有被包含的库的图形可能会快速的增长。当重复的库存在时，可能出现的情形将会持续上升。Maven 提供一些功能来控制可传递的依赖的程度:

|功能	|功能描述|
| ------------- |:-------------:|
|依赖调解	|当项目中出现多个版本构件依赖的情形，依赖调解决定最终应该使用哪个版本。目前，Maven 2.0只支持“短路径优先”原则，意思是项目会选择依赖关系树中路径最短的版本作为依赖。当然，你也可以在项目POM文件中显式指定使用哪个版本。值得注意的是，在Maven2.0.8及之前的版本中，当两个版本的依赖路径长度一致时，哪个依赖会被使用是不确定的。不过从Maven 2.0.9开始，POM中依赖声明的顺序决定了哪个版本会被使用，也叫作”第一声明原则”。|
|依赖管理	|在出现传递性依赖或者没有指定版本时，项目作者可以通过依赖管理直接指定模块版本。之前的章节说过，由于传递性依赖，尽管某个依赖没有被A直接指定，但也会被引入。相反的，A也可以将D加入```<dependencyManagement>```元素中，并在D可能被引用时决定D的版本号。|
|依赖范围	|你可以指定只在当前编译范围内包含合适的依赖。 下面会介绍更多相关的细节。|
|排除依赖	|任何可传递的依赖都可以通过 "exclusion" 元素被排除在外。举例说明，A 依赖 B， B 依赖 C，因此 A 可以标记 C 为 “被排除的”。|
|可选依赖	|如果项目Y依赖项目Z，项目Y的所有者可以使用”optional”元素来指定项目Z作为X的可选依赖。那么当项目X依赖项目Y时，X只依赖Y并不依赖Y的可选依赖Z。项目X的所有者也可以根据自己的意愿显式指定X对Z的依赖。（你可以把可选依赖理解为默认排除）。|

*dependency scope*:

|范围	|描述|
| ------------- |:-------------:|
|compile编译阶段	|如果没有提供一个范围，那该依赖的范围就是编译范围。编译范围依赖在所有的classpath 中可用，同时它们也会被打包。|
|provided供应阶段	|provided 依赖只有在当JDK 或者一个容器已提供该依赖之后才使用。例如， 如果你开发了一个web 应用，你可能在编译 classpath 中需要可用的Servlet API 来编译一个servlet，但是你不会想要在打包好的WAR 中包含这个Servlet API；这个Servlet API JAR 由你的应用服务器或者servlet 容器提供。已提供范围的依赖在编译classpath （不是运行时）可用。它们不是传递性的，也不会被打包。|
|runtime运行阶段	|runtime 依赖在运行和测试系统的时候需要，但在编译的时候不需要。比如，你可能在编译的时候只需要JDBC API JAR，而只有在运行的时候才需要JDBC驱动实现。|
|test测试阶段	|test范围依赖 在一般的编译和运行时都不需要，它们只有在测试编译和测试运行阶段可用。|
|system系统阶段	|system范围依赖与provided 类似，但是你必须显式的提供一个对于本地系统中JAR 文件的路径。这么做是为了允许基于本地对象编译，而这些对象是系统类库的一部分。这样的构件应该是一直可用的，Maven 也不会在仓库中去寻找它。如果你将一个依赖范围设置成系统范围，你必须同时提供一个 systemPath 元素。注意该范围是不推荐使用的（你应该一直尽量去从公共或定制的 Maven 仓库中引用依赖）。|

#### 4.3、外部依赖
通常情况下它会包含一些任何仓库无法使用，并且 maven 也无法下载的 jar 文件。如果你的代码正在使用这个库，那么 Maven 的构建过程将会失败，因为在编译阶段它不能下载或者引用这个库。
```xml
 <dependency>
             <groupId>ldapjdk</groupId>
             <artifactId>ldapjdk</artifactId>
             <scope>system</scope>
             <version>1.0</version>
             <systemPath>${basedir}\src\lib\ldapjdk.jar</systemPath>
          </dependency>
```

#### 4.4、快照（snapshot）
快照 vs 版本:
- 对于版本，Maven 一旦下载了指定的版本（例如 data-service:1.0），它将不会尝试从仓库里再次下载一个新的 1.0 版本。想要下载新的代码，数据服务版本需要被升级到 1.1。
- 对于快照，每次用户接口团队构建他们的项目时，Maven 将自动获取最新的快照（data-service:1.0-SNAPSHOT）。

#### 4.5、metadata.xml
Maven Repository Metadata is available in directories representing:

- an un-versioned artifact: it gives informations about available versions of the artifact,
- a snapshot artifact: it gives precise information on the snapshot,
- a group containing Maven plugins artifacts: it gives informations on plugins available in this group.

The metadata file name is:

- maven-metadata.xml in a remote repository,
- maven-metadata-<repo-id>.xml in a local repository, for metatada from a repository with repo-id identifier.

### 5、maven插件
---
*everything is plugin*

Maven 实际上是一个依赖插件执行的框架，每个任务实际上是由插件完成：
- 创建 jar 文件
- 创建 war 文件
- 编译代码文件
- 代码单元测试
- 创建工程文档
- 创建工程报告

插件通常提供了一个目标的集合，并且可以使用下面的语法执行：mvn [plugin-name]:[goal-name]

### 5.1、插件助手插件maven-help-plugin
Maven help 这个插件就是用来查询具体插件相关信息的，maven help 插件2.2版本有9个goals，下面重点说下describe这个goal 的用法：
*help:describe*
描述插件的属性。它不需要在项目目录下运行。但是你必须提供你想要描述插件的前缀或者 groupId和artifactId。通过 plugin 参数你可以指定你想要了解哪个插件，你可以传入插件的前缀（如 maven-war-plugin 插件的前缀就是war），或者可以是 groupId:artifact[:version] ，这里 version 是可选的

举例说明如下（以maven-war-plugin 为例 ）
 mvn help:describe -Dplugin=war
 或者
 mvn help:describe -Dplugin=org.apache.maven.plugins:maven-war-plugin
 或者
 mvn help:describe -DgroupId=org.apache.maven.plugins -DartifactId=maven-war-plugin
 
如果想了解详细信息，可以在其后加上-Ddetail 或者 -Dfull,如：mvn help:describe -Dplugin=org.apache.maven.plugins:maven-war-plugin -Ddetail

#### 5.2、打包插件assembly
使用Maven对Web项目进行打包，默认为war包；但有些时候，总是希望打成zip包(亦或其他压缩包)，maven-war-plugin插件就无能为力了，这时就用到了maven-assembly-plugin插件了，该插件能打包成指定格式分发包，更重要的是能够自定义包含/排除指定的目录或文件（遗留项目中,过滤配置文件时,或者仅仅需要发布图片或者CSS/JS等指定类型文件时,发挥作用)
该插件使用如下:
```xml
<build>
  <plugins>
    <plugin>
      <artifactId>maven-assembly-plugin</artifactId> <!-- 官网给出的配置，没有配置 groupId，这里也不配置 -->
      <version>2.4</version>
      <executions>
        <execution>
          <id>make-assembly</id> <!-- ID 标识，命名随意 -->
          <phase>package</phase> <!-- 绑定到 PACKAGE 生命周期阶段 -->
          <goals>
            <goal>single</goal>  <!-- 在 PACKAGE 生命周期阶段仅执行一次 -->
          </goals>
        </execution>
      </executions>
      <configuration>
        <descriptors>
          <descriptor>assembly.xml</descriptor> <!-- 自定义打包的配置文件 -->
        </descriptors>
        <appendAssemblyId>false</appendAssemblyId> <!-- 设为 FALSE, 防止 WAR 包名加入 assembly.xml 中的 ID -->
      </configuration>
    </plugin>
  </plugins>
</build>
```

### 6、build配置
---
#### 6.1、Build Settings
根据POM 4.0.0 XSD，build元素概念性的划分为两个部分：BaseBuild（包含poject build和profile build的公共部分，见下）和poject build包含的一些高级特性。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"  
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0  
                        http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    ...  
    <!-- "Project Build" contains more elements than just the BaseBuild set -->  
    <build>...</build>  
  
    <profiles>  
        <profile>  
            <!-- "Profile Build" contains a subset of "Project Build"s elements -->  
            <build>...</build>  
        </profile>  
    </profiles>  
</project>  
```
##### 6.1.1、The BaseBuild Element Set
```xml
<build>
  <defaultGoal>install</defaultGoal>
  <directory>${basedir}/target</directory>
  <finalName>${artifactId}-${version}</finalName>
  <filters>
    <filter>filters/filter1.properties</filter>
  </filters>
  ...
</build>
```

##### 6.1.2、Resources
```xml
 <build>
    ...
    <resources>
      <resource>
        <targetPath>META-INF/plexus</targetPath>
        <filtering>false</filtering>
        <directory>${basedir}/src/main/plexus</directory>
        <includes>
          <include>configuration.xml</include>
        </includes>
        <excludes>
          <exclude>**/*.properties</exclude>
        </excludes>
      </resource>
    </resources>
    <testResources>
      ...
    </testResources>
    ...
  </build>
```

*filtering*
filtering是maven的resource插件提供的功能，作用是用环境变量、pom文件里定义的属性和指定配置文件里的属性替换属性(*.properties)文件里的占位符(${jdbc.url})，具体使用如下：
在src/main/resources目录有个配置文件jdbc.properties，内容如下：

```
jdbc.url=${pom.jdbc.url}
jdbc.username=${pom.jdbc.username}
jdbc.passworkd=${pom.jdbc.password}
```

配置 resource 插件，启用filtering功能并添加属性到pom:

```xml
<project>
    ... 
    <properties>
        <pom.jdbc.url>jdbc:mysql://127.0.0.1:3306/dev</pom.jdbc.url>
        <pom.jdbc.username>root</pom.jdbc.username>
        <pom.jdbc.password>123456</pom.jdbc.password>
    </properties>
    <build>
      ...
        <filters>
            <filter>src/main/filters.properties</filter>
        </filters>
        <resources>
          <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
          </resource>
        </resources>
        ...
    </build>
    ...
</project>
```

##### 6.1.3、Plugins和Plugin Management
pluginManagement的元素的配置和plugins的配置是一样的，只是这里的配置只是用于集成，在孩子POM中指定使用
```xml
<build>
    ...
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <version>2.6</version>
        <extensions>false</extensions>
        <inherited>true</inherited>
        <configuration>
          <classifier>test</classifier>
        </configuration>
        <dependencies>...</dependencies>
        <executions>...</executions>
      </plugin>
    </plugins>
	<pluginManagement>
	</pluginManagement>	
  </build>
```

##### 6.1.4、The Build Element Set:
```xml
 <build>
    <sourceDirectory>${basedir}/src/main/java</sourceDirectory>
    <scriptSourceDirectory>${basedir}/src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${basedir}/src/test/java</testSourceDirectory>
    <outputDirectory>${basedir}/target/classes</outputDirectory>
    <testOutputDirectory>${basedir}/target/test-classes</testOutputDirectory>
    ...
  </build>
```

#### 6.2、自动化部署
一般情况下，在一个工程开发进程里，一次部署的过程包含需如下步骤：

- 合入每个子工程下的代码到 SVN 或者源代码库，并标记它。
- 从 SVN 下载完整的源代码。
- 构建应用程序。
- 保存构建结果为 WAR 或者 EAR 类型文件并存放到一个共同的指定的网络位置上。
- 从网络上获得该文件并且部署该文件到产品线上。
- 更新文档日期和应用程序的版本号。

### 7、maven聚合和继承
---
我们使用Maven应用到实际项目的时候，需要将项目分成不同的模块。这个时候，Maven的聚合特性能够把项目的各个模块聚合在一起构件，而Maven的继承特性则能帮助抽取各模块相同的依赖和插件等配置。在简化POM的同时，还能促进各个模块配置的一致性。下面以具体项目来讲解:

![](../image/223534_0RUu_865771.png)

user-parent的pom.xml详情如下:
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.liangbo.user</groupId>
  <artifactId>user-parent</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>pom</packaging>

  <modules>
      <module>../user-core</module>
      <module>../user-dao</module>
      <module>../user-log</module>
      <module>../user-service</module>
  </modules>
  
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <junit.version>4.10</junit.version>
    <mysql.driver>com.mysql.jdbc.Driver</mysql.driver>
    <mysql.url>jdbc:mysql://localhost:3306/mysql</mysql.url>
    <mysql.username>root</mysql.username>
    <mysql.password>password</mysql.password>
  </properties>
  
  <dependencyManagement>  
  </dependencyManagement>
  
  <build>
      <pluginManagement>
      </pluginManagement>
  </build>
</project>
```

下面来看下user-core项目pom.xml的配置:
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
      <groupId>com.liangbo.user</groupId>
      <artifactId>user-parent</artifactId>
      <version>0.0.1-SNAPSHOT</version>
      <relativePath>../user-parent/pom.xml</relativePath>
  </parent> 

  <artifactId>user-core</artifactId>
  <packaging>jar</packaging>

  <name>user-core</name>
  <url>http://maven.apache.org</url>

  <dependencies>
  </dependencies>
  
  <build>
      <plugins>
      </plugins>
  </build>
</project>
```

### 8、profile
---
#### 8.1、构建配置文件
构建配置文件是一组配置的集合，用来设置或者覆盖Maven构建的默认配置,使用构建配置文件，可以为不同的环境定制构建过程，例如 Producation 和 Development 环境，在profile里几乎可以定义所有在pom里的定义的内容（<dependencies>，<properties>，插件配置等等，不过不能再定义他自己了）。当一个profile被激活时，它定义的<dependencies>，<properties>等就会覆盖掉原pom里定义的相同内容，从而可以通过激活不同的profile来使用不同的配置。
```xml
<profiles>
      <profile>
      <id>test</id>
      <build>
      <plugins>
         <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-antrun-plugin</artifactId>
            <version>1.1</version>
            <executions>
               <execution>
                  <phase>test</phase>
                  <goals>
                     <goal>run</goal>
                  </goals>
                  <configuration>
                  <tasks>
                     <echo>Using env.test.properties</echo>
            <copy file="src/main/resources/env.test.propertiestofile
            ="${project.build.outputDirectory}/env.properties"/>
                  </tasks>
                  </configuration>
               </execution>
            </executions>
         </plugin>
      </plugins>
      </build>
      </profile>
   </profiles>
```
*Profile 激活，Maven 的 Profile 能够通过几种不同的方式激活:*
- 显式使用命令控制台输入,mvn install -Pproduction
- 通过 maven 设置
- 基于环境变量（用户 / 系统变量）
- 操作系统配置（例如，Windows family）
- 现存 / 缺失 文件

#### 8.2、profile + filtering

### 9、demo
---
#### 9.1、create project
Maven 使用原型（archetype）插件创建web工程，执行:

mvn archetype:generate -DgroupId=com.qianmo.demo -DartifactId=demo-server -Dversion=1.0.0-SNAPSHOT -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false

#### 9.2、import maven project
1. 通过idea new菜单，import maven项目
2. idea 设置
Import Maven projects automatically; add java package
3. 集成tomcat

参考:

- [Maven教程](http://wiki.jikexueyuan.com/project/maven/repositories.html)
- [POM Reference](https://maven.apache.org/pom.html#Build_Settings)