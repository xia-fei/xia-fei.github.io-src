---
title: YUM包管理
date: 2018-12-20 15:32:11
tags: Liunx
---
### rpm包
rpm是由红帽公司开发的软件包管理方式，使用rpm我们可以方便的进行软件的安装、查询、卸载、升级等工作。但是rpm软件包之间的依赖性问题往往会很繁琐,尤其是软件由多个rpm包组成时。


## rpm库的地址  
配置在
`/etc/yum.repos.d/*`
下面各个个文件中
<!-- more -->
```
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/mainline/centos/7/$basearch/
gpgcheck=0
enabled=1
```

** /etc/yum.repos.d/ **修改完仓库之后要清除下缓存才能搜索的到
`yum makecache`

yum 的repo 配置文件中可用的变量：
$releasever:当前OS的发行版的主版本号$arch: 平台，i386,i486,i586,x86_64等
$basearch ：基础平台；i386, x86_64

查看本机yum仓库
`yum repolist all`  
去掉`all` 就是查看已启用的文件服务器


`baseurl`就是存放**.rpm**文件的服务器  
可以打开nginx rpm包服务器看下  
[https://nginx.org/packages/mainline/centos/7/](https://nginx.org/packages/mainline/centos/7/)

yum search 其实就是去仓库地址扫描所有rpm文件

### rpm

```
[root@izwz92672j5u4uc9wvpopgz rpm]# rpm -qi nginx
Name        : nginx
Epoch       : 1
Version     : 1.15.5
Release     : 1.el7_4.ngx
Architecture: x86_64
Install Date: Thu 25 Oct 2018 09:45:45 AM CST
Group       : System Environment/Daemons
Size        : 2786120
License     : 2-clause BSD-like license
Signature   : RSA/SHA1, Tue 02 Oct 2018 11:48:24 PM CST, Key ID abf5bd827bd9bf62
Source RPM  : nginx-1.15.5-1.el7_4.ngx.src.rpm
Build Date  : Tue 02 Oct 2018 11:27:21 PM CST
Build Host  : centos74-amd64-builder-builder.gnt.nginx.com
Relocations : (not relocatable)
Vendor      : Nginx, Inc.
URL         : http://nginx.org/
Summary     : High performance web server
Description :
```
** 其中第一行**  
**name就是软件名，给yum search 时匹配的名字**



1.我的系统中安装了那些rpm软件包
>rpm -qa 列出所有安装过的包

2、如何获得某个软件包的文件全名。
>rpm -q mysql

3.查询rpm包详细信息
>rpm -qi nginx

4、一个rpm包中的文件安装到那里去了？
>rpm -ql 包名

5、查看包依赖
>rpm -qpR file.rpm　　　　　　　＃[查看包]依赖关系

其它命令
```
rpm 执行安装包
二进制包（Binary）以及源代码包（Source）两种。二进制包可以直接安装在计算机中，而源代码包将会由 RPM自动编译、安装。源代码包经常以src.rpm作为后缀名。
常用命令组合：
－ivh：安装显示安装进度--install--verbose--hash
－Uvh：升级软件包--Update；
－qpl： 列出RPM软件包内的文件信息[Query Package list]；
－qpi：列出RPM软件包的描述信息[Query Package install package(s)]；
－qf：查找指定文件属于哪个RPM软件包[Query File]；
－Va：校验所有的 RPM软件包，查找丢失的文件[View Lost]；
－e：删除包
rpm -q samba //查询程序是否安装
rpm -ivh /media/cdrom/RedHat/RPMS/samba-3.0.10-1.4E.i386.rpm //按路径安装并显示进度
rpm -ivh --relocate /=/opt/gaim gaim-1.3.0-1.fc4.i386.rpm    //指定安装目录
rpm -ivh --test gaim-1.3.0-1.fc4.i386.rpm　　　 //用来检查依赖关系；并不是真正的安装；
rpm -Uvh --oldpackage gaim-1.3.0-1.fc4.i386.rpm //新版本降级为旧版本
rpm -qa | grep httpd　　　　　 ＃[搜索指定rpm包是否安装]--all搜索*httpd*
rpm -ql httpd　　　　　　　　　＃[搜索rpm包]--list所有文件安装目录
rpm -qpi Linux-1.4-6.i368.rpm　＃[查看rpm包]--query--package--install package信息
rpm -qpf Linux-1.4-6.i368.rpm　＃[查看rpm包]--file
rpm -qpR file.rpm　　　　　　　＃[查看包]依赖关系
rpm2cpio file.rpm |cpio -div    ＃[抽出文件]
rpm -ivh file.rpm 　＃[安装新的rpm]--install--verbose--hash
rpm -ivh [url]http://mirrors.kernel.org/fedora/core/4/i386/os/Fedora/RPMS/gaim-1.3.0-1.fc4.i386.rpm[/url] 
rpm -Uvh file.rpm    ＃[升级一个rpm]--upgrade
rpm -e file.rpm      ＃[删除一个rpm包]--erase
常用参数：
Install/Upgrade/Erase options:
-i, --install                     install package(s)
-v, --verbose                     provide more detailed output
-h, --hash                        print hash marks as package installs (good with -v)
-e, --erase                       erase (uninstall) package
-U, --upgrade=<packagefile>+      upgrade package(s)
－-replacepkge                    无论软件包是否已被安装，都强行安装软件包
--test                            安装测试，并不实际安装
--nodeps                          忽略软件包的依赖关系强行安装
--force                           忽略软件包及文件的冲突
Query options (with -q or --query):
-a, --all                         query/verify all packages
-p, --package                     query/verify a package file
-l, --list                        list files in package
-d, --docfiles                    list all documentation files
-f, --file                        query/verify package(s) owning file
```


总结：
一般安装软件其实需要rpm包就可以了  
但是rpm依赖太多,依赖又需要个其它各仓库去下载。
YUM就是来解决这个问题  
`yum search nginx  `  
`yum install nginx `