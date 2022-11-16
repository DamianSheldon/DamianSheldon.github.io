---
layout: post
title: "broadleafcommerce study notes part one"
date: 2022-11-16 17:23:16 +0800
comments: true
categories: [Archives, Web Development]
keywords: broadleafcommerce
description: broadleafcommerce study notes part one
published: false
---

@SpringBootApplication
@EnableAutoConfiguration
public class AdminApplication {

    @Configuration
    @EnableBroadleafAdminAutoConfiguration
    public static class BroadleafFrameworkConfiguration {}

    public static void main(String[] args) {
        SpringApplication.run(AdminApplication.class, args);
    }
 
}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({
    EnableBroadleafAdminRootAutoConfiguration.BroadleafAdminRootAutoConfiguration.class,
    EnableBroadleafAdminServletAutoConfiguration.BroadleafAdminServletAutoConfiguration.class,
    BroadleafAdminAutoConfigurationOverrides.class
})
public @interface EnableBroadleafAdminAutoConfiguration {
    
    @ImportResource("classpath:/override-contexts/admin-root-autoconfiguration-overrides.xml")
    class BroadleafAdminAutoConfigurationOverrides {}
}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({
    EnableBroadleafRootAutoConfiguration.BroadleafRootAutoConfiguration.class,
    BroadleafAdminRootAutoConfiguration.class,
    BroadleafAdminRootAutoConfigurationOverrides.class
})
public @interface EnableBroadleafAdminRootAutoConfiguration {

    /**
     * We are deliberately leaving off the {@link org.springframework.context.annotation.Configuration} annotation since
     * this inner class is being included in the {@code Import} above, which interprets this as a
     * {@link org.springframework.context.annotation.Configuration}. We do this to avoid component scanning this inner class.
     */
    @Import(EnableBroadleafRootAutoConfiguration.BroadleafRootAutoConfiguration.class)
    @ImportResource(locations = {
            "classpath*:/blc-config/admin/framework/bl-*-applicationContext.xml",
            "classpath*:/blc-config/admin/early/bl-*-applicationContext.xml",
            "classpath*:/blc-config/admin/bl-*-applicationContext.xml",
            "classpath*:/blc-config/admin/late/bl-*-applicationContext.xml"
    }, reader = FrameworkXmlBeanDefinitionReader.class)
    class BroadleafAdminRootAutoConfiguration {

        @Bean
        public AdminOnlyTarget blAdminOnlyTarget() {
            return new AdminOnlyTarget();
        }
    }
    
    @ImportResource("classpath:/override-contexts/admin-root-autoconfiguration-overrides.xml")
    class BroadleafAdminRootAutoConfigurationOverrides {}
}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({
    BroadleafRootAutoConfiguration.class,
    BroadleafRootAutoConfigurationOverrides.class
})
public @interface EnableBroadleafRootAutoConfiguration {

    /**
     * We are deliberately leaving off the {@link org.springframework.context.annotation.Configuration} annotation since
     * this inner class is being included in the {@code Import} above, which interprets this as a
     * {@link org.springframework.context.annotation.Configuration}. We do this to avoid component scanning this inner class.
     */
    @ImportResource(locations = {
            "classpath*:/blc-config/framework/bl-*-applicationContext.xml",
            "classpath*:/blc-config/early/bl-*-applicationContext.xml",
            "classpath*:/blc-config/bl-*-applicationContext.xml",
            "classpath*:/blc-config/late/bl-*-applicationContext.xml"
    }, reader = FrameworkXmlBeanDefinitionReader.class)
    class BroadleafRootAutoConfiguration {}
    
    @ImportResource("classpath:/override-contexts/autoconfiguration-overrides.xml")
    class BroadleafRootAutoConfigurationOverrides { }
}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({
    EnableBroadleafServletAutoConfiguration.BroadleafServletAutoConfiguration.class,
    BroadleafAdminServletAutoConfiguration.class,
    BroadleafAdminServletAutoConfigurationOverrides.class
})
public @interface EnableBroadleafAdminServletAutoConfiguration {

    /**
     * We are deliberately leaving off the {@link org.springframework.context.annotation.Configuration} annotation since
     * this inner class is being included in the {@code Import} above, which interprets this as a
     * {@link org.springframework.context.annotation.Configuration}. We do this to avoid component scanning this inner class.
     */
    @Import(EnableBroadleafServletAutoConfiguration.BroadleafServletAutoConfiguration.class)
    @ImportResource(locations = {
            "classpath*:/blc-config/admin/framework/bl-*-applicationContext-servlet.xml",
            "classpath*:/blc-config/admin/early/bl-*-applicationContext-servlet.xml",
            "classpath*:/blc-config/admin/bl-*-applicationContext-servlet.xml",
            "classpath*:/blc-config/admin/late/bl-*-applicationContext-servlet.xml",
    }, reader = FrameworkXmlBeanDefinitionReader.class)
    class BroadleafAdminServletAutoConfiguration {}
    
