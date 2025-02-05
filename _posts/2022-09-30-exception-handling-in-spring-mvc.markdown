---
layout: post
title: "Spring MVC 中的异常处理"
date: 2022-09-30 10:06:54 +0800
comments: true
categories: [Archives, Web Development]
keywords: Spring MVC, exception handling
description: Exception handling in Spring MVC
---

使用 Spring 来开发 web 应用时很有必要建立一个统一的异常处理体系。想要建立这个体系，我们先要搞清楚 Spring MVC 中的异常处理机制。 Spring MVC 是基于 Servlet，所以它遵循 Servlet 规范。

Servlet 规范中有详细的错误处理说明，简单来说就是 Servlet 在处理请求时可能会抛出异常或者调用 `sendError` ，这时 Servlet-Container 就要产生相应的错误界面，错误界面是允许自定义的。Spring MVC 的核心之一是 DispatchServlet，它是一个前端控制器，所有的请求处理都由它来驱动，从名字可以看出，它也是一个 Servlet，所以它的错误处理机制自然要遵循 Servlet 规范。从完整性的角度来看，还一种错误处理方法，Servlet 可以自己设置 HTTP 的 status code 和 body，也就是不和 Servlet-Container 联动来处理错误，而是完全自主地处理。

我们先来看 DispatcherServlet 的异常处理机制，Spring 团队将异常处理功能集中到 HandlerExceptionResolver 接口的实现类中，DispatcherServlet 在初始化过程会把 IoC 容器中所有的 HandlerExceptionResolver 的实现类排好序后组装起来用于异常处理。

现在我们使用 Spring 来开发 web 应用时应该都会选择 Spring Boot 来配置 Spring，和异常相关的自动配置类为 ErrorMvcAutoConfiguration 和 WebMvcAutoConfiguration， 它们默认配置两个 HandlerExceptionResolver: DefaultErrorAttributes 和 HandleExceptionResolverComposite。

HandleExceptionResolverComposite 默认包含以下三个 HandlerExceptionResolver:  

> * ExceptionHandlerExceptionResolver matches uncaught exceptions against suitable @ExceptionHandler methods on both the handler (controller) and on any controller-advices.
> * ResponseStatusExceptionResolver looks for uncaught exceptions annotated by @ResponseStatus (as described in Section 1)
> * DefaultHandlerExceptionResolver converts standard Spring exceptions and converts them to HTTP Status Codes (I have not mentioned this above as it is internal to Spring MVC).

Spring 官方博客帮我们总结了 Spring Boot 默认配置的异常处理流程：

> 1.	In the event of any unhanded error, Spring Boot forwards internally to /error.
> 2.	Boot sets up a BasicErrorController to handle any request to /error. The controller adds error information to the internal Model and returns error as the logical view name.
> 3.	If any view-resolver(s) are configured, they will try to use a corresponding error-view.
> 4.	Otherwise, a default error page is provided using a dedicated View object (making it independent of any view-resolution system you may be using).
> 5.	Spring Boot sets up a BeanNameViewResolver so that /error can be mapped to a View of the same name.
> 6.	If you look in Boot’s ErrorMvcAutoConfiguration class you will see that the defaultErrorView is returned as a bean called error. This is the View bean found by the BeanNameViewResolver.

对于 Servlet-Container 层面的错误处理，Spring 官方博客的介绍如下：  

> Container-Wide Exception Handling
> 
> Exceptions thrown outside the Spring Framework, such as from a servlet Filter, are also reported by Spring Boot’s fallback error page.
> To do this Spring Boot has to register a default error page for the container. In Servlet 2, there is an `<error-page>` directive that you can add to your web.xml to do this. Sadly Servlet 3 does not offer a Java API equivalent. Instead Spring Boot does the following:
> 	
> * For a Jar application, with an embedded container, it registers a default error page using Container specific API.
> * For a Spring Boot application deployed as a traditional WAR file, a Servlet Filter is used to catch exceptions raised further down the line and handle it.

我们可以按照上述线索在 Spring Boot 的自动配置代码中找到相关的代码。

当开发 REST API 项目时，我希望业务抛出的异常能契合 Spring Boot 默认配置的异常处理机制，让整个异常体系尽量统一，接口返回给终端统一格式的错误信息，这样终端也能统一处理接口错误。那么我们应该如何做？

我们这里需要的是一个全局的异常处理机制，Spring MVC 提供给我们两种配置全局异常处理的方法：  

* 配置 HandlerExceptionResolver
* 使用 `@ControllerAdvice` 注解的类

相比之下，个人觉得使用 `@ControllerAdvice` 注解的类会方便一些，能达到感知框架的存在。我们可以定义一个异常处理基类，发布成一个库，然后在需要用到的项目中引入这个库，在项目中继承该基类定义一个 `@ControllerAdvice` 注解的异常处理类。  

选好全局异常处理机制后，那么我们应该如何来设计项目的业务异常类呢?   

Spring 官方博客给出了如下建议:

