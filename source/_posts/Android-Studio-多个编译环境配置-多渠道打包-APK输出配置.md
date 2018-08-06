---
title: Android Studio 多个编译环境配置 多渠道打包 APK输出配置
date: 2017-06-07 11:23:35
tags:
- Android
---
看完这篇你学到什么：

- 熟悉gradle的构建配置
- 熟悉代码构建环境的目录结构，你知道的不仅仅是只有src/main
- 开发、生成环境等等环境可以任意切换打包
- 多渠道打包
- APK输出文件配置

## 需求

一般我们开发的环境分为：debug 和 release，但是你想再分内测1环境、内测2环境等等怎么办呢？

这就需要依赖强大的gradle 来配置了。

相关的配置也可以参考[谷歌官方文档](https://developer.android.google.cn/studio/build/build-variants.html?hl=zh-cn)。
 
## 配置构建类型 `buildTypes`

您可以在模块级 `build.gradle` 文件的 `android {}` 代码块内部创建和配置构建类型。当您创建新模块时，Android Studio 会自动为您创建调试和发布这两种构建类型。尽管调试构建类型不会出现在构建配置文件中，Android Studio 会将其配置为 `debuggable true`。这样，您可以在安全的 Android 设备上调试应用并使用通用调试密钥库配置 APK 签署。

如果您希望添加或更改特定设置，您可以将调试构建类型添加到您的配置中。以下示例为调试构建类型指定了 `applicationIdSuffix`，并配置了一个使用调试构建类型中的设置进行初始化的`jnidebug`构建类型。

`applicationIdSuffix:` 字段表示，在不改变你默认的程序ID（包名）的情况下，为其添加后缀。比如你的包名是`com.rae.app`，但你想区分测试包和正式包的情况，这个时候将`applicationIdSuffix`设置为`.debug`，那么你的应用程序对应的包名就变成了`com.rae.app.debug`

```groovy
android {
    ...
    defaultConfig {...}
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }

        debug {
            applicationIdSuffix ".debug"
        }

        /**
         * The 'initWith' property allows you to copy configurations from other build types,
         * so you don't have to configure one from the beginning. You can then configure
         * just the settings you want to change. The following line initializes
         * 'jnidebug' using the debug build type, and changes only the
         * applicationIdSuffix and versionNameSuffix settings.
         */

        jnidebug {

            // This copies the debuggable attribute and debug signing configurations.
            initWith debug

            applicationIdSuffix ".jnidebug"
            jniDebuggable true
        }
    }
}
```

## 构建源集

我们都知道，源代码是放在`src/main` 文件夹下的，但是由于配置了不同的构建类型像想要区分不同的源文件怎么办呢？这个时候就可以在`src`对应你的`buildTypes`来建立文件夹了 ，更多参考[谷歌源集](https://developer.android.google.cn/studio/build/index.html?hl=zh-cn#sourcesets)。

Android Studio 按逻辑关系将每个模块的源代码和资源分组为源集。模块的 main/ 源集包括其所有构建变体共用的代码和资源。其他源集目录为可选项，在您配置新的构建变体时，Android Studio 不会自动为您创建这些目录。不过，创建类似于 main/ 的源集有助于让 Gradle 只应在构建特定应用版本时使用的文件和资源井然有序：

`productFlavor` 表示渠道包，可以看下面的`多渠道打包`

>`src/main/`

此源集包括所有构建变体共用的代码和资源。
    
>src/`<buildType>`/

创建此源集可加入特定构建类型专用的代码和资源。示例：`src/jnidebug`

>src/`<productFlavor>`/

创建此源集可加入特定产品风味专用的代码和资源。比如百度渠道包：`src/baidu`

>src/`<productFlavorBuildType>`/

创建此源集可加入特定构建变体专用的代码和资源。
例如，要生成应用的“完全调试”版本，构建系统需要合并来自以下源集的代码、设置和资源。比如：百度的开发环境包：`src/baiduDebug`



## 构建类型依赖配置
很多时候我们会把sdk或者api接口单独做成一个库，一般会有生产环境和测试环境之分，但在依赖的时候往往我们会像这样去引用：` compile project(':sdk')`，这样依赖的环境就是`release`，在开发调试的时候测试环境的时候就不行了。我们得换另外一种方式：

>`<buildType>`Compile project()

这样会根据不同的构建类型去依赖不同的包，比如我们测试环境的依赖包:`debugCompile project(':sdk')`，再比如上面的`jnidebug`：`jnidebugCompile project(':sdk')` 

那么问题来了，我当前的构建类型怎么对应到其他的`module`去呢？比如你的app要依赖sdk module 的debug 环境，那么你可以这么做：

`configuration`：目标`module`的`<buildType>`，比如你sdk 中`<buildType>`的`debug`构建类型

`debugCompile project(path: ':sdk', configuration: 'debug')`

>**综合示例：**

1、先看app这边的`build.gradle`配置：
```java
apply plugin: 'com.android.application'

android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            applicationIdSuffix '.debug'
            minifyEnabled false
        }
        
        // 自定义的构建类型，名字随便取，一定要有意义
        raedebug {
            initWith debug
            applicationIdSuffix '.raedebug'
        }
    }
}

dependencies {
    // 生成环境依赖
    releaseCompile project(path: ':sdk', configuration: 'release')
    // 测试环境依赖
    debugCompile project(path: ':sdk', configuration: 'debug')
    // 自定义构建类型依赖
    raedebugCompile project(path: ':sdk', configuration: 'uutest')
}

```
2、sdk module的`build.gradle`配置：


```java
apply plugin: 'com.android.library'

android {
       buildTypes {
        debug {
            debuggable true
            minifyEnabled false
        }
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        
        // 注意这里，跟第一点的 raedebugCompile project的configuration要匹配。
        uutest {
            initWith debug
        }
    }
}

```


## 多渠道打包 `productFlavors`

先看看`build.gradle`配置你就懂了

```java
android{

    // 渠道包定义，默认定义的名称就是渠道名称
    productFlavors {
 
        dev {} // 测试
        baidu {}        // 百度手机助手
        yinyongbao {}   // 应用宝
        m360 {}         // 360手机助手
        pp {}           // PP助手
        anzhi{}         // 安智市场
        xiaomi {}       // 小米商店
        letv {}         // 乐视商店
        huawei {}       // 华为商店
        lenovomm {}     // 联想乐商店
        other {}        // 其他市场
        official{}      // 官方版本
 
    }
 
    // 批量渠道包值替换
    productFlavors.all { flavor ->
        // 友盟、极光推送渠道包， UMENG_CHANNEL 是根据你AndroidManifest.xml来配置的，请看下面。
        flavor.manifestPlaceholders = [UMENG_CHANNEL: name, JPUSH_CHANNEL: name]
    }
}
```
`AndroidManifest.xml` 配置：
```xml

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.rae.demo">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
       
       <!--变量采用${变量名}这样来替换，不仅限与<meta-data /> 标签，任何你想替换的都行。-->
         <meta-data
            android:name="UMENG_APPKEY"
            android:value="${UMENG_APPKEY}"/>
        
        <meta-data
            android:name="UMENG_CHANNEL"
            android:value="${UMENG_CHANNEL}"/>
        
        <!--${变量随变换}-->   
        <activity
            android:name=".DemoActivity"
            android:label="${变量随变换}"/>
            
    </application>

</manifest>
```


sync gradle之后看看gradle projects 面板列表就多出了好到渠道的任务了，Build Variants 面板也相对应多了这些构建类型。



## APK输出配置

在结合到多渠道打包后，运营的那边希望我们给的渠道包是这种格式的`app-{版本号}-{渠道名称}.apk`，那我们来看看怎么来满足这个多渠道打包输出apk文件名修改的。

```java
android{

    // 输出文件配置
   applicationVariants.all { variant ->
        variant.outputs.each { output ->
            def outputFile = output.outputFile
            if (outputFile != null && outputFile.name.endsWith('.apk')) {
                def dirName = outputFile.parent // 输出文件夹所在的位置
            
                // 文件名修改
                def fileName = "app-${output.processResources.variantName}-${defaultConfig.versionName}-${variant.flavorName}.apk"
                
                // 比如不想这么麻烦，直接在后面加上版本号也行：
                // def fileName = outputFile.name.replace(".apk", "-${defaultConfig.versionName}.apk")
                
                
                output.outputFile = new File(dirName, fileName)
            }
        }
    }
}
```