    @ImportResource("classpath:/override-contexts/admin-servlet-autoconfiguration-overrides.xml")
    class BroadleafAdminServletAutoConfigurationOverrides {}
}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({
    BroadleafServletAutoConfiguration.class,
    BroadleafServletAutoConfigurationOverrides.class
})
public @interface EnableBroadleafServletAutoConfiguration {

    /**
     * We are deliberately leaving off the {@link org.springframework.context.annotation.Configuration} annotation since
     * this inner class is being included in the {@code Import} above, which interprets this as a
     * {@link org.springframework.context.annotation.Configuration}. We do this to avoid component scanning this inner class.
     */
    @ImportResource(locations = {
            "classpath*:/blc-config/framework/bl-*-applicationContext-servlet.xml",
            "classpath*:/blc-config/early/bl-*-applicationContext-servlet.xml",
            "classpath*:/blc-config/bl-*-applicationContext-servlet.xml",
            "classpath*:/blc-config/late/bl-*-applicationContext-servlet.xml"
    }, reader = FrameworkXmlBeanDefinitionReader.class)
    class BroadleafServletAutoConfiguration {}
    
    @ImportResource("classpath:/override-contexts/autoconfiguration-servlet-overrides.xml")
    class BroadleafServletAutoConfigurationOverrides { }
}

[org.springframework.boot.actuate.endpoint.web.servlet.AdditionalHealthEndpointPathsWebMvcHandlerMapping@690f7902, org.springframework.boot.actuate.endpoint.web.servlet.WebMvcEndpointHandlerMapping@5050dc97, org.springframework.boot.actuate.endpoint.web.servlet.ControllerEndpointHandlerMapping@279e7acb, org.springframework.web.servlet.handler.SimpleUrlHandlerMapping@bec7101, org.springframework.web.servlet.handler.SimpleUrlHandlerMapping@35f6893a, org.broadleafcommerce.openadmin.web.controller.AdminRequestMappingHandlerMapping@4e10eebf, 0
org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping@674982c3, org.springframework.web.servlet.function.support.RouterFunctionMapping@642b8b2, org.springframework.web.servlet.handler.SimpleUrlHandlerMapping@1dcb84df, org.broadleafcommerce.common.web.controller.FrameworkControllerHandlerMapping@68c89396,  Ordered.LOWEST_PRECEDENCE - 2
org.broadleafcommerce.openadmin.web.controller.AdminControllerHandlerMapping@44a1949b, Ordered.LOWEST_PRECEDENCE - 1
org.springframework.boot.autoconfigure.web.servlet.WelcomePageHandlerMapping@25faedfc]


org.broadleafcommerce.presentation.thymeleaf3.admin.config.Thymeleaf3AdminConfig

@Bean(name = {"blAdminThymeleafViewResolver", "thymeleafViewResolver"})
public BroadleafThymeleafViewResolver blAdminThymeleafViewResolver() {
    BroadleafThymeleafViewResolver view = new BroadleafThymeleafViewResolver();
    view.setTemplateEngine(templateEngine);
    view.setOrder(1);
    view.setCache(environment.getProperty("thymeleaf.view.resolver.cache", Boolean.class, true));
    view.setCharacterEncoding("UTF-8");
    view.setFullPageLayout("layout/fullPageLayout");
    Map<String, String> layoutMap = new HashMap<>();
    layoutMap.put("login/", "layout/loginLayout");
    layoutMap.put("views/", "NONE");
    layoutMap.put("modules/modalContainer", "NONE");
    view.setLayoutMap(layoutMap);
    return view;
}

select generatedAlias0 from PageImpl as generatedAlias0 where 1=1 order by generatedAlias0.id asc

insert into BLC_ADMIN_PERMISSION_ENTITY (ADMIN_PERMISSION_ENTITY_ID, CEILING_ENTITY, ADMIN_PERMISSION_ID) values (-30000, 'org.broadleafcommerce.cms.page.domain.PageTemplateImpl', -200);

insert into BLC_ADMIN_PERMISSION (ADMIN_PERMISSION_ID, DESCRIPTION, NAME, PERMISSION_TYPE, IS_FRIENDLY) VALUES (-40000, 'View Page Template', 'PERMISSION_PAGE_TEMPLATE_VIEW', 'READ', TRUE);

INSERT INTO BLC_ADMIN_PERMISSION_XREF (ADMIN_PERMISSION_ID, CHILD_PERMISSION_ID) VALUES (-40000, -200);

INSERT INTO BLC_ADMIN_ROLE_PERMISSION_XREF (ADMIN_ROLE_ID, ADMIN_PERMISSION_ID) VALUES (-1, -40000);

INSERT INTO BLC_ADMIN_SECTION (ADMIN_SECTION_ID, DISPLAY_ORDER, ADMIN_MODULE_ID, NAME, SECTION_KEY, URL, CEILING_ENTITY) VALUES (-40000, 1000, -2, 'Pag
e Template', 'PageTemplate', '/page-template', 'org.broadleafcommerce.cms.page.domain.PageTemplateImpl');

INSERT INTO BLC_ADMIN_SEC_PERM_XREF (ADMIN_SECTION_ID, ADMIN_PERMISSION_ID) VALUES (-40000, -40000);

