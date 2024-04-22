
耗时一个多星期终于基于这个版本一通修改编译出来了。目前主要是编译的Android arm64的。

[arm64-v8a] Compile : ijkj4a <= j4a_base.c
[arm64-v8a] Compile : ijkj4a <= AudioTrack.c
[arm64-v8a] Compile : ijkj4a <= MediaCodec.c
[arm64-v8a] Compile : ijkj4a <= PlaybackParams.c
[arm64-v8a] Compile : ijkj4a <= MediaFormat.c
[arm64-v8a] Compile : ijkj4a <= Build.c
[arm64-v8a] Compile : ijkj4a <= Buffer.c
[arm64-v8a] Compile : ijkj4a <= ByteBuffer.c
[arm64-v8a] Compile : ijkj4a <= Bundle.c
[arm64-v8a] Compile : ijkj4a <= ArrayList.c
[arm64-v8a] Compile : ijkj4a <= IMediaDataSource.c
[arm64-v8a] Compile : ijkj4a <= IAndroidIO.c
[arm64-v8a] Compile : ijkj4a <= AudioTrack.util.c
[arm64-v8a] Compile : ijkj4a <= ByteBuffer.util.c
[arm64-v8a] Compile : android-ndk-profiler <= prof.c
[arm64-v8a] Compile : ijkj4a <= IjkMediaPlayer.c
[arm64-v8a] Compile++ : ijksoundtouch <= AAFilter.cpp
[arm64-v8a] Compile++ : ijksoundtouch <= FIFOSampleBuffer.cpp
[arm64-v8a] Compile++ : ijksoundtouch <= cpu_detect_x86.cpp
[arm64-v8a] Compile++ : ijksoundtouch <= sse_optimized.cpp
[arm64-v8a] Compile++ : ijksoundtouch <= FIRFilter.cpp
[arm64-v8a] Compile++ : ijksoundtouch <= RateTransposer.cpp
[arm64-v8a] Compile++ : ijksoundtouch <= InterpolateCubic.cpp
[arm64-v8a] Compile++ : ijksoundtouch <= InterpolateShannon.cpp
[arm64-v8a] Compile++ : ijksoundtouch <= InterpolateLinear.cpp
[arm64-v8a] Compile++ : ijksoundtouch <= BPMDetect.cpp
[arm64-v8a] Compile++ : ijksoundtouch <= TDStretch.cpp
/home/cp/androidsoft/mijkplayer/android/ijkplayer/ijkplayer-arm64/src/main/jni/ijkmedia/ijksoundtouch/source/SoundTouch/TDStretch.cpp:66:13: warning: unused variable '_scanOffsets' [-Wunused-const-variable]
const short _scanOffsets[5][24]={
^
1 warning generated.
[arm64-v8a] Compile++ : ijksoundtouch <= PeakFinder.cpp
[arm64-v8a] Compile++ : ijksoundtouch <= mmx_optimized.cpp
[arm64-v8a] Compile++ : ijksoundtouch <= SoundTouch.cpp
[arm64-v8a] StaticLibrary : libcpufeatures.a
[arm64-v8a] StaticLibrary : libyuv_static.a
[arm64-v8a] Compile++ : ijksoundtouch <= ijksoundtouch_wrap.cpp
[arm64-v8a] StaticLibrary : libijkj4a.a
[arm64-v8a] StaticLibrary : libijksoundtouch.a
[arm64-v8a] StaticLibrary : libandroid-ndk-profiler.a
[arm64-v8a] SharedLibrary : libijksdl.so
[arm64-v8a] Install : libijksdl.so => libs/arm64-v8a/libijksdl.so
[arm64-v8a] SharedLibrary : libijkplayer.so
[arm64-v8a] Install : libijkplayer.so => libs/arm64-v8a/libijkplayer.so
/home/cp/androidsoft/mijkplayer/android

1.升级了opennssl：
IJK_OPENSSL_UPSTREAM=https://github.com/Bilibili/openssl.git
IJK_OPENSSL_FORK=https://gitee.com/mirrors/openssl.git
IJK_OPENSSL_COMMIT=OpenSSL_1_1_1w

ndk使用ndk21，会报找不到xxx-ar、xxx-runlib等ndk'编译脚本找不到，直接去ndk目录找对应的llvm脚本复制改文件名即可；也可以使用更高的版本，但是高版本llvm编译，需要大量修改编译脚本，本着能编译将就使用原则，没去尝试；
3.ijkplayer在sh compile-ijk.sh arm64 编译时会报错，ijkplayer里面的c代码和makefile都需要修改：
3.1其中一个注册协议解析的报错需要修改 ffmpeg6.1的源代码，参照这个 https://github.com/bilibili/ijkplayer/issues/3350；
3.2 ffmpeg6.1 接口有一些变更，需要同步修改 android\ijkplayer\ijkplayer-arm64\src\main\jni 源代码；
3.3 APP_STL := stlport_static 高版本ndk已废弃，需要修改APP_STL := c++_static；
3.4 在 mijkplayer/android/ijkplayer/ijkplayer-arm64/src/main/jni/ijkmedia/ijkplayer目录下的makefile文件的这个标记不能使用： LOCAL_CFLAGS += -std=c99；会报 error: invalid argument '-std=c99' not allowed with 'C++' 错误，因为mijkplayer/android/ijkplayer/ijkplayer-arm64/src/main/jni/ijkmedia/ijkplayer/ijkavutil/ijkstl.cpp 这个是c++语法文件，若有c99标记，编译器编译时无法兼容成c的语法。
3.5 在编译Android arm64的 ffmpeg时会报一个 ff_tx_codelet_list_float_x86 undefined的错误，这个应该是需要屏蔽不同abi耦合的，没有细究，不清楚ffmpeg编译为啥会出这问题，我的做法简单粗暴，直接在对应的源文件里添加一个对空函数体的对应函数名；

4.参照[本](https://github.com/1976222027/mijkplayer.git)项目作者的共享版本以及https://github.com/ShikinChen/ijkplayer-android 作者的静态库版本修改适配的ffmpeg6.1。

虽然ijk官方好久没更新了，希望大家群策群力，相互帮助，把ijk盘起来！若有需要，可以站内联系，或者建一个维护群。
另外，上面几楼的问题，在这里统一答复一下：修改ffmpeg的配置后，需要删除对应的abi目录下的config.h,这个就是根据脚本配置文件动态生成的控制ffmpeg里面相关所有子功能的c代码配置文件，每次改配置脚本需要重新生成；




# ijkplayer

 Platform | Build Guide
 -------- | ------------
 Android | [编译指南](build-android.md)
 iOS | [编译指南](buld-ios.md)

Video player based on [ffplay](http://ffmpeg.org)

### Download

- Android:
 - Gradle
```
# required
allprojects {
    repositories {
        maven {url 'https://jitpack.io'}
    }
}

dependencies {
    # required, enough for most devices.
    compile 'com.gitee.mahongyin:ijkplayer-java:0.1.0'
    compile 'com.gitee.mahongyin:ijkplayer-armv7a:0.1.0'

    # Other ABIs: optional
    compile 'com.gitee.mahongyin:ijkplayer-arm64:0.1.0'
    compile 'com.gitee.mahongyin:ijkplayer-x86:0.1.0'
    compile 'com.gitee.mahongyin:ijkplayer-x86_64:0.1.0'

    # ExoPlayer as IMediaPlayer: optional, experimental
    compile 'com.gitee.mahongyin:ijkplayer-exo:0.1.0'
    compile 'com.gitee.mahongyin:ijkplayer-exo2:0.1.0'
}
```
- iOS
 - in coming...

### My Build Environment
- Common
 - Mac OS X 10.11.5
- Android
 - [NDK r10e](http://developer.android.com/tools/sdk/ndk/index.html)
 - Android Studio 2.1.3
 - Gradle 2.14.1
- iOS
 - Xcode 7.3 (7D175)
- [HomeBrew](http://brew.sh)
 - ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
 - brew install git

### Latest Changes
- [NEWS.md](NEWS.md)

### Features
- Common公共
 - remove rarely used ffmpeg components to reduce binary size [config/module-lite.sh](config/module-lite.sh)
 - 删除很少使用的 ffmpeg 组件以减少二进制大小 [configmodule-lite.sh](configmodule-lite.sh)
 - workaround for some buggy online video.
 - 一些有问题的在线视频的解决方法。
- Android
 - platform: API 16~34
 - 平台：API 16~34
 - cpu: ARMv7a, ARM64v8a, x86 x86_64
 - api: [MediaPlayer-like](android/ijkplayer/ijkplayer-java/src/main/java/tv/danmaku/ijk/media/player/IMediaPlayer.java)
 - video-output: NativeWindow, OpenGL ES 2.0
 - 视频输出：NativeWindow、OpenGL ES 2.0
 - audio-output: AudioTrack, OpenSL ES
 - 音频输出：AudioTrack、OpenSL ES
 - hw-decoder: MediaCodec (API 16+, Android 4.1+)
 - 硬件解码器：MediaCodec（API 16+、Android 4.1+）
 - alternative-backend: android.media.MediaPlayer, ExoPlayer
 - 替代后端：android.media.MediaPlayer、ExoPlayer
- iOS
 - platform: iOS 7.0~17.x
 - iOS - 平台：iOS 7.0~17.x
 - cpu: armv7, arm64, i386, x86_64, (armv7s is obselete)
 - api: [MediaPlayer.framework-like](ios/IJKMediaPlayer/IJKMediaPlayer/IJKMediaPlayback.h)
 - video-output: OpenGL ES 2.0
 - audio-output: AudioQueue, AudioUnit
 - hw-decoder: VideoToolbox (iOS 8+)
 - alternative-backend: AVFoundation.Framework.AVPlayer, MediaPlayer.Framework.MPMoviePlayerControlelr (obselete since iOS 8)
    - cpu ：armv7、arm64、i386、x86_64、（armv7s 已过时） - api：[MediaPlayer.framework-like](iosIJKMediaPlayerIJKMediaPlayerIJKMediaPlayback.h) - 视频输出：OpenGL ES 2.0 - 音频输出：AudioQueue、AudioUnit - 硬件解码器： VideoToolbox (iOS 8+) - 替代后端：AVFoundation.Framework.AVPlayer、MediaPlayer.Framework.MPMoviePlayerControlelr（自 iOS 8 起已废弃）
### NOT-ON-PLAN
- obsolete platforms (Android: API-8 and below; iOS: pre-6.0)
- obsolete cpu: ARMv5, ARMv6, MIPS (I don't even have these types of devices…)
- native subtitle render
- avfilter support

### Before Build
```
# install homebrew, git, yasm
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install git
brew install yasm

# add these lines to your ~/.bash_profile or ~/.profile
# export ANDROID_SDK=<your sdk path>
# export ANDROID_NDK=<your ndk path>

# on Cygwin (unmaintained)
# install git, make, yasm
```

- If you prefer more codec/format
```
cd config
rm module.sh
ln -s module-default.sh module.sh
cd android/contrib
# cd ios
sh compile-ffmpeg.sh clean
```

- If you prefer less codec/format for smaller binary size (include hevc function)
```
cd config
# 删除默认的解码器
rm module.sh
# 创建一个软连接指向 module-lite-hevc.sh，这个可根据自己需求进行选择
ln -s module-lite-hevc.sh module.sh
cd android/contrib
# cd ios
sh compile-ffmpeg.sh clean
```

- If you prefer less codec/format for smaller binary size (by default)
```
cd config
rm module.sh
ln -s module-lite.sh module.sh
cd android/contrib
# cd ios
sh compile-ffmpeg.sh clean
```

- For Ubuntu/Debian users.
```
# choose [No] to use bash
sudo dpkg-reconfigure dash
```

- If you'd like to share your config, pull request is welcome.

### Build Android
```
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android
cd ijkplayer-android
git checkout -B latest k0.8.8

./init-android.sh

cd android/contrib
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all

cd ..
./compile-ijk.sh all

# Android Studio:
#     Open an existing Android Studio project
#     Select android/ijkplayer/ and import
#
#     define ext block in your root build.gradle
#     ext {
#       compileSdkVersion = 23       // depending on your sdk version
#       buildToolsVersion = "23.0.0" // depending on your build tools version
#
#       targetSdkVersion = 23        // depending on your sdk version
#     }
#
# If you want to enable debugging ijkplayer(native modules) on Android Studio 2.2+: (experimental)
#     sh android/patch-debugging-with-lldb.sh armv7a
#     Install Android Studio 2.2(+)
#     Preference -> Android SDK -> SDK Tools
#     Select (LLDB, NDK, Android SDK Build-tools,Cmake) and install
#     Open an existing Android Studio project
#     Select android/ijkplayer
#     Sync Project with Gradle Files
#     Run -> Edit Configurations -> Debugger -> Symbol Directories
#     Add "ijkplayer-armv7a/.externalNativeBuild/ndkBuild/release/obj/local/armeabi-v7a" to Symbol Directories
#     Run -> Debug 'ijkplayer-example'
#     if you want to reverse patches:
#     sh patch-debugging-with-lldb.sh reverse armv7a
#
# Eclipse: (obselete)
#     File -> New -> Project -> Android Project from Existing Code
#     Select android/ and import all project
#     Import appcompat-v7
#     Import preference-v7
#
# Gradle
#     cd ijkplayer
#     gradle

```


### Build iOS
```
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-ios
cd ijkplayer-ios
git checkout -B latest k0.8.8

./init-ios.sh

cd ios
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all

# Demo
#     open ios/IJKMediaDemo/IJKMediaDemo.xcodeproj with Xcode
# 
# Import into Your own Application
#     Select your project in Xcode.
#     File -> Add Files to ... -> Select ios/IJKMediaPlayer/IJKMediaPlayer.xcodeproj
#     Select your Application's target.
#     Build Phases -> Target Dependencies -> Select IJKMediaFramework
#     Build Phases -> Link Binary with Libraries -> Add:
#         IJKMediaFramework.framework
#
#         AudioToolbox.framework
#         AVFoundation.framework
#         CoreGraphics.framework
#         CoreMedia.framework
#         CoreVideo.framework
#         libbz2.tbd
#         libz.tbd
#         MediaPlayer.framework
#         MobileCoreServices.framework
#         OpenGLES.framework
#         QuartzCore.framework
#         UIKit.framework
#         VideoToolbox.framework
#
#         ... (Maybe something else, if you get any link error)
# 
```


### Support (支持) ###
- Please do not send e-mail to me. Public technical discussion on github is preferred.
- 请尽量在 github 上公开讨论[技术问题](https://github.com/bilibili/ijkplayer/issues)，不要以邮件方式私下询问，恕不一一回复。


### License

```
Copyright (c) 2017 Bilibili
Licensed under LGPLv2.1 or later
```

ijkplayer required features are based on or derives from projects below:
- LGPL
  - [FFmpeg](http://git.videolan.org/?p=ffmpeg.git)
  - [libVLC](http://git.videolan.org/?p=vlc.git)
  - [kxmovie](https://github.com/kolyvan/kxmovie)
  - [soundtouch](http://www.surina.net/soundtouch/sourcecode.html)
- zlib license
  - [SDL](http://www.libsdl.org)
- BSD-style license
  - [libyuv](https://code.google.com/p/libyuv/)
- ISC license
  - [libyuv/source/x86inc.asm](https://code.google.com/p/libyuv/source/browse/trunk/source/x86inc.asm)

android/ijkplayer-exo is based on or derives from projects below:
- Apache License 2.0
  - [ExoPlayer](https://github.com/google/ExoPlayer)

android/example is based on or derives from projects below:
- GPL
  - [android-ndk-profiler](https://github.com/richq/android-ndk-profiler) (not included by default)

ios/IJKMediaDemo is based on or derives from projects below:
- Unknown license
  - [iOS7-BarcodeScanner](https://github.com/jpwiddy/iOS7-BarcodeScanner)

ijkplayer's build scripts are based on or derives from projects below:
- [gas-preprocessor](http://git.libav.org/?p=gas-preprocessor.git)
- [VideoLAN](http://git.videolan.org)
- [yixia/FFmpeg-Android](https://github.com/yixia/FFmpeg-Android)
- [kewlbear/FFmpeg-iOS-build-script](https://github.com/kewlbear/FFmpeg-iOS-build-script) 

### Commercial Use
ijkplayer is licensed under LGPLv2.1 or later, so itself is free for commercial use under LGPLv2.1 or later

But ijkplayer is also based on other different projects under various licenses, which I have no idea whether they are compatible to each other or to your product.

[IANAL](https://en.wikipedia.org/wiki/IANAL), you should always ask your lawyer for these stuffs before use it in your product.
