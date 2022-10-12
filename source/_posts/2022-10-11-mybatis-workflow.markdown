---
layout: post
title: "MyBatis 的工作流程(一)"
date: 2022-10-11 16:06:03 +0800
comments: true
categories: [Archives, Web Development]
keywords: MyBatis, Spring Boot
description: MyBatis' Workflow
---

MyBatis 的工作流程主要是以下几步：  

* 加载配置并初始化
* 接收调用请求
* 处理操作请求
* 返回处理结果

本文我们先具体来看看它与 Spring Boot 集成时的初始化。  

MyBatis 官方团队有开发一个 Spring Boot Starter，我们通过它的代码来看下配置加载和初始化。配置的入口是 MybatisAutoConfiguration，它会按需创建 sqlSessionFactory 和 sqlSessionTemplate 这两个 bean 对象，同时它还包含一个内部配置类 MapperScannerRegistrarNotFoundConfiguration:  

```
@org.springframework.context.annotation.Configuration
@Import(AutoConfiguredMapperScannerRegistrar.class)
@ConditionalOnMissingBean({ MapperFactoryBean.class, MapperScannerConfigurer.class })
public static class MapperScannerRegistrarNotFoundConfiguration implements InitializingBean {

  @Override
  public void afterPropertiesSet() {
    logger.debug(
        "Not found configuration for registering mapper bean using @MapperScan, MapperFactoryBean and MapperScannerConfigurer.");
  }

}
```

MapperScannerRegistrarNotFoundConfiguration 导入了 AutoConfiguredMapperScannerRegistrar，它会向 IoC 容器注册 MapperScannerConfigurer。
<!--more-->
```
/**
 * This will just scan the same base package as Spring Boot does. If you want more power, you can explicitly use
 * {@link org.mybatis.spring.annotation.MapperScan} but this will get typed mappers working correctly, out-of-the-box,
 * similar to using Spring Data JPA repositories.
 */
public static class AutoConfiguredMapperScannerRegistrar implements BeanFactoryAware, ImportBeanDefinitionRegistrar {

  private BeanFactory beanFactory;

  @Override
  public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

    if (!AutoConfigurationPackages.has(this.beanFactory)) {
      logger.debug("Could not determine auto-configuration package, automatic mapper scanning disabled.");
      return;
    }

    logger.debug("Searching for mappers annotated with @Mapper");

    List<String> packages = AutoConfigurationPackages.get(this.beanFactory);
    if (logger.isDebugEnabled()) {
      packages.forEach(pkg -> logger.debug("Using auto-configuration base package '{}'", pkg));
    }

    BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(MapperScannerConfigurer.class);
    builder.addPropertyValue("processPropertyPlaceHolders", true);
    builder.addPropertyValue("annotationClass", Mapper.class);
    builder.addPropertyValue("basePackage", StringUtils.collectionToCommaDelimitedString(packages));
    BeanWrapper beanWrapper = new BeanWrapperImpl(MapperScannerConfigurer.class);
    Set<String> propertyNames = Stream.of(beanWrapper.getPropertyDescriptors()).map(PropertyDescriptor::getName)
        .collect(Collectors.toSet());
    if (propertyNames.contains("lazyInitialization")) {
      // Need to mybatis-spring 2.0.2+
      builder.addPropertyValue("lazyInitialization", "${mybatis.lazy-initialization:false}");
    }
    if (propertyNames.contains("defaultScope")) {
      // Need to mybatis-spring 2.0.6+
      builder.addPropertyValue("defaultScope", "${mybatis.mapper-default-scope:}");
    }
    registry.registerBeanDefinition(MapperScannerConfigurer.class.getName(), builder.getBeanDefinition());
  }

  @Override
  public void setBeanFactory(BeanFactory beanFactory) {
    this.beanFactory = beanFactory;
  }

}

```

MapperScannerConfigurer 是一个 BeanDefinitionRegistryPostProcessor，IoC 会调用它的 postProcessBeanDefinitionRegistry 方法来处理 bean 定义:  

