---
title: Android library 上传到Nexus 的Maven私库中
date: 2017-06-07 11:24:07
tags:
- Android
---
上传自己的库到maven中，我们可以借助Nexues Oss 来管理我们的私库，首先得安装好nexues oss，安装过程就不在这里描述了，详情可以查看

-  [Nexues安装过程](http://www.jianshu.com/p/37bebc28ab87)

安装好之后地址默认地址是：[http://localhost:8081](http://localhost:8081)，帐号密码为：admin/admin123，打开之后看到这个界面就成功安装了：

![](http://www.raeblog.com/blog/wp-content/uploads/2017/04/QQ20170412131016.png)

点击左边的`components `能看到所有的仓库，本地库的类型为hosted：

![](http://www.raeblog.com/blog/wp-content/uploads/2017/04/QQ20170412131016.png)

如果你要新建自己的maven 库可以登录后在设置中创建，点击设置的图表 -> `Repositories` ->`Create repository` -> `maven2 (hosted)` ->来到下图创建maven库：

![](http://www.raeblog.com/blog/wp-content/uploads/2017/04/QQ20170412131651.png)

好啦，到这里nexus 就配置成功了，找到你创建的库，复制URL，这个URL就是你的私库地址，在根目录下的`build.gradle`中就可以添加啦

```java
allprojects {
    repositories {
		    // 替换你的url
        maven { url 'http://localhost:8081/repository/maven-public/' }
        jcenter()
    }
}

```

![](http://www.raeblog.com/blog/wp-content/uploads/2017/04/QQ20170412132314.png)





打开要上传库的`build.gradle`在里面配置下面信息：

```java
apply plugin: 'com.android.library'

android {
   // ... 此处省略
}

uploadArchives {
    apply plugin: 'maven'
    repositories.mavenDeployer {
				// url 替换你的maven库地址
        repository(url: 'http://localhost:8081/repository/maven-rae/') {
            authentication(userName: 'nexus登录帐号', password: 'nexues登录密码')
        }
  
	     //  下面配置好之后在gradle就可以以这种形式引用啦：compile 'com.rae.core:ui:1.0.0'			 
        pom.project {
            version '1.0.0' // 版本号
            artifactId 'ui' // 库名称
            groupId 'com.rae.core' // 组织包名
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


重新`sync project`之后，在你的gradle面板中就可以看到upload任务了，双击它就可以上传了：

![](http://www.raeblog.com/blog/wp-content/uploads/2017/04/QQ20170412130533.png)


最后到你的maven中去看看有没有上传成功吧！

[END]

