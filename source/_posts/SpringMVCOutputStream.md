---
title: SpringMVC优雅的返回文件流
date: 2018-06-13 15:28:28
tags: Spring
---
springMVC下载文件方法很多，大致看了下，觉得下面这种方法最正规，最符合官网文档介绍。
代码主要有两部分组成  
`StreamingResponseBody` 用来异步下载文件，不占用tomcat工作住线程  
`ResponseEntity` 用来设置头信息,定义下载的文件名称
<!-- more -->

``` java
  @RequestMapping("/downloadCategoryExcel")
    public ResponseEntity<StreamingResponseBody> downloadCategoryExcel(final String param) {
        StreamingResponseBody streamingResponseBody = new StreamingResponseBody() {
            @Override
            public void writeTo(OutputStream outputStream) throws IOException {
                String categoryCode = JSON.parseObject(param).getString("categoryCode");
                Workbook workbook = excelCreateService.createCategoryTemplate(categoryCode);
                workbook.write(outputStream);
            }
        };
        ResponseEntity.BodyBuilder bodyBuilder=ResponseEntity.ok();
        bodyBuilder.contentType(MediaType.parseMediaType("application/vnd.ms-excel"));
        bodyBuilder.header("Content-Disposition", "attachment;filename=1.xlsx");
        bodyBuilder.body(streamingResponseBody);
        return bodyBuilder.build();
    }
```


[springMVC官网介绍](https://docs.spring.io/spring/docs/4.3.12.RELEASE/spring-framework-reference/htmlsingle/#mvc-ann-async-output-stream)
里面有有段话
>Note that `StreamingResponseBody` can also be used as the body in a `ResponseEntity` in order to customize the status and headers of the response.