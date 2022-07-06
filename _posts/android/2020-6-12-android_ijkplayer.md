---
layout:     post
title:      "『Android』 MediaPlayer + TextureView 实现视频播放器"
subtitle:   "基于 MediaPlayer/IjkPlayer + TextureView 实现。"
date:       2018-06-29 12:00:00
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Android
---


在开发中，我们知道，Android 中实现视频播放器的途径有两种：
-  `VideoView`：`VideoView` 使用比较简单，配合 `MediaController` 可以达到控制播放、暂停、快进、快退、切换视频、进度条显示等，具体使用在这里不在赘述了。
-  `MediaPlayer` + `SurfaceView `/ `TextureView`：本文主要通过此途径实现可自定义的视频播放器。

本文实现视频播放器，主要分为以下几部分：
- MediaPlayer 介绍和使用
- SurfaceView / TextureView 对比
- 简单视频播放器的实现
- 控制调节功能的实现

---
### 一、MediaPlayer 的介绍和使用

[MediaPlayer](https://developer.android.google.cn/reference/android/media/MediaPlayer) 可用于控制音频 / 视频文件和流的播放。

在下面的官网图中，是相关操作下 `MediaPlayer` 对象的生命周期和状态。
- 椭圆形表示 `MediaPlayer` 对象可能会存在的状态。
- 弧形表示对 `MediaPlayer` 对象的回放控制等操作，有两种类型的弧，单箭头表示同步方法调用，双箭头表示异步方法调用。

![MediaPlayer 的生命周期和状态](/img/android/android_ijkplayer/0.png)

#### MediaPlayer 的生命周期
配合官网图，我们了解下有关媒体播放器的生命周期。

- 当新建对象或者在创建之后调用 `reset()` 方法时，`MediaPlayer` 对象会处于 *Idle* 状态。在调用 `release()`之后处于 *End* 状态。

在这两种状态之间是 `MediaPlayer` 对象完整的生命周期。一旦不再使用 `MediaPlayer` 对象，需要立即调用 `release()` 释放资源。`MediaPlayer` 对象处于 *End* 状态，就无法再使用它，也无法将其恢复为其他任何状态。

- 某些回放控制操作可能由于各种原因而失败，例如，不支持的音频/视频格式，音频/视频的交错差，分辨率太高，流式传输超时等，在所有这些错误条件下，如果已经 `setOnErrorListener()` ，则内部播放器引擎将走到的`OnErrorListener.onError()` 的回调，我们可以在这里面做处理。

- `setDataSource()` 用于设置视频资源，只能在 *Idle* 状态下调用，其他状态下会抛出 `IllegalStateException`，调用后处于 *Initialized* 状态。

- `MediaPlayer` 需要 *Initialized* 状态下才能进行准备工作，`MediaPlayer` 提供了两个方法 `prepare()`（同步）、`prepareAsync()`（异步）使其进入 *Prepared* 状态，异步调用可以在 `OnPreparedListener` 接口的回调方法 `onPrepared()` 中进行 *Prepared* 之后的操作。

- 当 `MediaPlayer` 在 `Prepared` 状态之后可以调用 `start()` 进入 *Started* 状态，并开始播放视频，`isPlaying()` 可以查看 `MediaPlayer` 对象是否处于 *Started* 状态。在 *Started* 状态时，通过 `setOnBufferingUpdateListener()` 可以在 `OnBufferingUpdateListener.onBufferingUpdate()` 回调中获取流式传输 音频/视频 时跟踪其缓冲状态。

- 播放可以暂停和停止，并且可以调整当前播放位置。

可以通过 `pause()` 暂停播放。当调用 `pause()` 返回时，`MediaPlayer` 对象将进入 *Pause* 状态。注意在播放器引擎中，从 *Strated* 状态到 *Pause* 状态的转换是异步的，可能要花一些时间。调用 `start()` 会重新变为 *Started* 状态并开始播放。

可以通过 `stop()` 停止播放，这时，处于 *Started*，*Paused*，*Prepared* 或 *PlaybackCompleted* 状态的 `MediaPlayer` 都会进入 *Stopped* 状态。处于 *Stopped* 状态，就无法开始播放，直到调用 `prepare()` 或 `prepareAsync()` 将 `MediaPlayer` 对象重新设置为 *Prepared* 状态。

- 调整播放位置可以通过 `seekTo()` 方法，由于 `seekTo()` 是异步的，实际上查找需要一定时间才能完成，实际的查找位置完成时会走 `setOnSeekCompleteListener()` 的 `OnSeekComplete.onSeekComplete()` 回调。
 `seekTo()` 在 *Prepared*，*Paused* 和 *PlaybackCompleted* 状态下执行仍然会保持当前的状态。
实际的当前播放位置可以通过 `getCurrentPosition()` 获取。

- 当视频播放完成之后默认会走 `setOnCompletionListener()` 中的 `OnCompletion.onCompletion()` 回调，在回调后处于 *PlaybackCompleted* 状态。
如果设置 `setLooping()` 为 `true` 时，会在播放完成后重新变为 *Started* 状态并重新播放视频。
在 *PlaybackCompleted* 状态下，调用 `start()` 也可以从音频/视频源的开头重新开始播放。

### 二、SurfaceView / TextureView 对比

这里为什么有 `SurfaceView` 和 `TextureView` 两种方式呢？然后哪种方式更适合用在项目中呢？接下来我们通过对比来了解一下。

[SurfaceView](https://developer.android.google.cn/reference/kotlin/android/view/SurfaceView.html)：SurfaceView 是一个绘图的面板，可以控制此面板的格式，也可以控制其大小，通过线程将视频渲染到屏幕上。

[TextureView](https://developer.android.google.cn/reference/android/view/TextureView)：TextureView 可用于显示内容流，这样的内容流可以是视频或 `OpenGL` 场景、可以是来自本地数据源或者是远程数据源。

TextureView 只能在有硬件加速的窗口中使用，当软件渲染时，TextureView 不绘制任何内容。与 SurfaceView 不同，TextureView 不会创建单独的窗口，而是充当常规 `View`，所以可以对 TextureView 进行移动，转换，设置动画等操作。

如果项目需求简单，当然可以选择 SurfaceView，但是在这里选择了后者，原因是 TextureView 更适合视频流，以及具备普通 `View` 的特性和高自定义性，可以在项目中达到想要的变化效果。既然选定了 TextureView ，接下来看看如何使用。

#### TextureView 的使用

我们可以通过调用 `getSurfaceTexture()` 来获取 TextureView 的 `SurfaceTexture`。需要将 `TextureView` 附加到窗口（并 `onAttachedToWindow()` 已被调用）后，`SurfaceTexture` 才可用。因此，在 `SurfaceTexture` 可用时通过实现 `TextureView.SurfaceTextureListener` 来获取状态的监听。要注意，只有一个生产者可以使用 `TextureView`。例如，如果使用 `TextureView` 显示相机预览，则 `lockCanvas()` 无法同时在 `TextureView` 上绘制。

 `SurfaceTextureListener` 的回调。
 
```kotlin
    override fun onSurfaceTextureSizeChanged(surface: SurfaceTexture?, width: Int, height: Int) {
        //SurfaceTexture缓冲区大小更改时调用（这里的width、height是改变后的画布大小）
    }

    override fun onSurfaceTextureUpdated(surface: SurfaceTexture?) {
       //SurfaceTexture通过更新指定的值 时调用
    }

    override fun onSurfaceTextureDestroyed(surface: SurfaceTexture?): Boolean {
       //当指定的SurfaceTexture对象即将被销毁时调用。返回true，则调用此方法后，
      //表面纹理内不应进行任何渲染。如果返回false，则客户端需要SurfaceTexture.release()。
        return false
    }

    override fun onSurfaceTextureAvailable(surface: SurfaceTexture, width: Int, height: Int) {
        //当TextureView准备使用SurfaceTexture时调用（这里的width、height是原始画布大小）
    }
```


### 三、实现简单视频播放器

#### 流程图

![流程图](/img/android/android_ijkplayer/1.png)

#### 项目结构
项目结构的话借鉴 Google 官方 `MVP` 模式，能实现视频播放器解耦，功能操作和 UI 逻辑分离，使用契约类实现 `Presenter` 和 `Views`，`Presenter` 中实现各种功能逻辑，`Views` 负责 UI 展示。下面是项目的结构：

![JVideoView项目结构](/img/android/android_ijkplayer/2.jpg)

#### 功能实现
- 契约类 `JVideoViewContract` 的实现，分工明确，在 `Presenter` 中实现播放器功能逻辑。

```kotlin
    interface JVideoViewContract {
        interface Views {
            //设置presenter
            fun setPresenter(presenter: Presenter)
            //设置播放标题
            fun setTitle(title:String)
            //缓冲中
            fun buffering(percent:Int)
            /**
            * 其他状态下需实现的UI，例如：暂停、加载中、播放结束等
            */
        }

        interface Presenter {
            //实现订阅关系
            fun subscribe()
            //移除订阅关系
            fun unSubscribe()
            //开始播放
            fun startPlay(position: Int = 0)
            //暂停播放
            fun pausePlay()
            /**
            * 其他改变播放状态或者播放器参数的功能，比如初始化播放器、开始播放，滑动快进、音量调节等
            */
        }
    }
```

- 播放器的播放状态和播放模式等都写成常量封装在状态类 `JVideoState` 中，避免使用时过于混乱。

```kotlin
    class JVideoState {
         //播放器状态
         class PlayState{
         }

         // 播放模式
         class PlayMode{
         }

         //调节模式
         class PlayAdjust{
         }
    }
```

- 播放器状态常量 `PlayState`。

```kotlin
     /**
     * 播放器状态
     */
    class PlayState{
            companion object{
                //播放错误
                const val STATE_ERROR = -1
                //播放未开始
                const val STATE_IDLE = 0
                //播放准备中
                const val STATE_PREPARING = 1
                // 播放准备就绪
                const val STATE_PREPARED = 2
                // 开始播放
                const val STATE_START = 3
                //正在播放
                const val STATE_PLAYING = 4
                // 暂停播放
                const val STATE_PAUSED = 5
                //正在缓冲(播放器正在播放时，缓冲区数据不足，进行缓冲，缓冲区数据足够后恢复播放)
                const val STATE_BUFFERING_PLAYING = 6
                // 正在缓冲(播放器正在播放时，缓冲区数据不足，进行缓冲，此时暂停播放器，继续缓冲，缓冲区    
               //数据足够后恢复暂停
                const val STATE_BUFFERING_PAUSED = 7
                // 播放完成
                const val STATE_COMPLETED = 8
            }
        }
```

- 考虑到后期能运用在其他项目中，`JVideoView` 继承 `LinearLayout`、实现 `JVideoViewContract.Views`, `TextureView.SurfaceTextureListener` 接口，同时布局由控制器界面和播放界面组成。

```kotlin
class JVideoView : LinearLayout, JVideoViewContract.Views, TextureView.SurfaceTextureListener {
    /*具体逻辑*/
 override fun onSurfaceTextureSizeChanged(surface: SurfaceTexture?, width: Int, height: Int) {
        //SurfaceTexture缓冲区大小改变时
     
    }

    override fun onSurfaceTextureUpdated(surface: SurfaceTexture?) {
        // SurfaceTexture更新时
    }

    override fun onSurfaceTextureDestroyed(surface: SurfaceTexture?): Boolean {
        //幕布销毁时释放资源
        return false
    }

    override fun onSurfaceTextureAvailable(surface: SurfaceTexture, width: Int, height: Int) {
        //幕布准备完毕，这里是原始画布大小

    }
}
```

- 在 `JVideoView` 中实现对UI的显示控制。

```kotlin
    override fun setPresenter(presenter: JVideoViewContract.Presenter) {
        mPresenter = presenter
    }
    override fun setTitle(title: String) {

    }
    override fun preparedVideo(videoTime: String, max: Int) {
    }

    override fun startVideo(position: Int) {
    }

    override fun buffering(percent: Int) {
    }

    override fun continueVideo() {
    }

    override fun pauseVideo() {
    }

    override fun playing(videoTime: String, position: Int) {
    }

    override fun completedVideo() {
    }

    override fun showLoading(isShow: Boolean, text: String) {
    }
```

- 在 `JVideoViewPresenter` 中持有 `JVideoView` 和 `VideoRepository`（数据仓库）的引用，里面实现具体的播放功能。

```kotlin
class JVideoViewPresenter(
    private val mContext: Context,
    private val mView: JVideoViewContract.Views,
    private val mVideoRepository: VideoRepository
) : JVideoViewContract.Presenter {

    init {
        mView.setPresenter(this)
    }
/*其他操作*/
}
```

- `JVideoViewPresenter` 中实现数据的获取以及功能实现后调用 `JVideoViewContract.Views` 的方法。

```kotlin
    override fun subscribe() {
    }

    override fun unSubscribe() {
    }

    override fun startPlay(position: Int) {
         //开始播放
    }

    override fun pausePlay() {
        //暂停播放视频
    }
    override fun continuePlay() {
        //继续暂停播放视频
    }
    
    override fun onPause() {
        //onPause,可在此处暂停播放视频
    }
    
    override fun onResume() {
        //onResume，可在此处恢复播放视频
    }

    override fun getDuration(): Int {
        /*总进度获取*/
    }

    override fun getPosition(): Int {
        /*当前进度获取*/
    }

    override fun getBufferPercent(): Int {
        /*获取缓冲*/
    }

    override fun releasePlay(destroyUi: Boolean) {
        /*结束播放时释放资源，如对一些强引用的置空等*/
    }
```

- 在 `JVideoViewPresenter` 中实现数据源的获取。

```kotlin
    //获取数据源
    private fun loadVideosData() {
        mVideoRepository.getVideos(object : VideoDataSource.LoadVideosCallback {
            override fun onVideosLoaded(videos: List<Video>) {
                /*数据源获取成功*/
            }

            override fun onDataNotAvailable() {
                /*数据源获取失败*/
            }
        })
    }
```
- 关于 `MediaPlayer` 的设置
1. 设置播放器的幕布以及播放的 Url 以及相关的音频参数。

```kotlin
    //设置视频的url，本地或者网络
    mPlayer?.setDataSource(video.videoUrl)
    //设置渲染画板
    mPlayer?.setSurface(surface)
    //设置是否循环播放，默认可不写
    isLooping = false
    //设置播放类型、音频参数
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        val attributes = AudioAttributes.Builder()
                  .setContentType(AudioAttributes.CONTENT_TYPE_MUSIC)
                  .setFlags(AudioAttributes.FLAG_LOW_LATENCY)
                  .setUsage(AudioAttributes.USAGE_MEDIA)
                  .setLegacyStreamType(AudioManager.STREAM_MUSIC)
                  .build()
            mPlayer?. setAudioAttributes(attributes)
    } else {
      mPlayer?.setAudioStreamType(AudioManager.STREAM_MUSIC)
    }
    //设置是否保持屏幕常亮
    setScreenOnWhilePlaying(true)
```

2. 在设置完 `MediaPlayer` 的基本参数之后设置各种监听 `setOnCompletionListener`（播放完毕），`setOnSeekCompleteListener`（seekTo 查找完成），`setOnPreparedListener`（预加载完成），`setOnBufferingUpdateListener`（缓冲变化），`setOnErrorListener`（播放错误），` setOnInfoListener`（播放器信息），`setOnVideoSizeChangedListener`（视频尺寸修改），并修改相应的状态表示。

```kotlin
    //播放完成监听
    mPlayer?.setOnCompletionListener {
    }
     //seekTo()调用并实际查找完成之后
    mPlayer?.setOnSeekCompleteListener {
    }
     //预加载监听
    mPlayer?.setOnPreparedListener {
    }
    //相当于缓存进度条
    mPlayer?.setOnBufferingUpdateListener { mp, percent ->
    }
    //播放错误监听
    mPlayer?.setOnErrorListener { mp, what, extra ->
        true
    }
    //播放信息监听
    mPlayer?. setOnInfoListener { mp, what, extra ->
        true
    }
    //播放尺寸
    mPlayer?.setOnVideoSizeChangedListener { mp, width, height ->
        //这里是视频的原始尺寸大小
    }
```

3. 幕布准备完毕也就是前面的 `onSurfaceTextureAvailable()` 回调中让 `MediaPlayer` 进入 *Prepared* 状态。在这里用异步的方式 `prepareAsync`，在进入 `setOnPreparedListener`（预加载完成）之前都设为 `STATE_PREPARING`。

```kotlin
    //预加载监听
    mPlayer?.setOnPreparedListener {
        mPlayState = PlayState.STATE_PREPARED
        //预加载后播放
        mPlayer?.start()
    }
    //异步的方式装载流媒体文件
     prepareAsync()
```

4. 视频播放中会有缓冲，当缓冲区不足时会停止播放并进行缓冲加载，缓冲加载中如果用户未暂停则设为 `STATE_BUFFERING_PLAYING` 状态，如果暂停则设为 `STATE_BUFFERING_PAUSED` 状态，缓冲完成之后恢复到原来的播放状态。

```kotlin
    //播放信息监听
    mPlayer?. setOnInfoListener { mp, what, extra ->
        when (what) {
            MediaPlayer.MEDIA_INFO_VIDEO_RENDERING_START -> {
                // 播放器开始渲染
                mPlayState = PlayState.STATE_PLAYING
            }
            MediaPlayer.MEDIA_INFO_BUFFERING_START -> {
                // MediaPlayer暂时不播放，以缓冲更多的数据
                mPlayState = if (mPlayState == PlayState.STATE_PAUSED || mPlayState == PlayState.STATE_BUFFERING_PAUSED) {
                        PlayState.STATE_BUFFERING_PAUSED
                    } else {
                        PlayState.STATE_BUFFERING_PLAYING
                    }
             }
            MediaPlayer.MEDIA_INFO_BUFFERING_END -> {
                // 填充缓冲区后，MediaPlayer恢复播放/暂停
                if (mPlayState == PlayState.STATE_BUFFERING_PLAYING) {
                    mPlayState = PlayState.STATE_PLAYING
                }
                if (mPlayState == PlayState.STATE_BUFFERING_PAUSED) {
                    mPlayState = PlayState.STATE_PAUSED
                }
            }
            MediaPlayer.MEDIA_INFO_NOT_SEEKABLE -> {
                //无法seekTo
            }
        }
    }
```
- 到这里一个简单的播放器就完成了。但是~ ~，简单是不行的，单纯靠简单直接使用 `VideoView` 不就行了，所以还要实现拖动改变进度，全屏播放，滑动改变进度、音量和亮度调节。

### 四、调节控制功能的实现
- 拖动改变进度的实现，拖动主要是在控制器底部写一个 `SeekBar`，仿照平时看剧的 App 就行，设置 `SeekBar` 总进度和视频的总长度一致，拖动时在 `SeekBar` 的 `onStopTrackingTouch` 中通知播放器设置进度。

```kotlin
    override fun onStopTrackingTouch(seekBar: SeekBar) {
        //seekBar滑动中的回调
        mPlayer?.seekTo(seekBar.progress)
    }
```

- 全屏播放的实现，由于播放器是继承 `LinearLayout`，通过设置 `layoutParams` 达到全屏效果，记得一开始需要保存原始的 `params`，用于再次恢复正常模式。

```kotlin
    //进入全屏模式
    mPlayMode = PlayMode.MODE_FULL_SCREEN
    // 隐藏ActionBar、状态栏，并横屏
    (mContext as AppCompatActivity).supportActionBar?.hide()
    mContext.window.setFlags(
        WindowManager.LayoutParams.FLAG_FULLSCREEN,
        WindowManager.LayoutParams.FLAG_FULLSCREEN
    )
    mContext.requestedOrientation = ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE
    //设置为充满父布局
    val params = LinearLayout.LayoutParams(
        ViewGroup.LayoutParams.MATCH_PARENT,
        ViewGroup.LayoutParams.MATCH_PARENT
    )
    //隐藏虚拟按键，并且全屏
    mContext.window.decorView.systemUiVisibility =
        View.SYSTEM_UI_FLAG_HIDE_NAVIGATION or View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY or View.SYSTEM_UI_FLAG_FULLSCREEN
    //设置播放器为全屏
    (mView as LinearLayout).layoutParams = params
```

- 滑动改变进度、音量和亮度调节的实现，通过对整个播放器的根部局的 `setOnTouchListener` 进行滑动监听，在不同的位置滑动对播放器进行控制即可。`AudioManager` 的使用可以百度，具体代码可以移步项目地址，这里不再赘述。

```kotlin
    //亮度调节
    val params = (mContext as AppCompatActivity).window.attributes
    params.screenBrightness = light / 255f
    mContext.window.attributes = params
    // 音量调节
    mAudioManager?.setStreamVolume(AudioManager.STREAM_MUSIC, volume, 0)
```


**实现的界面**

![普通模式](/img/android/android_ijkplayer/3.jpg)

![全屏模式](/img/android/android_ijkplayer/4.jpg)


至此，一个简单的有功能的播放器实现了，可能或多或少有待改进的地方，后续仍然会进行优化，欢迎批评指正。

**项目地址：[Jvideoview](https://github.com/purejiang/Jvideoview)**

**参考文章：**
- [用 MediaPlayer + TextureView 封装一个完美实现全屏、小窗口的视频播放器](https://www.jianshu.com/p/420f7b14d6f6)
- [Android 多媒体：TextureView + MediaPlayer 实现视频播放器](https://www.jianshu.com/p/4a37ca48b16e)