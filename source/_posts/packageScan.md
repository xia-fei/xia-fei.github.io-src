---
title: java包扫描功能实现分析
date: 2018-12-25 13:50:28
tags: java
---

### 问题分析
输入包名 eg:com.xiafei  
返回包下面所有的类名

先总结再看过程

### 总结 

##### 1.包名换成路径名  
com/xiafei/ 

##### 2.获取系统中绝对路径  
Class.getResource("com/xiafei/")  

##### 3.获取文件夹下面所有class文件名  
File.listFiles()  
<!-- more -->
##### 4.获取文件名称  


##### Spring源码过程
```java
 public static void main(String[] args) throws IOException {
        PathMatchingResourcePatternResolver pathMatchingResourcePatternResolver=new PathMatchingResourcePatternResolver();
        Resource[]  resources=pathMatchingResourcePatternResolver.getResources("classpath*:com/wing/core/mock/generate/handler/*.class");
        for (int i = 0; i < resources.length; i++) {
            Resource resource = resources[i];
            System.out.println(resource.getURI());
        }
    }


    protected Resource[] findPathMatchingResources(String locationPattern) throws IOException {
        //文件夹路径
        String rootDirPath = determineRootDir(locationPattern);
        //匹配文件路径
        String subPattern = locationPattern.substring(rootDirPath.length());
        Resource[] rootDirResources = findAllClassPathResources(rootDirPath);
        Set<Resource> result = new LinkedHashSet<Resource>(16);
        for (Resource rootDirResource : rootDirResources) {
                result.addAll(doFindPathMatchingFileResources(rootDirResource, subPattern));
        return result.toArray(new Resource[result.size()]);
    }

    //根据文件夹名字找到Resouce也就是文件的绝对位置
    protected Resource[] findAllClassPathResources(String location) throws IOException {
        String path = location;
        if (path.startsWith("/")) {
            path = path.substring(1);
        }
        Set<Resource> result = doFindAllClassPathResources(path);
        if (logger.isDebugEnabled()) {
            logger.debug("Resolved classpath location [" + location + "] to resources " + result);
        }
        return result.toArray(new Resource[result.size()]);
    }


    //根据文件夹名字找到Resouce也就是文件的绝对位置
    protected Set<Resource> doFindAllClassPathResources(String path) throws IOException {
        Set<Resource> result = new LinkedHashSet<Resource>(16);
        ClassLoader cl = getClassLoader();
        Enumeration<URL> resourceUrls = (cl != null ? cl.getResources(path) : ClassLoader.getSystemResources(path));
        while (resourceUrls.hasMoreElements()) {
            URL url = resourceUrls.nextElement();
            result.add(convertClassLoaderURL(url));
        }
        return result;
    }


//扫描具体文件
protected Set<Resource> doFindPathMatchingFileResources(Resource rootDirResource, String subPattern)
            throws IOException {

        File rootDir;
        try {
            rootDir = rootDirResource.getFile().getAbsoluteFile();
        }
        catch (IOException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Cannot search for matching files underneath " + rootDirResource +
                        " because it does not correspond to a directory in the file system", ex);
            }
            return Collections.emptySet();
        }
        return doFindMatchingFileSystemResources(rootDir, subPattern);
    }

//扫描
    protected Set<Resource> doFindMatchingFileSystemResources(File rootDir, String subPattern) throws IOException {
        if (logger.isDebugEnabled()) {
            logger.debug("Looking for matching resources in directory tree [" + rootDir.getPath() + "]");
        }
        Set<File> matchingFiles = retrieveMatchingFiles(rootDir, subPattern);
        Set<Resource> result = new LinkedHashSet<Resource>(matchingFiles.size());
        for (File file : matchingFiles) {
            result.add(new FileSystemResource(file));
        }
        return result;
    }

    protected Set<File> retrieveMatchingFiles(File rootDir, String pattern) throws IOException {
        fullPattern = fullPattern + StringUtils.replace(pattern, File.separator, "/");
        Set<File> result = new LinkedHashSet<File>(8);
        //真正扫描
        doRetrieveMatchingFiles(fullPattern, rootDir, result);
        return result;
    }

    protected void doRetrieveMatchingFiles(String fullPattern, File dir, Set<File> result) throws IOException {
        
        //获取下面所有子文件
        File[] dirContents = dir.listFiles();
      
        Arrays.sort(dirContents);
        //一个模糊匹配功能
        for (File content : dirContents) {
            String currPath = StringUtils.replace(content.getAbsolutePath(), File.separator, "/");
            if (content.isDirectory() && getPathMatcher().matchStart(fullPattern, currPath + "/")) {
                else {
                    doRetrieveMatchingFiles(fullPattern, content, result);
                }
            }
            if (getPathMatcher().match(fullPattern, currPath)) {
                result.add(content);
            }
        }
    }

```



