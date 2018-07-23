---
title: maven基础知识
toc: true
comment: true
tags:
  - maven
  - maven基础知识
categories:
  - 工具学习
abbrlink: fcecdefc
date: 2018-07-22 11:42:00
---
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

## 构件、仓库、镜像仓库
1. 构件：pom.xml配置文件中的dependency，包含构件坐标等信息。
2. 仓库(repository)：本地仓库和远程仓库。先去本地仓库查询构件，如果没有就去远程仓库下载。
> maven提供了一个默认的全球中央仓库，在lib目录下jar包"maven-model-builder"中的pom-4.0.0.xml中配置了该仓库，所有项目的pom.xml都会继承该xml,默认就使用了该全球中央仓库

<div align=center >![maven-model-builder-w370](https://res.yangyuanming.com/images/post/maven-model-builder.png)

<div align=center>![maven-pom.4.0.0.xml-w370](https://res.yangyuanming.com/images/post/maven-pom.4.0.0.xml.png)


3. 镜像仓库：配置远程仓库的镜像，所有针对原仓库的访问将转到镜像仓库，原仓库的url设置无效。

* conf/settings.xml中配置镜像仓库，镜像可以有多个。

<div align=center>![maven-settings-w370](https://res.yangyuanming.com/images/post/maven-settings.png)

* mirror的mirrorOf不能和任何一个mirror的id相同。
* mirrorOf配置的是该镜像所匹配的远程仓库(id)。拦截对应的远程仓库，使所有针对原仓库的访问将转到镜像仓库。
* mirrorOf可以配置多个值，用逗号隔开

> 我配置了一个阿里云的mirror，mirrorOf配置的是central，则id是central的仓库将会转到阿里云的镜像下载构件，原仓库的url设置将失效。

* mirrorOf的值设置

> **rep1**:代表这个镜像仅仅针对rep1这个库

> **\***:代表匹配所有的库

> **\*,!rep1**:匹配所有的库，除了rep1 

> **external:\***:代表匹配任意不在localhost上的仓库，或不是基于文件的仓库。这个主要是看repository中的url判断的。

4. 更改仓库位置




