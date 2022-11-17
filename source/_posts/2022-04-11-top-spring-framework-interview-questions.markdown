---
layout: post
title: "(翻译)常见 Spring 框架面试题"
date: 2022-04-11 10:07:29 +0800
comments: true
categories: [Archives, Web Development]
keywords: Spring, Interview
description: Top Spring Framework Interview Questions
---

# 1.概述

在本教程中，我们将看看在求职面试中可能出现的一些最常见的与 Spring 有关的问题。

# 2. Spring Core

## Q1. 什么是 Spring 框架？
Spring是开发 Java 企业版应用程序最广泛使用的框架。此外，Spring 的核心功能可用于开发任何 Java 应用程序。

我们使用它的扩展功能在 Jakarta EE 平台之上构建各种网络应用。我们也可以在简单的独立应用程序中使用它的依赖注入功能。

## Q2. 使用 Spring 的好处是什么？
Spring 的目标是使 Jakarta EE的 开发更容易，所以我们来看看它的好处:  

* **轻量级**--在开发中使用该框架的开销很小
* **反转控制（IoC）**--Spring 容器负责连接各种对象的依赖关系，而不是创建或寻找依赖对象
* **面向切面编程（AOP）**--Spring支持AOP，将业务逻辑与系统服务分开
* **IoC容器**--管理 Spring Bean 的生命周期和项目特定的配置
* **MVC框架**--用于创建 Web 应用程序或 RESTful Web服务，能够返回XML/JSON响应
* **事务管理** -- 通过使用 Java 注解或 Spring Bean 的 XML 配置文件，减少JDBC操作、文件上传等方面的模板代码量
* **异常处理**--Spring 提供了一个方便的 API，用于将特定技术的异常转化为未检查的异常

## Q3. 您知道哪些 Spring 子项目？简要描述一下吧。

* **核心**--提供框架基础部分的关键模块，如 IoC 或 DI
* **JDBC**--实现了一个 JDBC 抽象层，不需要为特定的供应商数据库进行 JDBC 编码
* **ORM 集成** -- 为流行的对象关系映射 API 提供集成层，如 JPA、JDO 和 Hibernate
* **Web**--一个面向网络的集成模块，提供多部分文件上传、Servlet监听器和面向网络的应用程序上下文功能
* **MVC 框架**--一个实现模型-视图-控制器设计模式的 Web 模块
* **AOP 模块**--面向切面编程实现，允许定义干净的方法拦截器和切点
<!--more-->
## Q4. 什么是依赖性注入？
依赖注入是控制反转（IoC）的一个方面，它是一个一般的概念，即我们不手动创建我们的对象，而是描述它们应该如何被创建。然后IoC容器将在需要时实例化所需的类。

更多细节，请看[这里](https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)。  

## Q5. 我们如何在 Spring 中注入 Bean？
为了注入Spring Bean，有几个不同的选择。

* Setter 方法注入
* 构造器注入
* 字段注入

配置可以使用XML文件或注解来完成。

更多细节，请查看[这篇文章](https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)。  

## Q6. 哪种方式是注入 Bean 的最佳方式，为什么？
推荐的方法是对强制性的依赖使用构造函数参数，对选择性的依赖使用设置器。这是因为构造函数注入允许向不可变的字段注入值，使测试更容易。

## Q7. BeanFactory 和 ApplicationContext 之间的区别是什么？
BeanFactory 表示一个提供和管理 Bean 实例的容器接口。默认的实现是在调用 `getBean()`时懒惰地将 Bean 实例化。

相比之下，ApplicationContext 表示一个容纳了应用程序中元数据和 bean 等所有信息的容器接口。它也扩展了 BeanFactory 接口，但默认实现是在应用程序启动时迫切地实例化 Bean。然而，这种行为可以为单个 Bean 重写。

关于所有的区别，请参考[文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html)。  

译者点评：正如它们的名字一样，BeanFactory 是代表它是 Bean 工厂，它的主要功能是像工厂一样生产管理 Bean；ApplicationContext 则表示它是应用上下文，它是 BeanFactory 的子接口，扩展了很多面向应用的功能，它提供了国际化支持和框架事件体系，更易于创建实际应用。  

