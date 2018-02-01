# Android N中，关于多窗口模式的5个建议

  如果你已经挖掘了一些关于[Android N](https://www.youtube.com/watch?v=CsulIu3UaUM&utm_campaign=adp_series_prepareformultiwindow_032316&utm_source=medium&utm_medium=blog)的新特性，那么你可能会对多窗口支持（[multi-window support](https://developer.android.com/guide/topics/ui/multi-window.html?utm_campaign=adp_series_prepareformultiwindow_032316&utm_source=medium&utm_medium=blog)）感到一些不解。
  
  ![Multi-window support](https://cdn-images-1.medium.com/max/880/0*6ubw3wnJCWvjJYdU.)
  
  当多屏幕被分开之后，两个App就各在边变得可见了。看到它工作了之后这是很振奋人心的，我知道我要过一遍文档，看看有哪些新的具有魔法的API。
  
  事实证明，这里并没有太多新的API。只增加了一些XML属性来定制是否需要支持多窗口，同时Activity增加了一些方法去检查Activity是否在多窗口模式下。所以，魔力在哪里呢？魔力是一直都在啊。
  
  魔力体现在Android的资源系统（resource system）。其中一个非常强大的部分在于资源系统有能力提供可用的资源 - 它可以改变尺寸，布局，图片，菜单等等，这些都是基于不同的限定符（qualifiers）。
  
  **多窗口是通过根据窗口大小调整配置来利用系统资源的**，屏幕尺寸是很明显的一个，但是最小的宽度、高度，方向，也要在变更尺寸的时候更新。
  
  接下来，就让我们看看第一个Tip。
  
  ## Tip1：使用正确的上下文环境（Context）
  
  加载正确的资源需要正确的上下文（context）。如果你使用当前这个Activity上下文去加载布局，获取资源等等，那么这是没问题的。
  
  然而，如果你使用了`Application`的上下文去做和UI相关的事情，你会发现多窗口对于它所加载的资源是不清楚的。除了发现没有使用你的Acivity的主题问题之外，你会发现你可能完全用错了资源。**所以处理UI先关内容最好的Practice就是使用Activity的上下文环境。**
  
  ## Tip2：正确地处理配置（Configuration）变化
  
  有了正确的上下文环境，对于多窗口的尺寸来说你会拿到正确的资源（无论是之前全屏状态还是与其它App的分屏状态），对于重载这些资源的流程是基于你如何处理这些运行时变化（[runtime changes](https://developer.android.com/guide/topics/resources/runtime-changes.html?utm_campaign=adp_series_prepareformultiwindow_032316&utm_source=medium&utm_medium=blog)）。
  
  默认案例是你的Activity完全被销毁（destroy）且被重新创建（recreate），接着它会恢复你在[`onSaveInstanceState(Bundle outState)`](https://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle))方法中存储的任何状态，接着重新加载所有的资源和布局。它有一个非常棒的属性，你知道，在新的配置下任何内容都是一致的且任何类型的配置都是被处理过的。
  
  不言而喻，**任何一个配置的变化都应该是快速且无缝的。** 你要确保无需在`onResume()`中做过多的工作，也不用考虑使用[`loaders`](https://medium.com/google-developers/making-loading-data-on-android-lifecycle-aware-897e12760832)来包装在变化之后你的数据仍然存在。
  
在这种情况下，你仍然可以自己来处理配置变化，你的Activity（或者Fragment）仍然会接收到一个回调到[`onConfigurationChanged(Configuration newConfig)`](https://developer.android.com/reference/android/app/Activity.html#onConfigurationChanged(android.content.res.Configuration))，而不用采用将它们销毁（destroy）、重建（recreate），之后你可以手动地更新Views，重新加载资源等等。

为了捕捉多窗口（multi-window）相关的配置变化，你需要在配置清单中增加`android:configChanges`属性，它至少要有这些属性值：
```
<activity
  android:name=".MyActivity"
  android:configChanges="screenSize|smallestScreenSize|screenLayout|orientation"/>
```
你要确保每个发生变化的资源你都能够处理到。这其中就包括了要重新加载之前被认为是不会变的资源。考虑这样一个情景，有一个dimen在`values`和`values-sw600dp`。在非分屏的世界中，在程序运行时，你永远不用考虑在这两个值之间切换，因为最小的宽度（smallest width）是不会发生变化的（它总是你设备的最小宽度值）。然而，在多窗口的场景下，当窗口改变尺寸的时候，你需要且不得不切换这些尺寸。

## Tip3：处理所有的方向

回想一下我们讨论过的，当窗口大小变化时，屏幕方向改变的问题。这是对的：即使设备在横屏状态下，你的App也可以是竖屏的方向。

  竖屏只是意味着高度大于宽度，横屏是宽度大于高度。要在心中明确这两个定义，当屏幕被重新定义大小的时候，你的App可能会从一个方向转到另一个方向上去。这意味在屏幕方向的转变要尽可能变得顺畅。引用`split screen material design specs`的内容：
  
  > Changing a device's orientation should not cause UI to change unexpectedly. For example, an app displaying a video in one of the split screens(in portrait mode) should not begin playback in full-screen if the device rotates to landscape mode.

> 注意：当你的App在全屏模式下，如果你仍然希望有此类型的功能，你可以使用`inMultiWindowMode`方法去检查一下窗口具体在哪种模式下。

通过使用`android:screenOrientation`属性来锁定你的屏幕方向同样会被多窗口特性影响。对于那些不是以Android N为目标的App来说，增加`android:screenOrientation`意味着你将不会支持多窗口模式，这样是强迫用户远离多窗口模式的做法。但是在Android N上事情就发生了一些变化，不是不支持多窗口模式，而是在多窗口模式下，通过`android:screenOrientation`设置的任何方向都将被忽略。

请记住，在程序运行的时候，通过`setRequestedOrientation()`方法锁定的方向，在多窗口模式下，将不会有任何效果，无论是否是在Android N下。

> 注意：在Activity的配置清单中，对于你的目标Activity增加[`android:immersive`](http://blog.csdn.net/sdvch/article/details/44209959)属性的话，对于不是Android N的Apps来说，它都会让多窗口被禁止掉。而规则就像上边的`android:screenOrientation`一样。

## Tip4：对于全部的屏幕尺寸来说，构建一个响应式UI

对于分屏的设计来说，屏幕的方向并不是唯一要考虑的事情。对于平板电脑来说，多窗口会将界面缩放到一个非常小的尺寸（你是否已经是平台电脑的UI了呢，毕竟在1.4亿台设备中，有12.5%的占有率是很多很多的设备）。

如果你一直在构建[响应式的用户界面](https://material.io/guidelines/layout/responsive-ui.html)来响应屏幕的可用空间，并且你的用户界面具有相对类似的手机和平板电脑的布局(手机和平板的UI长的差不多），你会发现这是提前为多窗口做好了准备。作为建议，你可以这么做，将用户界面缩小到220dp的宽或高，并且从这个尺寸构建到全屏尺寸。

![Building a single responsive layout makes for smooth transitions as your app resizes.](https://cdn-images-1.medium.com/max/880/0*l8MfhEL7Gt1YtQY5.png)

然而，你的手机App和平板App的用户界面差别很大，不要让用户在两个界面之间来回切换，仍要坚持使用平板用户界面并努力使它缩小到一个尺寸。这里有很多的响应式用户界面可供考虑，它们可以使得窗口的变化成为一种无缝转换的体验，而不再需要Android N的API来支持。

## Tip5：被其它App启动的Activities总是要支持多窗口的

在多窗口的世界中，你的全部[任务](https://developer.android.com/guide/components/tasks-and-back-stack.html?utm_campaign=adp_series_prepareformultiwindow_032316&utm_source=medium&utm_medium=blog)都在同一个窗口中被表现出来。这就是为什么如果你想启动一个邻近的Activity，你就需要启动一个新的任务，也是一个新的窗口。

反过来说也是成立的。
> If you launch an activity within a task stack, the activity replaces the activity on the screen, inheriting all of its multi-window properties.

意思是如果你有一个Activity会被其它的App启动起来，你启动的这个Activity会继承相同的多窗口属性。它包含的属性有最小的尺寸。