```
/**
 * {@inheritDoc}
 *
 * @since 1.0.2
 */
@Override
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
  if (this.processPropertyPlaceHolders) {
    processPropertyPlaceHolders();
  }

  ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);
  scanner.setAddToConfig(this.addToConfig);
  scanner.setAnnotationClass(this.annotationClass);
  scanner.setMarkerInterface(this.markerInterface);
  scanner.setSqlSessionFactory(this.sqlSessionFactory);
  scanner.setSqlSessionTemplate(this.sqlSessionTemplate);
  scanner.setSqlSessionFactoryBeanName(this.sqlSessionFactoryBeanName);
  scanner.setSqlSessionTemplateBeanName(this.sqlSessionTemplateBeanName);
  scanner.setResourceLoader(this.applicationContext);
  scanner.setBeanNameGenerator(this.nameGenerator);
  scanner.setMapperFactoryBeanClass(this.mapperFactoryBeanClass);
  if (StringUtils.hasText(lazyInitialization)) {
    scanner.setLazyInitialization(Boolean.valueOf(lazyInitialization));
  }
  if (StringUtils.hasText(defaultScope)) {
    scanner.setDefaultScope(defaultScope);
  }
  scanner.registerFilters();
  scanner.scan(
      StringUtils.tokenizeToStringArray(this.basePackage, ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS));
}

```

从方法中我们可以知道，它使用 ClassPathMapperScanner 来扫描注册 Mapper，ClassPathMapperScanner 覆盖了父类 ClassPathBeanDefinitionScanner 的 doScan 方法：

```
/**
 * Calls the parent search that will search and register all the candidates. Then the registered objects are post
 * processed to set them as MapperFactoryBeans
 */
@Override
public Set<BeanDefinitionHolder> doScan(String... basePackages) {
  Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);

  if (beanDefinitions.isEmpty()) {
    LOGGER.warn(() -> "No MyBatis mapper was found in '" + Arrays.toString(basePackages)
        + "' package. Please check your configuration.");
  } else {
    processBeanDefinitions(beanDefinitions);
  }

  return beanDefinitions;
}

```

它在 processBeanDefinitions 方法中对 bean 定义进行了进一步的处理，把 bean 的 bean class 换成了 MapperFactoryBean，由它来生成创建对应的 Mapper。另外就是设置按类型自动连线 `definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE)`，我在跟代码的过程中一开始很奇怪 ClassPathMapperScanner 注册 MapperFactoryBean 的 bean 定义时 sqlSessionTemplate 为 null，那么是在什么时候设置的 sqlSessionTemplate 呢？经过一番代码追踪，发现是 beanDefinition 设置了 AutowireMode，AbstractAutowireCapableBeanFactory 会帮助自动关联。


```
private void processBeanDefinitions(Set<BeanDefinitionHolder> beanDefinitions) {
  AbstractBeanDefinition definition;
  BeanDefinitionRegistry registry = getRegistry();
  for (BeanDefinitionHolder holder : beanDefinitions) {
    definition = (AbstractBeanDefinition) holder.getBeanDefinition();
    boolean scopedProxy = false;
    if (ScopedProxyFactoryBean.class.getName().equals(definition.getBeanClassName())) {
      definition = (AbstractBeanDefinition) Optional
          .ofNullable(((RootBeanDefinition) definition).getDecoratedDefinition())
          .map(BeanDefinitionHolder::getBeanDefinition).orElseThrow(() -> new IllegalStateException(
              "The target bean definition of scoped proxy bean not found. Root bean definition[" + holder + "]"));
      scopedProxy = true;
    }
    String beanClassName = definition.getBeanClassName();
    LOGGER.debug(() -> "Creating MapperFactoryBean with name '" + holder.getBeanName() + "' and '" + beanClassName
        + "' mapperInterface");

    // the mapper interface is the original class of the bean
    // but, the actual class of the bean is MapperFactoryBean
    definition.getConstructorArgumentValues().addGenericArgumentValue(beanClassName); // issue #59
    definition.setBeanClass(this.mapperFactoryBeanClass);

    definition.getPropertyValues().add("addToConfig", this.addToConfig);

    // Attribute for MockitoPostProcessor
    // https://github.com/mybatis/spring-boot-starter/issues/475
    definition.setAttribute(FACTORY_BEAN_OBJECT_TYPE, beanClassName);

    boolean explicitFactoryUsed = false;
    if (StringUtils.hasText(this.sqlSessionFactoryBeanName)) {
      definition.getPropertyValues().add("sqlSessionFactory",
          new RuntimeBeanReference(this.sqlSessionFactoryBeanName));
      explicitFactoryUsed = true;
    } else if (this.sqlSessionFactory != null) {
      definition.getPropertyValues().add("sqlSessionFactory", this.sqlSessionFactory);
      explicitFactoryUsed = true;
    }

    if (StringUtils.hasText(this.sqlSessionTemplateBeanName)) {
      if (explicitFactoryUsed) {
        LOGGER.warn(
            () -> "Cannot use both: sqlSessionTemplate and sqlSessionFactory together. sqlSessionFactory is ignored.");
      }
      definition.getPropertyValues().add("sqlSessionTemplate",
          new RuntimeBeanReference(this.sqlSessionTemplateBeanName));
      explicitFactoryUsed = true;
    } else if (this.sqlSessionTemplate != null) {
      if (explicitFactoryUsed) {
        LOGGER.warn(
            () -> "Cannot use both: sqlSessionTemplate and sqlSessionFactory together. sqlSessionFactory is ignored.");
      }
      definition.getPropertyValues().add("sqlSessionTemplate", this.sqlSessionTemplate);
      explicitFactoryUsed = true;
    }

    if (!explicitFactoryUsed) {
      LOGGER.debug(() -> "Enabling autowire by type for MapperFactoryBean with name '" + holder.getBeanName() + "'.");
      definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);
    }

    definition.setLazyInit(lazyInitialization);

    if (scopedProxy) {
      continue;
    }

    if (ConfigurableBeanFactory.SCOPE_SINGLETON.equals(definition.getScope()) && defaultScope != null) {
      definition.setScope(defaultScope);
    }

    if (!definition.isSingleton()) {
      BeanDefinitionHolder proxyHolder = ScopedProxyUtils.createScopedProxy(holder, registry, true);
      if (registry.containsBeanDefinition(proxyHolder.getBeanName())) {
        registry.removeBeanDefinition(proxyHolder.getBeanName());
      }
      registry.registerBeanDefinition(proxyHolder.getBeanName(), proxyHolder.getBeanDefinition());
    }

  }
}

```