> As usual, Spring likes to offer you choice, so what should you do? Here are some rules of thumb. However if you have a preference for XML configuration or Annotations, that’s fine too.
> 
> * For exceptions you write, consider adding @ResponseStatus to them.
> * For all other exceptions implement an @ExceptionHandler method on a @ControllerAdvice class or use an instance of SimpleMappingExceptionResolver. You may well have SimpleMappingExceptionResolver configured for your application already, in which case it may be easier to add new exception classes to it than implement a @ControllerAdvice.
> * For Controller specific exception handling add @ExceptionHandler methods to your controller.
> * Warning: Be careful mixing too many of these options in the same application. If the same exception can be handed in more than one way, you may not get the behavior you wanted. @ExceptionHandler methods on the Controller are always selected before those on any @ControllerAdvice instance. It is undefined what order controller-advices are processed.
<!--more-->
这里我们重点来看第一条建议，他建议我们自己写的异常类可以考虑加上 `@ResponseStatus` 注解，这样 service 就可以往上层传递 HTTP 的 status 信息，然后可以根据异常的类型填充 body 信息。这样做当然可以，只是这样一来异常类数量容易膨胀，定义异常类也是很乏味。我觉得可以定义一个能表达 HTTP status, headers 和 body 信息的类，然后抛出它的实例，他的建议作为补充。  

设计好异常类层级后，接口出错时应该返回些什么信息给终端？

《Web API 的设计与开发》作者建议的单个和多个错误信息如下：  

```
// 单个
{
  "error": {
      "code": 2013,
      "message": "Bad authentication token",
      "info": "http://docs.example.com/api/v1/authentication"
  }
}
// 多个
{
  "errors": [
      {
          "code": 2013,
          "message": "Bad authentication token",
          "info": "http://docs.example.com/api/v1/authentication"
      }
  ]
}

```

同样，我还是希望业务异常产生的错误信息能够兼容 Spring Boot 默认产生的错误信息，这样终端可以统一处理错误信息。Spring Boot 默认产生的错误信息包含如下字段：


```
timestamp - The time that the errors were extracted
status - The status code
error - The error reason
exception - The class name of the root exception (if configured)
message - The exception message (if configured)
errors - Any ObjectErrors from a BindingResult exception (if configured)
trace - The exception stack trace (if configured)
path - The URL path when the exception was raised
```

综上所述，我们可以定义 REST API 的错误信息为 RestErrorInfo:  

```
public class RestErrorInfo {
    private Integer code;

    private String message;

    private String info;
    
    // 省略的 Constructor, Getter and Setter
}
```

异常类 ServiceException 为：  

```
package com.github.damiansheldon.exception;

import org.springframework.core.NestedRuntimeException;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;

public class ServiceException extends NestedRuntimeException {

    private HttpStatus status;

    private HttpHeaders headers;

    private RestErrorInfo errorInfo;

    private ServiceException(Builder builder) {
        super(builder.errorInfo.getMessage(), builder.cause);
        this.errorInfo = builder.errorInfo;
    }

    public HttpStatus getStatus() {
        return status;
    }

    public HttpHeaders getHeaders() {
        return headers;
    }

    public RestErrorInfo getErrorInfo() {
        return errorInfo;
    }

    public static class Builder {
        private HttpStatus status = HttpStatus.INTERNAL_SERVER_ERROR;
        private HttpHeaders headers;
        private RestErrorInfo errorInfo;
        private Throwable cause;

        public Builder(RestErrorInfo errorInfo) {
            this.errorInfo = errorInfo;
        }

        public Builder status(HttpStatus status) {
            this.status = status;
            return this;
        }

        public Builder headers(HttpHeaders headers) {
            this.headers = headers;
            return this;
        }

        public Builder cause(Throwable cause) {
            this.cause = cause;
            return this;
        }

        public ServiceException build() {
            return new ServiceException(this);
        }

    }

}

```

异常处理类为：   

```
package com.github.damiansheldon.exception;

import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.lang.Nullable;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.util.WebUtils;

public class ServiceExceptionHandler {

    @ExceptionHandler({ServiceException.class})
    public final ResponseEntity<Object> handleServiceException(ServiceException ex, WebRequest request) {
        return handleExceptionInternal(ex, ex.getErrorInfo(), ex.getHeaders(), ex.getStatus(), request);
    }

    protected ResponseEntity<Object> handleExceptionInternal(
            Exception ex, @Nullable Object body, HttpHeaders headers, HttpStatus status, WebRequest request) {

        if (HttpStatus.INTERNAL_SERVER_ERROR.equals(status)) {
            request.setAttribute(WebUtils.ERROR_EXCEPTION_ATTRIBUTE, ex, WebRequest.SCOPE_REQUEST);
        }
        return new ResponseEntity<>(body, headers, status);
    }
}

```

完整代码在[这里](https://github.com/DamianSheldon/Treasure)。  

#Reference

* [Exception Handling in Spring MVC](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc)  
* [Java™ Servlet Specification](https://javaee.github.io/servlet-spec/downloads/servlet-3.1/Final/servlet-3_1-final.pdf)  