##### Jar包扫描要复杂点  
关键点 URL目录转Resouce
org.springframework.core.io.support.PathMatchingResourcePatternResolver#doFindPathMatchingJarResources(org.springframework.core.io.Resource, java.net.URL, java.lang.String)

URL->JarFile->jarFile.entries()->entry.getName()
```java
protected Set<Resource> doFindPathMatchingJarResources(Resource rootDirResource, URL rootDirURL, String subPattern)
            throws IOException {

        // Check deprecated variant for potential overriding first...
        Set<Resource> result = doFindPathMatchingJarResources(rootDirResource, subPattern);
        if (result != null) {
            return result;
        }

        URLConnection con = rootDirURL.openConnection();
        JarFile jarFile;
        String jarFileUrl;
        String rootEntryPath;
        boolean closeJarFile;

        if (con instanceof JarURLConnection) {
           
            JarURLConnection jarCon = (JarURLConnection) con;
            ResourceUtils.useCachesIfNecessary(jarCon);
            jarFile = jarCon.getJarFile();
            jarFileUrl = jarCon.getJarFileURL().toExternalForm();
            JarEntry jarEntry = jarCon.getJarEntry();
            rootEntryPath = (jarEntry != null ? jarEntry.getName() : "");
            closeJarFile = !jarCon.getUseCaches();
        }
        else {
        
            String urlFile = rootDirURL.getFile();
            try {
                int separatorIndex = urlFile.indexOf(ResourceUtils.WAR_URL_SEPARATOR);
                if (separatorIndex == -1) {
                    separatorIndex = urlFile.indexOf(ResourceUtils.JAR_URL_SEPARATOR);
                }
                if (separatorIndex != -1) {
                    jarFileUrl = urlFile.substring(0, separatorIndex);
                    rootEntryPath = urlFile.substring(separatorIndex + 2);  // both separators are 2 chars
                    jarFile = getJarFile(jarFileUrl);
                }
                else {
                    jarFile = new JarFile(urlFile);
                    jarFileUrl = urlFile;
                    rootEntryPath = "";
                }
                closeJarFile = true;
            }
            catch (ZipException ex) {
                if (logger.isDebugEnabled()) {
                    logger.debug("Skipping invalid jar classpath entry [" + urlFile + "]");
                }
                return Collections.emptySet();
            }
        }

        try {
      
            if (!"".equals(rootEntryPath) && !rootEntryPath.endsWith("/")) {
                rootEntryPath = rootEntryPath + "/";
            }
            result = new LinkedHashSet<Resource>(8);
            for (Enumeration<JarEntry> entries = jarFile.entries(); entries.hasMoreElements();) {
                JarEntry entry = entries.nextElement();
                String entryPath = entry.getName();
                if (entryPath.startsWith(rootEntryPath)) {
                    String relativePath = entryPath.substring(rootEntryPath.length());
                    if (getPathMatcher().match(subPattern, relativePath)) {
                        result.add(rootDirResource.createRelative(relativePath));
                    }
                }
            }
            return result;
        }
        finally {
            if (closeJarFile) {
                jarFile.close();
            }
        }
    }
```





