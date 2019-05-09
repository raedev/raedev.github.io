---
title: Android Studio APK输出文件配置
date: 2017-06-07 11:22:03
tags:
- Android
---
```groovy
android{
  applicationVariants.all { variant ->
        variant.outputs.each { output ->
            def outputFile = output.outputFile
            if (outputFile != null && outputFile.name.endsWith('.apk')) {
                def dirName = APP_OUTPUT_DIR + "v${defaultConfig.versionName}"
                def fileName = outputFile.name.replace(".apk", "-${defaultConfig.versionName}.apk")
                output.outputFile = new File(dirName, fileName)
            }
        }
    }
```

<!-- more -->

`APP_OUTPUT_DIR` 在`gradle.properties`,如：

```ini
#小盘beta版 输出路径
APP_OUTPUT_DIR = F\:\\100bei\\xiaopan\\dev\\
```
