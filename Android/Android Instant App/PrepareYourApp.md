# 为你的App做准备

Instant App最好的用户体验就是帮助用户快速快速地完成一项任务（例如看一段视频或者进行一次购买），那么就可以让你的App开始为Instant App做准备，一般需要完成如下几步工作。对Android应用来说这些步骤是被认为最好的实践。

## 从你的App中移除掉任何你不需要的块

避免使用任何不清楚或无用的权限，没有用的组件，不需要的第三方依赖和库。将它们移除掉之后可以显著的减少APK的体积，提高性能。

## 让Android应用支持链接

Android Instant App使用URLs来完成全部的导航。当一个用户点击一个链接到Instant App的时候，他们会去到你App中一个指定的页面。如果链接失败了或者用户在一个不支持Instant App上点击了这个链接的话，手机中的浏览器会打开且展示给你一个web页面。同样，在一个Instant App内，一个activity不会直接启动另一个Activity，而是通过URL地址来请求到对应的那个Activity。

所有Instant App的可安装的版本都要实现**Android App Links**特性，它是在Android 6.0系统中被引入的。**App Links**所提供的主要机制是**URL连接到你的App中的某一个页面**。我们推荐你的App支持www和非www的域名。

### 处理Android应用的链接

要想让用户跟随设备上的链接进行引导，要记住一点：给用户想看到的东西。作为一个开发者，你可设置Android应用的链接，这样用户带着链接可以指定到这部分内容，通过App选择的对话框可以进行选择。因为Android应用链接是将一个HTTP的URL地址和一个网址关联的，用户即使没有安装你的App也可以直接到达这部分内容。

#### Deep linking （深链接）和Android应用链接

在深入去实现之前，我们先来理解一下两种不同的链接：
- **deep links**（深链接）：它可以带着用户直达你App中指定的这部分内容，在Android中，你可以通过添加`intent filters`和提取`intent`的数据让用户到达正确的Activity。然而，如果用户的手机上安装了其它的应用同样可以处理这个intent，那么用户可能不会直达你的App，例如用户点击了一个URL在一个银行系统应用中点击了一个URL，此时可能会弹出一个对话框让用户选择是使用浏览器还是让银行系统自己打开这个链接。

- **Android App links**（Android应用链接）：在Android 6.0系统（API 23）和更高版本的系统上，系统允许一个App**指定自己**为**默认的给定类型链接的处理器**。如果这个用户不希望这个App作为默认的处理器，他们可以从系统设置中更改这个行为。

Android应用链接提供了如下的便利：

- 安全和指定的：Android应用链接使用HTTP的URL，这些URL是链接到你自己的网站域名的。对于Android应用链接来说一个**要求**是：你要通过我们网站关联的方法之一来验证你的域名的所有权。

- 无缝的用户体验：因为Android应用链接使用一个单独的HTTP的URL，和你网站上的内容相同，这些没有安装的App的用户会登录到你的网站是去查看内容，而不是显示404，或者错误页面。

- Android Instant App的支持：随着Android Instant App的到来，用户可以直接使用你App的部分内容而无需安装它，为了给你的App添加Instant App特性，需要设置Android应用链接且访问[g.co/InstantApps](https://g.co/InstantApps)

