---
layout: post
title: "MyBatis Interview Questions"
date: 2022-04-26 10:18:15 +0800
comments: true
categories: [Archives, Web Development]
keywords: MyBatis, ORM, Mapper, SQL
description: MyBatis Interview Questions
published: false
---

#1.使用 JDBC 的基本步骤

使用 JDBC 大致可以分为六个步骤：

* 1. 加载驱动程序；
* 2. 获取数据连接；
* 3. 创建一个 Statement 对象；
* 4. 操作数据库，实现增删改查；
* 5. 获取结果集；
* 6. 关闭资源；

#2.什么是 Mybatis？

> MyBatis is a first class persistence framework with support for custom SQL, stored procedures and advanced mappings. MyBatis eliminates almost all of the JDBC code and manual setting of parameters and retrieval of results. MyBatis can use simple XML or Annotations for configuration and map primitives, Map interfaces and Java POJOs (Plain Old Java Objects) to database records.  

MyBatis是一个一流的持久性框架，支持自定义SQL、存储过程和高级映射。 MyBatis消除了几乎所有的JDBC代码以及参数的手动设置和结果的检索。MyBatis可以使用简单的XML或注释进行配置，并将基本类型、Map 接口和Java POJOs（Plain Old Java Objects）映射到数据库记录。

#3.什么是 ORM?

>Object–relational mapping (ORM, O/RM, and O/R mapping tool) in computer science is a programming technique for converting data between type systems using object-oriented programming languages.  -- Wikipedia  

ORM 是 Object-relational mapping 的缩写，翻译成中文就是对象关系映射，是一种使用面向对象编程语言在类型系统之间转换数据的编程技术。  

#4.说说 ORM 的优缺点

优点：  

* 简单系统可以提高开发效率，减少开发成本
* 使用面向对象编程的方式访问数据，不需要 SQL 编码
* 易于移植
* 不容易出现 SQL 注入

缺点：

* 对于有些需求实现起来会很笨重，不灵活
* 对于复杂的查询可能会有性能问题

##5.说说 Mybaits 的优缺点

优点：  

* 与 JDBC 相比，消除了 JDBC 大量样板代码，不需要手动设置参数，映射结果集
* 基于 SQL 语句编程，相当轻巧、灵活
* 易于学习掌握

缺点： 


* SQL 语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写 SQL 语句的功底有一定要求
* SQL 语句依赖于数据库，导致数据库移植性差，不能随意更换数据库


##6.Mybatis 是如何进行分页的？
 
Mybatis 使用 RowBounds 对象进行分页，它是针对 ResultSet 结果集执行的内存分页，而非物理分页，先把数据都查出来，然后再做分页。
 
可以在 sql 内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。


##7.分页插件的基本原理是什么？
 
分页插件的基本原理是使用 Mybatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql（SQL 拼接 limit），根据 dialect 方言，添加对应的物理分页语句和物理分页参数。  

##8.简述 Mybatis 的插件运行原理？
 
Mybatis 仅可以编写针对 ParameterHandler、ResultSetHandler、StatementHandler、Executor 这 4 种接口的插件，Mybatis 使用 JDK 的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能，每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是 InvocationHandler 的 invoke()方法，当然，只会拦截那些你指定需要拦截的方法。


##9.如何编写一个插件？
 
编写插件：实现 Mybatis 的 Interceptor 接口并复写 `intercept()` 方法，然后再给插件编写注解，指定要拦截哪一个接口的哪些方法即可，最后在配置文件中配置你编写的插件。


##10.Mybatis 动态 sql 有什么用？

便于处理有条件地拼接 SQL，把开发者从手动处理 SQL 拼接的痛苦中解脱出来。  

##11.Xml 映射文件中有哪些一级标签？

* **cache** – Configuration of the cache for a given namespace.
* **cache-ref** – Reference to a cache configuration from another namespace.
* **resultMap** – The most complicated and powerful element that describes how to load your objects from the database result sets.
* ~~**parameterMap** – Deprecated! Old-school way to map parameters. Inline parameters are preferred and this element may be removed in the future. Not documented here.~~
* **sql** – A reusable chunk of SQL that can be referenced by other statements.
* **insert** – A mapped INSERT statement.
* **update** – A mapped UPDATE statement.
* **delete** – A mapped DELETE statement.
* **select** – A mapped SELECT statement.

##12.Mybatis 是否支持延迟加载？
 
Mybatis 仅支持 association 关联对象和 collection 关联集合对象的延迟加载，association 指的就是一对一，collection 指的就是一对多查询。在 Mybatis 配置文件中，可以配置是否启用延迟加载`lazyLoadingEnabled=true|false`。

3.4.1 及以上的版本加入了 aggressiveLazyLoading 配置。

##13.延迟加载的基本原理是什么？
 
延迟加载的基本原理是，使用 CGLIB 创建目标对象的代理对象，当调用目标方法时，进入拦截器方法。
 
比如调用`a.getB().getName()`，拦截器 `invoke()`方法发现 `a.getB() `是 null 值，那么就会单独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，然后调用 `a.setB(b) `，于是 a 的对象 b 属性就有值了，接着完成`a.getB().getName()`方法的调用。

##14.mapper.xml 文件对应的 Mapper 接口原理是？
 