MapperFactoryBean 的 getObject 方法如下，它使用上文所述，容器中的 sqlSession bean 来创建 mapper，通常也就是 MybatisAutoConfiguration 定义的 sqlSession bean。

```
public T getObject() throws Exception {
  return getSqlSession().getMapper(this.mapperInterface);
}

```

MybatisAutoConfiguration 是定义 sqlSession bean 的代码如下：  


```
@Bean
@ConditionalOnMissingBean
public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
  ExecutorType executorType = this.properties.getExecutorType();
  if (executorType != null) {
    return new SqlSessionTemplate(sqlSessionFactory, executorType);
  } else {
    return new SqlSessionTemplate(sqlSessionFactory);
  }
}

```

它依赖的 SqlSessionFactory MybatisAutoConfiguration 也有定义:  

```
@Bean
@ConditionalOnMissingBean
public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
  SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
  factory.setDataSource(dataSource);
  factory.setVfs(SpringBootVFS.class);
  if (StringUtils.hasText(this.properties.getConfigLocation())) {
    factory.setConfigLocation(this.resourceLoader.getResource(this.properties.getConfigLocation()));
  }
  applyConfiguration(factory);
  if (this.properties.getConfigurationProperties() != null) {
    factory.setConfigurationProperties(this.properties.getConfigurationProperties());
  }
  if (!ObjectUtils.isEmpty(this.interceptors)) {
    factory.setPlugins(this.interceptors);
  }
  if (this.databaseIdProvider != null) {
    factory.setDatabaseIdProvider(this.databaseIdProvider);
  }
  if (StringUtils.hasLength(this.properties.getTypeAliasesPackage())) {
    factory.setTypeAliasesPackage(this.properties.getTypeAliasesPackage());
  }
  if (this.properties.getTypeAliasesSuperType() != null) {
    factory.setTypeAliasesSuperType(this.properties.getTypeAliasesSuperType());
  }
  if (StringUtils.hasLength(this.properties.getTypeHandlersPackage())) {
    factory.setTypeHandlersPackage(this.properties.getTypeHandlersPackage());
  }
  if (!ObjectUtils.isEmpty(this.typeHandlers)) {
    factory.setTypeHandlers(this.typeHandlers);
  }
  if (!ObjectUtils.isEmpty(this.properties.resolveMapperLocations())) {
    factory.setMapperLocations(this.properties.resolveMapperLocations());
  }
  Set<String> factoryPropertyNames = Stream
      .of(new BeanWrapperImpl(SqlSessionFactoryBean.class).getPropertyDescriptors()).map(PropertyDescriptor::getName)
      .collect(Collectors.toSet());
  Class<? extends LanguageDriver> defaultLanguageDriver = this.properties.getDefaultScriptingLanguageDriver();
  if (factoryPropertyNames.contains("scriptingLanguageDrivers") && !ObjectUtils.isEmpty(this.languageDrivers)) {
    // Need to mybatis-spring 2.0.2+
    factory.setScriptingLanguageDrivers(this.languageDrivers);
    if (defaultLanguageDriver == null && this.languageDrivers.length == 1) {
      defaultLanguageDriver = this.languageDrivers[0].getClass();
    }
  }
  if (factoryPropertyNames.contains("defaultScriptingLanguageDriver")) {
    // Need to mybatis-spring 2.0.2+
    factory.setDefaultScriptingLanguageDriver(defaultLanguageDriver);
  }

  return factory.getObject();
}

private void applyConfiguration(SqlSessionFactoryBean factory) {
  Configuration configuration = this.properties.getConfiguration();
  if (configuration == null && !StringUtils.hasText(this.properties.getConfigLocation())) {
    configuration = new Configuration();
  }
  if (configuration != null && !CollectionUtils.isEmpty(this.configurationCustomizers)) {
    for (ConfigurationCustomizer customizer : this.configurationCustomizers) {
      customizer.customize(configuration);
    }
  }
  factory.setConfiguration(configuration);
}

```

