---
abbrlink: '0'
---
---
title: maven常用构建命令及库的概念
comment: true
tags:
  - maven构建命令
  - maven库
categories:
  - 工具学习
  - maven
abbrlink: fcecdefc
date: 2018-07-22 11:42:00
    ---
[TOC]

## maven常用构建命令
1. 查看maven版本：mvn -v
2. 编译：mvn compile
3. 打包：mvn package
4. 测试：mvn test
5. 删除target目录：mvn clean
6. 安装jar包到本地仓库：mvn install  
<!--more-->
7. 自动建立目录结构的两种方式：

* mvn archetype:generate 按指示输入信息

* mvn archetype:generate  
-DgroupId=网址+项目名  
-DartifactId=项目名-模块名  
-Dversion=版本号  
-Dpackage=代码包名

-------

## 构件、仓库、镜像仓库

### 构件

pom.xml配置文件中的dependency，包含构件坐标等信息。
    
### 仓库(repository)
分为本地仓库和远程仓库。先去本地仓库查询构件，如果没有就去远程仓库下载。


> maven提供了一个默认的全球中央仓库，解压lib/maven-model-builder-version.jar，可以发现pom-4.0.0.xml在org/apache/maven/model下,pom-4.0.0.xml中配置了该仓库，所有项目的pom.xml都会继承该xml,默认就使用了该全球中央仓库。

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

    * **external:\***:代表匹配任意不在localhost上的仓库，或不是基于文件的仓库。这个主要是看repository中的url判断的。

### 更改本地仓库位置

> maven下载的构件默认放在~/.m2/repository下面，其中~代表用户目录。可以在conf/settings.xml中自定义本地仓库的位置。

* 从文档注释中复制localRepository标签，粘贴，填入自定义目录

```xml
<localRepository>/Users/yuanming/maven_repo</localRepository>
```

* 备份settings.xml到maven_repo文件夹(自定义仓库文件夹)，在IDE中设置settings file的路径为备份的settings.xml的路径。以后更新maven，不用重新配置settings.xml。
















