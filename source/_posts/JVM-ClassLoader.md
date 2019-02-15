---
title: JVM类加载器使用
date: 2018-12-14 19:58:15
tags: jvm
---
>Java类加载器算是一个老生常谈的问题

什么是类加载器？
这就是类加载器
把class文件的byte[] 加载成Class对象，加载到JVM里面
<!-- more -->
```
class MyClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        try {
            byte[] classBytes = Files.readAllBytes(Paths.get(ClassLoaderDevelop.class.getResource("/A.class").toURI()));
            return defineClass(name, classBytes, 0, classBytes.length);
        } catch (IOException | URISyntaxException e) {
            e.printStackTrace();
            return null;
        }

    }
}
```

为什么要弄这么都类加载器，还有层级关系，而不一股脑都加载到JVM里呢
1.定义一个层级就有优先级了，可以先加载啥，后加载啥，防止例如java.lang.String 根加载器的类，被第三方jar包冲掉
2.按需加载，需要的再加载，体现了java语言的动态特点


梳理结构

#### BootstrapClassLoader  
##### 加载路径：%JAVA_HOME%\lib
`System.out.println(System.getProperty("sun.boot.class.path"));`  
底层C++编写，不属于java类,根本没有对应的java类名
根类加载器，也叫引导类加载器、启动类加载器。  
主要加载rt.jar charsets.jar  
就是jdk自带的类

#### sun.misc.Launcher$ExtClassLoader
##### 加载路径：%JAVA_HOME%\lib\ext
`System.out.println(System.getProperty("java.ext.dirs"));`  
扩展类加载器

#### sun.misc.Launcher$AppClassLoader
##### 加载路径:%CLASS_PATH%
`System.out.println(System.getProperty("java.class.path"));`  
应用类加载器

### 双亲委派
简单说，ClassLoader是一个抽象类  
1.提供了一个`loadClass()`方法  
2.有一个`parent`字段  
作用：就是加载类的时候，先自下往上搜索一边。
都没有，再用自身这个类加载器去加载。

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        //进行类加载操作时首先要加锁，避免并发加载
        synchronized (getClassLoadingLock(name)) {
            //首先判断指定类是否已经被加载过
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        //如果当前类没有被加载且父类加载器不为null，则请求父类加载器进行加载操作
                        c = parent.loadClass(name, false);
                    } else {
                       //如果当前类没有被加载且父类加载器为null，则请求根类加载器进行加载操作
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                }

                if (c == null) {
                    long t1 = System.nanoTime();
                    //如果父类加载器加载失败，则由当前类加载器进行加载，
                    c = findClass(name);

                    //进行一些统计操作
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            //初始化该类
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```
1.如果要实现自己的类加载器且不破坏双亲委派模型，只需要继承ClassLoader类并重写findClass方法。  
2.如果要实现自己的类加载器且破坏双亲委派模型，则需要继承ClassLoader类并重写loadClass，findClass方法。  

#### 上下文类加载器
```java
//加载驱动程序
Class.forName("com.mysql.jdbc.Driver");
//连接数据库
Connection conn = DriverManager.getConnection(url, user, password);
```
这里面第1行按前面里理解 肯定是应用加载器
第二行`DriverManager`是数据java基础类库 由跟加载器加载
那么在DriverManager如何获取到应用加载器呢？
```java
Thread.currentThread().getContextClassLoader();
```
