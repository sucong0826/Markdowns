# 理解音频焦点

![Audio focus](https://cdn-images-1.medium.com/max/2000/1*2_mUAwAihjBYMszQCCL0Mw.png)

# 第一部分

在你的Android设备上有许多的App可以同时播放音频。当Android操作系统将所有的音频流混合在一起的时候，对于一个用户来说这可能就是有破坏性的。这样的结果就会导致一个很差的用户体验。为了有一个良好的用户体验，Android系统提供给了一个API来让所有的Apps之间分享音频焦点（audio focus），在同一时刻，只有一个App可以持有音频焦点。

这个系列文章的目的在于：
*  让你更好、更深入的理解什么是**音频焦点**。
*  为什么一个好的多媒体用户体验是非常重要的。
*  如何使用它。

此系列文章将包括：

  1. 运作一个良好的多媒体类App的重要性以及最常见的音频焦点使用场景（第一部分）
  2. 音频焦点对于多媒体应用程序用户体验重要性相关用例（第二部分）
  3. 在你的应用中实现音频焦点的三个步骤（第三部分）

音频焦点是具有协作性质的，而且它依赖于各个App遵守音频焦点的使用准则。系统不会强迫去遵守这些准则，如果一个应用想在失去音频焦点之后还可以继续的大声播放，这件事情是无法被阻止的。然而，这会导致一个非常糟糕的用户体验，对于用户想卸载你的应用来说这可是一个好机会。这有一些使用音焦点景去播放的场景。假设用户启动了你的App，这个App正在播放一段音频。当你的App需要输出音频的时候，它应该请求音频焦点。只有在被授权了音频焦点之后，才应该去播放音频。

## 场景1 - 当你的App正在播放音频，当前用户启动了另一个多媒体App来播放音频

### 如果你的App不处理音频焦点，会发生什么呢？

如果另一个App开始播放音频，它会覆盖你正在播放的音频。结果就是体验很差，谁的也听不清楚了。

![Playing simultaneously](https://cdn-images-1.medium.com/max/800/1*zaIB6fKmwSwhm_UM3Yox_A.png)

### 处理音频焦点的App会发生什么情况？

当其他的多媒体应用开始播放音频，它应该请求**永久的音频焦点**。一旦系统给了音频焦点，**它将开始播放。你的应用需要通过停止播放来响应永久音频焦点的丢失**，所以用户只会听到另一个App播放的音频。

![Playing simultaneously](https://cdn-images-1.medium.com/max/800/1*xk8Tio4_XxtmuoH9CK7qkQ.png)

现在，如果用户想在你的App中继续播放音频，之后你的App将会请求音频焦点。只有当你的音频焦点被授权之后，你的App才可以继续播放音频。其它的App也会通过停止播放音频来响应音频焦点的丢失。

## 场景2 - 你听音乐正嗨，突然来了电话

### 如果你的App不处理音频焦点会发生什么

当你的手机开始响铃时，除了铃声之外用户也会听到你的App中播放的音频，这当然不是一个好的用户体验。如果他们选择了拒绝接听，那么你的音频应该继续播放。如果他们选择了接通电话，你App中的音频会随着手机通话音频一起播放。当它们完成了通话，如果你的App不会自动恢复播放的话，这当然也不是一个好的用户体验。

![Playing simultaneously](https://cdn-images-1.medium.com/max/1000/1*_HjTvrT4locQYp8LHIMVrA.png)

### 如果你的App处理了音频焦点会发生什么情况呢？

当你的手机铃声响起（而且用户还没有接通），你的App应该响应一个暂时的音频焦点丢失，响应的方式可以是降低音量，可以通过降低当前音量的20%来完成响应，或者一齐暂停当前所有音频。

 - 如果这个用户拒绝了这通电话，那么你的App应该通过恢复音量来响应焦点的重新获得，或者恢复播放。


 - 如果这个用户接通了电话，那么系统会通知你丢失了音频焦点。那么你的音频应该暂停播放。当它们电话打完了，你的App应该通过以正常音量恢复播放来响应音频焦点的重新获得。

![Playing simultaneously](https://cdn-images-1.medium.com/max/1000/1*P1JDTh8I8XkDwXMPjGD2cg.png)

当你的App需要输出音频，它就要请求音频焦点。只有在被授予了音频焦点之后，它才应该播放音频。然而，你得到的音频焦点不会乖乖地等你把音频播完，其它的App也会请求这个音频焦点，并且会抢占了这个音频焦点。在这种情况下，你的App应该暂停播放或者降低音量来让用户更容易听到新的音频资源。

# 第二部分（更多场景）

## 场景3 - 在后台运行的导航类App提供了导航音频，而另一个App正在播放音频

### 如果不处理音频焦点会怎么样？

重叠的导航音频和音乐会使得用户分神。

### 应该如何处理音频焦点呢？

当导航App播报下一个指令的时候，你的App应该减小音量来响应焦点暂时的丢失（因为导航App正在播放音频）。但在一些情况中，你的App应该停止播放：播放音频书，播客，讲述。

当导航App的指令播放完毕之后，它会扔掉音频焦点（audio focus），而你的App会重新拿到这个焦点，并且应该恢复音量到原来的等级上去，作为重新获得音频焦点的响应。

## 场景4 - 当接通电话时，用户启动了一个会播放音频的Game

### 如果不处理音频会怎么样？
用户体验很差，因为音频和电话通话的声音叠加到了一起。

###  应该如何处理音频焦点呢？
在Android O的版本，这有一个关于音频焦点的特性，叫做“**取得被延迟的音频焦点**”。这个特性恰巧就是为这个场景创建的。看这个例子，当用户正在打电话，而且他有启动了一个游戏来玩，他们想打游戏但是又不想听到任何游戏的声音，但是电话结束后，游戏的音频又要回来让他们听到。

如果你的App想支持这个功能，并且用户正在打电话（已经获得了暂时的音频焦点）时播放音频，则会发生两件事情。

1. 当你的App请求永久的音频焦点时， 系统拒绝了授予，因为它被锁住了。系统的电话App已经取得了暂时的音频焦点。你的Game App不应该播放音频（它可能会在稍后的某个时间被授予音频焦点）。然而，在这种情况下，你的App是个游戏，可能会在没有音频的条件下持续工作。

2. 当电话结束了，你的App会被授予“获得音频焦点” `audio focus gain`。它的授权被延迟了一段时间（最初的请求是在用户打电话的时候）。你可以像处理当一个暂时性的音频焦点丢失后又重新获得焦点那样来处理这种场景。在这种情况下，它会开始播放音频。

而在Android O之前的版本并不支持这样的特性（`delayed audio focus gain`），在以前的版本中，当你的App试图开始播放音频而用户正在打电话，这个音频焦点的请求可能不会被简单的授权，而且即使通话结束，播放也不会开始。

## 场景5 - 导航类应用或者其他会生成音频提醒/提示的App

如果你正在构建的App会突然间产生一小段音频，那么使音频焦点在一个正确的顺序上对你来说是一个非常重要的事情，这会影响到用户体验。这类App是会生成一个提示音或者一个提醒音，或者这个App会在后台给出导航指令。

让我们说说，当你的App运行在后台，并且即将要生成一段音频。而且用户正在听音乐，或者一段播客，或者你的App生成了一段非常短的音频。

在你的App生成了这段音频，他应该请求一个暂时的音频焦点。只有在音频焦点被授权的时候你的App才应该播放音频。而刚刚播放正常的App，比如音乐App，则应该响应暂时的焦点丢失，然后音量减小。如果另一个App是播客类的App的话，则它应该暂停播放直到音频焦点重新获得之后才重新播放。如果你的App请求焦点失败，则会导致用户同时听到你的提示音频和其它App产生的音乐。

## 场景6 - 录音机或语音识别App

如果你正在建立一个的App是去录一小段音频，在录音期间，无论是系统的还是其它的App都不应该产生任何的声音（包括提示音或者多媒体播放），那么处理好音频焦点对于用户来说是至关重要的。这样的App包括录音机或语音识别App。

你的App应该去请求暂时的和独有的音频焦点。如果这个请求被授权，那么你就可以开始录音，系统不会产生任何的音频来干扰你的录音。在录音期间，如果其他的App去请求了焦点，那么它们的请求会被拒绝。如果用户完成了录音，那么你应该丢掉音频焦点，以至于系统App可以正常的播放声音。

### 小结2：
---
如果你的App需要输出音频，那么它就应该请求焦点（这里有不同类型的音频焦点可以请求）。

只有在被授权了音频焦点之后，它才可以播放音频。然而，在你获得了音频焦点之后，你并不会一直持有它直到播放结束，因为其它App可以请求音频焦点。

其它可能抢占音频焦点的App会申请音频焦点。在这种情况下，你的App应该暂停播放或者降低音量来让用户听到新的音频资源。

在Android O中，如果你的App在请求音频焦点后不能立即获得，系统会给你的App一个延迟的音频焦点。

# 第三部分（三个步骤实现音频焦点）

如果你的App没有合理的处理音频焦点，那么下面的图示就描述了用户可能会遇到的糟糕体验。

![If your app doesn't handle audio focus properly](https://cdn-images-1.medium.com/max/880/1*53tFOWaJmR_hrJq8QL0DHg.png)

既然你已经知道了做好一个音频播放类App的重要性，这种重要性也要体现在用户体验上，那就来看看如何做好一个妥善处理音频焦点的App。

在跳到代码之前，让我们来看一下下面的图是如何正确完成处理音频焦点步骤的。

![How to handle audio focus](https://cdn-images-1.medium.com/max/880/1*KdcNZbndhRg5ne18kquBKA.png)

###  步骤1：确定你的音频焦点请求
---

获取音频焦点的第一步，就是向系统发起请求。要牢记，你发起的请求并不意味着你的请求会得到结果。为了请求去获得音频焦点，你要说明App请求的意图。这里列举了一些常见的意图。

- 如果你的App是一个多媒体播放器、播客播放器这样占有音频焦点不定时长的应用（从用户选择你的App开始播放音频算起），这就是`AUDIOFOCUS_GAIN`。

- 你的App是暂时需要音频焦点么（可以是duck选项），这种情况下你的App可能是需要播放一段提示音，或者一个导航指示，或者要录一段音频。这就是`AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK`。

- 你的App也是暂时的需要音频焦点么？但是你不知道这段音频的播放会持续多久，可能很短，有可能很长，例如系统中的电话App会接通一个电话一样。这就是`AUDIOFOCUS_GAIN_TRANSIENT`。

- 如果你的App是暂时性获焦点的，但是使用多久无从而知，在这期间不允许有其他的音频产生，例如录音器这样的App。这就是`AUDIOFOCUS_GAIN_TRANSIENT_EXCLUSIVE`。

在Android O和之后的版本中，你必须创建一个`AudioFocusRequest`的对象去请求音频焦点（使用builder），这个对象中你必须指定你的App需要使用多久的音频焦点。接下来的代码片段中，会说明向系统申请永久的音频焦点的意图。

```
AudioManager mAudioManager = (AudioManager) mContext.getSystemService(Context.AUDIO_SERVICE);

AudioAttributes mAudioAttributes = new AudioAttributes.Builder()
  .setUsage(AudioAttributes.USAGE_MEDIA)
  .setContentType(AudioAttributes.CONTENTS_TYPE_MUSIC)
  .build();
  
AudioFocusRequest mAudioFocusRequest = new AudioFocusRequest.Builder(AudioManager.AUDIOFOCUS_GAIN)
  .setAudioAttributes(mAudioAttributes)
  .setAcceptsDelayedFocusGain(true)
  .setOnAudioFocusChangeListener(...) // Need to implement listener
  .build();
  
int focusRequest = mAudioManager.requestAudioFocus(mAudioFocusRequest);
mAudioManager.requestAudioFocus(mAudioFocusRequest);

switch (focusRequest) {
  case AudioManager.AUDIOFOCUS_REQUEST_FAILED:
    // DON'T START PLAYBACK
  case AudioManager.AUDIOFOCUS_REQUEST_GRANTED:
    // actually start playback
}
```
注意：

1. `AudioManager.AUDIOFOCUS_GAIN`是向系统请求永久性的音频焦点所使用的参数。你同样可以传递其他的int值，例如`AUDIOFOCUS_GAIN_TRANSIENT`或者`AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK`，如果你是想暂时的获得焦点的话，可以使用这两个参数。

2. 你必须传递一个`AudioManager.onAudioFocusChangedListener`的实现到`setOnAudioFocusChangedListener()`方法中。这是用来处理音频焦点发生变化时的代码，事件的发生是由系统来驱动的，而音频焦点发生变化则可能是由用户在其他的App引起的。例如，你的App可能已经获得了一个永久性的焦点，但是当用户启动了另一个App之后就把这个焦点给带走了。那么这个监听器的作用就是来处理焦点发生变化的。

3.  一旦你创建了`AudioFocusRequest`的对象，你可以用它来向`AudioManager`来请求音频焦点，通过`requestAudioFocus(...)`方法就可以做到。这个方法将会返回一个`int`值来表示你的音频焦点请求成功或者失败。只有它的返回值是`AUDIOFOCUS_REQUEST_GRANTED`，才可以立即播放。而且如果它是`AUDIOFOCUS_REQUEST_FAILED`，那么系统已经拒绝了你的请求。

在Android N和之前的版本，你可以不用`AudioFocusRequest`来声明这个意图，但是你仍要实现`AudioManager.OnAudioFocusChangedListener`。看如下代码。
```
AudioManager mAudioManager = (AudioManager) mContext.getSystemService(Context.AUDIO_SERVICE);
int focusReq = mAudioManager.requestAudioFocus(
  // need to implement listener
  AudioManager.STREAM_MUSIC,
  AudioManager.AUDIOFOCUS_GAIN
);
switch (focusReq) {
  case AudioManager.AUDIOFOCUS_REQUEST_FAILED:
    // DON'T START PLAYBACK
  case AudioManager.AUDIOFOCUS_REQUEST_GRANTED:
    // ACTUALLY START PLAYBACK 
}
```
接下来，我们需要实现`AudioManager.OnAudioFocusChangeListener`，这样我们就响应音频焦点的获得与丢失。

### 步骤2：响应音频焦点的状态变化
---
一旦你的App被授予了音频焦点（无论是暂时的还是永久的），它都是可以在任意时间被改变的。而且你的App一定要响应这个变化。这是发生在`onAudioFocusChangeListener`里面的实现。

下面的代码中包含了这个接口的实现。而且它处理了暂时的音频焦点丢失事件。同样，它也处理了由于用户在当前App暂停了播放引起的音频焦点变化和其它App（像Google Assistant）引起的暂时的音频焦点丢失。

```
private final class AudioFocusHelper implements AudioManager.OnAudioFocusChangeListener {
  
  private void abandonAudioFocus() {
    mAudioManager.abandonAudioFocus(this);
  }
  
  @Override
  public void onAudioFocusChange(int focusChange) {
    switch (focusChange) {
      case AudioManager.AUDIOFOCUS_GAIN:
        if (mPlayOnAudioFocus && !isPlay()) {
          play();
        } else if (isPlaying()) {
          setVolume(MEDIA_VOLUME_DEFAULT);
        }
        mPlayOnAudioFocus = false;
        break;
        
      case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:
        setVolume(MEDIA_VOLUME_DUCK);
        break; 
        
      case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT:
        if (isPlaying()) {
          mPlayOnAudioFocus = true;
          pause();
        }
        break;
        
      case AudioManager.AUDIOFOCUS_LOSS:
        mAudioManager.abandonAudioFocus(this);
        mPlayOnAudioFocus = false;
        stop();
        break;
    }
  }
}
```
当用户启动已暂停的播放时，应用程序的行为应该不同，从另一个应用程序请求暂时性的音频焦点并且播放暂停（不仅是音量减小的行为）。当用户发起的是暂停播放，你的App应该丢弃音频焦点。然而，如果你的App因为暂时的音频焦点丢失而暂停播放，那么接下来它就应该放弃音频焦点。这有一些使用案例来说明这一点：

让我们说一说如果你的App在后台播放着一段音频：

1. 当用户按下播放键，你的App回去请求永久性的音频焦点。现在，系统授权了这次讲求，将焦点给了你的App。
2. 现在，他们长按HOME键然后启动了Google Assistant。那么Google Assistant就去请求获得暂时的音频焦点。
3. 一旦系统授权了Google Assistant给它音频焦点，那么你的App中的`OnAudioFocusChangeListener`将会得到一个`AUDIOFOCUS_LOSS_TRANSIENT`事件。这里，你会暂停播放你的App中的音频，因为Google Assistant需要去录音。
4. 一旦Google Assistant完成了录音，它会丢弃音频焦点，然后你的App将会在`OnAudioFocusChangeListener`中被授权`AUDIOFOCUS_GAIN`。这里你就必须来决定是去恢复播放还是不恢复。这就是代码中那个`mPlayOnAudioFocus`flag所做的事情。

如下的代码是`pause`方法，在上面的代码中用到的：
```
public final void pause() {
    if (!mPlayOnAudioFocus) {
      mAudioFocusHelper.abandonAudioFocus();
    }
    onPause();
}
```
就是你看到的这样，当用户触发了暂停播放的时候，你的App就会丢掉焦点，但是当其他的App触发暂停播放时，你的App则不会丢掉焦点（当其它的App获取的`AUDIOFOCUS_GAIN_TRANSIENT`）。

####  对于暂时的音频焦点丢失，减小音量 vs 暂停播放

在监听器中，你可以选择暂停播放音频或者暂时的减小音量， 这取决于你要向用户传递一个什么样的体验。在Android O中，系统支持自动的音量减小，意思是系统会自动的减小音量而不同写额外的代码。在你的`OnAudioFocusChangeListener`中，就可以忽略`AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK`事件了。

在Android N之前的版本，你必须手动去实现音量减小。

#### 延迟获得

Android O 引入了一个概念，延迟获得音频焦点。为了实现这个特性，当你请求音频焦点的时候，你会得到一个`AUDIOFOCUS_REQUEST_DELAYED`结果：
```
public void requestPlayback() {
  int audioFocus = mAudioManager.requestAudioFocus(mAudioFocusRequest);
  switch (audioFocus) {
    case AudioManager.AUDIOFOCUS_REQUEST_FAILED:
      ...
    case AudioManager.AUDIOFOCUS_REQUEST_GRANTED:
      ...
      
    case AudioManager.AUDIOFOCUS_REQUEST_DELAYED:
      mAudioFocusPlaybackDelayed = true;
  }
}
```

在你的`OnAudioFocusChangeListener`实现中，当你响应了`AUDIOFOCUS_GAIN`，你一定要去检查这个`mAudioFocusPlaybackDelayed`的变量。
```
private void onAudioFocusChange(int focusChange) {
  switch (focusChange) {
    case AudioManager.AUDIOFOCUS_GAIN:
      logToUI("Audio focus: gained");
      if (mAudioFocusPlaybackDelayed || mAudioFocusResumeOnFocusGained) {
        mAudioFocusPlaybackDelayed = false;
        mAudioFocusResumeOnFocusGained = false;
        start();
      }
      break;
    
    case AudioManager.AUDIOFOCUS_LOSS:
      mAudioFocusResumeOnFocusGained = false;
      mAudioFocusPlaybackDelayed = false;
      stop();
      break;
      
    case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT:
      mAudioFocusResumeOnFocusGained = true;
      mAudioFocusPlaybackDelayed = false;
      pause();
      break;
      
    case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:
      pause();
      break;
  }
}
```

#### 步骤3 记得去丢弃音频焦点

当你的App完成了音频的播放之后，那么你应该通过如下的代码丢球音频焦点。
```
AudioManager.abandonAudioFocus(...);
```
在之前的步骤中，我们遇到了在用户启动暂定播放时，放弃音频焦点的情况，但是你的App在暂停时被其它的App暂时中断而保留了音频焦点。


完整的代码请查看：
https://github.com/googlesamples/android-media-controller#audio-focus