简单说：使用了 JDK 动态代理和反射，把接口和 xml 绑定在一起。

##15.Mapper 接口里的方法，参数不同时能重载吗？

不能。

* [Mappers should support polymorphic method invocations](https://github.com/mybatis/old-google-code-issues/issues/231)  

##16.`#{}`和 `${}`的区别是什么？
 
`${}`是字符串替换，`#{}`是预处理；
 
Mybatis 在处理`${}`时，就是把`{}`直接替换成变量的值。而 Mybatis 在处理 `#{}`时，会对 sql 语句进行预处理，将 sql 中的 `#{}`替换为?号，调用 PreparedStatement 的 set 方法来赋值；
 
使用 `#{}`可以有效的防止 SQL 注入，提高系统安全性。

##17.Mybatis 执行批量插入，能返回数据库主键列表吗？
能。

1. 对于支持生成自增主键的数据库：增加 useGenerateKeys 和 keyProperty ，`<insert>`标签属性。
2. 不支持生成自增主键的数据库：使用`<selectKey>`。


```
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username, password, email, bio) values
  <foreach item="item" collection="list" separator=",">
    (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
  </foreach>
</insert>

```

注意 Mybatis 的版本，官方在 3.3.1 版本中加入了批量新增返回主键 id 的功能 。

##18.不同的 Xml 映射文件，id 是否可以重复？

不同的 Xml 映射文件，如果配置了 namespace，那么 id 可以重复；如果没有配置 namespace，那么 id 不能重复；毕竟 namespace 不是必须的，只是最佳实践而已。
 
原因就是 `namespace+id` 是作为 `Map<String, MappedStatement>`的 key 使用的，如果没有 namespace，就剩下 id，那么，id 重复会导致数据互相覆盖。有了 namespace，自然 id 就可以重复，namespace 不同，`namespace+id` 自然也就不同。

##19.Mybatis 中 Executor 执行器的区别是？

Mybatis 有三种基本的 Executor 执行器，SimpleExecutor、ReuseExecutor、BatchExecutor。

* SimpleExecutor: 为每一个执行的语句创建一个新的PreparedStatement
* ReuseExecutor: 复用 PreparedStatement
* BatchExecutor: 对所有更新语句进行批处理，如果在它们之间执行 SELECT，则对它们进行必要的划分，以确保有一个易于理解的行为

##20.为什么说 Mybatis 是半自动 ORM 映射工具？
 
全自动 ORM 映射工具查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取。

而 Mybatis 在查询关联对象或关联集合对象时，需要手动编写 sql 来完成，所以，称之为半自动 ORM 映射工具。

##21.Mybatis 全局配置文件中有哪些标签？

* configuration
* properties
* settings
* typeAliases
* typeHandlers
* objectFactory
* plugins
* environments
* environment
* transactionManager
* dataSource
* databaseIdProvider
* mappers

##22.当实体类中的属性名和表中的字段名不一样时怎么办 ？

* 通过在查询的 sql 语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。
* 通过`<resultMap>`来映射字段名和实体类属性名的一一对应的关系。
* 使用注解时候，使用 Result，和`<resultMap>`类似。

##23.模糊查询 like 语句该怎么写?

第 1 种：在 Java 代码中添加 sql 通配符。

```
tring wildcardname = “%smi%”;
list<name> names = mapper.selectlike(wildcardname);
```

```
<select id=”selectlike”>
select * from foo where bar like #{value}
</select>
```

第 2 种：在 sql 语句中拼接通配符，会引起 sql 注入

```
String wildcardname = “smi”;
list<name> names = mapper.selectlike(wildcardname);
```

```
<select id=”selectlike”>
select * from foo where bar like "%"#{value}"%"
</select>
```


##24.Mybatis 构建步骤？

##25.简述一下 Mybatis 的手动编程步骤？

* 创建 SqlSessionFactory
* 通过 SqlSessionFactory 创建 SqlSession
* 通过 SqlSession 创建 Mapper
* 通过 Mapper 操作数据库
* 调用 `session.commit()` 提交事务
* 调用 `session.close()` 关闭会话


##26.Mybatis 工作的流程是？

* 加载配置并初始化
* 接收调用请求
* 处理操作请求
* 返回处理结果

##27.JDBC编程有哪些不足之处，MyBatis是如何解决这些问题的？

* 手动创建、释放数据库连接，容易出现数据库连接泄漏，频繁的创建也给系统带来很大开销，导致性能问题。MyBatis 通过配置数据库连接池来解决。
* 手动设置参数，映射结果比较烦琐。MyBatis 支持自动映射，也可以通过配置 resultMap 映射。
* 手动有条件的拼接 SQL 很痛苦。MyBatis 支持动态 SQL。  

##28.请说说MyBatis的工作原理

MyBatis 的工作原理是利用 JDK 代理技术为 Mapper 接口生成代理对象，调用 Mapper 方法操作数据库时，用方法名作为 id，到 Mapper XML 文件中找到对应的 SQL 语句，然后绑定参数，执行语句，映射结果。


# 原文

* [25 道 mybatis 面试题，不要说你不会](https://xie.infoq.cn/article/fde8156a2d5c52ccc1530e404)  



