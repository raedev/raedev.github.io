---
title: Android Studio 2.3 启动APP报错Not found; no service started.
date: 2017-06-07 10:54:18
tags:
- Android
---
升级android studio 2.3 之后，运行APP的时候报错 `Not found; no service started.`，具体的报错信息如下：

```code
Error while executing: am startservice com.yourpackagename/com.android.tools.fd.runtime.InstantRunService

Starting service: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER]cmp=com.yourpackagename/com.android.tools.fd.runtime.InstantRunService }

Error: Not found; no service started.
```

<!-- more -->

用模拟不会出现这个问题，真机调试一般都会出现这个问题。

**解决方案：**

-  `【推荐做法】`只要`允许应用自启动`就能解决了，一般很多手机一安装都不允许自动启动的。

- 普遍的做法是直接关闭`InstantRun`这个功能，但是不推荐这种做法，会导致每次调试都会重新安装APK。

前往 `File` -> `Settings` ->`Build,Exceution,Deployment` -> `InstantRun`  把`Enable Instant Run ····` 的勾去掉就行了

![QQ截图20170331093117](http://www.raeblog.com/blog/wp-content/uploads/2017/03/20170331093117.png)




参考文章：[http://blog.csdn.net/u013130865/article/details/61616587](http://blog.csdn.net/u013130865/article/details/61616587)