---
title: mongoDB部署笔记
date: 2018-12-19 17:51:52
tags: mongo
---
## mongodb启动方式

1  bin/mongod 

 默认数据存/data/db,  ctrl+c就退出

2.bin/mongod  --fork  

 
--syslog  默认日志位置  /var/log/messages

--fork 以守护进程运行
 <!-- more -->

## 关闭方式

 db.shutdownServer()

kill -2 pid

 
## 集群启动
### 1.副本集部署

https://docs.mongodb.com/v3.2/tutorial/deploy-replica-set/

每台上启动

mongod --replSet "rs0"
第一台初始化集群

rs.initiate() 
添加机器

rs.add("mongodb1.example.net")
 

### 2.分片部署

https://docs.mongodb.com/v3.2/tutorial/deploy-sharded-cluster-hashed-sharding/

启动配置服务器

mongod --configsvr --replSet <setname> --dbpath <path>

启动分片服务器

mongod --shardsvr --replSet <replSetname>
 

启动mongos

mongos --configdb <configReplSetName>/cfg1.example.net:27017,cfg2.example.net:27017
添加分片
sh.addShard( "<replSetName>/s1-mongo1.example.net:27017")
 

连接到mongos ip

sh.enableSharding("<database>")
 

hash分片

sh.shardCollection("<database>.<collection>", { <key> : "hashed" } )
 

range分片

sh.shardCollection("<database>.<collection>", { <key> : <direction> } )
eg: sh.shardCollection("records.people",{zipcode:1})
 

 

 

## 实战部署


启动1台 configServer
启动2台 sharedServer
启动1台 mongos

mkdir /data/configdb
mkdir /data/shardingdb

/data/mongo/bin/mongod --configsvr --dbpath /data/configdb --fork --syslog
/data/mongo/bin/mongod --shardsvr --dbpath /data/shardingdb --fork --syslog
/data/mongo/bin/mongos --configdb hmaster:28017

configdb data 
/data/configdb port=27019

sharding data
/data/shardingdb port=27018


mongos 
启用分片
sh.enableSharding("<database>")

建立hashed索引
db.user.ensureIndex({_id:"hashed"})

集合建立。hash分片
sh.shardCollection("<database>.<collection>", { _id : "hashed" } )

