# NIMSDK SDK

## 概述

[网易云信](http://netease.im)是由网易发布的一款 `IM` 云服务产品。此仓库是云信 `Android SDK`  的发布仓库。

## SDK 结构


libs 文件夹中，包含了网易云通信的 jar 文件，各 jni 库文件夹以及 SDK 依赖的第三方库。

如果需用网易云通信 SDK 提供的所有功能，将这些文件拷贝到你的工程的 libs 目录下，即可完成配置。列表如下：

```
libs
├── arm64-v8a
│   ├── libne_audio.so （高清语音录制功能必须）
│   ├── libnrtc_engine.so （音视频需要）
│   └── libnrtc_network.so （音视频需要）
│   └── librts_network.so （实时会话服务需要）
├── armeabi-v7a
│   ├── libne_audio.so
│   ├── libnrtc_engine.so
│   └── libnrtc_network.so
│   └── librts_network.so
├── x86
│   ├── libne_audio.so
│   ├── libnrtc_engine.so
│   └── libnrtc_network.so
│   └── librts_network.so
├── x86_64
│   ├── libne_audio.so
│   ├── libnrtc_engine.so
│   └── libnrtc_network.so
│   └── librts_network.so
│
├── nim-basesdk-6.7.0.jar （基础功能）
├── nim-chatroom-6.7.0.jar （聊天室需要）
├── nim-rts-6.7.0.jar （实时会话、文档转码需要）
├── nim-avchat-6.7.0.jar （音视频需要）
├── nim-lucene-6.7.0.jar （全文检索需要）
├── nrtc-sdk.jar（音视频需要）
```

以上文件列表中，jar 文件版本号可能会不同，子目录中的文件是 SDK 所依赖的各个 CPU 架构的 so 库。


> 按需配置 jar 包： 如果不需要聊天室功能，可以去掉 nim-chatroom-6.7.0.jar。 如果只需要 IM 基础功能和 音视频功能，可以去掉 nim-chatroom-6.7.0.jar，so 库需要加上 libnrtc*.so，还需加上 nim-avchat-6.7.0.jar 和 nrtc-sdk.jar。 如果不需要全文检索功能，可以去掉 nim-lucene-6.7.0.jar（该包有 1M+ 大小，如果没有用到消息全文检索功能，建议去掉）。

如果你使用的 IDE 是 Android Studio，要将 jni 库按照 IDEA 工程目录的结构，放置在对应的目录中（一般为 src/main/jniLibs）。或者在 build.gradle 中配置好 jniLibs 的 sourceSets（可参考 demo 的 build.gradle）。

## 集成

你可以通过[官网下载地址](http://netease.im/im-sdk-demo)下载最新版本，并添加到工程中，具体步骤参考[集成文档](http://dev.netease.im/docs/product/IM%E5%8D%B3%E6%97%B6%E9%80%9A%E8%AE%AF/SDK%E5%BC%80%E5%8F%91%E9%9B%86%E6%88%90/Android%E5%BC%80%E5%8F%91%E9%9B%86%E6%88%90/%E6%A6%82%E8%A6%81%E4%BB%8B%E7%BB%8D)。
