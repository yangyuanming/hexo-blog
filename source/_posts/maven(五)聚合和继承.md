---
title: maven(五):聚合和继承
comment: true
tags:
  - maven
  - maven聚合&继承
categories:
  - 工具学习
  - maven
date: 2018-08-12 20:22:00
---
## 聚合
Maven聚合（或者称为多模块），是为了能够使用一条命令就构建多个模块，方便快速构建项目，例如已经有两个模块，分别为A,B，我们需要创建一个额外的模块（假设名字为aggregator，然后通过该模块，来构建整个项目的所有模块，aggregator本身作为一个Maven项目，它必须有自己的POM,不过作为一个聚合项目，其POM又有特殊的地方，看下面的配置

```xml
<project
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/maven-v4_0_0.xsd>
        <modelVersion>4.0.0</modelVersion>
        <groupId>com.yangyuanming</groupId>
        <artifact>aggregator</artifact>
        <version>1.0.0-SNAPSHOT</version>
        <packaging>pom</packaging>
        <name>Aggregator</name>
        <modules>
            <module>../A</module>
            <module>../B</module>
        </modules>
</project>
```
必须声明`<packaging>`为pom，**对于聚合模块来说，其打包方式必须为pom**，否则无法构建。  
`<modules>`里的每一个`<module>`都可以用来指定一个被聚合模块，这里每个`<module>`的值都是一个当前pom的相对位置，本例中A、B和aggregator位于同一级目录下。
## 继承
为了消除多模块项目中的重复配置，类似于java，maven中也有继承，一次声明，多次使用。父项目的pom.xml声明的配置，子项目pom.xml不需要声明就可以直接使用，子项目的配置可以覆盖父项目的配置。很多时候，我们使用继承主要是为了方便管理引用的构件(依赖和插件)。
一个项目往往分为很多模块，而不同的模块中，引用的构件很多是一样的，使用继承就避免了在不同模块中重复引用的问题，同时也方便统一管理构件，构件版本号统一。
### 继承示例
> 父模块pom.xml

```xml
<project
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:shemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.yangyuanming</groupId>
    <artifactId>parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Parent</name>
    <dependencies>
        ....
    </dependencies>
    <build>
        <plugins>
            ....
        </plugins>
    </build>
</project>
```

`<packaging>`的值必须为pom，由于父模块只是为了帮助消除配置的重复，因此它本身不包含除POM之外的项目文件，可以把项目中src/main/java和src/test/java目录删除。**继承的是父模块的pom.xml**。

子模块pom.xml
```xml
<project>   
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:shemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/maven-v4_0_0.xsd">
    <parent>
        <groupId>com.yangyuanming<groupId>
        <artifactId>parent</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../parent/pom.xml</relativePath>
    </parent>
    
    <artifactId>A</artifactId>
    
    <name>artifactId-A</name>
    <dependencies>
        ....
    </dependencies>
    <build>
        <plugins>
            ....
        </plugins>
    </build>
</project>
```
上面POM中使用parent元素声明父模块，**paren下的子元素groupId、artifactId和version指定了父模块的坐标，这三个元素是必须的**。元素relativePath表示了**父模块POM**的相对路径。相对路径允许你选择一个不同的路径。默认值是../pom.xml。当项目构建时，Maven会首先根据relativePath检查父pom，然后在本地仓库，最后在远程仓库寻找父项目的pom。此例子父模块parent和子模块在同一级目录下。  
    
子模块会自动继承父模块的配置，子模块配置会覆盖从父模块继承来的配置。上面子模块child没有声明groupId，version，因为这个子模块隐式的从父模块继承了这两个元素，这也就消除了不必要的配置。上例中，父子模块使用了相同的groupId和version，如果遇到子模块需要使用和父模块不一样的groupId或者version的情况，可以在子模块中显式声明。子模块会自动继承父模块的插件和依赖，不需要再重新显式引入，这大大方便了多模块开发的管理。然而所有的子模块会将父pom.xml中定义的所有构件继承下来，不同模块所需的构件还是会有一点差别的，存在继承多余构件的问题，怎么办呢？看下一篇文章吧！
### 可继承的pom元素
* groupId:项目组ID,项目坐标的核心元素
* version:项目版本,项目坐标的核心元素
* description:项目的描述信息
* organnization:项目的组织信息
* inceptionYear:项目的创始年份
* url:项目的URL地址
* developers:项目的开发者信息
* contributors:项目的贡献者信息
* distributionManagement:项目的部署配置
* issueManagement:项目的缺陷跟踪系统信息
* ciManagement:项目的集成信息
* scm:项目的版本控制系统信息
* mailingLists:项目的邮件列表信息
* properties:自定义的Maven属性
* dependencies:项目的依赖配置
* dependencyManagement:项目的依赖管理配置
* repositories:项目的仓库配置
* build:包括项目的源码目录配置、输出目录配置、插件配置、插件管理配置等
* reporting:包括项目的报告输出目录配置，报告插件配置等。

## 参考资料
**lofty**，[Maven——聚合与继承](https://www.cnblogs.com/wangwei-beijing/p/6535084.html)


