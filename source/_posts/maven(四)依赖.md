---
title: 'maven(四):依赖'
comment: true
tags:
  - maven
  - maven依赖
categories:
  - 工具学习
  - maven
abbrlink: '7e899530'
date: 2018-08-12 14:00:00
---
## 依赖的配置
<!--more-->

```xml
project>
    ......
    <dependencies>
        <dependency>
            <groupId>...</groupId>
            <artifactId>...</artifactId>
            <version>...</version>
            <type>...</type>
            <scope>...</scope>
            <systemPath></systemPath>
            <optional>...<optional>
            <exclusions>
                <exclusion>...</exclusion>
            </exclusions>
            ....
        </dependency>
    </dependencies>
    .........
</project>
```
* groupId、artifactId、version：构成依赖的基本坐标
* type：依赖类型，默认类型是jar。它通常表示依赖的文件的扩展名，但也有例外。一个类型可以被映射成另外一个扩展名或分类器。对应于项目坐标中的packaging
* scope：依赖范围，默认compile，作用是控制依赖和3种classpath的关系，另外还会影响依赖传递，[maven官方介绍](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)。maven3种classpath:编译、运行、测试，各依赖范围与classpath的关系如下
    - compile:编译依赖范围。如果没有指定，就会默认使用该依赖范围，使用此依赖范围的Maven依赖，对于编译、测试、运行三种classpath都有效。此外，这些依赖关系会传播到依赖项目。
    - test:测试依赖范围。使用该范围的Maven依赖，只对于测试classpath有效，在编译主代码或者运行项目的使用时将无法使用此类依赖。典型的例子是JUnit,它只是在编译测试代码和运行测试的时候需要该依赖。这个范围是不可传递的。     
    - provided：已提供依赖范围。使用此依赖范围的Maven依赖，对于编译和测试的classpath有效，但是在运行时无效。典型的例子是servlet-api，编译和测试项目的时候需要改依赖，但是在运行项目的时候，由于容器已经提供，就不需要Maven重复地引入一遍。该依赖不可传递。
    - runtime:运行时依赖范围。使用此依赖范围的Maven依赖，对于测试和运行classpath有效，但是编译主代码时无效。典型的例子是JDBC驱动实现，项目主代码的编译只需要JDK提供的JDBC接口，只有在测试或者运行项目的时候才需要实现上述接口的JDBC驱动。
    - import(Maven 2.0.9及以上)：导入依赖范围。仅支持依赖类型为pom的依赖，该pom配置了<dependencyManagement>。该依赖范围不会对三种classpath产生实际的影响,不参与限制依赖关系的传递性。。
    - system:系统依赖范围，该范围与三种classpath的关系，和provided依赖范围完全一致。但是，使用system范围的依赖时必须通过systemPath元素显式的指定依赖文件的路径。由于此类依赖不是通过Maven仓库解析，而且往往与本机系统绑定，可能造成构建的不可移植，因此需要谨慎使用。systemPath元素可以引用环境变量，如：
            
    ```xml
    <dependency>
        <groupId>javax.sql</groupId>
        <artifactId>jdbc-stdext</artifactId>
        <version>2.0</version>
        <scope>system</scope>
        <systemPath>${java.home}/lib/rt.jar</systemPath>
    </dependency>
    ```
> 除import外，各依赖和3种classpath的关系如下

| 依赖范围 | 编译 | 测试 | 运行  | 例子 |
| --- | :-: | :-: | :-: | --- |
| compile | Y | Y | Y | spring-core |
| test | - | Y | - | junit |
| provided | Y | Y | - | servlet-api |
| runtime | - | Y | Y | JDBC驱动实现 |
| system | Y | Y | - | 本地的，Maven仓库之外的类库文件 |

* systemPath：仅供scope为system时使用。注意，不鼓励使用这个元素，并且在新的版本中该元素可能被覆盖掉。该元素为依赖规定了文件系统上的路径。需要绝对路径而不是相对路径。推荐使用属性匹配绝对路径，例如${java.home}。
* optional：可选依赖，控制依赖的可传递性。如果为true，则其子项不会引入该依赖。如果子项要引用该依赖，需要显式引用。
* exclusions：用于排除传递性依赖。A引入依赖B，而B依赖C，由于依赖会传递，A也会引入依赖C。如果A不想引入依赖C，则可以在引入B依赖时设置exclusion，排除传递依赖C。

## 依赖传递
### 例子解释
maven项目中可能会存在依赖多层引用，例如项目A依赖B,B依赖n个其他项目，A不需要手动再引用B所依赖的n个项目。Maven会解析各个直接依赖的POM，将那些必要的间接依赖，以传递性依赖的形式引入到当前的项目当中。
### 传递性依赖和依赖范围
每个作用域(除了import )以不同的方式影响传递依赖关系，如下表所示。如果将依赖项设置为左列中的范围，则该依赖项与顶部行中的范围的传递依赖项将导致主项目中的依赖项与交叉列出的范围。如果没有列出范围，这意味着将省略相关性。

|  | compile | provided | runtime | test |
| --- | --- | --- | --- | --- |
| compile | compile | - | runtime | - |
| provided | provided | - | provided | - |
| runtime | runtime | - | runtime | - |
| test | test | - | test | - |