## Q8. 什么是 Spring Bean？
Spring Bean 是由 Spring IoC 容器初始化的 Java 对象。

译者点评：个人觉得 Spring 官方文档的解释更好:  

> In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. 

在 Spring 中， 那些被 Spring IoC 容器管理并形成应用骨架的对象称为 beans。  

## Q9. Spring 框架中默认的 Bean 作用域是什么？

默认情况下，Spring Bean 被初始化为一个单例。

## Q10. 如何定义一个 Bean 的作用域？
为了设置Spring Bean的作用域，我们可以使用 `@Scope` 注解或在 XML 配置文件中使用 `"scope"` 属性。请注意，有五个支持的作用域。

* Singleton
* Prototype
* Request
* Session
* Global-session

关于差异，请看[这里](https://docs.spring.io/spring/docs/3.0.0.M4/reference/html/ch03s05.html)。  

## Q11. 单例 bean 是线程安全的吗？
单例 Bean 不是线程安全的，因为线程安全是关于执行的，而单例是一种专注于创建的设计模式。线程安全只取决于Bean的实现本身。

## Q12. Spring Bean 的生命周期是怎样的？
首先，Spring Bean 需要根据 Java 或 XML Bean 定义进行实例化。它可能还需要执行一些初始化，使其进入可用状态。之后，当不再需要该 Bean 时，它将被从IoC 容器中删除。

所有初始化方法的整个周期显示在图片中（[来源](http://www.dineshonjava.com/2012/07/bean-lifecycle-and-callbacks.html)）。

![Spring Bean Life Cycle](https://www.baeldung.com/wp-content/uploads/2017/06/Spring-Bean-Life-Cycle.jpg)  

译者点评：作者的回答在实际面试时可能会略显单薄，这里我尝试补充一下：  

首先应用上下文会读取配置元数据，然后解析元数据用 BeanDefinition 来表达 Bean 定义；之后先实例化实现了 BeanFactoryPostProcessor 接口的 Bean，排序后调用它们依次处理 Bean 定义信息；

如果 Bean 定义中存在实现了 InstantiationAwareBeanPostProcessor 接口的 Bean，则调用它的 `postProcessBeforeInstantiation` 方法；然后实例化 Bean，之后又调用 InstantiationAwareBeanPostProcessor 的 `postProcessAfterInstantiation` 和 `postProcessPropertyValues` 方法；

如果 Bean 实现了 BeanNameAware 接口则调用它的 `setBeanName` 方法；
如果 Bean 实现了 BeanFactoryAware 接口则调用它的 `setBeanFactory` 方法；
如果 Bean 实现了 ApplicationAware 接口则调用它 `setApplicationContext` 方法；

调用 BeanPostProcessor 的 `postProcessBeforeInitialization` 方法；

如果 Bean 实现 InitializingBean 的 `afterPropertiesSet` 方法；

调用 `init-method` 属性 或 `@postConstruct` 注解设置的初始化方法；

调用 BeanPostProcessor 的 `postProcessAfterInitialization` 方法；

如果 Bean 的作用域是原型则直接将准备就绪 Bean 对象返回给调用者；

如果 Bean 的作用域是单例则要先把它保存到容器的 Bean 对象缓存池中，然后将准备就绪的对象返回给调用者；

之后容器销毁时，如果 Bean 实现了 DisposableBean 的 `destroy` 接口；

调用 `destroy-method` 属性或 `@preDestroy` 注解设置的销毁方法；

## Q13. 什么是基于 Java 的 Spring 配置？
它是一种以类型安全的方式配置基于 Spring 的应用程序的方法。它是基于 XML 的配置的替代品。

另外，要把一个项目从 XML 配置迁移到 Java 配置，请参考[这篇文章](https://www.baeldung.com/spring-xml-vs-java-config)。  

## Q14. 我们可以在一个项目中拥有多个 Spring 配置文件吗？
是的，在大型项目中，建议拥有多个 Spring 配置以提高可维护性和模块化程度。

我们可以加载多个基于 Java 的配置文件:  

```
@Configuration
@Import({MainConfig.class, SchedulerConfig.class})
public class AppConfig {
```

或者我们可以加载一个XML文件，该文件将包含所有其他配置:  

```
ApplicationContext context = new ClassPathXmlApplicationContext("spring-all.xml");
```

在这个XML文件中，我们将有以下内容:

```
<import resource="main.xml"/>
<import resource="scheduler.xml"/>
```

## Q15. 什么是 Spring Security？
Spring Security 是 Spring 框架的一个独立模块，主要是在 Java 应用程序中提供认证和授权方法。它还负责处理大多数常见的安全漏洞，如CSRF攻击。

要在Web应用程序中使用Spring Security，我们可以通过简单的注解 @EnableWebSecurity 来开始。

欲了解更多信息，我们有一系列与[安全](https://www.baeldung.com/security-spring)有关的文章。

## Q16. 什么是 Spring Boot？
Spring Boot 是一个提供了一套预配置框架以减少模板配置的项目。这样，我们就可以用最少的代码来启动和运行一个 Spring 应用程序。

## Q17. 请说出 Spring 框架中使用的一些设计模式？

* **单例模式**--单例范围的 bean
* **工厂模式**-- Bean 工厂类
* **原型模式**（Prototype Pattern）--原型作用域的 Bean。
* **适配器模式**--Spring Web和Spring MVC
* **代理模式**--支持Spring面向切面编程
* **模板方法模式**--JdbcTemplate、HibernateTemplate等。
* **前端控制器**--Spring MVC DispatcherServlet
* **数据访问对象**--支持Spring DAO
* **模型视图控制器**--Spring MVC

译者点评：可能按照创建型、结构型和行为模式从 GoF 23 个设计模式中匹配会容易记忆点：

* **创建型模式**
  
  * 生成器(Builder)
  * 工厂(Factory)
  * 原型(Prototype)
  * 单例(Singleton)
  
* **结构型模式**
	
	* 适配器(Adatper)
 	* 组成(Composite)
 	* 外观(Facade)
 	* 代理(Proxy)
 
* **行为模式**
	
	* 职责链(Chain of responsibility)
	* 迭代器(Iterator)
	* 策略(Strategy)
	* 模板(Template Method)

## Q18. 原型作用域是如何工作的？
原型作用域意味着每次我们需要 Bean 的一个实例时，Spring都会创建一个新的实例并返回它。这与默认的单例作用域不同，在单例作用域中，每个 Spring IoC 容器只实例化一个对象实例。


# 3. Spring Web MVC

## Q19. 如何在 Spring Bean 中获取 ServletContext 和 ServletConfig 对象？
我们可以通过实现 Spring-aware 的接口来做到这一点。[这里](http://www.buggybread.com/2015/03/spring-framework-list-of-aware.html)有完整的列表。

我们也可以在这些 Bean 上使用 @Autowired 注解:  

```
@Autowired
ServletContext servletContext;

@Autowired
ServletConfig servletConfig;
```

## Q20. 什么是 Spring MVC 中的控制器？
简单地说，所有由 DispatcherServlet 处理的请求都会被引导到带有 `@Controller` 注解的类。每个控制器类都将映射一个或多个请求到方法中，这些方法处理和执行携带输入的请求。

退一步讲，我们建议看一下[典型的Spring MVC架构中的前端控制器](https://www.baeldung.com/spring-controllers)的概念。  

译者点评：个人觉得原文这题给的答案不是很好，题目是问什么是 Spring MVC 中的控制器，答案应该重点解释是什么，而且说所有由 DispatcherServlet 处理的请求都会被引导到带有 `@Controller` 注释的类太绝对了，例如 BeanNameUrlHandlerMapping 就支持将 URL 映射到对应名字的 bean。

Spring MVC 中的控制器是处理请求的组件，充当模型-视图-控制器模式中的控制器角色，通常是由 `@Controller` 的类。

## Q21. `@RequestMapping` 注解是如何工作的？
`@RequestMapping` 注解用于将 Web 请求映射到Spring 控制器方法。除了简单的用例之外，我们还可以用它来映射 HTTP 头，用 `@PathVariable` 来绑定URI的部分内容，以及用 URI 参数和 `@RequestParam` 注解来工作。

关于 `@RequestMapping` 的更多细节可以在[这里](https://www.baeldung.com/spring-requestmapping)找到。

更多关于 Spring MVC 的问题，请查看我们关于 [Spring MVC 面试问题](https://www.baeldung.com/spring-mvc-interview-questions)的文章。

译者点评：个人觉得这个题出得不怎么好，给的答案也有点答非所问。单纯说 `@RequestMapping` 注解是如何工作的？那答案应该重点说`@RequestMapping` 注解会将映射请求所需的匹配信息保留到 Java 运行时，出题者更多想考察的应该是 Spring MVC 是如何将请求映射到 `@RequestMapping` 注解的方法。

# 4. Spring Data Access

## Q22. 什么是 Spring JdbcTemplate 类以及如何使用它？
Spring JDBC 模板是数据库操作主要的API，我们可以通过它访问我们感兴趣的数据：

* 创建和关闭连接
* 执行语句和存储过程调用
* 遍历结果集并返回结果


为了使用它，我们需要定义 DataSource 的简单配置：

```
@Configuration
@ComponentScan("org.baeldung.jdbc")
public class SpringJdbcConfig {
    @Bean
    public DataSource mysqlDataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/springjdbc");
        dataSource.setUsername("guest_user");
        dataSource.setPassword("guest_password");
 
        return dataSource;
    }
}
```

如需进一步解释，请查看[这篇快速文章](https://www.baeldung.com/spring-jdbc-jdbctemplate)。  

## Q23. 如何在 Spring 中启用事务，其好处是什么？
有两种不同的方式来配置事务--使用注解或使用面向切面编程（AOP）--每种方式都有其优势。

根据官方文档，以下是使用 Spring Transactions 的好处。

* 在不同的事务API中提供一致的编程模型，如 JTA、JDBC、Hibernate、JPA 和 JDO
* 支持声明式事务管理
* 与 JTA 等一些复杂的事务 API 相比，为编程式事务管理提供了更简单的 API
* 与 Spring 的各种数据访问抽象结合得非常好

## Q24. 什么是 Spring DAO？
Spring 数据访问对象（DAO）是 Spring 为 JDBC、Hibernate 和 JPA 等数据访问技术提供的支持，其工作方式一致且简单。

有一个[完整的系列](https://www.baeldung.com/persistence-with-spring-series/)讨论了 Spring 的持久性，提供了一个更深入的解释。  

## 5. Spring Aspect-Oriented Programming

## Q25. 什么是面向切面编程（AOP）？
切面使跨领域的关注点模块化，如事务管理，它跨越多种类型和对象，在不修改受影响的类的情况下为已有的代码增加额外的行为。

下面是[基于切面的执行时间记录](https://www.baeldung.com/spring-aop-annotation)的例子。

## Q26. 什么是AOP中的Aspect、Advice、Pointcut和JoinPoint？

* Aspect -- 一个实现交叉关注的类，如事务管理
* Advice --当应用程序运行到与 Pointcut 相匹配的特定 JoinPoint 时被执行的方法
* Pointcut -- 一组与 JoinPoint 匹配的正则表达式，以确定是否需要执行 Advice
* JoinPoint -- 程序执行过程中的一个点，例如一个方法的执行或一个异常的处理

## Q27. 什么是编织？
根据[官方文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html)，编织是一个将各个切面与其他应用程序类型或对象联系起来以创建一个增强对象的过程。这可以在编译时、加载时或运行时完成。Spring AOP 和其他纯 Java AOP 框架一样，在运行时执行织入。  

#6. 总结

在这篇长文中，我们已经探讨了一些关于 Spring 技术面试最重要的问题。

我们希望这篇文章能对即将到来的Spring面试有所帮助。祝您好运!  

# 7. 原文

* [Top Spring Framework Interview Questions](https://www.baeldung.com/spring-interview-questions)  


