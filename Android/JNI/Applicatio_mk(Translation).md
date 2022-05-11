# Application.mk （译文）

#### 这个文档解释了Application.mk这个构建文件，它描述了你的app需要的本地模块（native modules）。一个模块可以是一个静态库（static library），一个共享库（shared library），或者一个可执行文件。

#### 我们推荐你先阅读[Concepts](https://developer.android.com/ndk/guides/concepts.html)和[Android.mk](https://developer.android.com/ndk/guides/android_mk.html)文件，这样做会帮助你最大化地理解这篇内容。

### 概览（Overview）
------
Application.mk是GNU MakeFile中的一个极小的片段，它定义了几个用来编译的变量。它通常放置在`$PROJECT/jni/`下，`$PROJECT`指向了你的应用工程目录。另一个可选择的地方是在`$NDK/apps/`的顶级目录下的一个子目录。例如：
```
$NDK/apps/<myapp>/Application.mk
```
`<myapp>`是一个描述你的app到NDK构建系统的一个短名。它不会真实的生成共享库或者你最终的包。

### 变量（Variables）
------
#### APP_PROJECT_PATH
这个变量存储了你的app的项目根目录的绝对路径。构建系统使用这个信息将生成的JNI共享库的剥离版本放置到一个APK生成工具已知的具体位置。

如果将`Application.mk`放置到`$NDK/apps/<myapp>/`下边，你一定要定义这个变量；如果放在`$PROJECT/jni/`下，它是可选择的。

#### APP_MODULES
如果定义了这个变量，它会告知`ndk-build`只构建相应的模块和那些相应模块所依赖的。它一定是一个**空白分隔符分割的**名称列表，这些名称是在`Android.mk`中的`LOCAL_MODULE`中定义的。

如果这个变量没有定义，`ndk-build`会寻找可安装的顶层模块列表，即列举在`Android.mk`中的模块和它直接包含的模块。即使被导入的模块不是顶层的。

一个可安装的模块（installable module）可能是一个共享库，或者一个可执行文件，将会在`libs/$ABI/`中生成一个文件。

如果这个变量没有定义，而且在工程中没有可安装的模块，那么`ndk-build`会构建所有顶层的静态库和它们的依赖。这些静态库通常放在`obj/`或者`obj-debug/`下。

#### APP_OPTIM
定义这个**可选变量**为`debug`或者`release`。当构建应用的模块时，使用这个变量来选择优化等级。

Release模式是默认值，使用这个模式会生成高度优化的二进制数。Debug模式会生成更适合于调试的非优化二进制数。

请注意你既可以调试release模式的二进制数也可以调试debug模式下的二进制数，然而，在调试时release模式下可提供的信息更少。例如release模式下构建系统优化出的一些变量，你是无法审查它们的。同样，代码的顺序重整会使逐步跟踪更加困难，stack trace也会变得不可信赖。

在应用中的`<application>`标签中声明`android:debuggable`将会使`APP_OPTIM`设置为`debug`取代`release`。通过设置`APP_OPTIM`为`release`来覆盖默认值。

#### APP_CFLAGS
这个变量**存储了一组C编译器的flags**，当编译任何模块的C或者C++的源代码时，构建系统将传递这些flags到编译器。如果应用需要这个变量，可以**使用它来改变给定模块的构建，而不用修改`Android.mk`文件本身**。

所有在这些flags中的路径应该相对于NDK顶层目录的。例如，如果你有如下的设置：
```
sources/foo/Android.mk
sources/bar/Android.mk
```
为了在`foo/Android.mk`中指定在编译过程中你想添加到`bar`资源的路径，你应该使用：
```
APP_CFLAGS += -Isources/bar
```
或者
```
APP_CFLAGS += -I$(LOCAL_PATH)/../bar
```
`-I../bar`将不会工作，因为它等价于`-I$NDK_ROOT/../bar`。
> NOTE：在android-ndk-1.5_r1的资源中，这个变量只在C上有效，在C++上无效。在这个版本之后，`APP_CFLAGS`在所有的构建系统中均有效。

#### APP_CPPFLAGS
这个变量只包含了一组C++编译器flags，当只构建C++资源时，构建系统会将这些flags传递给编译器。

>NOTE：在android-ndk-1.5_r1中，这个变量对于C/C++均有效。在所有的NDK序列版本中，`APP_CPPFLAGS`匹配了所有的Android构建系统。对于想应用在C/C++资源的flags，请使用`APP_CFLAGS`。

#### APP_LDFLAGS
一组连接器（linker）flags，当构建系统链接应用时，传入这些flags。这个变量只在构建共享库和可执行文件的时候有关联。当构建静态库的时候，构建系统会忽略这些flags。

### APP_BUILD_SCRIPT
默认地，构建系统在`jni/`目录下寻找一个名字为`Android.mk`的文件。

如果你想覆盖这个行为，你可以定义`APP_BUILD_SCRIPT`去指定一个可选构建脚本。构建系统总是解释一个非绝对路径，作为相对于NDK的顶层目录。