## 依赖冲突
maven的传递性依赖方便了我们开发，一般我们只关心直接依赖而不会关心传递依赖。但当传递性依赖发生问题时，我们需要清楚传递性依赖的来源。传递性依赖是可能存在冲突的，比如某个依赖路径可能存在多个相同依赖，只是版本号不同，那么使用哪个传递依赖呢？maven遵循两个原则：
### 第一原则：短路优先
例如，项目A存在以下传递依赖
`A-->B-->C-->D(1.0)`
`A-->C-->D(2.0)`
根据第一原则，项目A会使用D(2.0)
### 第二原则：声明优先
有时候存在路径长度相同的情况，这时遵循第二原则
`A-->B-->D(1.0)`
`A-->C-->D(2.0)`
假设`A-->B-->D(1.0)`先声明，那么项目A会使用D(1.0)
## 可选依赖
可选依赖是由`<dependency>`中的`<optional>`标签定义的，默认为false，如果声明为`<optional>true</optional>`,那么子项目将不会隐式引入该依赖，如果需要引入，需要在子项目的pom.xml中显式引入。
为什么需要设置可选依赖？可能项目A实现了两个特性，其中的特性一依赖于X，特性二依赖于Y,而且这两个特性是互斥的，用户不可能同时使用两个特性。比如A是一个持久层隔离工具包，它支持多种数据库，报错MySQL、Oracle,在构建这个工具包的时候，需要这两种数据库的驱动程序，但是在使用这个工具包的时候，只会依赖一种数据库。
在理想的情况下，是不应该使用可选依赖的。除非是某一个项目实现了多个互斥的特性，在面向对象设计中，有个单一职责性原则，意指一个类应该只要一项职责。
## 排除传递依赖
传递性依赖会给项目隐式引入很多依赖，大大方便了依赖的管理，但有时候会带来问题。例如存在以下依赖传递：  
`A-->B-->C`  
由于传递依赖C是SNAPSHOT版本，引入C会给项目A带来问题，实际上C存在release版的，引入release版的对C没有问题，这时就需要在引入依赖B的声明中排除C(SNAPSHOT),然后再显式引入C(release)。

```xml
<groupId>com.yangyuanming</groupId>
<artifactId>project-A</artifactId>
<version>1.0.0</version>
<dependencies>
    <dependency>
        <groupId>com.yangyuanming</groupId>
        <artifactId>project-B</artifactId>
        <version>1.0.0</version>
        <!--排除传递依赖-->
        <exclusions>
            <exclusion>
                <groupId>com.yangyuanming</groupId>
                <artifactId>project-C</artfactId>
            </exclusion>
        </exclusions>
    </dependency>
    <!--显式引入所需的依赖C稳定版-->
    <dependency>
        <groupId>com.yangyuanming</groupId>
        <artifactId>project-C</artifactId>
        <version>1.1.0</version>
    </dependency>
</dependencies>
```
上述代码中，项目A在引入依赖B时，使用`<exclusion>`排除了传递依赖C，然后再显式引入了另一个版本的依赖C。也可能存在其他原因不想引入某个传递性依赖，可以在`<exclusions>`中声明多个`<exclusion>`排除多个传递依赖。
声明`<exclusion>`时只需要指明`<groupId>`和`<artifactId>`
## 归并依赖
来自A项目的不同模块，版本号是相同的。当B项目需要引入A项目的多个模块时，一般要保证引入A项目的多个模块的版本号是相同的，为了方便B项目管理引入的多个模块(A项目)，需要做归并处理。  
举个例子，Spring Framework包含很多模块，例如core:2.5.6、bean:2.5.6、context:2.5.6和support:2.5.6,他们来着同一项目的不同模块，因此，所有这些依赖的版本都是相同的，而且，如果以后需要升级Spring Framework，这些依赖的版本会一起升级，这样我们就可以像Java中声明一个Constants一样，为依赖的项目的版本设置一个这样类似常量的东西。这样在升级Spring Framework的时候就需要修改一处，代码如下：

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.up366.feishu</groupId>
    <artfactId>feishu_v3</artifactId>
    <name>feishu_v3</name>
    <version>1.0.0-SNAPSHOT</version>
    
    <properties>
        <springframework.version>2.5.6</springframework.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifact>spring-core</artifact>
            <version>${springframework.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifact>spring-beans</artifact>
            <version>${springframework.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifact>spring-context</artifact>
            <version>${springframework.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifact>spring-context-support</artifact>
            <version>${springframework.version}</version>
        </dependency>
    </dependencies>
</project>
```
上面代码定义maven属性，定义了一个`<springframework.version>`，指定Spring Framework的版本号，然后在引入的依赖的`<version>`中引用该属性，达到统一管理版本号的作用。
## 优化依赖
查看已解析依赖可以使用命令，如下：

* 列表形式查看命令：mvn dependency:list
* 树状结构查看命令：mvn dependency:tree
* 依赖分析命令：mvn dependency:analyze
mvn dependency:analyze由两部分组成，首先是Used updeclared dependencies,意指项目中使用到的，没有显式声明的依赖。
其次是Unused declared dependencies，意指项目中没有使用的，但是显是声明了的依赖。需要注意的是，对于这一类依赖，我们不应该简单的直接删除其声明，而是应该仔细分析，由于dependency:analyze只会分析编译主代码和测试代码需要用到的依赖，一些执行测试和运行时需要的依赖它就发现不了。建议不要乱删，当然有时候确实能通过该信息找到一些没用的依赖，但一定要小心测试。
## 参考资料
**lofty**，[Maven——坐标与依赖](http://www.cnblogs.com/wangwei-beijing/p/6535073.html)
[Introduction to the Dependency Mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)



