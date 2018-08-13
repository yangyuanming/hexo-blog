---
title: maven(六):依赖管理和插件管理
comment: true
tags:
  - maven
  - maven依赖&插件管理
categories:
  - 工具学习
  - maven
date: 2018-08-13 15:29:00
---
## 依赖管理
### 继承dependencyManagement
通过继承可以减少子模块的重复配置，但会存在继承多余依赖的问题。maven提供了                               `<dependencyManagement>`,在**`<dependencyManagement>`元素下的依赖声明不会引入实际的依赖**，不过他能够管理`<denpendencies>`下的依赖。父模块定义`<dependencyManagement>`，子模块可以继承父模块，使用父模块依赖配置，同时保证子模块依赖使用的灵活度。
<!--more-->
举个例子，在父模块中定义：

```xml
<properties>
    <springframework.version>2.5.6</springframework.version>
    <junit.version>4.7</junit.version>
    <javax.mail.version>1.4.1</javax.mail.version>
</properties>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${springframework.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${springframework.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${springframework.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${springframework.version}</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
         <dependency>
            <groupId>javax.mail</groupId>
            <artifactId>mail</artifactId>
            <version>${javax.mail.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```
上面在父模块的`</dependencyManagement>`中定义了`<dependencies>`,依赖的版本号做了归并处理，方便后期修改。该配置不会引入依赖，子项目可以继承该配置。
下面是子项目的配置

```xml
<properties>
    <greenmail.version>1.3.1b</greenmail.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
    <dependency>
        <groupId>javax.icegreen</groupId>
        <artifactId>greenmail</artifactId>
        <version>${greenmail.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```
可以看到上面子模块的pom.xml的配置，子模块继承了父模块`<dependencyManagement>`的配置，但这只是依赖的配置，并不会真的引入依赖。对于父模块`<dependencyManagement>`已经配置的依赖，子模块仍然需要显式引入，但版本号不需要再定义，会直接使用父模块中定义的。相比于在父模块引入依赖，子模块全部继承的方式，这种方式更加灵活，可以自由引入所需依赖，虽然增加了显式引入的工作量，但避免了引入多余依赖。在多模块的项目中，这种方式可以灵活引入依赖和统一管理依赖配置。
> `<dependencyManagement>`是多模块项目中必不可少的，因为它可以有效地帮助我们维护依赖的一致性

### 引进dependencyManagement
除了继承的方式引入`<dependencyManagement>`，还可以引入**import** scope依赖的方式。被引进的依赖类型必须为pom，并且pom.xml中配置了`<dependencyManagement>`，在引进方的pom.xml中的`<dependencyManagement>`中才可以引入 **import** scope依赖，引进方其实引进的是被引进方pom.xml中`<dependencyManagement>`的配置。
为什么要使用引入import scope依赖的方式?
maven和java一样，无法使用多重继承。如果遇到一个模块很多的项目，如果都继承同一个父模块，那么将会在父模块的`<dependencyManagement>`中配置大量的依赖。如果想进行将这些依赖分类以进行更加清晰的管理，按照这种方式是不行的。在子模块`<dependencyManagement>`中引入import scope依赖可以解决这个为题。步骤如下：
* 在专门管理依赖的模块的pom中定义`<dependencyManagement>`
* 在需要引入依赖管理的模块的`<dependencyManagement>`以import scope的方式引入该pom

可以看出，第一步跟使用继承的方式一样，定义专门管理依赖的pom，定义`<dependencyManagement>`，只是第二步不同，依赖管理引入方不是通过继承的方式引入依赖管理的配置，而是在`<dependencyManagement>`中以引入**import** scope依赖的方式获得依赖管理的配置。下面例子示范：  
定义用于依赖管理的pom
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.yangyuanming</groupId>
  <artifactId>sample-dependency-infrastructure</artifactId>
  <packaging>pom</packaging>
  <version>1.0-SNAPSHOT</version>
  <dependencyManagement>
    <dependencies>
        <dependency>
          <groupId>junit</groupId>
          <artifactid>junit</artifactId>
          <version>4.8.2</version>
          <scope>test</scope>
        </dependency>
        <dependency>
          <groupId>log4j</groupId>
          <artifactid>log4j</artifactId>
          <version>1.2.16</version>
        </dependency>
    </dependencies>
  </dependencyManagement>
