---
title: 把自己的Library库上传到Maven中去
date: 2019-05-09 09:22:54
tags: View
---

一、在你要上传到maven的库中配置上传参数

![image.png](https://upload-images.jianshu.io/upload_images/2706530-84113f849b2e8466.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!-- more -->

```java

uploadArchives {
    apply plugin: 'maven'
    // 读取本地配置文件
    Properties properties = new Properties()
    properties.load(project.rootProject.file('local.properties').newInputStream())
    def userName = properties.getProperty('maven.user')
    def password = properties.getProperty('maven.password')
    def mavenUrl = properties.getProperty('maven.url')

    repositories.mavenDeployer {
        repository(url: mavenUrl) {
            authentication(userName: userName, password: password)
        }
        pom.project {
            // 注意：这里要修改一下你的库。比如：com.baidu:lib:1.0.0
            groupId 'com.github.raedev'
            artifactId 'session'
            version '1.0.0'
            packaging 'aar'
        }
    }

    task androidSourcesJar(type: Jar) {
        classifier = 'sources'
        from android.sourceSets.main.java.sourceFiles
    }
    artifacts {
        archives androidSourcesJar
    }
}
```

二、打开`local.properties` 配置好maven 参数

```java
maven.url=http://maven.baidu.com/repository/maven-baidu/
maven.user=填写你的Maven账号
maven.password=填写你的Maven密码
```

三、运行`uploadArchives`任务

![运行uploadArchives任务](https://upload-images.jianshu.io/upload_images/2706530-e5e54cb976f8280a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/360)



恭喜！前往你的maven库中去看看是否上传成功了。