#### APP_ABI
默认地，NDK构建系统会为`armeabi`应用二进制接口生成机器码。这样的机器码是对应于有软件浮点型运算的基于ARMv5TE的CPU。你可以使用`APP_ABI`来选择不同的应用二进制接口。表格1展示了`APP_ABI`的设置的不同指令集。

Instruction Set       | Value
--------              | ---
Hardware FPU instruction on ARMv7 based devices              | `APPABI := armeabi-v7a`
ARMv8 AArch64         | `APP_ABI := arm64-v8a`
IA-32                 | `APP_ABI := x86`
Intel64               | `APP_ABI := x86_64`
MIPS32                | `APP_ABI := mips`
MIPS64(r6)            | `APP_ABI := mips64`
All supported instruction sets | `APP_ABI := all`

>NOTE：`all`是从NDKr7开始可用的。

你同样可以指定多个值，将它们放置于同一行，空白符分割。例如
```
APP_ABI := armeabi armeabi-v7a x86 mips
```

对于列举的所有支持的ABI和它们的细节信息以及使用限制，请参照[ABI Management](https://developer.android.com/ndk/guides/abis.html)。

#### APP_PLATFORM
这个变量包含了你想支持的Android平台的最小版本号。例如，一个`android-15`的值指定了你的库在Android平台4.0.3(API level 15)以下是不可用的。想到得到一个完整的Android平台列表名称和相应的Android系统镜像，请看[Android NDK Native APIs](https://developer.android.com/ndk/guides/stable_apis.html)。

为了取代直接修改这个flag，你应该在你的`module-level`级别的`build.gradle`文件中，在`defaultConfig`块中设置`minSdkVersion`属性，或者设置`productFlavors`块。这样会使你确定你的库只能运行在那些平台版本允许安装的设备上。NDK构建工具链会遵循如下的逻辑，基于你正在构建的ABI和你指定的`minSdkVersion`，来为你的库选择最小的平台版本：
1. 如果对于ABI来说存在一个平台版本等于`minSdkVersion`，构建系统会使用这个版本。
2. 否则，如果对于ABI来说，存在平台版本低于`minSdkVersion`的话，构建系统使用那些版本中最高的平台版本。这个选择是合理的，因为一个平台版本的确实，严格意义上来说，这意味着相比前一个可用的版本来说对于本地平台API来说是没有变化的。
3. 否则，构建系统会使用高于`minSdkVersion`的下一个可用平台版本。

#### APP_STL
默认地，NDK构建系统为Android系统提供的C++运行时库（`system/lib/libstdc++.so`）提供C++文件。除此之外，你可以使用其他的实现，或者链接自己的应用上。使用`APP_STL`来选择其中的一个。关于支持的运行时信息和它们提供的功能，请看[NDK Runtimes and Features](https://developer.android.com/ndk/guides/cpp-support.html#runtimes)。

#### APP_SHORT_COMMANDS
对于整个工程中的`Application.mk`，它相当于`LOCAL_SHORT_COMMANDS`。更多信息，请看`Android.mk`中对`LOCAL_SHORT_COMMANDS`的文档。

#### NDK_TOOLCHAIN_VERSION
定义这个变量为`4.9`选择了GCC编译器的版本。定义这个变量为`clang`来选择C/C++等语言的编译器的版本,它是NDK r13之后的默认值。

#### APP_PIE
从Android 4.1（API level 16）开始，Android的动态链接支持位置无关（position-independent）的可执行文件。从Android 5.0（API level 21）开始，可执行文件要求位置无关（PIE）。为了使用PIE来构建你的可执行文件，设置`-fPIE`flag。这个flag使得通过随机化代码位置来挖掘（exploit）内存崩溃问题变得困难。默认地，如果你的平台目标是`android-16`或者更高，`ndk-build`会将这个值自动地设置为true，。同样你可以手动地设置`true`或者`false`。

这个flag只应用于可执行文件。当构建共享库或者静态库的时候，它不会产生影响。

#### APP_THIN_ARCHIVE
对于在工程中全部的静态库模块而言，在`Android.mk`文件中设置`LOCAL_THIN_ARCHIVE`的默认值。更多的信息，请看`Android.mk`中的`LOCAL_THIN_ARCHIVE`的文档。

### 附：为什么做这个翻译？
------
首先又要用到很久没有使用的JNI，要熟悉一下，后续我希望继续翻译`Android.mk`来熟悉这部分内容。其次，在我用`ndk-build`的时候遇到了一个问题，log如下：
```log
 No modules to build  your APP_MODULES definition is probably incorrect!
 make *** no rules to target...
```
经过Stackoverflow和Google的重重帮助，依然没有头绪。所以只能从原始文档出发，果然翻译的过程收到了效果，找到了解决方案，问题就出在配置`APP_MODULES`中。

```
...
APP_MODULES := hello-jni  // the wrong value I set before is 'jni'
APP_ABI := all
...
```
`APP_MODULES`的值只能是`Android.mk`中`LOCAL_MODULE`中定义的，所以它的值一定要和列表中定义的匹配才可以。

同时对于工程出现的复杂的`Application.mk`的各项参数要了解用途，才能快速的做出调整。

原文地址：https://developer.android.com/ndk/guides/application_mk.html
