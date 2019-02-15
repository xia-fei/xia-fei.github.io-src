---
title: RocketMQ多主部署情况分析
date: 2018-09-29 09:57:18
tags: mq
---
验证rocketMQ两主部署

准备工作两台服务器
A nameserver broker1
B broker2 
<!-- more -->
配置文件 集群名称一致,brokerId=0 相当于(主),brokerName不一致
```
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=0
```

nameserver启动
``` shell
[admin@172_30_229_146 bin]$ nohup sh mqnameserver &
```
两台主broker启动
``` shell
[admin@172_30_229_146 bin]$ nohup sh mqbroker -c ../conf/2m-noslave/broker-b.properties  -n localhost:9876 &
[admin@172_30_223_202 bin]$ nohup sh mqbroker -c ../conf/2m-noslave/broker-b.properties -n 172.30.229.146:9876 &
```


1.看下 broker 和nameserver部署情况
``` shell
[admin@172_30_229_146 bin]$ ./mqadmin clusterList -n localhost:9876
#Cluster Name     #Broker Name            #BID  #Addr                  #Version                #InTPS(LOAD)       #OutTPS(LOAD) #PCWait(ms) #Hour #SPACE
DefaultCluster    broker-a                0     172.30.229.146:10911   V4_3_0                   0.00(0,0ms)         0.00(0,0ms)          0 427206.54 -1.0000
DefaultCluster    broker-b                0     172.30.223.202:10911   V4_3_0                   0.00(0,0ms)         0.00(0,0ms)          0 427206.54 -1.0000

```
可知两台 broker 同一个集群都是主

创建topic 这里是关键` -c DefaultCluster`
``` shell
[admin@172_30_229_146 bin]$ ./mqadmin updateTopic  -c DefaultCluster -t xiafei -n localhost:9876
create topic to 172.30.229.146:10911 success.
create topic to 172.30.223.202:10911 success.
TopicConfig [topicName=xiafei, readQueueNums=8, writeQueueNums=8, perm=RW-, topicFilterType=SINGLE_TAG, topicSysFlag=0, order=false][admin@172_30_229_146 bin]$ 

```


### 客户端发消息测试
发第一条
```shell
[admin@172_30_229_146 bin]$ ./mqadmin topicStatus -t xiafei -n localhost:9876
#Broker Name                      #QID  #Min Offset           #Max Offset             #Last Updated
broker-a                          0     0                     0                       
broker-a                          1     0                     0                       
broker-a                          2     0                     0                       
broker-a                          3     0                     0                       
broker-a                          4     0                     0                       
broker-a                          5     0                     0                       
broker-a                          6     0                     0                       
broker-a                          7     0                     0                       
broker-b                          0     0                     0                       
broker-b                          1     0                     0                       
broker-b                          2     0                     0                       
broker-b                          3     0                     0                       
broker-b                          4     0                     0                       
broker-b                          5     0                     0                       
broker-b                          6     0                     1                       2018-09-26 14:50:45,747
broker-b                          7     0                     0                       
```
发第二条
``` shell
[admin@172_30_229_146 bin]$ ./mqadmin topicStatus -t xiafei -n localhost:9876
#Broker Name                      #QID  #Min Offset           #Max Offset             #Last Updated
broker-a                          0     0                     0                       
broker-a                          1     0                     0                       
broker-a                          2     0                     0                       
broker-a                          3     0                     0                       
broker-a                          4     0                     0                       
broker-a                          5     0                     0                       
broker-a                          6     0                     1                       2018-09-26 14:51:13,122
broker-a                          7     0                     0                       
broker-b                          0     0                     0                       
broker-b                          1     0                     0                       
broker-b                          2     0                     0                       
broker-b                          3     0                     0                       
broker-b                          4     0                     0                       
broker-b                          5     0                     0                       
broker-b                          6     0                     1                       2018-09-26 14:50:45,747
broker-b                          7     0                     0                       

```

#### 结论:
多主部署时,相当于分片.
消息会均衡的分散在个台broker上
至于 `多从` 情况就是从主上复制嘛!
然后选择同步或者异步复制

>由此可以部署上与kafka不一样的地方.就是在主从部署时,RocketMQ这个主服务器,不能同时作为主服务器或者从服务器.
>kafka是将各个topic的队列分散存在各个机器上.每个机器上队列可以是主,亦可以是从.比较灵活