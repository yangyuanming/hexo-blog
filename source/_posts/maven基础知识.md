---
title: maven核心知识
toc: true
comment: true
tags:
  - maven
  - maven核心知识
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

-------

## 构件、仓库、镜像仓库

### 构件

pom.xml配置文件中的dependency，包含构件坐标等信息。
    
### 仓库(repository)
分为本地仓库和远程仓库。先去本地仓库查询构件，如果没有就去远程仓库下载。


> maven提供了一个默认的全球中央仓库，在lib目录下jar包"maven-model-builder"中的pom-4.0.0.xml中配置了该仓库，所有项目的pom.xml都会继承该xml,默认就使用了该全球中央仓库


![maven-model-builder](https://res.yangyuanming.com/images/post/maven-model-builder.png)

![maven-pom.4.0.0.xml](https://res.yangyuanming.com/images/post/maven-pom.4.0.0.xml.png)


### 镜像仓库
配置远程仓库的镜像，所有针对原仓库的访问将转到镜像仓库，原仓库的url设置无效。


* conf/settings.xml中配置镜像仓库，镜像可以有多个。
* mirror的mirrorOf不能和任何一个mirror的id相同。
* mirrorOf配置的是该镜像所匹配的远程仓库(id)。拦截对应的远程仓库，使所有针对原仓库的访问将转到镜像仓库。
* mirrorOf可以配置多个值，用逗号隔开

> 我配置了一个阿里云的mirror，mirrorOf配置的是central，则id是central的仓库将会转到阿里云的镜像下载构件，原仓库的url设置将失效。

![maven-settings](https://res.yangyuanming.com/images/post/maven-settings.png)


* mirrorOf的值设置

    *  **rep1**:代表这个镜像仅仅针对rep1这个库。如果存在多个镜像的mirrorOf值都包含rep1，则按顺序匹配。

    * **\***:代表匹配所有的库。注意maven会优先匹配与仓库id完全的镜像。例如id为rep1的仓库会优先匹配mirrorOf也为rep1的镜像，如果没有才会匹配mirrorOf为*的镜像。
    * **\*,!rep1**:匹配所有的库，除了rep1 

    * **external:\***:代表匹配任意不在localhost上的仓库，或不是基于文件的仓库。这个主要是看repository中的url判断的。

### 更改仓库位置

> maven下载的构件默认放在/usernanme/.m2/repository下面，其中username代表用户目录。可以在conf/settings.xml中自定义仓库的位置。

* 从注释中复制localRepository标签，填入自定义目录
* 备份settings.xml到maven_repo文件夹。以后更新maven，不用重新配置settings.xml，复制一份回conf文件夹下即可。

![更改仓库位置](https://res.yangyuanming.com/images/post/更改仓库位置.png)

-------

## maven构建生命周期

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

    
-------

## 参考资料

>http://blog.sina.com.cn/s/blog_e01142dc0102wup3.html