SqlSessionTemplate 是一个代理对象，它的功能依赖 SqlSessionFactory，它的 getMapper 方法如下：

```
@Override
public <T> T getMapper(Class<T> type) {
  return getConfiguration().getMapper(type, this);
}
```

实现依赖 getConfiguration, 它的内容如下：

```
@Override
public Configuration getConfiguration() {
  return this.sqlSessionFactory.getConfiguration();
}

```

Configuration 就是使用的 sqlSessionFactory 的配置对象。  

现在我们聚焦到 SqlSessionFactory 的 Configuration 对象是怎么来的。 SqlSessionFactory 是由 SqlSessionFactoryBean，MybatisAutoConfiguration 有为它设置一个 Configuration :  

```
private void applyConfiguration(SqlSessionFactoryBean factory) {
  Configuration configuration = this.properties.getConfiguration();
  if (configuration == null && !StringUtils.hasText(this.properties.getConfigLocation())) {
    configuration = new Configuration();
  }
  if (configuration != null && !CollectionUtils.isEmpty(this.configurationCustomizers)) {
    for (ConfigurationCustomizer customizer : this.configurationCustomizers) {
      customizer.customize(configuration);
    }
  }
  factory.setConfiguration(configuration);
}
```

那么最终的 Configuration 就是这个外部设置的对象吗？我们继续看它的 getObject 方法:  

```
@Override
public SqlSessionFactory getObject() throws Exception {
  if (this.sqlSessionFactory == null) {
    afterPropertiesSet();
  }

  return this.sqlSessionFactory;
}
```

再看下它的 afterPropertiesSet 方法：   

```
public void afterPropertiesSet() throws Exception {
  notNull(dataSource, "Property 'dataSource' is required");
  notNull(sqlSessionFactoryBuilder, "Property 'sqlSessionFactoryBuilder' is required");
  state((configuration == null && configLocation == null) || !(configuration != null && configLocation != null),
      "Property 'configuration' and 'configLocation' can not specified with together");

  this.sqlSessionFactory = buildSqlSessionFactory();
}

```

是在 buildSqlSessionFactory 方法中创建的 sqlSessionFactory:  

