---
title: maven(三):pom.xml
comment: true
tags:
  - maven
  - pom.xml
categories:
  - 工具学习
  - maven
date: 2018-08-04 19:44:00
---

用eclipse自动生成的maven web项目是没有xml声明的，可以手动声明。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.yangyuanming</groupId>
  <artifactId>myblog</artifactId>
  <packaging>war</packaging>
  <version>0.0.1-SNAPSHOT</version>
  <name>myblog Maven Webapp</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <!-- 单元测试 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
        </dependency>
  </dependencies>
  <build>
    <finalName>myblog</finalName>
  </build>
</project>

```


maven3中classpath:编译、运行、测试
依赖范围的作用：控制依赖和3种classpath的关系
[maven官方文档](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)中对dependency scope的介绍
Dependency Scope的值
* compile 默认的scope，



