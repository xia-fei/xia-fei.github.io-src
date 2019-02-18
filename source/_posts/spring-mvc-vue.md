---
title: SpringMVC 配置html5 history模式
catalog: true
date: 2019-02-18 11:39:03
subtitle:
header-img:
tags: spring
---


##### 最简单方式：
配置404页面 
```java
   @Bean
    public ErrorPageRegistrar errorPageRegistrar(){
        return registry -> registry.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/index.html"));
    }
```
问题：
1. 所有404都回定向到html 报错一些不存在的js，jpg文件，都返回index.html里面内容了
2. html页面状态码是404 看起来很不和谐


##### 分析问题
springMVC 请求请求顺序
1. Controller
2. static-resources 有/**兜底

##### 解决一：(推荐)
我们可以在Controller里面配置一个区别于接口的 路径 返回index.html文件
例如
```java
@RequestMapping("/html/**");
```
感觉对路径侵入有点大必须html开头

换种匹配规则 所有 路径不带`.`的
```java
@RequestMapping("/**/{path:[^.]+}");
```

升级写法,写一个HandlerMapping放在`Controller`后面
```java
@Component
public class VueSinglePage extends AbstractUrlHandlerMapping {
    public VueSinglePage() {
        HttpRequestHandler httpRequestHandler = (request, response) -> {
            response.setContentType(MediaType.TEXT_HTML_VALUE);
            FileCopyUtils.copy(Objects.requireNonNull(Thread.currentThread().getContextClassLoader().getResourceAsStream("static/index.html")),
                    response.getOutputStream());
        };
        registerHandler("/**/{path:[^.]+}", httpRequestHandler);
        registerHandler("/", httpRequestHandler);
    }

    @Override
    public int getOrder() {
        return Ordered.LOWEST_PRECEDENCE - 1;
    }
}

```

##### 解决二
static-resources 把/**兜底 关掉   
写一个全匹配的HandlerMapping插入在12后面

```properties
spring.resources.add-mappings=false
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