```

/**
 * Build a {@code SqlSessionFactory} instance.
 *
 * The default implementation uses the standard MyBatis {@code XMLConfigBuilder} API to build a
 * {@code SqlSessionFactory} instance based on a Reader. Since 1.3.0, it can be specified a {@link Configuration}
 * instance directly(without config file).
 *
 * @return SqlSessionFactory
 * @throws Exception
 *           if configuration is failed
 */
protected SqlSessionFactory buildSqlSessionFactory() throws Exception {

  final Configuration targetConfiguration;

  XMLConfigBuilder xmlConfigBuilder = null;
  if (this.configuration != null) {
    targetConfiguration = this.configuration;
    if (targetConfiguration.getVariables() == null) {
      targetConfiguration.setVariables(this.configurationProperties);
    } else if (this.configurationProperties != null) {
      targetConfiguration.getVariables().putAll(this.configurationProperties);
    }
  } else if (this.configLocation != null) {
    xmlConfigBuilder = new XMLConfigBuilder(this.configLocation.getInputStream(), null, this.configurationProperties);
    targetConfiguration = xmlConfigBuilder.getConfiguration();
  } else {
    LOGGER.debug(
        () -> "Property 'configuration' or 'configLocation' not specified, using default MyBatis Configuration");
    targetConfiguration = new Configuration();
    Optional.ofNullable(this.configurationProperties).ifPresent(targetConfiguration::setVariables);
  }

  Optional.ofNullable(this.objectFactory).ifPresent(targetConfiguration::setObjectFactory);
  Optional.ofNullable(this.objectWrapperFactory).ifPresent(targetConfiguration::setObjectWrapperFactory);
  Optional.ofNullable(this.vfs).ifPresent(targetConfiguration::setVfsImpl);

  if (hasLength(this.typeAliasesPackage)) {
    scanClasses(this.typeAliasesPackage, this.typeAliasesSuperType).stream()
        .filter(clazz -> !clazz.isAnonymousClass()).filter(clazz -> !clazz.isInterface())
        .filter(clazz -> !clazz.isMemberClass()).forEach(targetConfiguration.getTypeAliasRegistry()::registerAlias);
  }

  if (!isEmpty(this.typeAliases)) {
    Stream.of(this.typeAliases).forEach(typeAlias -> {
      targetConfiguration.getTypeAliasRegistry().registerAlias(typeAlias);
      LOGGER.debug(() -> "Registered type alias: '" + typeAlias + "'");
    });
  }

  if (!isEmpty(this.plugins)) {
    Stream.of(this.plugins).forEach(plugin -> {
      targetConfiguration.addInterceptor(plugin);
      LOGGER.debug(() -> "Registered plugin: '" + plugin + "'");
    });
  }

  if (hasLength(this.typeHandlersPackage)) {
    scanClasses(this.typeHandlersPackage, TypeHandler.class).stream().filter(clazz -> !clazz.isAnonymousClass())
        .filter(clazz -> !clazz.isInterface()).filter(clazz -> !Modifier.isAbstract(clazz.getModifiers()))
        .forEach(targetConfiguration.getTypeHandlerRegistry()::register);
  }

  if (!isEmpty(this.typeHandlers)) {
    Stream.of(this.typeHandlers).forEach(typeHandler -> {
      targetConfiguration.getTypeHandlerRegistry().register(typeHandler);
      LOGGER.debug(() -> "Registered type handler: '" + typeHandler + "'");
    });
  }

  targetConfiguration.setDefaultEnumTypeHandler(defaultEnumTypeHandler);

  if (!isEmpty(this.scriptingLanguageDrivers)) {
    Stream.of(this.scriptingLanguageDrivers).forEach(languageDriver -> {
      targetConfiguration.getLanguageRegistry().register(languageDriver);
      LOGGER.debug(() -> "Registered scripting language driver: '" + languageDriver + "'");
    });
  }
  Optional.ofNullable(this.defaultScriptingLanguageDriver)
      .ifPresent(targetConfiguration::setDefaultScriptingLanguage);

  if (this.databaseIdProvider != null) {// fix #64 set databaseId before parse mapper xmls
    try {
      targetConfiguration.setDatabaseId(this.databaseIdProvider.getDatabaseId(this.dataSource));
    } catch (SQLException e) {
      throw new NestedIOException("Failed getting a databaseId", e);
    }
  }

  Optional.ofNullable(this.cache).ifPresent(targetConfiguration::addCache);

  if (xmlConfigBuilder != null) {
    try {
      xmlConfigBuilder.parse();
      LOGGER.debug(() -> "Parsed configuration file: '" + this.configLocation + "'");
    } catch (Exception ex) {
      throw new NestedIOException("Failed to parse config resource: " + this.configLocation, ex);
    } finally {
      ErrorContext.instance().reset();
    }
  }

  targetConfiguration.setEnvironment(new Environment(this.environment,
      this.transactionFactory == null ? new SpringManagedTransactionFactory() : this.transactionFactory,
      this.dataSource));

  if (this.mapperLocations != null) {
    if (this.mapperLocations.length == 0) {
      LOGGER.warn(() -> "Property 'mapperLocations' was specified but matching resources are not found.");
    } else {
      for (Resource mapperLocation : this.mapperLocations) {
        if (mapperLocation == null) {
          continue;
        }
        try {
          XMLMapperBuilder xmlMapperBuilder = new XMLMapperBuilder(mapperLocation.getInputStream(),
              targetConfiguration, mapperLocation.toString(), targetConfiguration.getSqlFragments());
          xmlMapperBuilder.parse();
        } catch (Exception e) {
          throw new NestedIOException("Failed to parse mapping resource: '" + mapperLocation + "'", e);
        } finally {
          ErrorContext.instance().reset();
        }
        LOGGER.debug(() -> "Parsed mapper file: '" + mapperLocation + "'");
      }
    }
  } else {
    LOGGER.debug(() -> "Property 'mapperLocations' was not specified.");
  }

  return this.sqlSessionFactoryBuilder.build(targetConfiguration);
}

```

