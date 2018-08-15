---
title: eclipse创建ssm框架的maven项目
comment: true
tags:
  - maven项目
  - eclipse
categories:
  - 工具学习
  - eclipse
date: 2018-08-15 21:00:00
---
本篇文章介绍用eclipse创建maven项目(web)，该项目有多个模块组成，举例时只用一个模块，使用jetty作为web server。主要讲解ssm框架配置文件、maven配置文件。
## eclipse创建maven项目(web)
### 创建步骤
* 新建一个maven project
* 到了“Select an Archetype”这一步，选择webapp
* 到了“Enter a group id for the artifact.”，填写Group Id和Artifact Id，Group Id采用“网址反写+项目名”命名方式，Artifact Id采用“项目名-模块名”命名方式。本例子的Group Id为com.yangyuanming.demo,Artifact Id为demo-A。
* 将项目转化为动态的web项目。项目右键，选择"Properties",在“Project Facets”中勾选“Dynamic Web Module”
* 完善项目目录结构。新建的目录结构一般不够完整，需要自己新建。
![项目目录结构](https://res.yangyuanming.com/images/post/project-cate.jpg)

| 目录 | 说明 |
| --- | --- |
| src | 根目录，下有test和main目录  |
| src/main | 主目录，放置java代码和资源文件 |
| src/main/java | 放置java代码， |
| src/main/resources | 存放资源文件，譬如各种的spring，mybatis，log配置文件。 |
| src/main/resources/mapper | 存放dao中每个方法对应的sql，在这里配置，无需写daoImpl。 |
| src/main/resources/spring | 存放spring相关的配置文件，有dao service web三层。 |
| src/main/webapp | 用来存放前端的静态资源，如jsp js css,还有WEB-INF目录 |
| src/main/webapp/resources | 用来存放前端的静态资源，如jsp js css |
| src/main/webapp/WEB-INF | 外部浏览器无法访问的目录，可以把jsp放在这里，另外就是web.xml了。 |
| src/main/sql | 存放数据库创建文件 |
| src/test | 测试代码存放目录 |
| src/test/java | 测试java代码 |
| src/test/resources | 很少用到，一般都有，规范 |
| target | maven构建输出目录 |

* 修改部署目录。在项目属性的"Deployment Assembly"中，把测试相关的目录remove，部署项目时一般用不到。
* 完善src/main/java下的包目录。
![包结构](https://res.yangyuanming.com/images/post/project-pkg.jpg)


| 包名 | 名称 | 作用 |
| --- | --- | --- |
| dao | 数据访问层（接口） | 负责数据交互，可以是数据库操作，也可以是文件读写操作，<br>甚至是redis缓存操作。有人叫做dal或者数据持久层都差不多意思。<br>为什么没有daoImpl，因为我们用的是mybatis，所以可以直接<br>在配置文件中实现接口的每个方法。 |
| entity | 实体类 | 一般与数据库的表相对应，封装dao层取出来的数据为一个对象，<br>也就是我们常说的pojo，一般只在dao层与service层之间传输。 |
| dto | 数据传输层 | 用于service层与web层之间传输，为什么不直接用entity（pojo）？<br>其实在实际开发中发现，很多时间一个entity并不能满足我们的<br>业务需求，可能呈现给用户的信息十分之多，这时候就有了dto，<br>也相当于vo |
| service | 业务逻辑（接口） | 写我们的业务逻辑，也有人叫bll，在设计业务接口时候应该站在<br>“使用者”的角度。 |
| impl | 业务逻辑（实现) | 业务接口实现，一般事务控制是写在这里 |
| web | 控制器 | 	controller，springmvc就是在这里发挥作用的 |


 







<!--more-->
## 参考资料
>**李奕锋**,[手把手教你整合最优雅SSM框架：SpringMVC + Spring + MyBatis](https://blog.csdn.net/qq598535550/article/details/51703190)