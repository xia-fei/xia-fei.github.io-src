---
title: maven仓库的研究
date: 2018-06-23 18:19:57
tags: maven
---
经常有时候下载特别慢。于是乎仔细看了下maven的仓库配置,发现
##### maven 仓库地址有如下配置:  
<!-- more -->
**pom.xml**
``` xml
<pluginRepositories>
    <pluginRepository>
      <id>spring-snapshots</id>
      <name>Spring Snapshots</name>
      <url>https://repo.spring.io/snapshot</url>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
    </pluginRepository>
<pluginRepositories>    

<repositories>
    <repository>
      <id>spring-snapshots</id>
      <name>Spring Snapshots</name>
      <url>https://repo.spring.io/snapshot</url>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
    </repository>
</repositories>
```
**settings.xml**
``` xml
<mirror>
    <id>CN</id>
    <name>OSChina Central</name>
    <url>http://maven.oschina.net/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
<profile>
     <id>dev</id>
     <repositories>
         <repository>
             <id>nexus-public</id>
             <name>Public Repositories</name>
             <url>http://192.168.2.126:8081/nexus/content/groups/public/</url>
         </repository>
    </repositories>
     <pluginRepositories>
         <pluginRepository>
             <id>public</id>
             <name>Public Repositories</name>
             <url>http://192.168.2.126:8081/nexus/content/groups/public/</url>
         </pluginRepository>
     </pluginRepositories>
     <activeProfiles>
        <activeProfile>dev</activeProfile>
    </activeProfiles>
</profile>   
```
首先我们有几个问题：
1. 这么多配仓库的地方,配置的是什么仓库?
2. 多个仓库使用哪一个,优先级是什么?

第一个问题
 配的仓库其实就两种  
`<repositories>` 就一般的远程仓库  
`<pluginRepositories>` 这个是下载插件的仓库    
私服和第三方远程仓库都是 远程仓库  只不过私服会缓存下来，方便第二次下载jar包局域网更快

第二个优先级
1. 按照`settings.xml` 的`<repositories>` 里的顺序从上往下
2. `pom.xml` 的`<repositories>` 优先级是最低的  
也就是说你先 使用指定的仓库 直接配置到`<repositories>` 第一个最快捷，更合理的方式是使用`<activeProfiles>`来切换吧

分享一个国内非常快的aliyun maven仓库,大家可以试试效果
```xml
<repository>
      <id>alibaba-nexus</id>
      <name>alibaba maven nexus</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <releases>
        <enabled>true</enabled>
      </releases>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
```



**仓库分类**
1. 本地仓库 
2. 远程仓库
    1.  中央仓库
    2.  私服
    3.  其它公共仓库  





**本地仓库** 就是本地文件夹配置在`settings.xml`里的 `<localRepositoryD:\maven_new_repository</localRepository`

**中央仓库** 只得就是maven刚安装好后的maven中央仓库`<url>http://repo.maven.apache.org/maven2</url>`  

**远程仓库** 通过`<repositories>  <repository>` 配置的就是远程仓库

**私服** 这是一种特殊的远程仓库,简单来讲就是为了缓存 其它远程仓库的请求
1. 减少重复请求造成的外网带宽消耗
2. 有些构件无法从外部仓库获得的时候，我们可以把这些构件部署到内部仓库(私服)中，供内部maven项目使用
3. 降低中央仓库的负荷：maven中央仓库被请求的数量是巨大的，配置私服也可以大大降低中央仓库的压力
**插件仓库** `<pluginRepository>` 这个是配置插件的下载仓库
学习下**repositories**配置
``` xml
<repositories>  
        <repository>  
            <id>jboss</id>  
            <name>JBoss Repository</name>  
            <url>http://repository.jboss.com/maven2/</url>  
            <releases>  
                <updatePolicy>daily</updatePolicy><!-- never,always,interval n -->  
                <enabled>true</enabled>  
                <checksumPolicy>warn</checksumPolicy><!-- fail,ignore -->  
            </releases>  
            <snapshots>  
                <enabled>false</enabled>  
            </snapshots>  
            <layout>default</layout>  
        </repository>  
    </repositories>  
```
`<updatePolicy>`元素：表示更新的频率，值有：never, always,interval,daily, daily 为默认值
`<checksumPolicy>`元素：表示maven检查和检验文件的策略，warn为默认值
远程仓库的认证配置在  
``` xml
 <server>  
       <id>same with repository id in pom</id>  
       <username>username</username>  
       <password>pwd</password>  
 </server>  
```
**注：这里的id必须与POM中需要认证的repository元素的Id一致。**

**如何将生成的项目部署到远程仓库**
完成这项工作，也需要在POM中进行配置，这里有新引入了一个元素：`<distributionManagement>`
distributionManagement包含了2个子元素：`repository`和`snapshotRepository`, 前者表示发布版本构件的仓库，后者表示快照版本的仓库
这两个元素都需要配置 id(该远程仓库的唯一标识)，name，url(表示该仓库的地址)
向远程仓库中部署构件，需要进行认证。配置同上



##### maven到底是如何从仓库中解析构件的呢？
1. 依赖范围是system的时候->**本地文件系统**
2. 尝试直接从**本地仓库**寻找构件，如果发现相应构件，则解析成功
3. 在**本地仓库**不存在相应的构件情况下，如果依赖的版本是显示的发布版本构件，则**遍历所有的远程仓库**