从这段代码可知，由于 MybatisAutoConfiguration 有设置一个 Configuration ，所以 SqlSessionFactoryBean 使用的就是设置的这个 Configuration。  

sqlSessionFactoryBuilder 默认是 SqlSessionFactoryBuilder，它的赋值代码为：  

```
private SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
```

它的 build 方法实现为:  

```
public SqlSessionFactory build(Configuration config) {
  return new DefaultSqlSessionFactory(config);
}
```

因此创建的是 DefaultSqlSessionFactory 对象。

现在 SqlSessionFactory, SqlSession, Configuration 的实现来源都清楚了，我们继续来看创建 mapper 的代码，看 mapper 的实现是什么？ 

```
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
  return mapperRegistry.getMapper(type, sqlSession);
}
```

mapperRegistry 的实现是 MapperRegistry，它的 getMapper 方法如下：

```
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
  final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
  if (mapperProxyFactory == null) {
    throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
  }
  try {
    return mapperProxyFactory.newInstance(sqlSession);
  } catch (Exception e) {
    throw new BindingException("Error getting mapper instance. Cause: " + e, e);
  }
}
```

它是从 knownMappers 取出类型对应的 MapperProxyFactory，那么它是何时如何注册的呢？  

前面我们说过， Mapper 对应向 IoC 注册的的 bean 定义是 MapperFactoryBean 对象，MapperFactory 继承自 SqlSessionDaoSupport，而 SqlSessionDaoSupport 继承自 DaoSupport，DaoSupport 实现 InitializingBean 接口，所以在实例化 mapper 时会实例化出 MapperFactory，并先调用它的 afterPropertiesSet 方法：  

```
@Override
public final void afterPropertiesSet() throws IllegalArgumentException, BeanInitializationException {
	// Let abstract subclasses check their configuration.
	checkDaoConfig();

	// Let concrete implementations initialize themselves.
	try {
		initDao();
	}
	catch (Exception ex) {
		throw new BeanInitializationException("Initialization of DAO failed", ex);
	}
}

```

MapperFactory 覆盖了 checkDaoConfig 方法:  

```
protected void checkDaoConfig() {
  super.checkDaoConfig();

  notNull(this.mapperInterface, "Property 'mapperInterface' is required");

  Configuration configuration = getSqlSession().getConfiguration();
  if (this.addToConfig && !configuration.hasMapper(this.mapperInterface)) {
    try {
      configuration.addMapper(this.mapperInterface);
    } catch (Exception e) {
      logger.error("Error while adding the mapper '" + this.mapperInterface + "' to configuration.", e);
      throw new IllegalArgumentException(e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
}

```

也就是在这个方法中向 Configuration 添加了 mapper 对应的 MapperProxyFactory 类型:  

```
public <T> void addMapper(Class<T> type) {
  mapperRegistry.addMapper(type);
}

public <T> void addMapper(Class<T> type) {
  if (type.isInterface()) {
    if (hasMapper(type)) {
      throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
    }
    boolean loadCompleted = false;
    try {
      knownMappers.put(type, new MapperProxyFactory<>(type));
      // It's important that the type is added before the parser is run
      // otherwise the binding may automatically be attempted by the
      // mapper parser. If the type is already known, it won't try.
      MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
      parser.parse();
      loadCompleted = true;
    } finally {
      if (!loadCompleted) {
        knownMappers.remove(type);
      }
    }
  }
}

```

从上述代码可以看出 mapper 对应的 MapperProxyFactory 类型是用定义的 mapper 接口类型参数化的 MapperProxyFactory，它的 `newInstance(SqlSession sqlSession)` 方法如下：

```
public T newInstance(SqlSession sqlSession) {
  final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
  return newInstance(mapperProxy);
}

```

最后为 mapper 接口创建向 mapperProxy 实例转发调用的代理对象。

```
protected T newInstance(MapperProxy<T> mapperProxy) {
  return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
}
```

也就是说，mapper 接口类型对象等效于一个 MapperProxy 对象。至此，MyBatis 的初始化过程就梳理清楚了。  

