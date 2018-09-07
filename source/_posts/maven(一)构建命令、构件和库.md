---
title: 
```maven(一):构建命令、构件和库
```
comment: true
tags:
  - maven
  - maven构建命令
  - maven库
categories:
  - 工具学习
  - maven
date: 2018-07-22 11:42:00
---

## maven常用构建命令
1. 查看maven版本：`mvn -v`
2. 编译：`mvn compile`
3. 打包：`mvn package`  
<!--more-->
4. 测试：`mvn test`
5. 删除target目录：`mvn clean`
6. 安装jar包到本地仓库：`mvn install` 
7. 自动建立目录结构的两种方式：

* `mvn archetype:generate` 按指示输入信息 
* `mvn archetype:generate` 

```shell
-DgroupId=网址+项目名  
-DartifactId=项目名-模块名  
-Dversion=版本号  
-Dpackage=代码包名
```
## 构件、仓库、镜像仓库

### 构件
#### 定义

* 在Maven中，任何依赖(jar包,tomcat等)、插件，或构建的输出都可成为构件。

* Maven在某个统一的位置存储所有项目的共享的构件，这个统一的位置，我们就称之为仓库。（仓库就是存放依赖和插件的地方）

* 任何的构件都有唯一的坐标，Maven根据这个坐标定义了构件在仓库中的唯一存储路径。
* 坐标的组成：
    * `groupId` 当前Maven构件隶属的项目名。实际开发中，项目往往会模块化开发，如spring-core,spring-aop等，他们都是Spring项目下不同的模块。命名方式与Java包名类似，通常是项目名+域名的反向书写。(必须)
    * `artifactId`：隶属项目中的模块名。(必须)
    * `version`：当前版本。(必须)
    * `packaging`：打包方式，如jar,war... 。默认为jar(必须)
    * `classifier`：帮助定义构建输出的一些附属构件。如spring-core.jar，还生成有文档javadoc.jar，源码sources.jar。
    
#### Maven构件在仓库中的存储路径
* 基于groupId准备路径，将句点分隔符转成路径分隔符，就是将  "."  转换成 "/" ; example： org.testng --->org/testng

* 基于artifactId准备路径，将artifactId连接到后面：org/testng/testng

* 使用version准备路径，将version连接到后面：org/testng/testng/5.8

* 将artifactId于version以分隔符连字号连接到后面：org/testng/testng/5.8/tesng-5.8

* 判断如果构件有classifier，就要在 第4项 后增加 分隔符连字号 再加上 classifier，org/testng/testng/5.8/tesng-5.8-jdk5

* 检查构件的extension，如果extension存在，则加上句点分隔符和extension，而extension是由packing决定的，org/testng/testng/5.8/tesng-5.8-jdk5.jar 

#### 特性    
* 构件具有依赖传递。例如：项目依赖构件A，而构件A又依赖B，Maven会将A和B都视为项目的依赖。
* 依赖之间存在版本冲突时，Maven会依据 “短路优先” 原则加载构件。如果引用的路径长度相同时，遵循“声明优先”的原则。此外，我们也可以在POM.XML中，使用<exclusions></exclusions>显式排除某个版本的依赖，以确保项目能够运行。
    * 项目依赖构件A和B，构件A → C → D(version:1.0.0)，构件B → D(version:1.1.0)，此时，Maven会优先解析加载D(version:1.1.0)。
    * 项目依赖构件A和B，构件A → D(version:1.0.0)， 构件B → D(version:1.1.0)，此时，Maven会优先解析加载D(version:1.0.0)。
* 构件的依赖范围。Maven在项目的构建过程中，会编译三套classpath，分别对应：编译期，运行期，测试期。而依赖范围，就是为构件指定它可以作用于哪套classpath。
       
### 仓库(repository)
分为本地仓库和远程仓库。先去本地仓库查询构件，如果没有就去远程仓库下载。

> maven提供了一个默认的全球中央仓库，解压lib/maven-model-builder-version.jar，可以发现pom-4.0.0.xml在org/apache/maven/model下,pom-4.0.0.xml中配置了该仓库，所有项目的pom.xml都会继承该xml,默认就使用了该全球中央仓库。用户可以在pom.xml中自定义远程仓库。

**中央仓库配置如下：**

```xml
<repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
</repository>
```
### 镜像仓库
配置远程仓库的镜像，所有针对原仓库的访问将转到镜像仓库，原仓库的url设置无效。

* conf/settings.xml中配置镜像仓库，镜像可以有多个。

* mirror的mirrorOf不能和任何一个mirror的id相同。  

* mirrorOf配置的是该镜像所匹配的远程仓库(id)。拦截对应的远程仓库，使所有针对原仓库的访问将转到镜像仓库。

* mirrorOf可以配置多个值，用逗号隔开

> 默认是没有配置mirror的，为了加速构件和插件的下载速度，我配置了一个阿里云的mirror，mirrorOf配置的是central，则id是central的仓库(中央仓库)将会转到阿里云的镜像下载构件，原仓库的url设置将失效。

```xml
<mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

* mirrorOf的值设置

    *  **rep1**:代表这个镜像仅仅针对rep1这个库。如果存在多个镜像的mirrorOf值都包含rep1，则按顺序匹配。

    * **\***:代表匹配所有的库。注意maven会优先匹配mirrorOf值与仓库id完全相同的镜像。例如id为rep1的仓库会优先匹配mirrorOf也为rep1的镜像，如果没有才会匹配mirrorOf为*的镜像。
    * **\*,!rep1**:匹配所有的库，除了rep1 

    * external:*:代表匹配任意不在localhost上的仓库，或不是基于文件的仓库。这个主要是看repository中的url判断的。

### 更改本地仓库位置

> maven下载的构件默认放在~/.m2/repository下面，其中~代表用户目录。可以在conf/settings.xml中自定义本地仓库的位置。

* 从文档注释中复制localRepository标签，粘贴，填入自定义目录

```xml
<localRepository>/Users/yuanming/maven_repo</localRepository>
```

* 备份settings.xml到maven_repo文件夹(自定义仓库文件夹)，在IDE中设置settings file的路径为备份的settings.xml的路径。以后更新maven，不用重新配置settings.xml。

## 参考资料

>**易枫**,[Maven之构件](https://www.cnblogs.com/Maple-leaves/p/5785885.html)
>**chengfangjunmy**,[Maven](https://blog.csdn.net/chengfangjunmy/article/details/61192021)
















