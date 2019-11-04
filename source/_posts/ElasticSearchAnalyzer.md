---
title: ES分词器笔记
catalog: true
date: 2019-11-01 14:59:13
subtitle:
tags: es
---

# ES分词器

## 关键组件组成
### char filter  字符过滤器
对文本进行字符过滤处理，如处理文本中的html标签字符。处理完后再交给tokenizer进行分词。一个analyzer中可包含0个或多个字符过滤器，多个按配置顺序依次进行处理。
例如 去掉html标记

### tokenizer 分词器
对文本进行分词 例如 按单词的空格  
一个analyzer必需且只可包含一个tokenizer

### token filter  词项过滤器
对分过的词进行过滤处理 如转小写、同义词处理,停用词。
>停用词 就是用来过滤掉无用的词 比如英文的 `the` `a` 



## 	索引分词器设置

```
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": { ... custom character filters ... },
            "tokenizer":   { ...    custom tokenizers     ... },
            "filter":      { ...   custom token filters   ... },
            "analyzer":    { ...    custom analyzers      ... }
        }
    }
}
```

```
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type":       "mapping",
                    "mappings": [ "&=> and "]
            }},
            "filter": {
                "my_stopwords": {
                    "type":       "stop",
                    "stopwords": [ "the", "a" ]
            }},
            "analyzer": {
                "my_analyzer": {
                    "type":         "custom",
                    "char_filter":  [ "html_strip", "&_to_and" ],
                    "tokenizer":    "standard",
                    "filter":       [ "lowercase", "my_stopwords" ]
            }}
}}}
```


## 内置组件

### char filter
html_strip ：过滤html标签，解码HTML entities like &amp;.
mapping ：用指定的字符串替换文本中的某字符串。
pattern_replace ：进行正则表达式替换。

### Tokenizer
- Letter Tokenizer
- Lowercase Tokenizer
- Whitespace Tokenizer
- UAX URL Email Tokenizer
- Classic Tokenizer
- Thai Tokenizer
- NGram Tokenizer
- Edge NGram Tokenizer
- Keyword Tokenizer
- Pattern Tokenizer
- Simple Pattern Tokenizer
- Simple Pattern Split Tokenizer
- Path Hierarchy Tokenizer

### Token Filter
Lowercase Token Filter ：lowercase 转小写  
Stop Token Filter ：stop 停用词过滤器  
Synonym Token Filter： synonym 同义词过滤器  



### Analyzer
- Standard Analyzer 	 
- Simple Analyzer  
- Whitespace Analyzer  
- Stop Analyzer  
- Keyword Analyzer  
- Pattern Analyzer  
- Language Analyzers  
- Fingerprint Analyzer   


### 参考文档

[https://www.cnblogs.com/leeSmall/p/9195782.html](https://www.cnblogs.com/leeSmall/p/9195782.html)

