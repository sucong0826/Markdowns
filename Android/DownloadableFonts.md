# 利用兼容库支持可下载字体

在2017年的Google I/O大会上，对于开发者来说以令人激动的消息是，从Android O开始，Android系统开始支持自定义的字体，同时，Android的兼容库也一直兼容到API 14。现在，你可以从[Google Fonts](https://fonts.google.com)的成千上万款字体中，为你的App选择一个最合适的字体。

那么记下来我们来说说使用[Downloadable Fonts（可下载字体）](https://developer.android.com/guide/topics/ui/look-and-feel/downloadable-fonts.html#adding-certificates)的优势。

## 可下载的字体的优势

你可以去看看Google官方的文档，附上链接[Downloadable Fonts（可下载字体）](https://developer.android.com/guide/topics/ui/look-and-feel/downloadable-fonts.html#adding-certificates)。

- 可下载的字体可以减小APK的体积。（因为你可以不用把.ttf/.otf这样的字体文件打包在你的APK中了。）
  > 这里附上一个文章，关于ttf与otf字体的比较。[otf/ttf/ttc格式字体的区别](https://jingyan.baidu.com/article/5d6edee2fe14f299eadeec1c.html)

- 同一台设备上的Apps可以从一个单一资源中共享字体，这种方式解决了因字体资源文件过多占用了用户过多的存储空间的问题。

对于原生支持的字体，Android系统尤其强调了第二点。来简单的看一下：

![Android's process to get a font](../pngs/AndroidProcessToGetAFont.png)

你可以看到，所有请求字体的Apps都会走到一个**Font Provider（字体提供者）**，而**Fonts Contract**指定一个**Font Provider**（也就说设备上的所有App都会通过FontsContract来找到一个Font Provider）。因此，如果一种字体已经被任意一个App（App X）请求过了，那么如果AppY再次请求的话就不会触发这个字体的下载任务了，而是使用在系统中缓存的字体文件。
