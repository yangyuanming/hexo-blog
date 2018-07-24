---
title: maven生命周期和插件
comment: true
tags:
  - maven生命周期
  - maven插件
categories:
  - 工具学习
  - maven
abbrlink: b37b3bbf
date: 2018-07-24 07:30:00
---

## maven生命周期

maven抽象出了3套生命周期，其具体实现是依赖于[maven插件](http://maven.apache.org/plugins/index.html)。每套生命周期是相互独立的，都由一组阶段(Phase)组成。每套生命周期中的阶段是有顺序的，后面阶段依赖于前面的阶段，执行后面阶段会自动执行之前的阶段，但不会触发不同生命周期的阶段。

**下面是三个生命周期及其包含的阶段。**

### Clean Lifecycle 
清理项目，在进行真正的构建之前进行一些清理工作。

*  pre-clean     执行clean前需要完成的工作  

*  clean     clean上一次构建生成的所有文件  

*  post-clean    执行clean后需要立刻完成的工作  
 
    
>这里的clean就是指的mvn clean。在一套生命周期内，运行某个阶段会自动按序运行之前阶段，mvn clean=mvn pre-clean clean

### Default Lifecycle
构建的核心部分，编译，测试，打包，部署等等。

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

## 插件


## 参考资料

>http://blog.sina.com.cn/s/blog_e01142dc0102wup3.html


