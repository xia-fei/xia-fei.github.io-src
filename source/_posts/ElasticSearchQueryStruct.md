---
title: ElasticSearch查询结构
catalog: true
date: 2020-09-19 19:40:32
subtitle: ES查询语法结构体系
header-img:
tags: elasticsearch
---

query结构

- query
    - 全文检索相关
    - 条件过滤相关 
    - 组合上面两个 bool query 
        - must (and)
        - must_not (not)
        - should   (or)
        - filter  (条件过滤与term类似,性更高)
 - aggregations 聚合
 - post_filter 结果过滤,聚合不过滤    
        
   
        
- 全文检索  
    - match  (全文检索,会对输入结果分词 相当于分词or)
    - match_phrase (短语查询,会对内容整个去匹配,相当于全部and)
    - query_string (查询语法)
    
    
- 条件判断查询  [文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/term-level-queries.html#term-level-query-types)
    - Exists
    - Fuzzy
    - IDs
    - Prefix
    - Range
    - Regexp
    - Term
    - Terms
    - Terms set
    - Type Query
    - Wildcard
    