---
title: Nginx location总结
catalog: true
date: 2019-10-08 20:22:22
subtitle:
header-img:
tags: nginx
---



### 语法规则
```
location [ = | ~ | ~* | ^~ ] uri { ... }
```

简单来说 匹配它匹配的默认就分两种

##### 1.前缀匹配
类似下面这种
`location /user {}`


##### 2.正则匹配
`location ~ /u.*`
`location ~* /u.*`

##### 3.精确匹配
`location = /user.html`


### 优先级

`^~` 这个优先级最高

然后按照
`正则>前缀匹配`

要想前缀匹配 大于正则 使用 `^~` 

### pass_proxy 末尾加`/` 作用

`pass_proxy http:192.168.0.55:3000`  
`pass_proxy http:192.168.0.55:3000/`

加`/`的作用就是 会去掉前缀匹配的部分,然后去反向代理后端服务器
```
location /a/ {
	pass_proxy http:192.168.0.55:3000/
}

```

### alias
相当于丢弃 location 部分URL

例如:  
用户请求 `/a/ccc`
加`/` 实际到后端服务请求是 `/ccc`
不加`/` 实际请求是`/a/ccc`


