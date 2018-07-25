---
title: maven生命周期和插件
comment: true
tags:
  - maven生命周期
  - maven插件目标
  - maven插件配置
  - maven插件解析
  - maven
categories:
  - 工具学习
  - maven
date: 2018-07-24 07:30:00
---

## maven生命周期

maven抽象出了3套生命周期，其具体实现是依赖于[插件](https://maven.apache.org/plugins/index.html)。每套生命周期是相互独立的，都由一组阶段(Phase)组成。每套生命周期中的阶段是有顺序的，后面阶段依赖于前面的阶段，执行后面阶段会自动执行之前的阶段，但不会触发不同生命周期的阶段。

**下面是三个生命周期及其包含的阶段。**

### Clean Lifecycle 
清理项目，在进行真正的构建之前进行一些清理工作。

*  pre-clean     执行clean前需要完成的工作  

*  clean     clean上一次构建生成的所有文件  

*  post-clean    执行clean后需要立刻完成的工作  

 <!--more-->
    
>这里的clean就是指的mvn clean。在一套生命周期内，运行某个阶段会自动按序运行之前阶段，mvn clean=mvn pre-clean clean。


### Default Lifecycle
**构建的核心部分**，编译，测试，打包，部署等等。

* validate      验证项目是否正确，并且所有必要的信息可用于完成构建过程

* initialize    建立初始化状态，例如设置属性

* generate-sources
  
* process-sources 
  
* generate-resources

* process-resources     复制并处理资源文件，至目标目录，准备打包。

* compile     编译项目的源代码。

* process-classes

* generate-test-sources

* process-test-sources 

* generate-test-resources

* process-test-resources     复制并处理资源文件，至目标测试目录。

* test-compile     编译测试源代码。

* process-test-classes

* test     使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。

* prepare-package

* package     提取编译后的代码，并在其分发格式打包，如JAR，WAR或EAR文件

* pre-integration-test     完成执行集成测试之前所需操作。例如，设置所需的环境

* integration-test

* post-integration-test     完成集成测试已全部执行后所需操作。例如，清理环境

* verify        运行任何检查，验证包是有效的，符合质量审核规定

* install     将包安装至本地仓库，以让其它项目依赖。

* deploy     将最终的包复制到远程的仓库，以让其它开发人员与项目共享    
  
### Site Lifecycle   
生成项目报告，站点，发布站点。

* pre-site     执行一些需要在生成站点文档(html)之前完成的工作

* site     生成项目信息的站点文档

* post-site     执行一些需要在生成站点文档之后完成的工作，并且为部署做准备

* site-deploy     将生成的站点文档部署到特定的服务器上


-------


## maven插件目标
maven本质是一个插件框架，maven每个生命周期的每个阶段(phase)默认绑定了一个或多个插件中的一个或多个目标。用户可以自行配置或编写插件。
**一个插件对应一个或多个目标，一个插件可以绑定多个生命周期阶段。**


-------


##  两种方式调用插件目标
### 插件目标绑定maven生命周期阶段  
   这分为内置绑定和自定义绑定。
     
* 内置绑定。maven的生命周期的阶段已经默认和一些插件的目标进行了绑定。例如Maven默认将maven-compiler-plugin的compile目标与compile生命周期阶段绑定，因此命令mvn compile实际上是先定位到compile这一生命周期阶段，然后再根据绑定关系调用maven-compiler-plugin的compile目标。  
*    自定义绑定。在pom.xml中进行配置，我们可以根据需要将任何插件目标绑定到任何生命周期的任何阶段。如：将maven-source-plugin的jar-no-fork目标绑定到default生命周期的package阶段，这样，以后在执行mvn package命令打包项目时，在package**阶段之后**会执行源代码打包。  
**自定义绑定的插件目标是在绑定的生命周期阶段之后执行的**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>2.2.1</version>
            <executions> <!--执行-->
                <execution>
                    <id>attach-source</id>
                    <!-- 要绑定到的生命周期的阶段 -->
                    <phase>package</phase>
                    <goals>
                        <!-- 要绑定的插件的目标，在maven官网plugins上可以查到 -->
                        <goal>jar-no-fork</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
    ……
</build>
```

### 运行命令时直接指定插件目标(与生命周期无关)  
`mvn 插件目标前缀(prefix):插件目标`  
各插件目标的命令在官网可以查。例如mvn archetype:generate 就表示调用maven-archetype-plugin的generate目标，这种**带冒号的调用方式与生命周期无关**。


-------


## 插件配置

完成插件和生命周期的绑定后，用户还可以配置插件目标的参数，进一步调整插件目标所执行的任务，以满足项目的需求。几乎所有的Maven插件的目标都有一些可配置的参数，用户可以通过命令行和POM配置的方式来配置这些参数。
### 命令行插件配置
用户可以在Maven命令中使用-D参数，并伴随一个参数键=参数值得形式，来配置插件目标的参数。

例如，maven-surefire-plugin提供了一个maven.test.skip参数，当其值为true的时候，就会跳过执行测试，于是在运行命令的时候，加上如下-D参数就能跳过测试。

```
mvn install -Dmaven.test.skip=true
```

参数-D是Java自带的，其功能是通过命令行设置一个Java系统属性，Maven简单的重用了该参数，在准备插件的时候检查系统属性，便实现了插件参数的配置。
### 在POM中插件全局配置
并不是所有的插件参数都适合从命令行配置，有些参数的值从项目创建到项目发布都不会改变，或者说很少改变，对于这种情况，在POM文件中一次性配置就显然比重负在命令行输入要方便。

用户可以在声明插件的时候，对此插件进行一个全局配置。也就是说，所有该基于该插件目标的任务，都会使用这些配置。例如我们通常会需要配置maven-compiler-plugin告诉它配置Java1.5版本的源文件，生成与JVM1.5兼容的字节码文件，代码如下：

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artfactId>
            <version>2.1</version>
            <configuration>
                <source>1.5</source>
                <target>1.5</target>
            </configration>
        </plugin>
    </plugins>
</build>
```
 
这样，不管绑定到compile阶段的maven-compiler-plugin:compile任务，还是绑定到test-compiler阶段的maven-compiler-plugin:testCompiler任务，这都能够使用该配置，基于Java1.5版本进行编译。

### POM中插件任务配置
除了为插件配置全局的参数，用户还可以为某个插件任务配置特定的参数。以maven-antrun-plugin为例，它有一个目标run,可以用来在Maven中调用Ant任务。用户将maven-antrun-plugin:run绑定到多个生命周期阶段上，再加以不同的配置，就可以让Maven在不同的生命周期执行不同的任务，代码如下:

```
<build>
    <plugins>
        <groupId>org.apache.maven.plugins<groupId>
        <artifactId>maven-antrun-plugin</artifactId>
        <version>1.3</version>
        <executions>
            <execution>
                <id>ant-validate</id>
                <phase>validate<phase>
                <goals>
                    <goal>run</goal>
                </goals>
                <configuration>
                    <tasks>
                        <echo>Im'bound to validate phase</echo>
                    </tasks>
                </configurationo>
            </execution>
            <execution>
                <id>ant-verify</id>
                <phase>verify</phase>
                <goals>
                    <goal>run</goal>
                </goals>
                <configuration>
                    <tasks>
                        <echo>I'm bound to verify phase</echo>
                    </tasks>
                </configuration>
            </execution>
        </executions>
    </plugins>
</build>
```
上述代码片段中，首先，maven-antrun-plugin:run与validate绑定，从而构成一个id为ant-validate的任务。插件全局配置中的configuration元素位于plugin元素下面，而这里的configuration元素则位于execution元素下，表示这是特定任务的配置，而非插件整体的配置。这个ant-validate任务配置了一个echo Ant任务，向命令行输出一段文字，表示该任务是绑定到validate阶段的。第二个任务的id为ant-verify，它绑定到了verify阶段，同样它也输出一段文字到命令行，告诉该任务绑定到了verify阶段。
## 获取插件信息
仅仅理解如何配置和使用插件是不够的，当遇到一个构建任务的时候，用户还需要知道去哪里寻找合适的插件，以帮助完成任务，找到正确的插件之后，还要详细了解该插件的配置点。由于Maven的插件非常多，这其中大部分没有完善文档，因此，使用正确的插件并进行正确的配置，其实并不是一件容易的事。
### 在线插件信息
基本所有的主要的Maven插件都来自Apache和Codehaus。由于Maven本身是属于Apache软件基金会的，因此他有很多的官方的插件，每天都有成千上万的Maven用户在使用这些插件，他们具有非常好的的稳定性。
[官网插件介绍](https://maven.apache.org/plugins/index.html)  
[插件列表](http://repo1.maven.org/maven2/org/apache/maven/plugins)  
### 使用maven-help-plugin描述插件
除了访问在线的插件文档之外，还可以借助maven-help-plugin来获取插件的详细信息。。可以运行一下命令来获取maven-compiler-plugin2.1版本的信息：
`mvn help:describe-Dplugin=org.apache.maven.plugins:maven-compiler-plugin:2.1`
这里执行的是maven-help-plugins的describe目标，在参数的plugin中输入需要描述插件的groupId、artfactId和version。Maven在命令行输出maven-compiler-plugin的简要信息。

在描述插件的时候，还可以省去版本信息，让Maven自动获取最新版本来进行表述。例如：
`mvn help:describe-Dplugin=org.apache.maven.plugins:maven-compiler-plugin`
进一步简化，可以使用插件目标前缀替换坐标。例如：
`mvn help:describe-Dplugin=compiler`
如果仅仅想描述某个插件目标的信息，可以加上goal参数：
`mvn help:describe-Dplugin=compiler-Dgoal=compile`
如果想让maven-help-plugin输出更详细的信息，可以加上detail参数：
`mvn help:describe -Dplugin=compiler-Ddetail`
### 从命令行调用插件
如果在命令行运行mvn -h来显示mvn命令帮助，可以看到如下的信息：

```

usage: mvn [options] [<goal(s)>] [<phase(s)>]
 
Options:
 -am,--also-make                        If project list is specified, also
                                        build projects required by the
                                        list
 -amd,--also-make-dependents            If project list is specified, also
                                        build projects that depend on

...
```
该信息告诉了我们mvn命令的基本用法，options表示可用的选项，mvn命令有20多个选项，除了选项之外，mvn命令后面可以添加一个或者多个goal和phase，他们分别是指插件目标和生命周期阶段

可以通过mvn命令激活生命周期阶段，从而执行那些绑定在生命周期阶段上的插件目标。但Maven还支持直接从命令行调用插件目标。Maven支持这种方式是因为有些任务不适合绑定在生命周期上，例如maven-help-plugin:describe，我们不需要在构建项目的时候去描述插件信息，又如maven-dependency-plugin:tree,我们也不需要在构建项目的时候去显示依赖树，因此这些插件目标应该通过如下方式使用：

```
mvn help:describe-Dplugin=compiler
mvn dependency:tree
```
不过这里有个疑问，describe是maven-help-plugin的目标没错，但是冒号前面的help是什么呢？它既不是groupId，也不是artifactId,Maven是如何根据该信息找到对应版本插件的呢？同理为什么不是maven-dependency-plugin:tree,而是dependency:tree

解答该疑问之前，可以尝试一下如下命令：

```
mvn org.apache.maven.plugins:maven-help-plugin:2.1:describe-Dplugin=compiler

mvn org.apache.maven.plugins:maven-dependency-plugin:2.1:tree
```
这两条命令就比较容易理解了，插件的groupId、artifactId、version以及goal都得以清晰描述。它们的效果与之前的两条命令基本是一样的，但是显然前面的命令更简洁，更容易记忆和使用。为了达到该目的，Maven引入了目标前缀的概念help是maven-help-plugin的目标前缀，dependency是maven-dependency-plugin的前缀，有了插件前缀，Maven就能找到对应的artifactId。不过，除了artifactId,Maven还需要得到groupId和version才能精确定位到某个插件。
## 插件解析机制
### 仓库元数据
* 插件元数据

**主要用于解释插件版本**

> 在远程仓库存放的位置结构:http://repo1.maven.org/maven2/groupId/artifactId/maven-metadata.xml  

这里的groupId指的就是构件的groupId，artifactId指的是构件的artifactId，例如插件maven-compiler-plugin的元数据链接:http://repo1.maven.org/maven2/org/apache/maven/plugins/maven-compiler-plugin/maven-metadata.xml  
内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<metadata>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <versioning>
    <latest>3.7.0</latest>
    <release>3.7.0</release>
    <versions>
      <version>2.0-beta-1</version>
      <version>2.0</version>
      <version>2.0.1</version>
      <version>2.0.2</version>
      <version>2.1</version>
      <version>2.2</version>
      <version>2.3</version>
      <version>2.3.1</version>
      <version>2.3.2</version>
      <version>2.4</version>
      <version>2.5</version>
      <version>2.5.1</version>
      <version>3.0</version>
      <version>3.1</version>
      <version>3.2</version>
      <version>3.3</version>
      <version>3.5</version>
      <version>3.5.1</version>
      <version>3.6.0</version>
      <version>3.6.1</version>
      <version>3.6.2</version>
      <version>3.7.0</version>
    </versions>
    <lastUpdated>20170904193138</lastUpdated>
  </versioning>
</metadata>
```
* 插件前缀元数据

**主要用于解释插件前缀**

> 在远程仓库存放的位置结构:http://repo1.maven.org/maven2/groupId/maven-metadata.xml

我们使用一般使用官方的插件就够了，官方插件默认的groupId为org.apache.maven.plugins，对应的链接为http://repo1.maven.org/maven2/org/apache/maven/plugins/maven-metadata.xml  
下面是前缀元数据xml文件截取的部分内容


```xml
<?xml version="1.0" encoding="UTF-8"?>
<metadata>
  <plugins>
    <plugin>
      <name>Apache Maven ACR Plugin</name>
      <prefix>acr</prefix>
      <artifactId>maven-acr-plugin</artifactId>
    </plugin>
    <plugin>
      <name>Apache Maven Ant Plugin</name>
      <prefix>ant</prefix>
      <artifactId>maven-ant-plugin</artifactId>
    </plugin>
    <plugin>
      <name>Maven ANTLR Plugin</name>
      <prefix>antlr</prefix>
      <artifactId>maven-antlr-plugin</artifactId>
    </plugin>
    ……
  </plugins>
</metadata>
```

### 插件仓库
与依赖仓库一样，插件构件同样基于坐标存储在Maven仓库中，在需要的时候Maven会从本地仓库查找插件，如果不存在，则从远程仓库查找。找到插件之后，再下载到本地仓库使用。

需要注意的是，Maven会区别对待依赖的远程仓库与插件的远程仓库。前面提到如何配置远程仓库，但是这种配置只对一般依赖有效果，当Maven需要的依赖在本地仓库不存在时，它会去所配置的远程仓库中查找，可是当Maven需要的插件在本地仓库存在时，他就不会去那些远程仓库查找。

不同于repositories及其repository子元素，插件的远程仓库使用pluginRepositories和pluginReposirory配置，例如，Maven的超级pom:pom-4.0.0.xml配置了如下的插件远程仓库,代码如下：

```xml
<pluginRepositories>
    <pluginRepository>
        <id>central</id>
        <name>Maven Plugin Repository</name>
        <url>http://repo1.maven.org/maven2</url>
        <layout>default</layout>
        <snapshots>
            <enabled>false</enabled>
        <snapshots>
        <releases>
            <updatePolicy>never</updatePolicy>
        </releases>
    </pluginRepository>
</pluginRepositories>
```
一般来说，中央仓库所包含的插件完全能够满足我们的需要，因此也不需要配置其他的插件仓库。只有在很少的情况下，项目使用的插件无法在中央仓库找到，或者自己编写的插件，这个时候可以参考上述的配置，在POM或者settings.xml中加入其他的插件仓库配置。
### 插件的默认groupId
在POM配置中配置插件的时候，如果该插件是Maven的官方插件（即如果其groupId为org.apache.maven.plugins），就可以省略groupId配置，见代码清单：

```
<build>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.1</version>
            <configuration>
                <source>1.5</source>
                <target>1.5</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```
上面省略了配置maven-compiler-plugin的groupId，Maven在解析该插件的时候，会自动用默认groupId org.apache.maven.plugins补齐。但是并不推荐使用Maven的这一机制，这样虽然可以省略一些配置，但是这样的配置会让团队中不熟悉Maven的成员感到费解，况且能省略的配置也就仅仅一行而已。
### 解析插件版本
> 同样是为了简化插件的配置和使用，在用户没有提供插件版本的情况下，Maven会自动解析插件版本。
> 首先，Maven的超级POM中为所有核心插件设定了版本,超级POM是所有Maven项目的父POM，所有项目都会继承这个超级POM配置，因此，即使用户不加任何配置，Maven使用核心插件的时候，他们的版本都已经确定了，这些插件包括maven-clean-plugin、maven-compiler-plugin、maven-surefire-plugin等。

上面说法是来自其他博客，我表示质疑，我使用的是maven3.5.4，在超级pom中并没有发现其为所有核心插件设定了版本。对于超级pom中没有设定版本的核心插件，没有指定版本时应该使用release版本。

如果用户使用某个插件时没有设定版本，而这个插件又不属于核心插件范畴，Maven就会去检查所有仓库中的可用版本，然后做出选择。以maven-compiler-plugin为例，他在中央仓库的[仓库元数据](http://repo1.maven.org/maven2/org/apache/maven/plugins/maven-compiler-plugin/maven-metadata.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<metadata>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <versioning>
    <latest>3.7.0</latest>
    <release>3.7.0</release>
    <versions>
      <version>2.0-beta-1</version>
      <version>2.0</version>
      <version>2.0.1</version>
      <version>2.0.2</version>
      <version>2.1</version>
      <version>2.2</version>
      <version>2.3</version>
      <version>2.3.1</version>
      <version>2.3.2</version>
      <version>2.4</version>
      <version>2.5</version>
      <version>2.5.1</version>
      <version>3.0</version>
      <version>3.1</version>
      <version>3.2</version>
      <version>3.3</version>
      <version>3.5</version>
      <version>3.5.1</version>
      <version>3.6.0</version>
      <version>3.6.1</version>
      <version>3.6.2</version>
      <version>3.7.0</version>
    </versions>
    <lastUpdated>20170904193138</lastUpdated>
  </versioning>
</metadata>
```
Maven遍历本地仓库和所有远程插件仓库，将该路径下的仓库元数据归并后，就能计算出latest和release的值。latest表示所有仓库中该构件的最新版本，而release表示最新的非快照版本。在Maven2中，插件的版本会被解析至latest。也就是说，当用户使用某个非核心插件且没有声明版本的时候，Maven会将版本解析为所有可用仓库中的最新版本，而这个版本也可能是快照版本的。

当插件的版本为快照版本的时，就会出现潜在的问题。Maven会基于更新策略，检查并使用快照的更新。某个插件可能昨天还好好的，今天就出错了。其原因是因为这个版本的插件发生了变化，为了防止这类问题，Maven3调整了解析机制，当插件没有声明版本的时候，不再解析至latest，而是使用release。这样就可以避免由于快照频繁更新而导致的插件行为不稳定。

依赖Maven解析插件版本其实是不推荐的做法，即使Maven3将版本解析到最新的非快照版本，也还是唯有潜在的不稳定性。例如，可能某个构件发布了一个新的版本，而这个版本的行为与之前的的版本发生了变化，这种变化就可能导致项目构建失败。因此，使用插件的时候，应该一直显式的设定版本，这也解释了Maven为什么要在超级POM中为核心插件设定版本。

### 解析插件前缀
mvn命令行支持使用插件前缀来简化插件的调用，现在解释Maven如何根据插件前缀解析到插件的坐标的。

插件前缀与groupId:artifactId是一一对应的，这种匹配关系存储在仓库元数据中。这里的仓库元数据不是groupId/artifactId/maven-metadata.xml，而是groupId/maven-metadata.xml。当Maven解析前缀:

* 首先会基于默认的groupId(org.apache.maven.plugins)归并所有插件仓库的元数据org/apache/maven/plugins/maven-metadata.xml
* 其次检查归并后的元素，根据前缀(prefix)找到对应的artifactId；
* 结合artifactId和groupId，最后获取version，这是就得到了完整的插件坐标。
* 若org/apache/maven/plugins/maven-metadata.xml没有记录该插件前缀，则接着检查其他groupId下的元数据，如org/codehaus/mojo/maven-metadata.xml以及用户自定义的插件。若所有元数据都不包含该前缀，则报错。
插件前缀元数据部分内容:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<metadata>
  <plugins>
    <plugin>
      <name>Apache Maven ACR Plugin</name>
      <prefix>acr</prefix>
      <artifactId>maven-acr-plugin</artifactId>
    </plugin>
    <plugin>
      <name>Apache Maven Ant Plugin</name>
      <prefix>ant</prefix>
      <artifactId>maven-ant-plugin</artifactId>
    </plugin>
    <plugin>
      <name>Maven ANTLR Plugin</name>
      <prefix>antlr</prefix>
      <artifactId>maven-antlr-plugin</artifactId>
    </plugin>
    ……
  </plugins>
</metadata>
```

## 参考资料

>http://blog.sina.com.cn/s/blog_e01142dc0102wup3.html
>https://www.cnblogs.com/wangwei-beijing/p/6535081.html


