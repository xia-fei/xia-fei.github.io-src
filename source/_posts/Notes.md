---
title: 技术笔记Notes
catalog: true
date: 2019-11-04 09:54:23
subtitle:
header-img:
tags: mongodb,servlet
---

dubbo RPCcontext 分析
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gi5ixfmoncj31lc0u07bg.jpg)

# MongoDB实现case  when 和 id in的查询

```javascript
db.store.aggregate([{
    $project: {
        "nczStoreId": 1,
        "storeName": 1,
        "groupIds": 1,
        "priority": {
            $cond: { if: { $setIsSubset: [ "$groupIds",[1000] ]}, then: 1, else: 0 }
        }
    }
}])
```


放回 当groupIds集合中包含1000 返回auth=true	

$cond 判断表达式文档
[https://docs.mongodb.com/v3.2/meta/aggregation-quick-reference/#expressions](https://docs.mongodb.com/v3.2/meta/aggregation-quick-reference/#expressions)

## 它有下面几种表达式判断

#### Boolean Expressions 
$and $or 用于表达式嵌套
result: { $and: [ { $gt: [ "$qty", 100 ] }, { $lt: [ "$qty", 250 ] } ] }

#### Set Expressions 	 
判断集合是否包含		$setIsSubset

#### ComparisonExpressions
$eq $gt

#### Arithmetic Expressions
用来做计算$abs $floor

#### String Expressions
字符串运算
$concat $substr

#### Text Search expressions
$meta

#### Array Expressions
$size	

#### Date Expressions

等等	


### JavaAPI实现
```java
    BasicDBObject bson = new BasicDBObject();
        bson.put("$eval","db.store.aggregate([{\n" +
                "    $project: {\n" +
                "        \"nczStoreId\": 1,\n" +
                "        \"storeName\": 1,\n" +
                "        \"groupIds\": 1,\n" +
                "        \"auth\": {\n" +
                "            $cond: { if: { $setIsSubset: [ \"$groupIds\",[1000] ]}, then: true, else: false }\n" +
                "        }\n" +
                "    }\n" +
                "}])");
        Object o=mongoTemplate.getDb().runCommand(bson);
        System.out.println(o.toString());
```


# Servlet笔记

### HttpServletRequest

GET http://localhost:8080/test?a=1

### request.getRequestURL();
http://localhost:8080/test

### request.getRequestURI();
/test

### request.getQueryString();
a=1








