---
title: 搞定springMVC vue history分析
date: 2019-01-15 17:05:30
tags: spring-mvc
---


## 动机
解决 vue 路由 history模式  
将所有`/**/*` 定位为到同一个html文件
<!-- more -->

要解决的问题  
1. 此路由匹配优先级得在controller之后  
2. 此路由匹配优先级得再 静态资源之前,应为springMVC默认 有/**兜底

springMVC简单来说就是一个Servlet  
`org.springframework.web.servlet.DispatcherServlet`
一切代码都是围绕`protected void doDispatch(HttpServletRequest request, HttpServletResponse response)`展开


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

## 解决一

返回指定html文件
```
public class HtmlRequestMapping implements HttpRequestHandler {
    private String htmlPath;

    public HtmlRequestMapping(String htmlPath) {
        this.htmlPath = htmlPath;
    }

    @Override
    public void handleRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType(MediaType.TEXT_HTML_VALUE);
        FileCopyUtils.copy(Thread.currentThread().getContextClassLoader().getResourceAsStream(htmlPath),
                response.getOutputStream());
    }
}

```

根据请求规则返回指定文件
``` java
public class VueSinglePage extends AbstractUrlHandlerMapping {



    public VueSinglePage() {
        registerHandler("/**/{path:[^.]+}",new HtmlRequestMapping("public/dist/index.html"));
        registerHandler("/",new HtmlRequestMapping("public/dist/index.html"));
    }

    @Override
    public int getOrder() {
        return Ordered.LOWEST_PRECEDENCE-1;
    }
}
```

## 解决二 (推荐)
思路会按照controller->static-mapping-> index.html 这个优先级来

```properties
spring.resources.add-mappings=false
```
```java
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**/*.css", "/**/*.js", "/**/*.ttf", "/**/*.woff", "/**/*.ico").addResourceLocations("classpath:/public/dist/");
    }
```
```java

    @Bean
    HandlerMapping vueHandlerMapping() {
        return new VueHandlerMapping();
    }

    class VueHandlerMapping implements HandlerMapping, Ordered {
        @Override
        public HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
            return new HandlerExecutionChain(new HttpRequestHandler() {
                @Override
                public void handleRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                    response.setContentType(MediaType.TEXT_HTML_VALUE);
                    FileCopyUtils.copy(Thread.currentThread().getContextClassLoader().getResourceAsStream("public/dist/index.html"),
                            response.getOutputStream());
                }
            });
        }

        @Override
        public int getOrder() {
            return Ordered.LOWEST_PRECEDENCE;
        }
    }

```
