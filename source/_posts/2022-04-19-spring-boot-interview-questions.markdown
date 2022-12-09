---
layout: post
title: "(翻译)Spring Boot 面试题"
date: 2022-04-19 10:03:03 +0800
comments: true
categories: [Archives, Web Development]
keywords: Spring Boot, Spring, Interview
description: Spring Boot Interview Questions
---

#1. 概述
自推出以来，Spring Boot一直是Spring生态系统中的一个重要角色。这个项目凭借其自动配置能力使我们的生活变得更加轻松。

在本教程中，我们将介绍一些在求职面试中可能出现的与Spring Boot有关的最常见问题。

#2. 题目

##Q1. 什么是Spring Boot，其主要特点是什么？
Spring Boot本质上是一个建立在Spring框架之上的快速应用开发框架。凭借其自动配置和嵌入式应用服务器支持，再加上其享有的大量文档和社区支持，Spring Boot是迄今为止Java生态系统中最受欢迎的技术之一。

这里有几个突出的特点。

* **启动器** -- 一组依赖性描述符，可以一次性包括相关的依赖性
* **自动配置** -- 一种基于classpath上的依赖关系自动配置应用程序的方法
* **执行器** -- 获得生产就绪的功能，如监控
* **安全性**
* **日志**

译者点评：  

官方文档总结的特点：  

> * 	Create stand-alone Spring applications 
> *	Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files) 
> *	Provide opinionated 'starter' dependencies to simplify your build configuration 
> *	Automatically configure Spring and 3rd party libraries whenever possible 
> *	Provide production-ready features such as metrics, health checks, and externalized configuration 
> *	Absolutely no code generation and no requirement for XML configuration 

> * 创建独立的Spring应用程序

> * 直接嵌入Tomcat、Jetty或Undertow（不需要部署WAR文件）

> * 提供有主见的 "启动器 "依赖，以简化你的构建配置

> * 尽可能地自动配置Spring和第三方库

> * 提供生产就绪的功能，如度量、健康检查和外部化配置

> * 完全没有代码生成，也不需要XML配置


##Q2. Spring和Spring Boot之间的区别是什么？
Spring框架提供了多种功能，使Web应用的开发更加容易。这些功能包括依赖性注入、数据绑定、面向切面编程、数据访问等等。

多年来，Spring越来越复杂，这种应用所需的配置量可能令人生畏。这就是Spring Boot的用武之地--它使配置一个Spring应用程序变得轻而易举。

从本质上讲，Spring是没有主见的，而Spring Boot对平台和库有主见，让我们快速上手。

下面是Spring Boot带来的两个最重要的好处。

* 根据它在classpath上找到的库自动配置应用程序
* 提供生产中的应用程序常见的非功能特性，如安全或健康检查