</project>
```
在需要引入依赖管理的模块中采用import scope的方式导入该pom

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
          <groupId>com.yangyuanming</groupId>
          <artifactid>sample-dependency-infrastructure</artifactId>
          <version>1.0-SNAPSHOT</version>
          <type>pom</type>
          <scope>import</scope>
        </dependency>
    </dependencies>
  </dependencyManagement>

  <dependency>
    <groupId>junit</groupId>
    <artifactid>junit</artifactId>
  </dependency>
  <dependency>
    <groupId>log4j</groupId>
    <artifactid>log4j</artifactId>
  </dependency>
```
跟继承一样，导入的`<dependencyManagement>`，并不会真的引入依赖，还是需要再显式引入依赖。相比于继承的方式，采用import scope依赖的方式可以更好分类管理依赖。父模块的pom将会更清晰，不再对依赖进行管理，交给专门的packaging为pom的来管理依赖。我们可以定义多个用于管理依赖的pom，进行更加细化的依赖管理,可以满足多模块的需求，实现清晰化管理依赖。
### dependencyManagement和dependencies区别
* `<dependencies>`相对于`<dependencyManagement>`，所有生命在dependencies里的依赖都会自动引入，并默认被所有的子项目继承。
* `<dependencyManagement>`里只是声明依赖的配置，并不自动实现引入，因此子项目需要显式声明需要用的依赖。如果不在子项目中声明依赖，是不会引入该依赖的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会引入该依赖，并且version和scope都读取自父pom;另外如果子项目中指定了版本号，那么将不会使用父pom中对于该依赖的配置。前面说的是继承的方式，采用import scope依赖也遵循这个原则。

## 插件管理
maven提供了`<dependencyManagement>`进行依赖管理，同时也提供了`<pluginManagement>`帮忙管理插件。在`<pluginManagement>`中定义的只是插件的配置，不会真的引入插件，需要显式引入才行。这跟使用`<dependencyManagement>`的道理是类似的。
在父模块中定义:

```xml
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>2.1.1</version>
                <executions>
                    <execution>
                        <id>attach-sources</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>jar-no-fork</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```
子模块

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
通过继承的方式，子模块继承了父模块的插件管理配置，子模块中声明的插件如果没有版本号，根据groupId和artifactId去匹配插件管理的配置，如果有匹配的会将相应的配置应用到该插件。上面例子的maven-source-plugin只有声明的groupId和artifactId，那么会在插件管理中查找相应的配置，此例中，父模块中`<pluginManagement>`中可以找到匹配的插件，那么相应的配置信息也会被该插件使用。  

一个项目中所有模块中使用的依赖配置应该是一样的，应该要进行统一，但插件配置很多时候是不一样的。对于插件管理，不能随便将一个插件相关的配置都提取到父模块的pom中，因为所有的子模块都会继承。一般来说，同一项目中所有模块使用的插件的版本号应该要是相同的，不然容易出错，插件版本号推荐在父模块的`<pluginManagement>`进行统一配置。对于插件的其他配置信息，只有在所有的子模块中都适用的情况下才能在父模块的`<pluginManagement>`进行统一配置。
**关于`<pluginManagement>`，Maven并没有提供与import scope依赖类似的方式管理，那我们只能借助继承关系**，不过好在一般来说插件配置的数量远没有依赖配置那么多，因此这也不是一个问题。

## 参考资料
**许晓斌**，[Maven实战（三）——多模块项目的POM重构](http://www.infoq.com/cn/news/2011/01/xxb-maven-3-pom-refactoring)
**lofty**，[Maven——聚合与继承](https://www.cnblogs.com/wangwei-beijing/p/6535084.html)