---
title: springMVC请求流程
date: 2019-01-15 17:05:30
tags: spring
---




springMVC简单来说就是一个Servlet  
`org.springframework.web.servlet.DispatcherServlet`
一切代码都是围绕`protected void doDispatch(HttpServletRequest request, HttpServletResponse response)`展开
<!-- more -->

## 代码机构
- org.springframework.web.servlet.HandlerMapping
    - org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping  **controller**
        - org.springframework.web.method.HandlerMethod  **controller-method**
    - org.springframework.web.servlet.handler.SimpleUrlHandlerMapping
        - org.springframework.web.HttpRequestHandler
            - org.springframework.web.servlet.resource.ResourceHttpRequestHandler 

- org.springframework.web.servlet.HandlerAdapter
    - org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter
    - org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter

首先会有很多个`org.springframework.web.servlet.HandlerMapping`
```java
public interface HandlerMapping {
    HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
}
```
![](https://xiafei-web.oss-cn-hangzhou.aliyuncs.com/file-server/1547537892389.png)  
**Controller**会抽象成一个`org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping`

**Method**会抽象成`org.springframework.web.method.HandlerMethod`储存在
`org.springframework.web.servlet.handler.AbstractHandlerMethodMapping.MappingRegistry`


静态资源
```xml
<mvc:resources mapping="/resources/**" location="/public, classpath:/static/" cache-period="31556926" />
```
这种静态资源会抽象成`org.springframework.web.servlet.handler.SimpleUrlHandlerMapping`
里面静态资源路劲抽象成
`org.springframework.web.HttpRequestHandler`
```java
public interface HttpRequestHandler {
    void handleRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```

```java
public abstract class AbstractUrlHandlerMapping extends AbstractHandlerMapping implements MatchableHandlerMapping {

    private Object rootHandler;

    private boolean useTrailingSlashMatch = false;

    private boolean lazyInitHandlers = false;

    //存放着
    private final Map<String, Object> handlerMap = new LinkedHashMap<String, Object>();
    }
```

路径匹配  
```java
new AntPathMatcher(String pattern, String path).match();
new UrlPathHelper().getLookupPathForRequest(HttpServletRequest request);
```
