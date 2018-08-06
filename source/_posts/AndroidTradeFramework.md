---
title: Android 金融类项目模块化架构
date: 2018-01-16 10:08:46
tags:
- Android
- 金融
---


# 一、前言

在以往的开发中，我们通常会使用MVC的模式进行开发，这样导致了Activity处理的逻辑非常的复杂，而且耦合度非常高，代码结构混乱、层次不清，各业务技术方案不统一，冗余代码充斥项目的各个角落；甚至连基本的包结构也是胡乱不堪，项目架构更是无从谈起。大家只不过是不停地往上堆砌代码添加新功能罢了。

其中业务层是一种非标准的 MVC 架构，Activity 和 Fragment 承担了 View 和 Controller 的职责：


![](//upload-images.jianshu.io/upload_images/2706530-0fdb81de65035098.png)
<div class="image-caption">传统的MVC模式</div>

为了适应项目快速开发以及项目中代码的复用，解决项目中的耦合度。我们不断引入了 Retrofit、UniversalImageLoader、OkHttp、ButterKnife 等一系列成熟的开源库，同时我们也开发了自己的 UI 组件库 UIComponent、基础工具库 CommonUtils、基于第三方地图封装的 MapSDK、即时聊天模块 ChatLibrary 等等。这样就由基础组件层、业务组件层和业务层组成的三层架构。如下图：

![](//upload-images.jianshu.io/upload_images/2706530-2dd70709f5031bc3.png)
<div class="image-caption">MVP模式</div>

前面这种分层的架构本身是没太大问题的，即使到了现在我们的业务项目也已然是基于这种分层的架构来构建的，只不过在不断的迭代中我们做了些许调整（分层架构后面在介绍组件化和模块化的时候会详细介绍）。但是随着业务的不断迭代,我们慢慢发现业务层这种非标准的 MVC 架构带来了种种影响团队开发效率的问题：

Activity 和 Fragment 越来越多的同时承担了 Controller 和 View 的职责，导致他们变得及其臃肿且难以维护；

由于 Controller 和 View 的揉合，导致单元测试起来很困难；

回调嵌套太多，面对负责业务时的代码逻辑不清晰，难以理解且不利于后期维护；

各层次模块之间职责不清晰等等

# 二、项目整体架构

**整体项目架构如下图：**

![](//upload-images.jianshu.io/upload_images/2706530-2284d27eb4438440.png)
<div class="image-caption">整体项目架构</div>

# 三、项目分层说明

## 整体项目分层：

**View Layer**: 只负责 UI 的绘制呈现，包含 Fragment 和一些自定义的 UI 组件，View 层需要实现 ViewInterface 接口。Activity 在项目中不再负责 View 的职责，仅仅是一个全局的控制者，负责创建 View 和 Presenter 的实例；

**Model Layer**: 负责检索、存储、操作数据，包括来自网络、数据库、磁盘文件和 SharedPreferences 的数据；

**Presenter Layer**: 作为 View Layer 和 Module Layer 的之间的纽带，它从 Model 层中获取数据，然后调用 View 的接口去控制 View；

**Contract**: 我们参照 Google 的 Demo 加入契约类 Contract 来统一管理 View 和 Presenter 的接口，使得某一功能模块的接口能更加直观的呈现出来，这样做是有利于后期维护的。

### 整体项目架构分为3层：

- 模块层
- 业务逻辑层
- 基础组件层

为什么要这么分？首先从整体模式上看跟微盘的架构大同小异，很多模块都是相同的，可能UI展现跟接口数据不大一样，业务逻辑是差不多的，但是目前很难再从以前的微盘中直接重用。所以分成单独的模块。

第一利于团队多模块开发，第二利于开发速度，只针对单个模块进行编译，编译速度提升了。第三每个模块都可以单独成为APK运行，方便代码调试。第四就是易用性和重用性高。

## 模块层

对整个APP进行功能拆分，单独成立模块，每一个模块都独立依赖基础组件和业务组件。我们可以把 Basic Component Layer 和 Business Component Layer 放在一起看做是一层SDK，新的业务或者项目只需要依赖 SDK 就好。甚至我们可以做得更极致一些，开发一套自己的组件管理平台，业务方可以根据自己的需求选择自己需要的组件，定制业务专属的SDK。业务端和SDK 的关系如下图所示：

![](//upload-images.jianshu.io/upload_images/2706530-c9b9eb047dbbb9b6.jpg)
<div class="image-caption">业务端和SDK 的关系</div>

## 业务逻辑层

封装了与模块层的数据交和UI回调，实际上就相当于Presenter的职责。调用接口以及数据处理都在这一层里面做，最终把结果回调给界面。

另外的职责就是封装常用的公共模块如数据库操作，缓存操作，HTTP请求等。

这一层可以使用RxJava，可以很好的解决嵌套回调的问题。[RxJava系列的文章可以参考这里](https://link.jianshu.com?t=https://zhuanlan.zhihu.com/p/20687178)

各 Layer 间严禁反向依赖：每个层要进行依赖，先画好层于层直接的调用关系，禁止相互依赖。有相互依赖的把公共部分单独拆分。

## 基础组件层

这一层比较好理解，封装基础组件，比如模块化需要用到的Router来连接，并且可以管理Activity的生命周期。还有一些基础UIWidget库，比如股票的图表。

## 依赖管理

项目间的依赖通过私有maven库进行管理，特别是sdk跟presenter这一层，强制把UI跟逻辑分离。使用maven的一个弊端就是需要频繁的上传跟重新build。前期可以在项目用**complie project(':sdk')** 来依赖。

## API 接口层

API接口作为核心的一层，每个模块都需要调用该层，我们采用分功能来设计接口，并提供统一的接口工厂来获取接口的实例。UML图如下：

![](//upload-images.jianshu.io/upload_images/2706530-bb187404a13c6fb0.png)
<div class="image-caption"></div></div>

接口层编码的时候要注意：

**全站使用HTTPS证书**

如何避免回调Listener的内存泄漏？（原因：回调都是通过onSuccess()方法去处理，很容易引用到Context，而导致Http线程没办法退出）

封装成基础类去发请求，可以控制token失效重新发请求的过程。

采用Retrofit2 +okhttp3

**接口缓存策略（参考Volley的缓存）：**

如果缓存中存在，先从缓存读取。

每一个请求都有缓存时间，缓存过期或者失效后重新获取。

可以配置每个请求启用或者禁用缓存，比如一些增删改查操作就不需要用到缓存。列表的形式一般要缓存。

缓存默认是关闭的，根据需要来给请求缓存。

## Model

一般情况下我们的实体层（entity、bean、model）这些都是跟sdk处于一层的，为了避免每个模块为了使用实体层而引用sdk，所以要把这个实体层单独一层出来，避免相互之间有反向依赖的可能性。

除了常用的实体层之外，Model层还具有负责检索、存储、操作数据，包括来自网络、数据库、磁盘文件和 SharedPreferences 的数据的功能，只是都归根到Model这一块来。实际上他们都是单独开来的。

## UIWidget

View划分成若干小模块，不单单可以使用当前项目，更为了以后方便集成到其他项目当中去。

![](//upload-images.jianshu.io/upload_images/2706530-e74d1c82bb6cf15b.png)
<div class="image-caption">UIWidget</div>

## Router 路由管理

组件化和模块化使得程序更加灵活，为了避免在app对各个模块以及组件的依赖当组件发生改变的时候代码修改很大。所以由RouterManger去统一管理各个模块组件之间的跳转。

实现方式可以采用[ARouter](https://link.jianshu.com?t=https://github.com/alibaba/ARouter)，支持Url方式的跳转。

![](//upload-images.jianshu.io/upload_images/2706530-b5bee6a64f546df2.png)
<div class="image-caption">APP路由管理</div>


# 四、项目安全说明

## 接口安全

接口使用HTTPS加密证书进行传输，并进行用户鉴权，用户鉴权方面则打算采用Token方式。用户登录之后分配一个accessToken和一个refreshToken，accessToken用于发起用户请求，refreshToken用于更新accessToken。accessToken会设置有效期，可以设为24小时。而用户退出登录之后，accessToken和refreshToken都将作废。重新登录之后会分配新的accessToken和refreshToken。

然后，我还打算在App层级分配AppKey和AppSecret，Android和iOS分别分配一对。每次向服务端发送请求时，AppKey都必须带上，服务端会对相应的AppKey进行校验。而AppSecret则需要安全保存在客户端，也不能在网络上进行传输，防止泄露。AppSecret只用于加密一些安全性级别较高的数据，以及为URL生成签名。URL签名算法步骤如下：

将所有参数按参数名进行升序排序；

将排序后的参数名和值拼接成字符串stringParams，格式：key1value1key2value2...；

在上一步的字符串前面拼接上请求URI的Endpoint，字符串后面拼接上AppSecret，即：stringURI + stringParams + AppSecret；

使用AppSecret为密钥，对上一步的结果字符串使用HMAC算法计算MAC值，这个MAC值就是签名。

鉴权流程如下：

![](//upload-images.jianshu.io/upload_images/2706530-97716f435c4f3e76.png)
<div class="image-caption">接口鉴权流程</div>

**最后总结设计为：**

接口采用HTTPS和签名证书进行传输

接口参数通过排序 &amp; URL &amp; AppSecret 最终通过HMAC加密生成sign参数

利用底层JNI的so库提供接口进行加密，保证密钥的安全性

敏感字段so加密传输，比如设计密码、银行卡号、身份证号码等有关用户安全信息的字段

**数据安全利用底层JNI实现，打包成so库提供JAVA接口调用。把公钥信息、加密算法放到so库里面防止反编译信息泄漏。**

**APK安全**

android的apk文件实际上是压缩文件，很容易被反编译工具进行反编译，像微信这些apk都能被反编译。为了加强APK的安全性，设计如下：

发布时对代码进行混淆编译

第三方APK文件加固

第三方加固网站：

[爱加密](https://link.jianshu.com?t=http://www.ijiami.cn/)

[360加固助手](https://link.jianshu.com?t=http://jiagu.360.cn/)

[梆梆安全](https://link.jianshu.com?t=https://www.bangcle.com/)

# 五、模块说明

**行情模块（Quotation Module）**

行情使用**WSS**安全传输。

行情以后台服务的形式，在Application onCreate() 的时候就开始建立WebSocket连接。没有订阅行情的时候关闭行情连接，当订阅的时候再次开启行情连接。目的为了节省流量以及手机电量。

因为行情基本上每个页面都会用到，所以采用发布订阅的观察者模式进行设计，实现原理：

每个Presenter都可以注册自己需要的行情（Quotation Filter），当WebSocket 接收到对应的Quotation Filter的时候调用EventBus中间件发送消息。最终会回调到该类的定义的事件方法中去。

把数据处理好前端需要展示的数据实体返回。如行情的状态是上涨还是下跌的状态。

统一使用QuotationManager来管理行情的连接、订阅；抽象成QuotationAction 来管理注册不同的行消息，解析返回。

**行情服务接口方法：**

start() 启动服务

stop() 停止服务

getStatus() 获取当前状态

register(object handler, QuotationAction action) 订阅行情

unregister(); 反注册行情

![](//upload-images.jianshu.io/upload_images/2706530-4a9bf1897132ebe6.png)
<div class="image-caption">行情模块</div>

## 用户模块（User Module）

依赖项：API

![](//upload-images.jianshu.io/upload_images/2706530-e9c2a30d595e5f84.png)
<div class="image-caption">用户模块</div>

说明：

User Manager ：用户管理，**管理当前用户**的登录状态、用户信息、用户操作（退出登录，状态维持，Token刷新）

User Module：用户模块，跟用户相关的各个功能的业务处理、数据处理

## 交易模块（Trade Module）

依赖项：API、行情模块

其他模块需要调用交易模块都是通过路由跳转的方式调用不会直接调用到内部方法里面，所以交易模块重点还是调用API进行数据处理：

![](//upload-images.jianshu.io/upload_images/2706530-42e3b8b5ecf96445.png)
<div class="image-caption">交易模块</div>

本文参考链接：

[安居客 Android 项目架构演进](https://link.jianshu.com?t=http://mp.weixin.qq.com/s?__biz=MzA4NTQwNDcyMA==&amp;mid=2650662653&amp;idx=1&amp;sn=e15a36e4460eb3d1890d92aa921c0962&amp;chksm=87d13ba2b0a6b2b43c24abfa56c6fc78fcae2746cb4812cc87a8e8495e0231bde6bdbfe8957b&amp;mpshare=1&amp;scene=23&amp;srcid=0401P1BfNuztcl0iX8HY6x0D#rd)

[Android组件化项目详细实施方案](https://link.jianshu.com?t=http://mp.weixin.qq.com/s?__biz=MzI2OTQxMTM4OQ==&amp;mid=2247484720&amp;idx=1&amp;sn=6626e31308d58ec9f2caa770006a3220&amp;chksm=eae1f062dd9679746008678118859a0c732db80ed3413cb651f92753f63910d84ebca8a4793f&amp;mpshare=1&amp;scene=23&amp;srcid=0401fX5isQMtiX6nRAPaKDB8#rd)

[App项目实战之路(二):API篇](https://link.jianshu.com?t=http://keeganlee.me/post/practice/20160812)

[APK 的自我保护](https://link.jianshu.com?t=http://bbs.pediy.com/thread-183116.htm)

[如果对MVP模式还不够了解的可以参考google官方例子](https://link.jianshu.com?t=https://github.com/googlesamples/android-architecture)