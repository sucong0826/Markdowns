# \#Google# Android Instant App的概览

Android InstantApp是当你去启动一个URL，作为响应，会运行一个原生的Android应用，而不用将这个应用安装在手机上。Instant App可以使用很多Android的API，同时，在Android Studio上可以构建Instant Apps。

Instant App是如何工作的呢？当Google Play接收到一个URL的请求之后，会根据这个URL去匹配一个Instant App，Google Play会必要的代码文件发送到设备上。

> Android Instant App只运行在Android 5.0以上或者更高的版本上。（同时还要支持Google Play才行）

## Apps是某种功能（特性）
---
对于开发者和用户来说，Android Instant App是一种全新的且独有的方式。在介绍Instant App的核心概念之前，这里有些基础的术语来帮助大家理解。

在一个非常基础的等级上，Apps至少需要具有一种特性或者内容：比如在地图上找到一个点，发送邮件，或者阅读当日新闻。还有很多的App会提供更多的功能。

例如，一个地图软件会允许用户去查找附近的餐馆或者通过邮件发送一个地点链接。这些类似的功能都是这款App中的一些功能。

在使用Instant App的时候，用户可以使用一个应用中的某一个功能，而无需将整个App安装到手机上。当用户从Instant App上请求一个功能来使用的时候，他们只会接收到这一部分的代码来运行程序，不多也不少。当用户使用完之后，系统会处理该功能的代码。

让我们回到刚才的例子中，这个地图的Instant App可以将每一个功能点单独暴露出来作为一个单独的、离散的个体。用户可以下载下来并且只使用这个地点查找的功能，或者使用地点分享的功能。一旦用户转换到其它的App去使用，系统可以安全地移除掉这部分更能的代码。

每一个Instant App都应该需要至少一个Activity来充当这个功能的入口点。这个入口点活动托管了对应的功能点的UI和用户流程。当用户在他们的设备上启动了这个功能的时候，首先看到的就是这个入口功能点Activity。一个功能可以有多个Activity充当入口点，但是对于功能来说它只需要一个。

## 功能模块与功能APKs
---
为了提供这些模块的下载，你需要将你的App打散成一个个小的模块，然后将它们重构为`feature modules`。

当你在构建一个Instant App的程序时，构建的结果是许多个Instant App的APK，它包括一个或者多个功能点的APK。每一个功能的APK都是从工程中的某一个模块构建的，下载过后，它们都可以作为一个单独的Instant App来运行。

每一个Instant App都有且仅有一个`base feature APK`（基础功能APK）。如果你的Instant App只包含一个功能的话，那么你只需要这一个基础功能的APK。其他功能的APK是可选的。如果你的Instant App有多个功能的话，那么这个基础功能的APK会包含共享的资源和代码文件，其它的功能都会依赖它。无论其它的功能是否会被用户请求到，基础功能的APK是总要被下载的。

除了基础功能APK之外，你也可以拥有其他功能的APKs。附件功能的APK可以包含App中代码的一部分来对应一个功能。这个功能的APK包含了当前功能的入口点Activity和此功能需要的资源。

如果用户从一个Instant App中请求一个功能，他们会得到两个功能的APK：对应功能的APK和基础功能APK。如果同一个用户从同一个Instant App中请求了另一个功能，他们接收到只会是这个功能的APK，因为基础功能APK已经下载好了。当然，如果你的Instant App只有一个功能的话，用户只会接受到这一个基础功能的APK。

下面的这个图阐述了Instant App APK和功能APK之间的关键。

![Relationship between Instant App APK and feature APK](https://developer.android.com/topic/instant-apps/images/aia_features_diagram.png)

## 从Google Play请求一些功能
---
要从Google Play上下载一个Instant App的某一个功能，用户需要点击一个链接去下载。当Google Play发现一个满足此链接的Instant App之后，Google Play将会发送这个功能的APK到当前的设备上，然后Android系统会启动这个功能。如果Google Play没有找到这个Instant App，它会提醒Android系统，之后系统会广播一个`intent`也由系统来处理这个URL。

由于这个原因，**每一个入口点Activity都需要被设置为可寻址的**：它需要去响应一个独一无二的URL。**如果一个Instant App中的功能URL地址是共享一个域的，则每个功能需要对应于该域名内的不同路径**。

我们用之前地图应用做例子，这个应用有三个独立的功能：
- 位置查找
- 附近的餐馆
- 位置分享

每一个功能都对应一组资源且在一个web域名内，如`example.com`。Instant App需要指在当前域名下指定一个不同的域名给每一个功能点。

Feature    | URL adress
--------| ---
Location finder | `http://example.com/finder `
Nearby | `http://example.com/restaurants`
Share Location | `http://example.com/share`

正如前面提到的，一个单一的功能会有多个入口点Activities。例如，一个功能点可能会有两个相关的activities来让用户切换，每个Activity都要自己的URL地址，在多匹配的情况下，需要为这些入口点Activity准备一个清单，清单包括每个Activity的路径和优先级。

例如，假设这个位置定位功能有两个入口点Activities，一个查找页面，一个详情页面。对于详情页面应该和查找页面是相似的，而详情页面需要在URL的最后添加上一个数字的ID。

| Activity |URL address                    |URL path     | Priority|
| :------- |:------------------------------|:----------- | :-------|
| Search   |http://example.com/finder/     |\'/finder/'  | 1       |
| Details  |http://example.com/finder/\<ID>|\'/finder/*' | 100     |

如果Google Play接受到了一个URL的请求 `http://example.com/finder/1234`，这个URL会匹配到这两个Activity。Google Play需要从中选择一个作为作为入口点Activity。因为这个Instant App这个详情页面的**优先级要高于**搜索页面的优先级，那么Google Play会告诉系统指定**详情页面去启动**这个功能。