请查看我们的其他教程，了解[普通Spring和Spring Boot的详细比较](https://www.baeldung.com/spring-vs-spring-boot)。

<!--more-->

##Q3. 我们如何用Maven设置Spring Boot应用程序？
我们可以像对待其他库一样，将Spring Boot纳入Maven项目中。不过，最好的方法是从spring-boot-starter-parent项目中继承，并声明对Spring Boot starters的依赖关系。这样做可以让我们的项目重用Spring Boot的默认设置。

继承spring-boot-starter-parent项目很简单，我们只需要在pom.xml中指定一个父元素:  

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0.RELEASE</version>
</parent>
```

我们可以在Maven中心找到最新版本的spring-boot-starter-parent。

使用启动器父项目很方便，但并不总是可行的。例如，如果我们公司要求所有项目都继承自标准POM，那么我们仍然可以通过使用[自定义父项目](https://www.baeldung.com/spring-boot-dependency-management-custom-parent)来受益于Spring Boot的依赖性管理。

##Q4. 什么是Spring Initializr？
Spring Initializr是一种创建Spring Boot项目的便捷方式。

我们可以去[Spring Initializr](https://start.spring.io/)网站，选择一个依赖管理工具（Maven或Gradle）、一种语言（Java、Kotlin或Groovy）、一种打包方案（Jar或War）、版本和依赖性，然后下载项目。

这为我们创建了一个骨架项目，节省了设置时间，这样我们就可以集中精力添加业务逻辑。

即使我们使用IDE的（如STS或带有STS插件的Eclipse）新项目向导来创建Spring Boot项目，它也会在底下使用Spring Initializr。

##Q5. 外面有哪些Spring Boot启动器？
每个启动器都扮演着一站式服务的角色，提供我们需要的所有 Spring 技术。其他所需的依赖项也会被拉进来，并以一致的方式进行管理。

所有启动器都在org.springframework.boot组下，其名称以spring-boot-starter-开头。这种命名模式使我们很容易找到启动器，特别是在使用支持按名称搜索依赖关系的IDE时。

在写这篇文章的时候，有超过50个启动器供我们使用。这里，我们将列出最常见的：

* **spring-boot-starter**：核心启动器，包括自动配置支持、日志和YAML
* **spring-boot-starter-aop**: 使用Spring AOP和AspectJ进行面向方面的编程
* **spring-boot-starter-data-jpa**：用于在Hibernate中使用Spring Data JPA。
* **spring-boot-starter-security**: 用于使用Spring Security。
* **spring-boot-starter-test**: 用于测试Spring Boot应用程序
* **spring-boot-starter-web**: 用于使用Spring MVC构建Web（包括RESTful）应用程序。

有关启动程序的完整列表，请参见该[资源库](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters)。

要找到关于Spring Boot启动程序的更多信息，请看[Spring Boot启动程序介绍](https://www.baeldung.com/spring-boot-starters)。


##Q6. 如何禁用一个特定的自动配置？
如果我们想禁用一个特定的自动配置，我们可以使用@EnableAutoConfiguration注解的exclude属性来设置它。

例如，这个代码片断禁用了DataSourceAutoConfiguration:  

```
// other annotations
@EnableAutoConfiguration(exclude = DataSourceAutoConfiguration.class)
public class MyConfiguration { }
```

如果我们用@SpringBootApplication注解启用了自动配置--该注解将@EnableAutoConfiguration作为元注解--我们可以用同名的属性禁用自动配置:  

```
// other annotations
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class MyConfiguration { }
```

我们也可以用spring.autoconfigure.exclude环境属性禁用自动配置。在application.properties文件中的配置与之前做的设置是做相同的事情:  

```
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

##Q7. 如何注册一个自定义的自动配置？
要注册一个自动配置类，我们必须在META-INF/spring.factories文件的EnableAutoConfiguration键下列出其完全限定名称:  

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.baeldung.autoconfigure.CustomAutoConfiguration
```

如果我们用Maven构建项目，该文件应放在resources/META-INF目录下，在打包阶段最终会出现在上述位置。

##Q8. 如何告诉自动配置在Bean存在时退缩？
为了指示自动配置类在Bean已经存在时退缩，我们可以使用 @ConditionalOnMissingBean 注解。

这个注解最值得注意的属性是:  

* value -- 要检查的Bean的类型
* name -- 要检查的bean的名字

当放在一个用@Bean装饰的方法上时，目标类型默认为该方法的返回类型:  

```
@Configuration
public class CustomConfiguration {
    @Bean
    @ConditionalOnMissingBean
    public CustomService service() { ... }
}
```

##Q9. 如何将Spring Boot Web应用部署为Jar和War文件？
传统上，我们将Web应用打包成WAR文件，然后将其部署到外部服务器上。这样做可以让我们在同一台服务器上安排多个应用程序。在CPU和内存稀缺的时候，这是一个节省资源的好方法。

但事情已经发生了变化。现在计算机硬件相当便宜，人们的注意力已经转向服务器配置。在部署过程中，配置服务器的一个小错误可能会导致灾难性的后果。

Spring通过提供一个插件，即spring-boot-maven-plugin，将网络应用打包成可执行的JAR，来解决这个问题。

要包含这个插件，只需在pom.xml中添加一个插件元素:  

```
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

有了这个插件，我们在执行打包阶段后会得到一个扁平的JAR。这个JAR包含所有必要的依赖，包括一个嵌入式服务器。因此，我们不再需要担心配置外部服务器的问题。

然后我们可以像运行普通的可执行JAR一样运行该应用程序。

注意，pom.xml文件中的打包元素必须设置为jar来构建JAR文件:  

```
<packaging>jar</packaging>
```

如果我们不包括这个元素，它也默认为jar。

要构建一个WAR文件，我们把包装元素改为war:  

```
<packaging>war</packaging>
```

并将容器的依赖关系从打包的文件中删除:  

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

在执行Maven打包阶段后，我们会有一个可部署的WAR文件。

##Q10. 如何在命令行应用程序中使用Spring Boot？
就像其他Java程序一样，Spring Boot命令行应用程序必须有一个main方法。

这个方法作为一个入口点，它调用`SpringApplication#run`方法来启动应用程序:  

```
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class);
        // other statements
    }
}
```

然后SpringApplication类启动Spring容器并自动配置Bean。

注意我们必须向run方法传递一个配置类，作为主要的配置源。按照惯例，这个参数就是入口类本身。

在调用运行方法后，我们可以像普通程序一样执行其他语句。

##Q11. 外部配置的可能来源是什么？
Spring Boot提供了对外部配置的支持，使我们能够在不同的环境中运行同一个应用程序。我们可以使用属性文件、YAML文件、环境变量、系统属性和命令行选项参数来指定配置属性。

然后，我们可以使用@Value注解、通过[@ConfigurationProperties注解](https://www.baeldung.com/configuration-properties-in-spring-boot)的绑定对象或环境抽象来访问这些属性。

##Q12. Spring Boot支持宽松的绑定是什么意思？
Spring Boot中的宽松绑定适用于[配置属性的类型安全绑定](https://www.baeldung.com/configuration-properties-in-spring-boot)。

通过宽松的绑定，属性的键不需要与属性名完全匹配。这样的环境属性可以用camelCase、kebab-case、snake_case，或者用大写字母，用下划线隔开单词。

例如，如果一个带有 @ConfigurationProperties 注解的 bean 类中的一个属性被命名为 myProp，它可以被绑定到这些环境属性中的任何一个：myProp、my-prop、my_prop 或 MY_PROP。

##Q13. Spring Boot DevTools的用途是什么？
Spring Boot开发者工具，或称DevTools，是一套使开发过程更容易的工具。

为了包括这些开发时的功能，我们只需要在pom.xml文件中添加一个依赖项:  

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

如果应用程序在生产中运行，spring-boot-devtools模块会自动禁用。重新打包的归档文件也默认排除了这个模块。所以，它不会给我们的最终产品带来任何开销。

默认情况下，DevTools应用适合于开发环境的属性。这些属性禁用了模板缓存，启用了网络组的调试日志，诸如此类。因此，我们有了这种合理的开发时配置，而无需设置任何属性。

只要classpath上的文件发生变化，使用DevTools的应用程序就会重新启动。这在开发中是一个非常有帮助的功能，因为它可以快速反馈修改情况。

默认情况下，静态资源，包括视图模板，不会引发重启。相反，资源变化会触发浏览器刷新。请注意，只有在浏览器中安装了LiveReload扩展，以便与DevTools包含的嵌入式LiveReload服务器进行交互时，这才会发生。

关于这一主题的进一步信息，请参见[Spring Boot DevTools概述](https://www.baeldung.com/spring-boot-devtools)。  

##Q14. 如何编写集成测试？
在为Spring应用程序运行集成测试时，我们必须有一个ApplicationContext。

为了让我们的生活更轻松，Spring Boot为测试提供了一个特殊的注解--@SpringBootTest。该注解从其classes属性所指示的配置类中创建一个ApplicationContext。

如果classes属性没有设置，Spring Boot会搜索主要的配置类。搜索从包含测试的包开始，直到找到一个用@SpringBootApplication或@SpringBootConfiguration注解的类。

有关详细说明，请查看我们的[Spring Boot测试教程](https://www.baeldung.com/spring-boot-testing)。  

##Q15. Spring Boot Actuator的用途是什么？
从本质上讲，Actuator通过启用生产就绪的功能，使Spring Boot应用程序活起来。这些功能使我们能够在应用程序在生产中运行时监控和管理它们。

将Spring Boot Actuator集成到一个项目中非常简单。我们所要做的就是在pom.xml文件中包括spring-boot-starter-actuator启动器:  

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Spring Boot Actuator可以使用HTTP或JMX端点来暴露操作信息。但大多数应用程序都会选择HTTP，其中端点的身份和/actuator前缀构成了一个URL路径。

下面是Actuator提供的一些最常见的内置端点。

* **env** 暴露环境属性
* **health** 显示应用程序的健康信息
* **httptrace** 显示HTTP跟踪信息
* **info** 显示任意的应用程序信息
* **metrics** 显示度量信息
* **loggers** 显示并修改应用程序中的记录器配置
* **mappings** 显示所有@RequestMapping路径的列表

请参考我们的[Spring Boot Actuator教程](https://www.baeldung.com/spring-boot-actuators)，了解详细情况。

##Q16. 配置Spring Boot项目时，属性和YAML哪个更好？
与属性文件相比，YAML有很多优点。

* 更加清晰，可读性更好
* 非常适合分层的配置数据，也可以用更好、更易读的格式来表示
* 支持 map、列表和标量类型
* 可以在同一个文件中包含多个[配置文件](https://www.baeldung.com/spring-profiles)（从Spring Boot 2.4.0开始，属性文件也可以这样做了）

但是，由于它的缩进规则，编写它可能有点困难，而且容易出错。

有关细节和工作样本，请参考我们的[Spring YAML与属性](https://www.baeldung.com/spring-yaml-vs-properties)教程。  

##Q17. Spring Boot提供了哪些基本注解？
Spring Boot提供的主要注释位于org.springframework.boot.autoconfigure及其子包中。

以下是一些基本的注释：

* **@EnableAutoConfiguration** -- 使Spring Boot在其classpath上寻找自动配置 Bean 并自动应用它们。
* **@SpringBootApplication** -- 表示Boot Application的主类。该注解将@Configuration、@EnableAutoConfiguration和@ComponentScan注解与它们的默认属性结合起来。

[Spring Boot注解](https://www.baeldung.com/spring-boot-annotations)提供了对这一主题的更多见解。

##Q18. 如何改变Spring Boot中的默认端口？
我们可以通过以下方式改变嵌入Spring Boot中的服务器的默认端口。

* 使用属性文件 -- 我们可以在application.properties（或application.yml）文件中使用属性server.port来定义。
* 通过编程 -- 在我们的主@SpringBootApplication类中，我们可以在SpringApplication实例上设置server.port。
* 使用命令行 -- 当以jar文件的形式运行应用程序时，我们可以将server.port设置为java命令参数:  

```
java -jar -Dserver.port=8081 myspringproject.jar
```

##Q19. Spring Boot支持哪些嵌入式服务器，以及如何改变默认值？
截至目前，Spring MVC支持Tomcat、Jetty和Undertow。Tomcat是Spring Boot的Web Starter支持的默认应用服务器。

Spring WebFlux支持Reactor Netty、Tomcat、Jetty和Undertow，其中Reactor Netty为默认。

在Spring MVC中，如果要改变默认，比方说改变为Jetty，我们需要排除Tomcat，在依赖关系中包括Jetty:  

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

同样，要把WebFlux的默认值改为UnderTow，我们需要排除Reactor Netty，并把UnderTow纳入依赖关系。

[比较Spring Boot中的嵌入式Servlet容器](https://www.baeldung.com/spring-boot-servlet-containers)有更多关于我们可以在Spring MVC中使用的不同嵌入式服务器的细节。  

##Q20. 为什么我们需要Spring Profiles？
在为企业开发应用程序时，我们通常要处理多种环境，如开发、QA和生产。这些环境的配置属性是不同的。

例如，我们可能在开发中使用嵌入式H2数据库，但开发中可能有专有的Oracle或DB2。即使DBMS在不同的环境中是相同的，URLs也肯定是不同的。

为了使这个问题简单明了，Spring提供了配置文件，以帮助分离每个环境的配置。因此，可以将这些属性保存在不同的文件中，如application-dev.properties和application-prod.properties，而不是通过编程来维护这些属性。默认的application.properties使用spring.profiles.active指向当前活动的配置文件，这样就可以获得正确的配置。

[Spring Profiles](https://www.baeldung.com/spring-profiles)给出了关于这个主题的全面观点。  

#3. 总结
本文介绍了技术面试中可能出现的关于Spring Boot的一些最关键问题。

我们希望这些问题能够帮助你找到理想的工作。

#4. 原文  

* [Spring Boot Interview Questions](https://www.baeldung.com/spring-boot-interview-questions)  




