# 网易云信安卓SDK开发指南

## <span id="SDK 概述">SDK 概述</span>

网易云信 SDK 为移动应用提供一个完善的 IM 系统开发框架，屏蔽掉 IM 系统的复杂细节，对外提供较为简洁的 API 接口，方便第三方应用快速集成 IM 功能。SDK 提供的能力如下：

- 网络连接管理
- 登录认证服务
- 消息传输服务
- 消息存储服务
- 消息检索服务
- 群组管理/聊天室/会话服务
- 用户资料、好友关系托管服务
- 实时语音视频通话
- 实时会话（白板）服务

## <span id="开发准备">开发准备</span>

首先从[网易云信官网](http://netease.im/im-sdk-demo "target=_blank")下载 Android SDK。开发者可以根据实际需求，配置类库。

注：SDK 最低要求 Android 2.3, 其中网络音视频通话和白板最低要求 Android 4.0。

### <span id="类库配置">类库配置（2.5以下版本）</span>

 SDK 包的libs文件夹中，包含了网易云信的 jar 文件，各 jni 库文件夹以及 SDK 依赖的第三方库，列表如下：
 
```
libs
├── armeabi
│   ├── libne_audio.so （高清语音录制功能必须）
│   └── libcosine.so （Android 后台保活需要）
│   ├── libnrtc_engine.so （音视频需要）
│   └── libnrtc_network.so （音视频需要）
│   └── librts_network.so （实时会话服务需要）
├── armeabi-v7a
│   ├── libne_audio.so
│   └── libcosine.so
│   ├── libnrtc_engine.so
│   └── libnrtc_network.so
│   └── librts_network.so
├── x86
│   ├── libne_audio.so
│   └── libcosine.so
│   ├── libnrtc_engine.so
│   └── libnrtc_network.so
│   └── librts_network.so
├── nim-sdk-1.0.0.jar
├── nrtc-sdk.jar（音视频需要）
└── cosinesdk.jar (Android 后台保活需要)
```

将这些文件拷贝到你的工程的 libs 目录下，即可完成配置。

以上文件列表中，nim-sdk-1.0.0.jar (版本号可能会不同)为网易云信 SDK，子目录中的文件是 SDK 所依赖的各个 CPU 架构的 so 库。

> 注意：如果你只需要 SDK 的基础功能（不含音视频及实时会话服务），则 so 库只需要 libne_audio.so 和 libcosine.so 两个，没有 libnrtc\*.so、librts\*.so，可以去掉 nrtc-sdk.1.0.0.jar；
如果需要音视频功能，so 库需要加上 libnrtc\*.so，还需加上 nrtc-sdk.jar；
如果需要实时会话（白板）服务，so 库需要加上librts\*.so；
如果不需要安卓保活功能，可以去掉 libcosine.so 和 cosinesdk.jar ( AndroidManifest.xml 文件中相关的安卓保活的配置需要删去)。 

> 特别说明：目前 Android 系统存在 armeabi armeabi-v7a x86 mips arm64-v8a x86_64 mips64 等多种Native库支持。
云信 SDK 目前只提供其中常用的 armeabi-v7a armeabi x86 三种32位库文件，但是可以以32位兼容模式运行在64位设备上，不会存在功能上的异常现象。当云信 SDK 运行在64位设备上时，应确保没有
arm64-v8a x86_64 库存在，否则 Android 系统将使用64位模式启动程序，而云信 SDK 目前不支持此种模式。详细说明见[云信Android SDK 64位库兼容问题说明](http://bbs.netease.im/read-tid-267 "target=_blank")

如果你的 APP 的 libs 里面只包含 armeabi 一个文件夹，为了保证在 arm-v7a 上有较好的性能，以及兼容各个平台，可将各目录下的 so 文件名改为原文件名加上"_{arch_of_cpu}"，然后统一放到 armeabi 目录下，SDK 也会加载到正确版本的so库。改名后的目录结构如下：

```
libs
├── armeabi
│   ├── libne_audio_armeabi.so
│   ├── libcosine_armeabi.so
│   ├── libnrtc_engine_armeabi.so
│   ├── libnrtc_network_armeabi.so
│   ├── librts_network_armeabi.so
│   ├── libne_audio_armeabi-v7a.so
│   ├── libcosine_armeabi-v7a.so
│   ├── libnrtc_engine_armeabi-v7a.so
│   └── libnrtc_network_armeabi-v7a.so
│   ├── librts_network_armeabi-v7a.so
├── x86(如果不考虑支持x86，可不包含)
│   ├── libne_audio_x86.so
│   ├── libcosine_x86.so
│   ├── libnrtc_engine_x86.so
│   └── libnrtc_network_x86.so
│   ├── librts_network_x86.so
├── nim-sdk-1.0.0.jar
├── nrtc-sdk.jar
└── cosinesdk.jar
```

如果你使用的 IDE 是 Android Studio，要将 jni 库按照 IDEA 工程目录的结构，放置在对应的目录中（一般为 src/main/jniLibs）。或者在 build.gradle 中配置好 jniLibs 的 sourceSets（可参考 demo 的 build.gradle）。

### 类库配置（2.5及以上版本）

网易云信 Android SDK v2.5及以上分为两种 SDK 包下载，第一种包含全部功能：IM + 聊天室 + 实时音视频 + 教学白板。第二种包含部分功能，包含：IM + 聊天室。

 SDK 包的libs文件夹中，包含了网易云信的 jar 文件，各 jni 库文件夹以及 SDK 依赖的第三方库。
 
第一种，包含全部功能的 SDK 包。如果需用云信 SDK 提供的所有功能，将这些文件拷贝到你的工程的 libs 目录下，即可完成配置。列表如下：

```
libs
├── armeabi
│   ├── libne_audio.so （高清语音录制功能必须）
│   └── libcosine.so （Android 后台保活需要）
│   ├── libnrtc_engine.so （音视频需要）
│   └── libnrtc_network.so （音视频需要）
│   └── librts_network.so （实时会话服务需要）
├── armeabi-v7a
│   ├── libne_audio.so
│   └── libcosine.so
│   ├── libnrtc_engine.so
│   └── libnrtc_network.so
│   └── librts_network.so
├── x86
│   ├── libne_audio.so
│   └── libcosine.so
│   ├── libnrtc_engine.so
│   └── libnrtc_network.so
│   └── librts_network.so
├── nim-basesdk-1.0.0.jar （基础功能）
├── nim-chatroom-1.0.0.jar （聊天室需要）
├── nim-rts-1.0.0.jar （实时会话服务需要）
├── nim-avchat-1.0.0.jar （音视频需要）
├── nim-lucene-1.0.0.jar （全文检索需要）
├── nrtc-sdkjar（音视频需要）
└── cosinesdk.jar (Android 后台保活需要)
```

第二种，只包含 IM 基础功能和聊天室功能的 SDK 包。如果只需要 IM 基础功能和聊天室功能，只需要将下面这些文件拷贝到你的工程的 libs 目录下，即可完成配置。列表如下：

```
libs
├── armeabi
│   ├── libne_audio.so （高清语音录制功能必须）
│   └── libcosine.so （Android 后台保活需要）
├── armeabi-v7a
│   ├── libne_audio.so
│   └── libcosine.so
├── x86
│   ├── libne_audio.so
│   └── libcosine.so
├── nim-basesdk-1.0.0.jar （基础功能）
├── nim-chatroom-1.0.0.jar （聊天室需要）
└── cosinesdk.jar (Android 后台保活需要)
```

以上文件列表中，jar文件版本号可能会不同，子目录中的文件是 SDK 所依赖的各个 CPU 架构的 so 库。

> 按需配置jar包：
如果不需要聊天室功能，可以在IM和聊天室的基础包中，去掉nim-chatroom-1.0.0.jar。
如果只需要 IM 基础功能和 音视频功能，可以在IM和聊天室的基础包中，去掉nim-chatroom-1.0.0.jar，so 库需要加上 libnrtc\*.so，还需加上nim-avchat-1.0.0.jar 和 nrtc-sdk.jar；
如果不需要安卓保活功能，可以去掉 libcosine.so 和 cosinesdk.jar ，以及 assets 文件夹中的cosine文件夹( AndroidManifest.xml 文件中相关的安卓保活的配置需要删去)。 
如果不需要全文检索功能，可以去掉 nim-lucene-1.0.0.jar（该包有 1M+ 大小，如果没有用到消息全文检索功能，建议去掉）。

***其他配置及相关说明，请参考上一节***

### <span id="通过Gradle集成SDK">通过Gradle集成SDK</span>

网易云信 Android SDK 2.5.0以上支持通过 Gradle 集成 SDK。

首先，在整个工程的 build.gradle 文件中，配置repositories，使用 jcenter 或者 maven ，二选一即可，如下：

```
allprojects {
    repositories {
        jcenter() // 或者 mavenCentral()
    }
}
```

第二步，在主工程的 build.gradle 文件中，添加 dependencies。根据自己项目的需求，添加不同的依赖即可。注意：版本号必须一致，这里以1.0.0版本为例：

```
dependencies {
    compile fileTree(dir: 'libs', include: '*.jar')
    // 添加依赖。注意，版本号必须一致。
    // 基础功能 (必需)
    compile 'com.netease.nimlib:basesdk:1.0.0'
    // 音视频需要
    compile 'com.netease.nimlib:avchat:1.0.0'
    // 聊天室需要
    compile 'com.netease.nimlib:chatroom:1.0.0'
    // 实时会话服务需要
    compile 'com.netease.nimlib:rts:1.0.0'
    // 全文检索服务需要
    compile 'com.netease.nimlib:lucene:1.0.0'
}
```

> 再次注意：依赖包的版本号必须一致。

### <span id="权限与组件">权限与组件</span>

在 `AndroidManifest.xml` 中加入以下配置:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	   package="xxx">

	<!-- 权限声明 -->
	<!-- 访问网络状态-->
	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
	<!-- 控制呼吸灯，振动器等，用于新消息提醒 -->
    <uses-permission android:name="android.permission.FLASHLIGHT" />
    <uses-permission android:name="android.permission.VIBRATE" />
    <!-- 外置存储存取权限 -->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

    <!-- 多媒体相关 -->
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>

    <!-- 如果需要实时音视频通话模块，下面的权限也是必须的。否则，可以不加 -->
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS"/>
    <uses-permission android:name="android.permission.BROADCAST_STICKY"/>
    <uses-feature android:name="android.hardware.camera" />
    <uses-feature android:name="android.hardware.camera.autofocus" />
    <uses-feature android:glEsVersion="0x00020000" android:required="true" />

    <!-- SDK 权限申明, 第三方 APP 接入时，请将 com.netease.nim.demo 替换为自己的包名 -->
    <!-- 和下面的 uses-permission 一起加入到你的 AndroidManifest 文件中。 -->
    <permission
        android:name="com.netease.nim.demo.permission.RECEIVE_MSG"
        android:protectionLevel="signature"/>
    <!-- 接收 SDK 消息广播权限， 第三方 APP 接入时，请将 com.netease.nim.demo 替换为自己的包名 -->
     <uses-permission android:name="com.netease.nim.demo.permission.RECEIVE_MSG"/>

    <application
        ...>
        <!-- APP key, 可以在这里设置，也可以在 SDKOptions 中提供。
            如果 SDKOptions 中提供了，取 SDKOptions 中的值。 -->
        <meta-data
            android:name="com.netease.nim.appKey"
            android:value="key_of_your_app" />

        <!-- 声明云信后台服务，如需保持后台推送，使用独立进程效果会更好。 -->
        <service 
            android:name="com.netease.nimlib.service.NimService"
            android:process=":core"/>
            
        <service
            android:name="com.netease.nimlib.service.NimService$Aux"
            android:process=":core"/>

        <!-- 云信SDK的监视系统启动和网络变化的广播接收器，用户开机自启动以及网络变化时候重新登录，
            保持和 NimService 同一进程 -->
        <receiver android:name="com.netease.nimlib.service.NimReceiver"
            android:process=":core"
            android:exported="false">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
            </intent-filter>
        </receiver>

        <!-- 云信进程间通信 Receiver -->
        <receiver android:name="com.netease.nimlib.service.ResponseReceiver"/>

        <!-- 安卓保活配置 -->
        <service
            android:name="com.netease.cosine.core.CosineService"
            android:process=":cosine">
        </service>

        <receiver
            android:name="com.netease.cosine.target.CosineReceiver"
            android:exported="true"
            android:process=":cosine">
        </receiver>

        <meta-data
            android:name="com.netease.cosine.target"
            android:value=""/>
        <meta-data
            android:name="com.netease.cosine.target.receiver"
            android:value="com.netease.nimlib.service.NimReceiver"/>

    </application>
</manifest>
```

### <span id="混淆配置">混淆配置</span>

如果你的 apk 最终会经过代码混淆，请在 proguard 配置文件中加入以下代码:

```
-dontwarn com.netease.**
-dontwarn io.netty.**
-keep class com.netease.** {*;}
#如果 netty 使用的官方版本，它中间用到了反射，因此需要 keep。如果使用的是我们提供的版本，则不需要 keep
-keep class io.netty.** {*;}

#如果你使用全文检索插件，需要加入
-dontwarn org.apache.lucene.**
-keep class org.apache.lucene.** {*;}

```

### <span id="总体接口介绍">总体接口介绍</span>

网易云信 SDK 提供了两类接口供开发者调用：一类是第三方 APP 主动发起请求，第二类是第三方 APP 作为观察者监听事件和变化。第一类接口名均以 **Service** 结尾，例如 `AuthService` ，第二类接口名均以 **ServiceObserver** 结尾，例如 `AuthServiceObserver`，个别太长的类名则可能直接以 **Observer** 结尾，比如 `SystemMessageObserver`。

可以通过 `NIMClient` 的 `getService` 接口获取到各个服务实例，例如：通过 `NIMClient.getService(AuthService.class)` 获取到 `AuthService` 服务实例，通过 `NIMClient.getService(AuthServiceObserver.class)` 获取到 `AuthServiceObserver` 观察者接口。

SDK 由于需要保持后台运行，典型场景下会在独立进程中运行，第一类接口基本上都是从主进程发起调用，然后在后台进程执行，最后再将结果返回给主进程。因此，如无特殊说明，所有接口均为异步调用，开发者无需担心调用 SDK 接口阻塞 UI 的问题。如果调用的接口需要一些后期操作，包括结果回调，取消调用等，此类接口会返回一个 `InvocationFuture` 对象。如果开发者关心调用的结果，可在此返回的接口中设置回调函数。
接口回调接口为 `RequestCallback` ，有3个接口需要实现：onSuccess, onFailed, onException，分别对应成功，失败，以及出现异常。另外还提供了一个更简洁的接口 `RequestCallbackWrapper`，作为 `RequestCallback` 的包裹，封装了上面的 3 个接口，然后将他们合并成一个 onResult 接口，在参数上做不同结果的区分。

还有一类接口，耗时会很长，可能需要传输大量数据，或者要发起网络连接，比如上传下载，登录等。对于这类接口，返回值会是一个 `AbortableFuture` 对象，该接口继承自 `InvocationFuture`，增加了一个 `abort()` 方法，可取消之前的请求。

观察者接口的方法名都是以 observe 开头，并包含一个 register 参数，该值为`true`时，为注册观察者，为`false`时，注销观察者。开发者要在不需要观察者时，主动注销，以免造成资源泄露。

> 注意：除了 `NIMClient.init` 接口外，其他 SDK 暴露的接口都只能在 UI 进程调用。如果 APP 包含远程 service，该 APP 的 Application 的 onCreate 会多次调用。因此，如果需要在 onCreate 中调用除 init 接口外的其他接口，应先判断当前所属进程，并只有在当前是 UI 进程时才调用。判断代码如下：

```java
public static boolean inMainProcess(Context context) {
    String packageName = context.getPackageName();
    String processName = SystemUtil.getProcessName(context);
    return packageName.equals(processName);
}

/**
 * 获取当前进程名
 * @param context
 * @return 进程名
 */
public static final String getProcessName(Context context) {
    String processName = null;

    // ActivityManager
    ActivityManager am = ((ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE));

    while (true) {
        for (ActivityManager.RunningAppProcessInfo info : am.getRunningAppProcesses()) {
            if (info.pid == android.os.Process.myPid()) {
                processName = info.processName;
                break;
            }
        }

        // go home
        if (!TextUtils.isEmpty(processName)) {
            return processName;
        }

        // take a rest and again
        try {
            Thread.sleep(100L);
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
    }
}
```

SDK 提供的接口主要按照业务进行分类，大致说明如下：
- `AuthService`：用户认证服务接口，提供登录注销接口。
- `AuthServiceObserve`: 用户状态观察者接口。
- `MsgService`: 消息服务接口，用于发送消息，管理消息记录等。同时还提供了发送自定义通知的接口。
- `MsgServiceObserve`: 接收消息，消息状态变化等观察者接口。
- `LuceneService`: 聊天消息全文检索接口。
- `TeamService`: 群组服务接口，用于发送群组消息，管理群组和群成员资料等。
- `TeamServiceObserve`: 群组和群成员资料变化观察者。
- `SystemMessageService`: 管理系统通知接口。
- `SystemMessageObserve`: 系统通知观察者。
- `FriendService`: 好友关系托管接口，目前支持添加、删除好友、获取好友列表、黑名单、设置消息提醒。
- `FriendServiceObserve`: 好友关系变更、黑名单变更通知观察者。
- `UserService`: 用户资料托管接口，提供获取用户资料、修改个人资料等。
- `UserServiceObserve`: 用户资料托管接口，提供获取用户资料、修改个人资料等。
- `AVChatManager`: 语音视频通话接口。
- `RTSManager`: 实时会话接口。
- `NosService`: 网易云存储服务，提供文件上传和下载。

### <span id="SDK 数据缓存目录结构">SDK 数据缓存目录结构</span>

当收到多媒体消息后，SDK 会负责下载这些多媒体文件，同时 SDK 还要记录一些关键的 log，因此 SDK 需要一个数据缓存目录。
该目录可以在 SDK 初始化时通过 `SDKOptions#sdkStorageRootPath` 进行设置。
如果不设置，则默认为“/{外卡根目录}/{app\_package\_name}/nim/”，其中外卡根目录获取方式为 Environment.getExternalStorageDirectory().getPath()。
如果你的 APP 需要清除缓存功能，可扫描该目录下的文件，按照你们的规则清理即可。 在 SDK 初始化完成后可以通过 `NimClient#getSdkStorageDirPath` 获取 SDK 数据缓存目录。

SDK数据缓存目录下面包含如下子目录：
- log: SDK日志包含一个文件：nim\_sdk.log，大小一般不超过 8M。音视频通话的日志包含3个文件：avchat\_n.log, nrtc\_engine.log, nrtc\_net.log。
默认路径为：**/{外卡根目录}/{app\_package\_name}/nim/log/**，如果在 SDKOptions 中配置过 sdkStorageRootPath，那么日志路径为“{sdkStorageRootPath}/log/”。
- file: 文件消息文件
- image: 图片消息文件
- audio：语音消息文件
- video：视频消息文件
- thumb：图片/视频缩略图文件

## <span id="初始化 SDK">初始化 SDK</span>

在你的程序的 Application 的 `onCreate` 中，加入网易云信 SDK 的初始化代码：

```
public class NimApplication extends Application {

	public void onCreate() {
		// ... your codes
		
		// SDK初始化（启动后台服务，若已经存在用户登录信息， SDK 将完成自动登录）
		NIMClient.init(this, loginInfo(), options());
		
		// ... your codes
		if (inMainProcess()) {
			// 注意：以下操作必须在主进程中进行
            // 1、UI相关初始化操作
            // 2、相关Service调用
        }
	}

	// 如果返回值为 null，则全部使用默认参数。
	private SDKOptions options() {
		SDKOptions options = new SDKOptions();

	    // 如果将新消息通知提醒托管给 SDK 完成，需要添加以下配置。否则无需设置。
	    StatusBarNotificationConfig config = new StatusBarNotificationConfig();
	    config.notificationEntrance = WelcomeActivity.class; // 点击通知栏跳转到该Activity
	    config.notificationSmallIconId = R.drawable.ic_stat_notify_msg;
	    // 呼吸灯配置
        config.ledARGB = Color.GREEN;
        config.ledOnMs = 1000;
        config.ledOffMs = 1500;
        // 通知铃声的uri字符串
        config.notificationSound = "android.resource://com.netease.nim.demo/raw/msg";
	    options.statusBarNotificationConfig = config;

	    // 配置保存图片，文件，log 等数据的目录
	    // 如果 options 中没有设置这个值，SDK 会使用下面代码示例中的位置作为 SDK 的数据目录。
	    // 该目录目前包含 log, file, image, audio, video, thumb 这6个目录。
	    // 如果第三方 APP 需要缓存清理功能， 清理这个目录下面个子目录的内容即可。
	    String sdkPath = Environment.getExternalStorageDirectory() + "/" + getPackageName() + "/nim";
	    options.sdkStorageRootPath = sdkPath;

	    // 配置是否需要预下载附件缩略图，默认为 true
	    options.preloadAttach = true;

	    // 配置附件缩略图的尺寸大小。表示向服务器请求缩略图文件的大小
	    // 该值一般应根据屏幕尺寸来确定， 默认值为 Screen.width / 2
	    options.thumbnailSize = ${Screen.width} / 2;

        // 用户资料提供者, 目前主要用于提供用户资料，用于新消息通知栏中显示消息来源的头像和昵称
        options.userInfoProvider = new UserInfoProvider() {
             @Override
             public UserInfo getUserInfo(String account) {
                 return null;
             }

        	 @Override
        	 public int getDefaultIconResId() {
            	 return R.drawable.avatar_def;
        	 }

             @Override
             public Bitmap getTeamIcon(String tid) {
                 return null;
             }

			 @Override
        	 public Bitmap getAvatarForMessageNotifier(String account) {
             	 return null;
        	 }

             @Override
             public String getDisplayNameForMessageNotifier(String account, String sessionId,
                SessionTypeEnum sessionType) {
                 return null;
             }
         };
	     return options;
	}

	// 如果已经存在用户登录信息，返回LoginInfo，否则返回null即可
    private LoginInfo loginInfo() {
	    return null;
	}
}
```

再次提醒，如果你的 APP 会启动多个进程，此处无需做特殊处理，在每次 Application 的 `onCreate` 调用初始化代码即可。SDK 会处理好进程关系，并只在需要的进程中初始化。但是，要注意不要在 SDK 进程中再调用各种 Service 提供的接口，也不要在 SDK 进程中做 UI 相关的初始化动作，以免造成资源浪费。判断当前进程是否是在主进程的代码见[总体接口介绍](#总体接口介绍)。

## <span id="登录与登出">登录与登出</span>

### <span id="手动登录">手动登录</span>

一般 APP 在首次登录、切换帐号登录、注销重登时需要手动登录，开发者需要调用 `AuthService` 提供的 `login` 接口主动发起登录请求。该接口返回类型为 `AbortableFuture`，允许用户在后面取消登录操作。如果服务器一直没有响应，30 秒后 RequestCallback 的 onFailed 会被调用，参数为 408 （网络连接超时）。

```java
public class LoginActivity extends Activity {
	public void doLogin() {
		LoginInfo info = new LoginInfo(); // config...
		RequestCallback<LoginInfo> callback =
			new RequestCallback<LoginInfo>() {
			// 可以在此保存LoginInfo到本地，下次启动APP做自动登录用
		};
		NIMClient.getService(AuthService.class).login(info)
				.setCallback(callback);
	}
}
```
登录成功后，可以将用户登录信息 LoginInfo 信息保存到本地，下次启动APP时，读取本地保存的 LoginInfo 进行自动登录。

说明：在手动登录过程中，如果网络断开或者与云信服务器建立连接失败，会返回登录失败（错误码 415），在线状态切换为 NET_BROKEN；
如果连接建立成功，SDK 发出登录请求后云信服务器一直没有响应，那么 30s 后将导致登录超时，那么会返回登录失败（错误码 408），在线状态切换为 UNLOGIN。

> 注意：从SDK 2.2.0版本开始， LoginInfo 中添加了可选属性 AppKey，支持在登录的时候设置 AppKey；如果不填，则优先使用 SDKOptions 中配置的 AppKey；如果也没有，则使用 AndroidManifest.xml 中配置的 AppKey（默认方式）。建议使用默认方式。

> 特别提醒: 登录成功之前，调用服务器相关请求接口（由于与云信服务器连接尚未建立成功，会导致发包超时）会报408错误；调用本地数据库相关接口（手动登录的情况下数据库未打开），会报1000错误，建议用户在登录成功之后，再进行相关接口调用。

### <span id="自动登录">自动登录</span>

如果上次登录已经存在用户登录信息，那么在初始化 SDK 时传入 `LoginInfo`，SDK 后台会自动登录，并在登录发起前即打开相关账号的数据库，供上层调用。开发者此时无需再手动调用登录接口，可以跳过登录界面直接进入主界面。

进入主界面后，可以通过监听用户在线状态（每次注册用户在线状态监听都会立即回调通知当前的用户在线状态），或者主动获取当前用户在线状态，来判断自动登录是否成功。

在初始化 SDK 时自动登录示例：

```java
public class NimApplication extends Application {

	public void onCreate() {
		// ... your codes

		// SDK初始化（启动后台服务，若已经存在用户登录信息，SDK 将完成自动登录）
		NIMClient.init(this, loginInfo(), options());

		// ... your codes
	}

	private LoginInfo loginInfo() {
	    // 从本地读取上次登录成功时保存的用户登录信息
        String account = Preferences.getUserAccount();
        String token = Preferences.getUserToken();

        if (!TextUtils.isEmpty(account) && !TextUtils.isEmpty(token)) {
            DemoCache.setAccount(account.toLowerCase());
            return new LoginInfo(account, token);
        } else {
            return null;
        }
    }
}
```
说明：在自动登录过程中，如果没有网络或者网络断开或者与云信服务器建立连接失败，会上报在线状态 NET_BROKEN，表示当前网络不可用，当网络恢复的时候，会触发断网自动重连；如果连接建立成功但登录超时，会上报在线状态 UNLOGIN，并触发自动重连，无需上层手动调用登录接口。

> 特别提醒: 在自动登录成功前，调用服务器相关请求接口（由于与云信服务器连接尚未建立成功，会导致发包超时）会报408错误。但可以调用本地数据库相关接口获取本地数据（自动登录的情况下会自动打开相关账号的数据库）。自动登录过程中也会有用户在线状态回调。

### <span id="监听用户在线状态">监听用户在线状态</span>

登录成功后，SDK 会负责维护与服务器的长连接以及断线重连等工作。当用户在线状态发生改变时，会发出通知。此外，自动登录过程中也会有状态回调。开发者可以通过加入以下代码监听用户在线状态改变：

```java
NIMClient.getService(AuthServiceObserver.class).observeOnlineStatus(
	new Observer<StatusCode> () {
		public void onEvent(StatusCode status) {
			Log.i("tag", "User status changed to: " + status);
			if (code.wontAutoLogin()) {
                // 被踢出、账号被禁用、密码错误等情况，自动登录失败，需要返回到登录界面进行重新登录操作
            }
		}
}, true);
```

被踢出的情况说明：
1. 当用户在线时被踢出，会立刻收到被踢出的状态变更通知；
2. 当用户离线后在其他设备成功登录，又在本设备重新自动登录时，也会收到被踢出的状态变更通知。

开发者也可以主动获取当前用户在线状态：

```java
StatusCode status = NIMClient.getStatus();
```

### <span id="数据同步状态通知">数据同步状态通知</span>

登录成功后，SDK 会立即同步数据（用户资料、用户关系、群资料、离线消息、漫游消息等），同步开始和同步完成都会发出通知。

注册登录同步状态通知：

```java
NIMClient.getService(AuthServiceObserver.class).observeLoginSyncDataStatus(new Observer<LoginSyncStatus>() {
    @Override
    public void onEvent(LoginSyncStatus status) {
        if (status == LoginSyncStatus.BEGIN_SYNC) {
            LogUtil.i(TAG, "login sync data begin");
        } else if (status == LoginSyncStatus.SYNC_COMPLETED) {
            LogUtil.i(TAG, "login sync data completed");
        }
    }
}, register);
```

同步开始时，SDK 数据库中的数据可能还是旧数据（如果是首次登录，那么 SDK 数据库中还没有数据，重新登录时 SDK 数据库中还是上一次退出时保存的数据）。

同步完成时， SDK 数据库已完成更新。

在同步过程中，SDK 数据的更新会通过相应的 XXXServiceObserver 接口发出数据变更通知。

一般来说， APP 开发者在登录完成后可以开始构建数据缓存：
- 登录完成后立即从 SDK 读取数据构建缓存，此时加载到的可能是旧数据；
- 在 Application 的 onCreate 中注册 XXXServiceObserver 来监听数据变化，那么在同步过程中， APP 会收到数据更新通知，此时直接更新缓存。当同步完成时，缓存也就构建完成了。

### <span id="多端登录">多端登录</span>

登录成功后，可以注册多端登录状态观察者。当有其他端登录或者注销时，会通过此接口通知到UI。登录成功后，如果有其他端登录着（在线），也会发出通知。返回的 `OnlineClient` 能够获取当前同时在线的客户端类型和操作系统。

```java
NIMClient.getService(AuthServiceObserver.class).observeOtherClients(new Observer<List<OnlineClient>>() { ... }, true);
```

如果需要主动踢掉当前同时在线的其他端, 需要传入 `OnlineClient`：

```java
NIMClient.getService(AuthService.class).kickOtherClient(onlineClient).setCallback(new RequestCallback<Void>() { ... });
```

当被其他端踢掉，可以通过在线状态观察者来接收监听消息：

```java
NIMClient.getService(AuthServiceObserver.class).observeOnlineStatus(
	new Observer<StatusCode> () {
		public void onEvent(StatusCode status) {
			// 判断在线状态，如果为被其他端踢掉，做登出操作
		}
}, true);
```

云信内置多端登录互踢策略为：移动端( Android 、 iOS )互踢，桌面端( PC 、 Web )互踢，移动端和桌面端共存（ 可以采用上述 kickOtherClient 主动踢下共存的其他端）。

如果当前的互踢策略无法满足业务需求的话，可以联系我们取消内置互踢，根据多端登录的回调和当前的设备列表，判断本设备是否需要被踢出。如果需要踢出，直接调用登出接口并在界面上给出相关提示即可。

### <span id="登出">登出</span>

如果用户手动登出，不再接收消息和提醒，开发者可以调用 logout 方法，该方法没有回调。

> 注意: 登出操作，不要放在 Activity(Fragment) 的 onDestroy 方法中。

```java
NIMClient.getService(AuthService.class).logout();
```

### <span id="离线查看数据">离线查看数据</span>

对于一些弱IM场景，需要在登录成功前或者未登录状态下访问指定账号的数据（聊天记录、好友资料等）。 SDK 提供两种方案：

1. 使用自动登录。在登录成功前，可以访问 SDK 服务来读取本地数据（但不能发送数据）。

2. 使用 AuthService#openLocalCache 接口打开本地数据，这是个同步方法，打开后即可读取 SDK 数据库中的记录。可以通过注销来切换账号查看本地数据。

```java
NIMClient.getService(AuthService.class).openLocalCache(account);
```

### <span id="断线重连机制">断线重连机制</span>

SDK 提供三种断线重连的策略（重新建立与云信服务器的连接并重新登录）：

1\. 当网络由连通变为断开时，SDK 会启动立即上报网络断开的状态，并启动重连定时器，采用特定的策略并根据当前网络状态进行重连（如果 APP 处于后台，重连时间间隔会较长）。

2\. SDK会监听设备的网络连接状况，当监听到手机断网重连上网络的通知后，会立即进行重连并登录。

3\. 应用长时间处于后台（后台进程可能活着但网络连接被系统切断）后切回到前台（恢复网络连通），SDK 监测到当前处于未登录状态，会在短时间内进行重连。

## <span id="基础消息功能">基础消息功能</span>

### <span id="消息功能概述">消息功能概述</span>

SDK 提供一套完善的消息传输管理服务，包括收发消息，存储消息，上传下载附件，管理最近联系人等。

SDK 原生支持发送文本，语音，图片，视频，文件，地理位置，提醒等 7 种类型消息，同时支持用户发送自定义的消息类型。

网易云信消息对象均为 `IMMessage`，不同消息类型以 `MsgTypeEnum` 作区分，消息内容根据类型不同也不一样。文本消息最为简单，消息内容就是 `content` 字符串。其他消息类型均带有一个消息附件对象 `MsgAttachment`，该对象在传输时一般序列化为 json 格式字符串。内建的消息附件主要有以下几种：
- LocationAttachment： 位置消息附件对象类型。
- FileAttachment:  文件消息附件对象类型（继承该附件， SDK 在发送消息时将自动上传文件）。
- ImageAttachment：图片消息附件对象类型。
- AudioAttachment：音频消息附件对象类型。
- VideoAttachment：视频消息附件对象类型。
- NotificationAttachment：通知类消息附件基类，目前主要用于群类各种操作事件的通知。该类型事件仅由服务器生成和发送，客户端不会生成该类型的消息。

### <span id="发送消息">发送消息</span>

发送原生支持的消息类型流程比较简单，先通过 `MessageBuilder` 提供的接口创建消息对象，然后调用 `MsgService` 的 `sendMessage` 接口发送出去即可。下面给出发送各种类型消息的示例代码：

```java
// 创建文本消息
IMMessage message = MessageBuilder.createTextMessage(
	sessionId, // 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID
	sessionType, // 聊天类型，单聊或群组
	content // 文本内容
	);
// 发送消息。如果需要关心发送结果，可设置回调函数。发送完成时，会收到回调。如果失败，会有具体的错误码。
NIMClient.getService(MsgService.class).sendMessage(message);

// 发送其他类型的消息代码类似，演示如下
// 创建地理位置消息
IMMessage message = MessageBuilder.createLocationMessage(
	sessionId, // 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID
	sessionType, // 聊天类型，单聊或群组
	latitude, // 纬度
	longitude, // 经度
	address // 地址信息描述
	);
NIMClient.getService(MsgService.class).sendMessage(message);

// 创建图片消息
IMMessage message = MessageBuilder.createImageMessage(
	sessionId, // 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID
	sessionType, // 聊天类型，单聊或群组
	file, // 图片文件对象
	displayName // 文件显示名字，如果第三方 APP 不关注，可以为 null
	);
NIMClient.getService(MsgService.class).sendMessage(message);

// 创建音频消息
IMMessage message = MessageBuilder.createAudioMessage(
    sessionId, // 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID
	sessionType, // 聊天类型，单聊或群组
    file, // 音频文件
    duration // 音频持续时间，单位是ms
    );
NIMClient.getService(MsgService.class).sendMessage(message);

// 创建视频消息
IMMessage message = MessageBuilder.createVideoMessage(
	sessionId, // 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID
	sessionType, // 聊天类型，单聊或群组
	file, // 视频文件
	duration, // 视频持续时间
	width, // 视频宽度
	height, // 视频高度
	displayName // 视频显示名，可为空
	);
NIMClient.getService(MsgService.class).sendMessage(message);

// 创建提醒消息（主要用于会话内的通知提醒，例如进入会话时出现的欢迎消息，
// 或是会话过程中命中敏感词后的提示消息等场景，也可以用自定义消息实现，但相对于Tip消息实现比较复杂）
// 注意：提醒消息不支持setAttachment（如果要使用Attachment请使用自定义消息）。
IMMessage message = MessageBuilder.createTipMessage(
	sessionId,   // 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID
	sessionType, // 聊天类型，单聊或群组
	);
NIMClient.getService(MsgService.class).sendMessage(message);
```

- 消息支持扩展字段，扩展字段分为服务器扩展字段（ RemoteExtension ）和本地扩展字段（ LocalExtension ），最大长度1024字节。对于服务器扩展字段，该字段会发送到其他端，而本地扩展字段仅在本地有效。对于这两种扩展字段， SDK 都会存储在数据库中。示例如下：

```java
IMMessage msg = MessageBuilder.createCustomMessage(...);
Map<String, Object> data = new HashMap<>();
data.put("key1", "ext data");
data.put("key2", 2015);
msg.setRemoteExtension(data); // 设置服务器扩展字段

NIMClient.getService(MsgService.class).sendMessage(msg, false);
```

利用扩展字段开发者可以实现一些特殊场景，例如群@功能：

1\. 当发送群消息@某人的时候，可以通过扩展字段带上被@的账号列表发送出去；群成员收到群消息时，查看扩展字段的@账号列表里有没有自己，如果有，则界面上做被@的提醒。

2\. 此外，最近会话列表项也提供扩展标签和扩展字段，可以满足开发者在最近会话列表中做@提醒的需求：
在监听到最新联系人有更新时，可以根据最近一条消息的 ID 查询出该条消息，查看扩展字段是否包含@账号列表，如果包含自己，可以在扩展字段上做一个标记，如果有新消息到来（未进入聊天界面查看时），要继续保持最近会话列表里面有@提醒，就可以参考扩展字段的标记，做界面的上被@的提醒。

3\. 1.8.0版本后消息提醒支持定制，开发者可以自行定制@消息在通知栏提醒时需要展示的文案。

- 发送消息时可以设置推送文案（最大长度200字节）和自定义推送属性（最大长度2048字节），请注意最大长度的限制，如果超过 SDK 将会抛出 IllegalArgumentException 。

设置了推送文案后，接收方收到消息时，在通知栏提醒中会显示该文案，如果不设置则采用 SDK 默认的文案（当然，通知栏提醒也可以由开发者自行实现）。

自定义推送属性，参数为 Map<String,Object> ，（ SDK 底层会将此 Map 转成 JsonObject 进行传输）， 接收方收到消息时，可以获取此 Map。示例如下：

```java
IMMessage msg = MessageBuilder.createCustomMessage(...);
msg.setPushContent("收到一条自定义消息");
Map<String, Object> data = new HashMap<>();
data.put("key1", "playload1");
data.put("key2", 2015);
msg.setPushPayload(data);

NIMClient.getService(MsgService.class).sendMessage(msg, false);
```

- 发送消息时可以设置消息配置选项 `CustomMessageConfig`，主要用于设定消息的声明周期，是否需要推送，是否需要计入未读数等，目前支持的配置选项有：

1\. enableHistory ：该消息是否允许在消息历史中拉取，如果为 false，通过 MsgService#pullMessageHistory 拉取的结果将不包含该条消息。默认为 true 。

2\. enableRoaming ：该消息是否需要漫游。如果为 false ，一旦某一个客户端收取过该条消息，其他端将不会再漫游到该条消息。默认为 true 。

3\. enableSelfSync ：多端同时登录时，发送一条消息后，是否要同步到其他同时登录的客户端。默认为 true 。

4\. enablePush ： 该消息是否进行推送（消息提醒）。默认为 true 。

5\. enablePushNick ： 该消息是否需要推送昵称（针对iOS客户端有效），如果为false，那么对方收到消息后，iOS端将不显示推送昵称。默认为 true 。

6\. enableUnreadCount ：该消息是否要计入未读数，如果为 true ，那么对方收到消息后，最近联系人列表中未读数加1。默认为 true 。

7\. enableRoute ：该消息是否支持路由，如果为 true，默认按照 app 的路由开关（如果有配置抄送地址则将抄送该消息）。默认为 true 。

示例如下：

```java
IMMessage msg = MessageBuilder.createCustomMessage(...);
CustomMessageConfig config = new CustomMessageConfig();
config.enableUnreadCount = false; // 该消息不计入未读数
msg.setConfig(config);

NIMClient.getService(MsgService.class).sendMessage(msg, false);
```

- 发送消息后，还需要持续关注消息的发送进度，以更新发送界面。发送消息的接口可以设置回调函数，并在发送完成时回调，通知上层消息发送结果（如果出错会返回具体错误码），该回调函数没有发送状态变化通知和进度通知。
如果需要，开发者需要通过观察者接口来完成。这样，开发者在任意界面都能接收到消息状态的改变。示例代码如下：

```java
// 监听消息发送状态的变化通知
NIMClient.getService(MsgServiceObserve.class).observeMsgStatus(
	new Observer<IMMessage> {
		public void onEvent(IMMessage message) {
			// 参数为有状态发生改变的消息对象，其 msgStatus 和 attachStatus 均为最新状态。
			// 发送消息和接收消息的状态监听均可以通过此接口完成。
		}
	}
}, true);

// 如果发送的多媒体文件消息，还需要监听文件的上传进度。
NIMClient.getService(MsgServiceObserve.class).observerAttachProgress(
	new Observer(AttachProgress progress) {
		public void onEvent(AttachProgress progress) {
			// 参数为附件的传输进度，可根据 progress 中的 uuid 查找具体的消息对象，更新 UI
			// 上传附件和下载附件的进度监听均可以通过此接口完成。
		}
	}, true);
```

- 保存消息到本地

1\. 如果第三方APP想保存消息到本地，可以调用 MsgService#saveMessageToLocal ，该接口保存消息到本地数据库，但不发送到服务器端。该接口将消息保存到数据库后，如果需要通知到UI，可将参数 notify 设置为 true ，此时会触发 #observeReceiveMessage 通知。

2\. 此接口在 1.8.0 版本及以上支持设置是否计入未读数（默认计入未读数），若需要不计入未读数，传入的 IMMessage 中的 CustomMessageConfig 的 enableUnreadCount 需要设置为 false 。

```
NIMClient.getService(MsgService.class).saveMessageToLocal(msg, true);
```

3\. 2.8.0 版本及以上，新增可以设置消息存储时间点的保存消息到本地的接口，其他使用方式与旧的接口一致。常用场景：撤回消息时，在撤回的位置（有可能为消息列表中间）插入消息。

```
NIMClient.getService(MsgService.class).saveMessageToLocal(message, true, time);
```

### <span id="接收消息">接收消息</span>

通过添加消息接收观察者，在有新消息到达时，第三方 APP 就可以接收到通知。代码如下：

```java
Observer<List<IMMessage>> incomingMessageObserver =
	new Observer<List<IMMessage>>() {
	    @Override
	    public void onEvent(List<IMMessage> messages) {
		    // 处理新收到的消息，为了上传处理方便，SDK 保证参数 messages 全部来自同一个聊天对象。
        }
    }
	NIMClient.getService(MsgServiceObserve.class)
		.observeReceiveMessage(incomingMessageObserver, true);
```

该代码的典型场景为消息对话界面，在界面 `onCreate` 里注册消息接收观察者，在 `onDestroy` 中注销观察者。在收到消息时，判断是否是当前聊天对象的消息，如果是，加入到列表中显示。
- 多媒体文件下载

如果接收到消息是带文件附件的多媒体消息，SDK 默认会在后台自动下载附件：如果是语音消息，直接下载文件，如果是图片或视频消息，下载缩略图文件。开发者可在 `SDKOptions` 中关闭自动下载，并在用户翻阅到对应消息，再通过以下代码手动下载。如果自动下载或手动下载失败，也可以通过这段代码重新下载：

```java
// 下载之前判断一下是否已经下载。若重复下载，会报错误码414。（以SnapChatAttachment为例）
// 错误码414可能是重复下载，或者下载参数错误。
private boolean isOriginImageHasDownloaded(final IMMessage message) {
    if (message.getAttachStatus() == AttachStatusEnum.transferred &&
        !TextUtils.isEmpty(((SnapChatAttachment) message.getAttachment()).getPath())) {
        return true;
    }
    return false;
}
// 下载附件，参数1位消息对象，参数2为是下载缩略图还是下载原图。
// 因为下载的文件可能会很大，这个接口返回类型为 AbortableFuture ，允许用户中途取消下载。
AbortableFuture future = NIMClient.getService(MsgService.class).downloadAttachment(message, true);
```
- 获取多媒体文件

多媒体文件收到之后，会进行自动下载或手动下载。需要在下载完成之后，才能获取到多媒体文件路径，并刷新界面。通过监听消息状态的变化，来查看是否下载完成，示例代码如下：

```
// 监听消息状态变化
NIMClient.getService(MsgServiceObserve.class).observeMsgStatus(statusObserver, register);

private Observer<IMMessage> statusObserver = new Observer<IMMessage>() {
        @Override
        public void onEvent(IMMessage msg) {
            // 1、根据sessionId判断是否是自己的消息
            // 2、更改内存中消息的状态
            // 3、刷新界面
        }
    };
```

### <span id="最近会话">最近会话</span>

最近会话 `RecentContact` ，也可称作会话列表或者最近联系人列表，它记录了与用户最近有过会话的联系人信息，包括联系人帐号、联系人类型、最近一条消息的时间、消息状态、消息内容、未读条数等信息。
`RecentContact` 中还提供了一个扩展标签 tag（用于做联系人置顶、最近会话列表排序等扩展用途）和一个扩展字段 extension （是一个Map，可用于做群@等扩展用途），并支持动态的更新这两个字段。
最近会话列表由 SDK 维护并提供查询、监听变化的接口，只要与某个用户或者群组有产生聊天（自己发送消息或者收到消息）， SDK 会自动更新最近会话列表并通知上层，开发者无需手动更新。
某些场景下，开发者可能需要手动向最近会话列表中插入一条会话项（即插入一个最近联系人），例如：在创建完高级群时，需要在最近会话列表中显示该群的会话项。由创建高级群完成时并不会收到任何消息， SDK 并不会立即更新最近会话，此时要满足需求，可以在创建群成功的回调中，插入一条本地消息， 即调用 MsgService#saveMessageToLocal。

> 注意：最近会话是本地的，不会漫游。漫游与消息相关，与最近会话无关。

- 获取最近会话列表：

```java
 NIMClient.getService(MsgService.class).queryRecentContacts()
	 .setCallback(new RequestCallbackWrapper<List<RecentContact>>() {
	   @Override
       public void onResult(int code, List<RecentContact> recents, Throwable e) {
            // recents参数即为最近联系人列表（最近会话列表）
       }
    });
```

- 监听最近会话变更

在收发消息的同时，SDK 会更新对应聊天对象的最近联系人资料。当有消息收发时，SDK 会发出最近联系人更新通知：

```java
//  创建观察者对象
Observer<List<RecentContact>> messageObserver =
	new Observer<List<RecentContact>>() {
	    @Override
	    public void onEvent(List<RecentContact> messages) {
	    }
    }
//  注册/注销观察者
NIMClient.getService(MsgServiceObserve.class)
	.observeRecentContact(messageObserver, register);
```

- 获取会话未读数总数，总两种方法：

1\. 通过接口直接获取：

```java
int unreadNum = NIMClient.getService(MsgService.class).getTotalUnreadCount();
```

2\. 对最近联系人列表中的每个最近联系人的未读数进行求和：

```java
int unreadNum = 0;
for (RecentContact r : items) {
    unreadNum += r.getUnreadCount();
}
```

> 说明：多端同时登录时，在其他端进行查看，客户端不会进行未读数清零操作。

- 设置当前会话

如果用户在开始聊天时，开发者调用了 `setChattingAccount` 接口，SDK会自动管理消息的未读数。当收到新消息时，自动将未读数清零。如果第三方 APP 需要不进入聊天窗口，就需要将未读数清零，可以通过调用如下接口来实现：

```java
// 触发 MsgServiceObserve#observeRecentContact(Observer, boolean) 通知，
// 通知中的 RecentContact 对象的未读数为0
NIMClient.getService(MsgService.class).clearUnreadCount(account, sessionType);
```

如果需要在最近联系人列表界面显示当前消息状态，还需要增加消息状态监听，操作见[发送消息](#发送消息) 一节。

- 移除最近会话列表中的项

MsgService 提供了两种方法： deleteRecentContact 和 deleteRecentContact2，区别在于后者会触发 MsgServiceObserve#observeRecentContactDeleted 通知。

```java
NIMClient.getService(MsgService.class).deleteRecentContact(recent);
```

- 删除指定最近联系人的漫游消息

不删除本地消息，但如果在其他端登录，当前时间点该会话已经产生的消息不漫游。

```
NIMClient.getService(MsgService.class)
	.deleteRoamingRecentContact(contactId, sessionTypeEnum)
	.setCallback(new RequestCallback<Void>() { ... });
```

### <span id="自定义消息">自定义消息</span>

除了内建消息类型以外，SDK 还支持收发自定义消息类型。SDK 不负责定义和解析自定义消息的具体内容，解释工作由开发者完成。SDK 会将自定义消息存入消息数据库，会和内建消息一并展现在消息记录中。
为了使用更加方便，自定义消息采用附件的方式展示给开发者。体现在 `IMMessage` 类中，自定义消息的内容会被解析为 `MsgAttachment` 对象，由于 SDK 并不知道自定义消息的格式，第三方 APP 需要注册一个自定义消息解析器。当第三方 APP 调用查询查询消息历史的接口，或者是 SDK 收到消息通知第三方 APP 时，就能将自定义消息内容转换为一个 `MsgAttachment` 对象，然后就可以同操作图片消息等带附件消息一样，操作自定义消息了。
与 IMMessage 一致，为了增加自定义消息的灵活性，SDK 还提供了对其生命周期和推送参数的一些配置选项，开发者可以自主设定一条自定义消息是否要保存云端消息记录，是否要漫游，发送时是否要多端同步等的配置选项 `CustomMessageConfig`。详见上文发消息章节。

创建和发送自定义消息的代码如下：

```java
IMMessage message = MessageBuilder.createCustomMessage(
	sessionId, // 会话对象
	sessionType, // 会话类型
	attachment, // 自定义消息附件
    config // 自定义消息的参数配置选项
	)
NIMClient.getService(MsgService.class).sendMessage(message);
```

#### <span id="示例一（剪刀石头布）">示例一（剪刀石头布）</span>

在 demo 中，我们提供了一个用自定义消息实现的“剪刀石头布”的游戏，下面以此为例，详细解析其实现步骤。

首先，我们先定义一个自定义消息附件的基类，负责解析你的自定义消息的公用字段，比如类型等。还可以定义一些公共接口，用于一些便利性的调用。

> 注意: 实现 `MsgAttachment ` 接口的成员都要实现 Serializable。

```java
// 先定义一个自定义消息附件的基类，负责解析你的自定义消息的公用字段，比如类型等等。
public abstract class CustomAttachment implements MsgAttachment {

	// 自定义消息附件的类型，根据该字段区分不同的自定义消息
    protected int type;

    CustomAttachment(int type) {
        this.type = type;
    }

	// 解析附件内容。
    public void fromJson(JSONObject data) {
        if (data != null) {
            parseData(data);
        }
    }

	// 实现 MsgAttachment 的接口，封装公用字段，然后调用子类的封装函数。
    @Override
    public String toJson(boolean send) {
        return CustomAttachParser.packData(type, packData());
    }

    // 子类的解析和封装接口。
    protected abstract void parseData(JSONObject data);
    protected abstract JSONObject packData();
}
```

然后，继承这个基类，实现“剪刀石头布”的附件类型。注意，成员变量都要实现 Serializable。

```java
public class GuessAttachment extends CustomAttachment {

	// 猜拳类型枚举
    public enum Guess {
        Shitou(1),
        Jiandao(2),
        Bu(3),
        ;
    }

    private Guess value;

    public GuessAttachment() {
        super(CustomAttachmentType.Guess);
        random();
    }

	// 解析猜拳类型具体数据
    @Override
    protected void parseData(JSONObject data) {
        value = Guess.enumOfValue(data.getIntValue("value"));
    }

	// 数据打包
    @Override
    protected JSONObject packData() {
        JSONObject data = new JSONObject();
        data.put("value", value.getValue());
        return data;
    }

    private void random() {
        int value = new Random().nextInt(3) + 1;
        this.value = Guess.enumOfValue(value);
    }
}

```

第三步，实现自定义消息的附件解析器。

```java
public class CustomAttachParser implements MsgAttachmentParser {

	// 根据解析到的消息类型，确定附件对象类型
    @Override
    public MsgAttachment parse(String json) {
        CustomAttachment attachment = null;
        try {
            JSONObject object = JSON.parseObject(json);
            int type = object.getInteger("type");
            JSONObject data = object.getJSONObject(KEY_DATA);
            switch (type) {
                case CustomAttachmentType.Guess:
                    attachment = new GuessAttachment();
                    break;
                default:
                    attachment = new DefaultCustomAttachment();
                    break;
            }

            if (attachment != null) {
                attachment.fromJson(data);
            }
        } catch (Exception e) {

        }

        return attachment;
    }

	public static String packData(int type, JSONObject data) {
        JSONObject object = new JSONObject();
        object.put(KEY_TYPE, type);
        if (data != null) {
            object.put(KEY_DATA, data);
        }

        return object.toJSONString();
    }
}
```

最后，将该附件解析器注册到 SDK 中。为了保证生成历史消息时能够正确解析自定义附件，注册一般应放在 Application 的 onCreate 中 的主进程判断语句内完成。

```java
NIMClient.getService(MsgService.class).registerCustomAttachmentParser(new CustomAttachParser()); // 监听的注册，必须在主进程中。
```

#### <span id="示例二（阅后即焚）">示例二（阅后即焚）</span>

若需要发送文件类型消息，例如图片等，可以参考阅后即焚的实现。具体实现步骤如下：

第一步，定义一个自定义的附件类型，并继承 `FileAttachment`。注意，成员变量都要实现 Serializable。

```
public class SnapChatAttachment extends FileAttachment {

	private static final String KEY_PATH = "path";
    private static final String KEY_SIZE = "size";
    private static final String KEY_MD5 = "md5";
    private static final String KEY_URL = "url";

    public SnapChatAttachment() {
        super();
    }

    public SnapChatAttachment(JSONObject data) {
        load(data);
    }

    @Override
    public String toJson(boolean send) {
        JSONObject data = new JSONObject();
        try {
	        // 重发使用本地路径
	        if (!send && !TextUtils.isEmpty(path)) {
                data.put(KEY_PATH, path);
            }

            if (!TextUtils.isEmpty(md5)) {
                data.put(KEY_MD5, md5);
            }
			// 注意：这段代码一定要写。
			// SDK在调toJson的时候 父类FileAttachemnt的url才有值。
			// 这个值是sdk自动赋值的。
            data.put(KEY_URL, url);
            data.put(KEY_SIZE, size);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return CustomAttachParser.packData(CustomAttachmentType.SnapChat, data);
    }

    private void load(JSONObject data) {
	    path = data.getString(KEY_PATH);
        md5 = data.getString(KEY_MD5);
        url = data.getString(KEY_URL);
        size = data.containsKey(KEY_SIZE) ? data.getLong(KEY_SIZE) : 0;
    }
}
```

第二步，实现自定义消息的附件解析器。

```java
public class CustomAttachParser implements MsgAttachmentParser {

	// 根据解析到的消息类型，确定附件对象类型
    @Override
    public MsgAttachment parse(String json) {
        CustomAttachment attachment = null;
        try {
            JSONObject object = JSON.parseObject(json);
            int type = object.getInteger("type");
            JSONObject data = object.getJSONObject(KEY_DATA);
            switch (type) {
                case CustomAttachmentType.SnapChat:
                    return new SnapChatAttachment(data);
                default:
                    attachment = new DefaultCustomAttachment();
                    break;
            }

            if (attachment != null) {
                attachment.fromJson(data);
            }
        } catch (Exception e) {

        }

        return attachment;
    }
    ...
}
```

最后，将该附件解析器注册到 SDK 中。为了保证生成历史消息时能够正确解析自定义附件，注册一般应放在 Application 的 onCreate 中完成。

```java
NIMClient.getService(MsgService.class).registerCustomAttachmentParser(new CustomAttachParser()); // 监听的注册，必须在主进程中。
```

### <span id="Tip消息">Tip消息</span>

Tip 消息主要用于会话内的通知提醒，可以看做是自定义消息的简化，有独立的消息类型 MsgTypeEnum.tip 。
区别于自定义消息，Tip 消息暂不支持 setAttachment，如果要使用 Attachment 请使用自定义消息。
Tip 消息使用场景例如：进入会话时出现的欢迎消息，或是会话过程中命中敏感词后的提示消息等场景，当然也可以用自定义消息实现，只是相对复杂一些。

```java
// 演示：发送一条在线的Tip消息
IMMessage msg = MessageBuilder.createTipMessage(getAccount(), getSessionType());
msg.setContent("这Tip消息内容");
sendMessage(msg);
```

```java
// 演示：向群里插入一条Tip消息，使得该群能立即出现在最近联系人列表（会话列表）中，满足部分开发者需求。
Map<String, Object> content = new HashMap<>(1);
content.put("content", "成功创建高级群");
IMMessage msg = MessageBuilder.createTipMessage(team.getId(), SessionTypeEnum.Team);
msg.setRemoteExtension(content);
CustomMessageConfig config = new CustomMessageConfig();
config.enableUnreadCount = false;
msg.setConfig(config);
msg.setStatus(MsgStatusEnum.success);
NIMClient.getService(MsgService.class).saveMessageToLocal(msg, true);
```

### <span id="更新消息">更新消息</span>

目前仅提供消息状态和消息本地扩展字段的更新（更新 SDK 数据库中对应字段的值）：

- 消息状态、消息附件状态、消息附件内容的更新接口为 MsgService#updateIMMessageStatus。

- 消息本地扩展字段 LocalExtension 的更新接口为 MsgService#updateIMMessage。

### <span id="已读回执">已读回执</span>

网易云信提供点对点消息的已读回执。注意：此功能仅对 P2P 消息中有效。

在会话界面中调用发送已读回执的接口并传入最后一条消息，即表示这之前的消息都已读，对端将收到此回执。

发送消息已读回执的一般场景：
- 进入 P2P 聊天界面（如果没有收到新的消息，反复进入调用发送已读回执接口， SDK 会自动过滤，只会发送一次给云信服务器）。
- 处于聊天界面中，收到当前会话新消息时。

```java
/**
* 发送消息已读回执
* @param sessionId 会话ID（聊天对象账号）
* @param message 已读的消息(一般是当前接收的最后一条消息）
*/
NIMClient.getService(MsgService.class).sendMessageReceipt(sessionId, message);
```

监听已读回执

监听到已读回执，根据 IMMessage 中的 `isRemoteRead()` 方法来判断该条消息是否已读，并刷新界面。

一般场景：一个会话显示一个已读回执（比该已读回执对应的消息时间早的消息都是已读），那么刷新界面时，可以倒序查找第一条 isRemoteRead 接口返回 true 的消息打上已读回执标记。注意：当删除带有已读回执标记的消息时，也应该刷新界面。

第三方可以根据已读回执观察者来监听这个消息，示例如下：

```java
// 注册/注销观察者
NIMClient.getService(MsgServiceObserve.class).observeMessageReceipt(messageReceiptObserver, register);

private Observer<List<MessageReceipt>> messageReceiptObserver = new Observer<List<MessageReceipt>>() {
        @Override
        public void onEvent(List<MessageReceipt> messageReceipts) {
            receiveReceipt();
        }
    };
```

### <span id="转发消息"> 转发消息 </span>

网易云信支持消息转发功能，不支持通知消息和音视频消息的转发，其他消息类型均支持。

首先，通过 `MessageBuilder` 创建一个待转发的消息，参数为想转发的消息，转发目标的聊天对象id， 转发目标的会话类型。然后，通过 `MsgService#sendMessage` 接口，将消息发送出去。

```
// 创建待转发消息
IMMessage message = MessageBuilder.createForwardMessage(forwardMessage, sessionId, sessionTypeEnum);
if (message == null) {
	Toast.makeText(container.activity, "该类型不支持转发", Toast.LENGTH_SHORT).show();
	return;
}
NIMClient.getService(MsgService.class).sendMessage(message, false);
```

### <span id="清空本地所有消息记录">清空本地所有消息记录</span>

网易云信支持清空本地数据库中的所有消息记录。在清空数据记录的同时，可选择是否要同时清空最近联系人列表数据库。若最近联系人列表也被清空，会触发MsgServiceObserve#observeRecentContactDeleted(Observer, boolean)通知

```
/**
* 清空本地所有消息记录
* @param clearRecent 若为true，将同时清空最近联系人列表数据
*/
public void clearMsgDatabase(boolean clearRecent);
```

### <span id="消息过滤">消息过滤</span>

支持单聊和群聊的通知类型消息过滤，支持音视频类型消息过滤。

通知消息是指 IMMessage#getMsgType 为 MsgTypeEnum#notification。 SDK 在 2.4.0 版本后支持上层指定过滤器决定是否要将通知消息，音视频消息存入 SDK 数据库（并通知上层收到该消息）。请注意，注册过滤器的时机，建议放在 Application 的 onCreate 中， SDK 初始化之后。

示例：SDK 过滤群头像变更通知和过滤音视频类型消息。

```java
// 在 Application启动时注册，保证漫游、离线消息也能够回调此过滤器进行过滤。注意，过滤器的实现不要有耗时操作。
NIMClient.getService(MsgService.class).registerIMMessageFilter(new IMMessageFilter() {
        @Override
        public boolean shouldIgnore(IMMessage message) {
            if (UserPreferences.getMsgIgnore() && message.getAttachment() != null) {
                if (message.getAttachment() instanceof UpdateTeamAttachment) {
                    UpdateTeamAttachment attachment = (UpdateTeamAttachment) message.getAttachment();
                    for (Map.Entry<TeamFieldEnum, Object> field : attachment.getUpdatedFields().entrySet()) {
                        if (field.getKey() == TeamFieldEnum.ICON) {
                            return true; // 过滤
                        }
                    }
                } else if (message.getAttachment() instanceof AVChatAttachment) {
                    return true; // 过滤
                }
            }
            return false; // 不过滤
        }
    });
```

### <span id="消息撤回">消息撤回</span>

- 发起消息撤回

云信支持发送消息的撤回。超过限定时间（默认时间为2分钟）的撤回会失败，并返回错误码508。消息撤回后，未读数不发生变化，通知栏提醒的文案变为：撤回了一条消息。

几种不能撤回的情况：

1、消息为空，不能撤回
2、消息没有发送成功，不能撤回
3、消息发起者不是自己，不能撤回
4、会话 sessionId 是自己，不能撤回

```
/**
*  消息撤回
*
* @param message 待撤回的消息
* @return InvocationFuture 可设置回调函数，监听发送结果。
*/
NIMClient.getService(MsgService.class)
	.revokeMessage(message).setCallback(new RequestCallback<Void>() { ... });
```

- 监听消息撤回

消息被撤回，会收到消息撤回通知。当开发者收到消息撤回通知，可以在界面上做相应的消息删除等操作。

添加消息撤回通知的观察者代码如下：

```
NIMClient.getService(MsgServiceObserve.class).observeRevokeMessage(revokeMessageObserver, register);

Observer<IMMessage> revokeMessageObserver = new Observer<IMMessage>() {
        @Override
        public void onEvent(IMMessage message) {
            // 执行消息撤回后的界面操作
        }
    };
```

## <span id="消息全文检索">消息全文检索</span>

云信 SDK 2.7.0 加入基于 Lucene 的全文检索插件，支持聊天消息的全文检索，目前开放的查询接口适用于两类需求（与微信类似）：

- 需求1：

检索所有会话，返回所有匹配关键字的会话、每个会话匹配关键字的消息条数。如果会话只有一条消息匹配，那么直接返回该消息内容，返回的会话数量可以指定，也可以一次性列出所有会话。会话间的排序规则，按照每个会话最近一条匹配消息的时间倒序排列。需要支持多个关键字查询，采用空格隔开，关键字之间是 AND 关系。

- 需求2：

检索单个会话，返回所有匹配关键字的消息，并高亮显示被击中的关键字，可以跳转到该消息的上下文。

云信 SDK 目前主要针对上述两种需求提供查询服务，只要集成了全文检索插件，SDK 会自动同步所有聊天记录到全文检索索引中。

注意：全文检索插件最低需要 Android 14 (Android 4.0)，APP 构建时 targetSdkVersion 强烈建议指向 19 及以上。

在 2.8.0 版本我们已经对依赖的 Lucene 源码做了大幅度的精简，约 1M+ 的大小，对于大的工程，仍然可能造成方法数 65535 的限制，此时可能需要 Multi dex 的支持。

### <span id="插件集成">插件集成</span>

有两种方式，选其一即可：

方式一： libs 中引入 nim-lucene-2.8.0.jar。

方式二： 在 build.gradle 中集成：

```
dependencies {
    ...
    compile 'com.netease.nimlib:lucene:2.8.0'
}
```

如果构建 APP 时有使用代码混淆，需要在proguard.cfg中加入

```
-dontwarn org.apache.lucene.**
-keep class org.apache.lucene.** {*;}
```

### <span id="接口说明">接口说明</span>

全文检索接口为：LuceneService，具体 API 如下（下面只列出异步版本，同步版本 API 也提供），针对需求2，强烈建议您使用分页查询接口：

```java
/**
 * 检索所有会话，返回每个会话与检索串匹配的消息数及最近一条匹配的消息记录。（异步函数）
 *
 * @param query 待检索的字符串
 * @param limit 最多返回多少条记录
 * @return InvocationFuture 可以设置回调函数，回调时返回聊天消息全文检索结果集
*/
public InvocationFuture<List<MsgIndexRecord>> searchAllSession(String query, int limit);

/**
 * 检索指定的会话，返回该会话中与检索串匹配的所有消息记录。（异步函数）
 *
 * @param query       待检索的字符串
 * @param sessionType 待检索的会话类型（个人/群组）
 * @param sessionId   待检索的会话ID
 * @return 聊天消息全文检索结果集
*/
public InvocationFuture<List<MsgIndexRecord>> searchSession(String query, SessionTypeEnum sessionType, String sessionId);

/**
 * 指定会话关键字查询（分页返回匹配记录）（异步）
 *
 * @param query       待检索的字符串
 * @param sessionType 待检索的会话类型（个人/群组）
 * @param sessionId   待检索的会话ID
 * @param pageIndex   页码（从第一页开始）
 * @param pageSize    分页大小
 * @return InvocationFuture 可以设置回调函数，回调时返回聊天消息全文检索结果集
*/
public InvocationFuture<List<MsgIndexRecord>> searchSessionPage(String query, SessionTypeEnum sessionType, String sessionId, int pageIndex, int pageSize);

/**
 * 指定会话关键字查询（分页查询：根据锚点，返回下一页匹配记录）（异步）
 *
 * @param query       待检索的字符串
 * @param sessionType 待检索的会话类型（个人/群组）
 * @param sessionId   待检索的会话ID
 * @param anchor      首页传null，下一页传上一页的最后一条记录
 * @param pageSize    分页大小
 * @return InvocationFuture 可以设置回调函数，回调时返回聊天消息全文检索结果集
*/
public InvocationFuture<List<MsgIndexRecord>> searchSessionNextPage(String query, SessionTypeEnum sessionType, String sessionId, MsgIndexRecord anchor, int pageSize);

/**
 * 指定会话关键字查询匹配记录总数（同步）
 *
 * @param query       待检索的字符串
 * @param sessionType 待检索的会话类型（个人/群组）
 * @param sessionId   待检索的会话ID
 * @return 匹配的记录总数
*/
public int searchSessionMatchCount(String query, SessionTypeEnum sessionType, String sessionId);

/**
 * 指定会话关键字查询匹配记录总页数（同步）
 *
 * @param query       待检索的字符串
 * @param sessionType 待检索的会话类型（个人/群组）
 * @param sessionId   待检索的会话ID
 * @param pageSize    分页每页记录数
 * @return 匹配的记录总页数
*/
public int searchSessionPageCount(String query, SessionTypeEnum sessionType, String sessionId, int pageSize);

/**
 * 获取所有缓存数据的大小
 * @return 缓存数据字节数
 */
public long getCacheSize();

/**
 * 删除所有缓存数据
 */
public void clearCache();
```

## <span id="消息提醒">消息提醒</span>

集成网易云信 Android SDK 的 APP 运行起来时，会有个后台进程（push 进程），该进程保持了与云信 Server 的长连接。只要这个 push 进程活着（云信提供安卓保活机制），就能接收云信 Server 推过来的消息，进行通知栏提醒。

### <span id="消息提醒场景">消息提醒场景</span>

- 需要消息提醒的场景：

1\. APP 在后台时。

2\. 在前台与 A 聊天但收到非 A 的消息时（与 iOS 不一样）。

3\. 在非聊天界面且非最近会话界面时。

- 不需要消息提醒的场景：

1\. 如果用户正在与某一个人聊天，当这个人的消息到达时，是不应该有通知栏提醒的。

2\. 如果用户停留在最近联系人列表界面，收到消息也不应该有通知栏提醒（但会有未读数变更通知）。

云信 SDK 提供内置的消息提醒功能，如需使用，开发者需要在进出聊天界面以及最近联系人列表界面时，通知 SDK。相关接口如下：

```java
// 进入聊天界面，建议放在onResume中
NIMClient.getService(MsgService.class).setChattingAccount(account， sessionType);

// 进入最近联系人列表界面，建议放在onResume中
NIMClient.getService(MsgService.class).setChattingAccount(MsgService.MSG_CHATTING_ACCOUNT_ALL, SessionTypeEnum.None);

// 退出聊天界面或离开最近联系人列表界面，建议放在onPause中
NIMClient.getService(MsgService.class).setChattingAccount(MsgService.MSG_CHATTING_ACCOUNT_NONE, SessionTypeEnum.None);
```

### <span id="内置消息提醒定制">内置消息提醒定制</span>

云信 SDK 提供内置的消息提醒（通知栏提醒）功能，并提供以下四个维度的定制，开启/关闭内置的消息提醒、更新消息提醒配置接口如下：

```java
// 开启/关闭通知栏消息提醒
NIMClient.toggleNotification(enable);

// 更新消息提醒配置 StatusBarNotificationConfig
NIMClient.updateStatusBarNotificationConfig(config);
```

#### 接收消息时本地全局配置（不可漫游）

在 SDKOption 中有通知栏提醒配置选项 StatusBarNotificationConfig：
* 通知栏弹出时的小图标（ticker），默认用App的图标。
* 是否需要响铃（指定铃声）
* 是否需要震动
* 呼吸灯颜色、闪烁时长
* 免打扰（是否开启免打扰、设置免打扰开始结束时间），在免打扰时间段的不进行通知栏提醒。
* 点击通知栏要跳转到哪里（一般来说会跳转到主界面，然后根据对应消息的发送者，跳转到指定的P2P聊天界面）

注意: StatusBarNotificationConfig 中的notificationEntrance 字段指明了点击通知需要跳转到的Activity，Activity启动后可以获取收到的消息：

``` java
ArrayList<IMMessage> messages = (ArrayList<IMMessage>)
getIntent().getSerializableExtra(NimIntent.EXTRA_NOTIFY_CONTENT); // 可以获取消息的发送者，跳转到指定的单聊、群聊界面。
```

#### 可漫游的消息提醒配置

- 个人消息提醒配置（支持漫游）

支持对用户开启或关闭消息提醒，关闭后，收到该用户发来的消息时，不再进行SDK内置的通知栏消息提醒。

```java
NIMClient.getService(FriendService.class).setMessageNotify(account, checkState).setCallback(new RequestCallback<Void>() {});
```

- 群消息提醒配置（支持漫游）

群聊消息提醒可以单独打开或关闭，关闭提醒之后，用户仍然会收到这个群的消息，但是SDK内置的通知栏提醒将不会触发。如果开发者自行实现通知栏提醒，可通过 Team 的 mute 接口获取是否开启消息提醒，并决定是不是要显示通知。
开发者可通过调用以下接口打开或关闭群聊消息提醒：

```java
NIMClient.getService(TeamService.class).muteTeam(teamId, mute);
```

#### 接收消息时定制提醒内容

针对不同的消息类型，通知栏显示不同的提醒内容。按照以下优先级显示：

1\. 发送方可以设置了推送文案，如果设置，那么通知栏显示该推送文案。

对于 SDK 1.7.0 及以上版本，开发者可以调用 `IMMessage` 的 `setPushContent` 接口设置推送文案；

对于低于 1.7.0 的早期版本，开发者可以调用 `IMMessage` 的 `setContent` 接口设置推送文案：对于文本消息，该接口会同时修改消息内容和提醒内容，对于其他格式消息，该接口仅修改提醒内容。如果接收方是 iOS 客户端，消息推送的内容遵从相同的规则：如果设置了 `setContent` 字段，则使用设置的字符串作为推送内容，否则使用默认提醒内容。

2\. （ SDK 1.8.0 及以上版本支持）本地定制的通知栏提醒文案，目前支持配置Ticker文案（通知栏弹框条显示内容）和通知内容文案（下拉通知栏显示的通知内容）， SDK 会在收到消息时回调 `MessageNotifierCustomization` 接口， 开发者可以根据昵称和收到的消息（消息类型、会话类型、发送者、消息扩展字段等）来决定要显示的通知内容。示例如下：

```java
public class NimApplication extends Application {
    public void onCreate() {
        ...
        NIMClient.init(this, getLoginInfo(), getOptions());
        ...
    }

    private SDKOptions getOptions() {
        SDKOptions options = new SDKOptions();
        ...

        // 定制通知栏提醒文案（可选，如果不定制将采用SDK默认文案）
        options.messageNotifierCustomization = messageNotifierCustomization;
        return options;
    }

    private MessageNotifierCustomization messageNotifierCustomization = new MessageNotifierCustomization() {
        @Override
        public String makeNotifyContent(String nick, IMMessage message) {
            return null; // 采用SDK默认文案
        }

        @Override
        public String makeTicker(String nick, IMMessage message) {
            return null; // 采用SDK默认文案
        }
    };
}
```

3\. 如果上述两点都不定制(返回null)，将显示默认提醒内容：

文本消息：文本消息内容。

文件消息：{说话者}发来一条文件消息

图片消息：{说话者}发来一条图片消息

语音消息：{说话者}发来一条语音消息

视频消息：{说话者}发来一条视频消息

位置消息：{说话者}分享了一个地理位置

通知消息：{说话者}: 通知消息

提醒消息：{说话者}: 提醒消息

自定义消息：{说话者}: 自定义消息

除文本消息外，开发者可以通过  `NimStrings` 类修改这些默认提醒内容。

#### 接收消息时定制通知栏的头像

云信支持定制通知栏显示的头像(用户头像、群头像)，在 UserInfoProvider 接口下提供方法：

```
/**
 * 如果根据用户账号找不到UserInfo的avatar时，显示的默认头像资源ID
 *
 * @return 默认头像的资源ID
 */
int getDefaultIconResId();

/**
 * 为通知栏提供用户头像（一般从本地缓存中取，若未下载或本地不存在，返回null，通知栏将显示默认头像）
 *
 * @return 头像位图
 */
Bitmap getAvatarForMessageNotifier(String account);

/**
 * 为通知栏提供消息发送者显示名称（例如：如果是P2P聊天，可以显示备注名、昵称、帐号等；如果是群聊天，可以显示群昵称，备注名，昵称、帐号等）
 *
 * @param account     消息发送者账号
 * @param sessionId   会话ID（如果是P2P聊天，那么会话ID即为发送者账号，如果是群聊天，那么会话ID就是群号）
 * @param sessionType 会话类型
 * @return 消息发送者对应的显示名称
 */
String getDisplayNameForMessageNotifier(String account, String sessionId, SessionTypeEnum sessionType);

/**
 * 根据群组ID获取群组头像位图。头像功能可由app自己拼接或自定义，也可以直接使用预置图片作为头像
 * 为通知栏提供群头像（一般从本地缓存中取，若未下载、未合成或者本地缓存不存，请返回预置的群头像资源ID对应的Bitmap）
 *
 * @param tid 群组ID
 * @return 群组头像位图
 */
Bitmap getTeamIcon(String tid);
```
实现上述需要的方法，在 SDKOptions 中配置 UserInfoProvider 实例，在 SDK 初始化时传入 SDKOptions 方可生效。

需要注意的是，上述返回头像 Bitmap 的函数，请尽可能从内存缓存里拿头像，如果读取本地头像可能导致 UI 进程阻塞，从而导致通知栏提醒延时弹出。

#### 发送消息时指定消息提醒

发送消息时可以设置消息配置选项 CustomMessageConfig，可以设定是否需要推送，是否需要计入未读数等：

enablePush ： 该消息是否进行推送（消息提醒）。默认为 true 。

enableUnreadCount ：该消息是否要计入未读数，如果为 true ，那么对方收到消息后，最近联系人列表中未读数加1。默认为 true 。

### <span id="自行实现消息提醒">自行实现消息提醒</span>

如果 SDK 内建的消息提醒不能满足你的需求，你可以关闭 SDK 内置的消息提醒，自行实现。收到新消息时，上层有两种方式得到通知，然后给出通知栏提醒：

1\. 添加消息接收观察者，在观察者的 `onEvent` 中实现状态栏提醒。注册注销方式详见[接收消息](#接收消息)一节。注意：只有 SDK 1.4.0 及以上版本才能使用该方式，1.4.0 以下的版本使用此方式有可能会漏掉通知。

2\. SDK 收到新消息后，还会发送一个广播，在广播接收者的 `onReceive` 中实现自己的状态栏提醒。该广播的 ACTION 为 APP 的包名加上 `NimIntent#ACTION_RECEIVE_MSG`，例如 demo 的为："com.netease.nim.demo.ACTION.RECEIVE\_MSG"。使用这种方式时，开发者需要在 AndroidManifest.xml 文件中注册此广播的接收者，并添加对应的广播权限。

### <span id="PC/Web端在线配置推送">PC/Web端在线配置推送</span>

支持配置桌面端（PC/Web）在线时，移动端是否需要推送。

```
/**
* 设置桌面端(PC/WEB)在线时，移动端是否需要推送
* @param isOpen true 桌面端在线时移动端不需推送；false 桌面端在线时移动端需推送
* @return InvocationFuture 可以设置回调函数。成功会返回成功信息，错误会返回相应的错误码。
*/
NIMClient.getService(SettingsService.class)
	.updateMultiportPushConfig(isOpen)
	.setCallback(new RequestCallback<Void>() { ... });
```

支持查询当前配置的推送状态。true 桌面端在线时移动端不需推送；false 桌面端在线时移动端需推送

```
NIMClient.getService(SettingsService.class).isMultiportPushOpen()
```

## <span id="历史记录">历史记录</span>

### <span id="本地记录">本地记录</span>

- 查询消息历史

SDK 提供了一个根据锚点查询本地消息历史记录的接口，根据提供的方向 (direct)，查询比 anchor 更老 (QUERY\_OLD) 或更新 (QUERY\_NEW) 的最靠近anchor 的 limit 条数据。调用者可使用 asc 参数指定结果排序规则，结果使用 time 作为排序字段。

当进行首次查询时，锚点可以用使用 `MessageBuilder#createEmptyMessage` 接口生成。查询结果不包含锚点。

```java
/**
 * @param anchor IMMessage 查询锚点
 * @param direction QueryDirectionEnum 查询方向
 * @param limit int 查询结果的条数限制
 * @param asc boolean 查询结果的排序规则，如果为 true，结果按照时间升级排列，如果为 false，按照时间降序排列
 * @return 调用跟踪，可设置回调函数，接收查询结果
 */
NIMClient.getService(MsgService.class).queryMessageListEx(anchor, direction, limit, asc);
```

SDK 还提供了按照关键字搜索聊天记录的功能，可以对指定的聊天对象，输入关键字进行消息内容搜索。查询方向为从后往前。以锚点 anchor 作为起始点开始查询，返回最多 limit 条匹配 keyword 的记录。该接口目前仅搜索文本类型的消息，匹配规则为文本内容包含 keyword，目前仅支持精确匹配，不支持模糊匹配和拼音匹配。
由于 SDK 并不存储用户数据，因此 keyword 不会匹配用户资料。如果调用者希望查询指定用户的说话记录，可提供 fromAccounts 参数。如果提供的 fromAccounts 参数不为空，那么 anchor 对应的会话 (由 sessionId 和 sessionType决定) 的消息记录中，凡是消息说话者在 fromAccounts 列表中的记录，也会被当做匹配结果，加入搜索结果中。

```java
/**
 * @param keyword String 文本消息的搜索关键字
 * @param fromAccounts List<String> 消息说话者帐号列表，如果消息说话在该列表中，
 * 那么无需匹配 keyword，对应的消息记录会直接加入搜索结果中。
 * @param anchor IMMessage 搜索的消息锚点
 * @param limit int 搜索结果的条数限制
 * @return 调用跟踪，可设置回调函数，接收查询结果
 */
 NIMClient.getService(MsgService.class).searchMessageHistory(keyword, fromAccounts, anchor, limit)
	 .setCallback(new RequestCallbackWrapper<List<IMMessage>>(){ ... });
```

此外，SDK 还提供了按照关键字全局搜索聊天记录的接口：MsgService#searchAllMessageHistory，用法与上述类似。目前暂不提供索引方式的全文检索功能，该全局搜索接口需要开发者评估大数据情况下的性能开销。

- 删除消息记录：

```java
// 删除单条消息
NIMClient.getService(MsgService.class).deleteChattingHistory(message);
// 删除与某个聊天对象的全部消息记录
NIMClient.getService(MsgService.class).clearChattingHistory(account, sessionType);
```

> 注意：当调用 MsgService#clearChattingHistory 接口删除与某个对象的全部聊天记录后，最近会话列表（最近联系人列表）中对应的项不会被移除，但会清空最近的消息内容，包括消息的时间显示（但 RecentContact 的 tag, extension 字段作为开发者的扩展字段，不会被清除。开发者可使用 tag, extension 进行最近会话列表显示定制）。如果需要移除最近会话项，请调用 MsgService#deleteRecentContact 接口。

- 通过消息 uuid 查询消息

```
// 通过uuid批量获取IMMessage(异步版本)
List<String> uuids = new ArrayList<>();
uuids.add(message.getUuid());
NIMClient.getService(MsgService.class).queryMessageListByUuid(uuids);

// 通过uuid批量获取IMMessage(同步版本)
NIMClient.getService(MsgService.class).queryMessageListByUuidBlock(uuids);
```

- 通过消息类型查询消息

通过消息类型从本地消息数据库中查询消息历史。查询范围由 msgTypeEnum 参数和 anchor 的 sessionId 决定。该接口查询方向为从后往前。以锚点 anchor 作为起始点（不包含锚点），往前查询最多 limit 条消息。

```
/**
* @param msgTypeEnum MsgTypeEnum 消息类型
* @param anchor IMMessage        搜索的消息锚点
* @param limit int               搜索结果的条数限制
* @return
*/
NIMClient.getService(MsgService.class).queryMessageListByType(msgTypeEnum, anchor, limit);
```

- 根据时间点搜索消息历史

该接口查询方向以某个时间点为基准从后往前，返回最多limit条匹配key的记录。该接口目前仅搜索文本类型的消息，匹配规则为文本内容包含keyword，仅支持精确匹配，不支持模糊匹配和拼音匹配。 由于sdk并不存储用户数据，因此keyword不会匹配用户资料。如果调用者希望查询指定用户的说话记录，可提供fromAccounts参数。如果提供的fromAccounts参数不为空，那么凡是消息说话者在fromAccounts列表中的记录，也会被当做匹配结果，加入搜索结果中。

```
/**
* @param keyword      文本消息的搜索关键字
* @param fromAccounts 消息说话者帐号列表，如果消息说话在该列表中，那么无需匹配keyword，对应的消息记录会直接加入搜索结果集中。
* @param time         查询范围时间点，比time小（从后往前查）
* @param limit        搜索结果的条数限制
* @return InvocationFuture
*/
NIMClient.getService(MsgService.class).searchAllMessageHistory(keyword, fromAccounts, time, limit)
	.setCallback(new RequestCallbackWrapper<List<IMMessage>>(){ ... });
```


### <span id="云端记录">云端记录</span>

网易云信还提供云端消息记录的功能，根据应用选择的套餐类型，保存不同时间长度范围的云端消息记录。SDK 提供了查询云端消息历史的接口，查询以锚点 anchor 作为起始点（不包含锚点），根据 direction 参数，往前或往后查询由锚点到 toTime 之间的最多 limit 条消息。查询范围由 toTime 和 limit 共同决定，以先到为准。如果到 toTime 之间消息大于 limit 条，返回 limit 条记录，如果小于 limit 条，返回实际条数。当已经查询到头时，返回的结果列表的 size 可能会比 limit 小。当进行首次查询时，锚点可以用使用 `MessageBuilder#createEmptyMessage` 接口生成。查询结果不包含锚点。该接口的最后一个参数可用于控制是否要将拉取到的消息记录保存到本地，如果选择保存到了本地，在使用拉取本地消息记录的接口时，也能取到这些数据。

```java
/**
 * 从服务器拉取消息历史记录。
 * @param anchor    IMMessage 起始时间点的消息，不能为 null。
 * @param toTime    long 结束时间点单位毫秒
 * @param limit     int 本次查询的消息条数上限(最多 100 条)
 * @param direction QueryDirectionEnum 查询方向，
 * QUERY_OLD 按结束时间点逆序查询，逆序排列；
 * QUERY_NEW 按起始时间点正序起查，正序排列
 * @param persist   boolean 通过该接口获取的漫游消息记录，要不要保存到本地消息数据库。
 * @return InvocationFuture
 */
NIMClient.getService(MsgService.class).pullMessageHistoryEx(anchor, toTime, limit, direction, persist);
```

上面是一个全接口，参数比较复杂。如果只需要类似于在聊天窗口下拉刷新的效果，可以使用下面这个简单版本：

```java
/**
 * 从服务器拉取消息历史记录。该接口查询方向为从后往前。以锚点 anchor 作为起始点（不包含锚点），
 * 往前查询最多 limit 条消息。
 * @param anchor    IMMessage 查询锚点。
 * @param limit     int 本次查询的消息条数上限(最多 100 条)
 * @param persist   boolean 通过该接口获取的漫游消息记录，要不要保存到本地消息数据库。
 * @return InvocationFuture
 */
NIMClient.getService(MsgService.class).pullMessageHistory(anchor, limit, persist);
```

## <span id="语音录制及回放">语音录制及回放</span>

### <span id="录制">录制</span>

网易云信 SDK 提供了一套录制高清语音的接口 `AudioRecorder` ，用于采集，编码，存储高清语音数据，并提供过程回调，供开发者进行自由的界面展现。
Recorder 使用示例代码如下：

```java

// 定义录音过程回调对象
IAudioRecordCallback callback = new IAudioRecordCallback () {

	void onRecordReady() {
		// 初始化完成回调，提供此接口用于在录音前关闭本地音视频播放（可选）
	}

    void onRecordStart(File audioFile, RecordType recordType) {
	    // 开始录音回调
    }

    void onRecordSuccess(File audioFile, long audioLength, RecordType recordType) {
	    // 录音结束，成功
    }

    void onRecordFail() {
	    // 录音结束，出错
    }

    void onRecordCancel() {
	    // 录音结束， 用户主动取消录音
    }

    void onRecordReachedMaxTime(int maxTime) {
	    // 到达指定的最长录音时间
    }
};
// 初始化recorder
AudioRecorder recorder = new AudioRecorder(
	context,
	RecordType.AAC, // 录制音频类型（aac/amr)
	maxDuration, // 最长录音时长，到该长度后，会自动停止录音
	callback // 录音过程回调
	);
// 开始录音
if (!recorder.startRecord()) {
	// 开启录音失败。
}
// 结束录音, 正常结束，或者取消录音
recorder.completeRecord(cancel);
```

- 在录音过程中可以获取当前录音时最大振幅（40ms更新一次数据），接口为 `AudioRecorder#getCurrentRecordMaxAmplitude` 。

### <span id="回放">回放</span>

网易云信的语音消息格式有 aac 和 amr 两种格式可选，由于 2.x 系统的原生 MediaPlayer 不支持 aac 格式，因此 SDK 也提供了一个 AudioPlayer 来播放网易云信的语音消息。同时，将 MediaPlayer 的接口进行了一些封装，使得在会话场景下播放语音更加方便。
使用示例代码如下：

```java
// 定义一个播放进程回调类
OnPlayListener listener = new OnPlayListener() {

    // 音频转码解码完成，会马上开始播放了
    public void onPrepared() {}

    // 播放结束
    public void onCompletion() {}

    // 播放被中断了
    public void onInterrupt() {}

    // 播放过程中出错。参数为出错原因描述
    public void onError(String error){}

    // 播放进度报告，每隔 500ms 会回调一次，告诉当前进度。 参数为当前进度，单位为毫秒，可用于更新 UI
    public void onPlaying(long curPosition) {}
};

// 构造播放器对象
AudioPlayer player = new AudioPlayer(context, filePath, listener);

// 开始播放。需要传入一个 Stream Type 参数，表示是用听筒播放还是扬声器。取值可参见
// android.media.AudioManager#STREAM_***
// AudioManager.STREAM_VOICE_CALL 表示使用听筒模式
// AudioManager.STREAM_MUSIC 表示使用扬声器模式
player.start(streamType);

// 如果中途切换播放设备，重新调用 start，传入指定的 streamType 即可。player 会自动停止播放，然后再以新的 streamType 重新开始播放。
// 如果需要从中断的地方继续播放，需要外面自己记住已经播放过的位置，然后在 onPrepared 回调中调用 seekTo
player.seekTo(pausedPosition);

// 主动停止播放
player.stop();
```

## <span id="群组功能">群组功能</span>

### <span id="群组功能概述">群组功能概述</span>

网易云信 SDK 提供了普通群 (`TeamTypeEnum#Normal`)，以及高级群 (`TeamTypeEnum#Advanced`)两种形式的群聊功能。高级群拥有更多的权限操作，两种群聊形式在共有操作上保持了接口一致。在群组中，当前会话的 ID 就是群组的 ID。

- 普通群

开发手册中所提及的普通群都等同于 Demo 中的讨论组。普通群没有权限操作，适用于快速创建多人会话的场景。每个普通群只有一个管理员。管理员可以对普通群进行增减员操作，普通成员只能对普通群进行增员操作。在添加新成员的时候，并不需要经过对方同意。

- 高级群

高级群在权限上有更多的限制，权限分为群主、管理员、以及群成员。2.4.0之前版本在添加成员的时候需要对方接受邀请；2.4.0版本之后，可以设定被邀请模式（是否需要对方同意）。高级群可以覆盖所有普通群的能力，建议开发者创建时选用高级群。

### <span id="群聊消息">群聊消息</span>

群聊消息收发和管理和单人聊天完全相同，仅在 `SessionTypeEnum` 上做了区分，详见[基础消息功能](#基础消息功能)节。

### <span id="关闭群聊消息提醒">关闭群聊消息提醒</span>

群聊消息提醒可以单独打开或关闭，关闭提醒之后，用户仍然会收到这个群的消息。如果开发者使用的是云信内建的消息提醒，用户收到新消息后不会再用通知栏提醒，如果用户使用的 iOS 客户端，则他将收不到该群聊消息的 APNS 推送。如果开发者自行实现状态栏提醒，可通过 `Team` 的 `mute` 接口获取提醒配置，并决定是不是要显示通知。群聊消息提醒设置可以漫游。
开发者可通过调用一下接口打开或关闭群聊消息提醒：

```java
NIMClient.getService(TeamService.class).muteTeam(teamId, mute);
```

### <span id="获取群组">获取群组</span>

SDK 提供了两个获取自己加入的所有群的列表的接口，一个是获取所有群（包括高级群和普通群）的接口，另一个是根据类型获取列表的接口。开发者可根据实际产品需求选择使用。

注意：这里获取的是所有我加入的群列表（退群、被移除群后，将不在返回列表中），如果需要自己退群或者被移出群的群资料，请使用下面的 queryTeam 接口。

获取所有我加入的群：

```java
NIMClient.getService(TeamService.class).queryTeamList()
	.setCallback(new RequestCallbackWrapper<List<Team>>() { ... });
```

也可以直接使用这个函数的同步版本：

```java
List<Team> teams = NIMClient.getService(TeamService.class).queryTeamListBlock();
```

按照类型获取自己加入的群列表：

```java
NIMClient.getService(TeamService.class).queryTeamListByType(type)
	.setCallback(new RequestCallback<List<Team>>() { ... });
```

根据群ID查询群资料：

如果本地没有群组资料，则去服务器查询。如果自己不在这个群中，该接口返回的可能是过期资料，如需最新的，请调用 `searchTeam` 接口去服务器查询。 此外 queryTeam 接口也有同步版本： queryTeamBlock 。

```java
NIMClient.getService(TeamService.class).queryTeam(teamId).setCallback(new RequestCallbackWrapper<Team>() {
    @Override
    public void onResult(int code, Team t, Throwable exception) {
        ...
    }
});
```

群资料 SDK 本地存储说明：

1\. 解散群、退群或者被移出群时，本地数据库会继续保留这个群资料，只是设置了无效标记，此时依然可以通过 queryTeam 查出来该群资料，只是 isMyTeam 返回 false 。 如果用户手动清空全部本地数据，下次登录同步时，服务器不会将无效的群资料同步过来，将无法取得已退出群的群资料。

2\. 群解散后，通过 `searchTeam` 接口从服务器查询将返回 null 。

### <span id="创建群组">创建群组</span>

网易云信群组分为两类：普通群和高级群，两种群组的消息功能都是相同的，区别在于管理功能。普通群所有人都可以拉人入群，除群主外，其他人都不能踢人；固定群则拥有完善的成员权限体系及管理功能。创建群的接口相同，传入不同的类型参数即可。

注意：群扩展字段最大长度为1024字节，若超限，服务器将返回414。

```java
// 群组类型
TeamTypeEnum type = TeamTypeEnum.Advanced;
// 创建时可以预设群组的一些相关属性，如果是普通群，仅群名有效。
// fields 中，key 为数据字段，value 对对应的值，该值类型必须和 field 中定义的 fieldType 一致
HashMap<TeamFieldEnum, Serializable> fields = new HashMap<TeamFieldEnum, Serializable>();
fields.put(TeamFieldEnum.Name, teamName);
fields.put(TeamFieldEnum.Introduce, teamIntroduce);
fields.put(TeamFieldEnum.VerifyType, verifyType);
NIMClient.getService(TeamService.class).createTeam(fields, type, "", accounts)
     setCallback(new RequestCallback<Team> { ... });
```

### <span id="加入群组">加入群组</span>

```java
NIMClient.getService(TeamService.class)
	.applyJoinTeam(teamId, reason)
	.setCallback(new RequestCallback<Team>() { ... });
```

### <span id="解散群组">解散群组</span>

高级群的群主可以解散群：

```java
NIMClient.getService(TeamService.class).dismissTeam(teamId)
	.setCallback(new RequestCallback<Void>() { ... });
```

### <span id="拉人入群">拉人入群</span>

普通群所有人都可以拉人入群，SDK 2.4.0之前版本高级群仅管理员和拥有者可以邀请人入群， SDK 2.4.0及以后版本高级群在创建时可以设置群邀请模式，支持仅管理员或者所有人均可拉人入群。

```java
NIMClient.getService(TeamService.class).addMembers(teamId, accounts)
    .setCallback(new RequestCallback<Void>() {
            @Override
            public void onSuccess(Void param) {
				// 返回onSuccess，表示拉人不需要对方同意，且对方已经入群成功了
            }

            @Override
            public void onFailed(int code) {
	            // 返回onFailed，并且返回码为810，表示发出邀请成功了，但是还需要对方同意
            }

            @Override
            public void onException(Throwable exception) {
				...
            }
        });
```

普通群可直接将用户拉入群聊，拉人成功，直接返回onSuccess。

高级群不能直接拉入，发出邀请成功会返回onFailed，并且返回码为810（这是一个特例，与其他接口成功直接返回 onSuccess 有所不同）。

被邀请的人会收到一条系统通知 (`SystemMessage`)，类型为 `SystemMessageType#TeamInvite`。用户接受该邀请后，才真正入群。可以通过 `SystemMessage#getAttachObject`  获取服务器设置的扩展字段，类型为TeamInviteNotify。

用户入群（普通群被拉人，高级群接受邀请）后，在收到之后的第一条消息时，群内所有成员（包括入群者）会收到一条入群消息，附件类型为 `MemberChangeAttachment`。若通知类型为`NotificationType#InviteMember`，可以通过`MemberChangeAttachment#getExtension` 获取服务器设置的扩展字段。

### <span id="踢人出群">踢人出群</span>

普通群仅拥有者可以踢人，高级群拥有者和管理员可以踢人，且管理员不能踢拥有者和其他管理员。

```java
NIMClient.getService(TeamService.class).removeMember(teamId, account)
	.setCallback(new RequestCallback<Void>() { ... });
```

踢人后，群内所有成员(包括被踢者)会收到一条消息类型为 `notification` 的 `IMMessage`，类型为 `NotificationType#KickMember`, 附件类型为 `MemberChangeAttachment`。可以通过`MemberChangeAttachment#getExtension` 获取服务器设置的扩展字段。

### <span id="主动退群">主动退群</span>

普通群群主可以退群，若退群，该群没有群主。高级群除群主外，其他用户均可以主动退群：

```java
NIMClient.getService(TeamService.class).quitTeam(teamId)
	.setCallback(new RequestCallback<Void>() { ... });
```

退群后，群内所有成员(包括退出者)会收到一条消息类型为 `notification` 的 `IMMessage`，附件类型为 `MemberChangeAttachment`。

### <span id="转让群组">转让群组</span>

高级群拥有者可以将群的拥有者权限转给群内的其他成员，转移后，被转让者变为新的拥有者，原拥有者变为普通成员。原拥有者还可以选择在转让的同时，直接退出该群。

```java
/**
 * 拥有者将群的拥有者权限转给另外一个人，转移后，另外一个人成为拥有者。
 *     原拥有者变成普通成员。若参数quit为true，原拥有者直接退出该群。
 * @param teamId 群ID
 * @param account 新任拥有者的用户帐号
 * @param quit 转移时是否要同时退出该群
 * @return InvocationFuture 可以设置回调函数，如果成功，视参数 quit 值：
 *     quit为false：参数仅包含原拥有着和当前拥有者的(即操作者和 account)，权限已被更新。
 *     quit为true: 参数为空。
 */
 NIMClient.getService(TeamService.class)
	 .transferTeam(teamId, account, quit)
	 .setCallback(new RequestCallback<List<TeamMember>>() { ... });
```

### <span id="验证入群邀请">验证入群邀请</span>

收到入群邀请后，用户可在系统通知中看到该邀请，并选择接受或拒绝：

```java
SystemMessage message;
RequestCallback<Void> callback;
// 接受邀请
NIMClient.getService(TeamService.class)
	.acceptInvite(message.getTargetId(), message.getFromAccount())
	.setCallback(callback);
// 拒绝邀请,可带上拒绝理由
NIMClient.getService(TeamService.class)
	.declineInvite(message.getTargetId(), message.getFromAccount(), reason)
	.setCallback(callback);
```

接受邀请后，用户真正入群。如果拒绝邀请，邀请该用户的管理员会收到一条系统通知，类型为 `SystemMessageType#DeclineTeamInvite`。

### <span id="验证入群申请">验证入群申请</span>

用户发出申请后，所有管理员都会收到一条系统通知，类型为 `SystemMessageType#TeamApply`。管理员可选择同意或拒绝：

```java
SystemMessage message;
RequestCallback<Void> callback;
// 同意申请
NIMClient.getService(TeamService.class)
	.passApply(message.getTargetId(), message.getFromAccount())
	.setCallback(callback);
// 拒绝申请，可填写理由
NIMClient.getService(TeamService.class)
	.rejectApply(message.getTargetId(), message.getFromAccount(), reason)
	.setCallback(callback);
```

任意一管理员操作后，其他管理员再操作都会失败。

如果同意入群申请，群内所有成员(包括申请者)都会收到一条消息类型为 `notification` 的 `IMMessage`，附件类型为 `MemberChangeAttachment`。

如果拒绝申请，申请者会收到一条系统通知，类型为 `SystemMessageType#RejectTeamApply`。

### <span id="编辑群组资料">编辑群组资料</span>

每次仅修改群的一个属性，可修改的属性查看 `TeamFieldEnum` ，包括群名，群头像，群简介，群公告，群扩展字段，申请加入群组的验证模式，群邀请模式，群被邀请模式，群资料修改模式，群资料扩展字段修改模式。注意： 其中的 `Ext_Server` 群服务器扩展字段，仅限服务端修改。

更新的value值，对应的数据类型，同样查看 `TeamFieldEnum` 。例如：群名 Name(ITeamService.TinfoTag.NAME, String.class)，表示 value 值的数据类型为 String。验证类型 VerifyType(ITeamService.TinfoTag.JOIN_MODE, VerifyTypeEnum.class)，表示 value 值的数据类型为VerifyTypeEnum。

更新群头像请注意: 需要先调用 `NosService#upload` 方法，将头像图片成功上传到 nos。再将 nos 的 url 地址更新到群资料。

```java
// 每次仅修改群的一个属性，可修改的属性包括：群名，介绍，公告，验证类型等。
NIMClient.getService(TeamService.class).updateTeam(teamId, TeamFieldEnum, value)
	.setCallback(new RequestCallback<Void>() {...});
```
当然，SDK 也支持批量更新群组资料，可一次性更新多个字段的值，见 `TeamService#updateTeamFields`。

修改后，群内所有成员会收到一条消息类型为 `notification` 的 `IMMessage`，带有一个消息附件，类型为 `UpdateTeamAttachment`。如果注册了群组资料变化观察者，观察者也会收到通知，见[监听群组资料变化](#监听群组资料变化)。

### <span id ="修改成员的群昵称">修改成员的群昵称</span>

群主和管理员修改群内其他成员的群昵称，仅群主和管理员拥有权限。

群主可以修改所有人的群昵称。管理员只能修改普通群成员的群昵称。

```
/**
* 修改成员的群昵称
* @param teamId  所在群组ID
* @param account 要修改的群成员帐号
* @param nick    新的群昵称
* @return InvocationFuture 可以设置回调函数，监听操作结果
*/
NIMClient.getService(TeamService.class).updateMemberNick(teamId, account, nick).setCallback(new RequestCallback<Void>() {...});
```

### <span id="修改自己的群昵称">修改自己的群昵称</span>

```
/**
* 修改自己的群昵称
* @param teamId  所在群组ID
* @param nick    新的群昵称
* @return InvocationFuture 可以设置回调函数，监听操作结果
*/
NIMClient.getService(TeamService.class).updateMyTeamNick(teamId, nick).setCallback(new RequestCallback<Void>() {...});
```

### <span id="修改自己的群成员扩展字段">修改自己的群成员扩展字段</span>

```
/**
* 修改自己的群成员扩展字段（自定义属性）
*
* @param teamId    所在群组ID
* @param extension 新的扩展字段（自定义属性）类型：Map<String,Object>
* @return InvocationFuture 可以设置回调函数，监听操作结果
*/
NIMClient.getService(TeamService.class).updateMyMemberExtension(teamId, extMap).setCallback(new RequestCallback<Void>() {...});
```
修改后，群成员会收到群成员资料变更通知。

### <span id="监听群组资料变化">监听群组资料变化</span>

由于获取群组资料需要跨进程异步调用，开发者最好能在第三方 APP 中做好群组资料缓存，查询群组资料时都从本地缓存中访问。在群组资料有变化时，SDK 会告诉注册的观察者，此时，第三方 APP 可更新缓存，并刷新界面。

```java
// 创建群组资料变动观察者
Observer<List<Team>> teamUpdateObserver = new Observer<List<Team>>() {
    @Override
    public void onEvent(List<Team> teams) {
    }
};
// 注册/注销观察者
NIMClient.getService(TeamServiceObserver.class).observeTeamUpdate(teamUpdateObserver, register);

// 创建群组被移除的观察者。在退群，被踢，群被解散时会收到该通知。
Observer<Team> teamRemoveObserver = new Observer<Team>() {
    @Override
    public void onEvent(Team team) {
    // 由于界面上可能仍需要显示群组名等信息，因此参数中会返回 Team 对象。
    // 该对象的 isMyTeam 接口返回为 false。
    }
};
// 注册/注销观察者
NIMClient.getService(TeamServiceObserver.class).observeTeamRemove(teamRemoveObserver, register);
```

群被解散后，群内所有成员都会收到一条消息类型为 `notification` 的 `IMMessage`，带有一个消息附件，类型为 `DismissAttachment`，原群主为该消息的 fromAccount。如果注册了群组被移除的观察者，这个观察者会收到通知。

### <span id="监听群成员资料变化">监听群成员资料变化</span>

由于获取群成员资料需要跨进程异步调用，开发者最好能在第三方 APP 中做好群成员资料缓存，查询群成员资料时都从本地缓存中访问。在群成员资料有变化时，SDK 会告诉注册的观察者，此时，第三方 APP 可更新缓存，并刷新界面。

```java
// 群成员资料变化观察者通知。群组添加新成员，成员资料变化会收到该通知。
// 返回的参数为有更新的群成员资料列表。
Observer<List<TeamMember>> memberUpdateObserver = new Observer<List<TeamMember>>() {
	@Override
	public void onEvent(List<TeamMember> members) {
	}
};
// 注册/注销观察者
NIMClient.getService(TeamServiceObserver.class).observeMemberUpdate(memberUpdateObserver, register);

// 移除群成员的观察者通知。
private Observer<TeamMember> memberRemoveObserver = new Observer<TeamMember>() {
	@Override
	public void onEvent(TeamMember member) {
	}
};
// 注册/注销观察者
NIMClient.getService(TeamServiceObserver.class).observeMemberRemove(memberRemoveObserver, register);
```

### <span id="管理群组权限">管理群组权限</span>

高级群中，拥有者可以增加和删除管理员。

```java
/**
 * 拥有者添加管理员
 * @param teamId 群 ID
 * @param accounts 待提升为管理员的用户帐号列表
 * @return InvocationFuture 可以设置回调函数,如果成功，参数为新增的群管理员列表
 */
NIMClient.getService(TeamService.class)
	.addManagers(teamId, accounts)
	.setCallback(new RequestCallback<List<TeamMember>>() {
});
/**
 * 拥有者撤销管理员权限 <br>
 * @param teamId 群ID
 * @param managers 待撤销的管理员的帐号列表
 * @return InvocationFuture 可以设置回调函数，如果成功，参数为被撤销的群成员列表(权限已被降为Normal)。
 */
 NIMClient.getService(TeamService.class)
	.removeManagers(teamId, managers)
	.setCallback(new RequestCallback<List<TeamMember>>() {
});
```

操作完成后，群内所有成员都会收到一条消息类型为 `notification` 的 `IMMessage`，附件类型为 `MemberChangeAttachment`。

### <span id="群成员禁言">群成员禁言</span>

群组支持管理员对普通成员的禁言、解除禁言操作。

```java
/**
* 禁言、解除禁言
*
* @param teamId  群组ID
* @param account 被禁言、被解除禁言的账号
* @param mute    true表示禁言，false表示解除禁言
* @return InvocationFuture 可以设置回调函数，监听操作结果
*/
NIMClient.getService(TeamService.class).muteTeamMember(teamId, account, mute).setCallback(new RequestCallback<Void>(){...});
```
操作完成后，群内所有成员都会收到一条消息类型为 `notification` 的 `IMMessage`，附件类型为 `MuteMemberAttachment`，通过附件可以知道具体哪个用户被禁言或者解除禁言。

### <span id="获取群组成员">获取群组成员</span>

由于群组成员数据比较大，且除了进入群组成员列表界面外，其他地方均不需要群组成员列表的数据，因此 SDK 不会在登录时同步群组成员数据，而是按照按需获取的原则，当上层主动调用获取指定群的群组成员列表时，才判断是否需要同步。获取群组成员的示例代码如下：

```java
// 该操作有可能只是从本地数据库读取缓存数据，也有可能会从服务器同步新的数据，因此耗时可能会比较长。
NIMClient.getService(TeamService.class).queryMemberList(teamId)
	.setCallback(new RequestCallback<List<TeamMember>>() {
            @Override
            public void onSuccess(List<TeamMember> members) {
                updateTeamMember(members);
            }
        });
```

根据群ID和账号查询群成员资料：

如果本地群成员资料已过期， SDK 会去服务器获取最新的。 queryTeamMember 还有同步版本 queryTeamMemberBlock 。

```java
NIMClient.getService(TeamService.class).queryTeamMember(teamId, account)
    .setCallback(new RequestCallbackWrapper<TeamMember>() {
    @Override
    public void onResult(int code, TeamMember member, Throwable exception) {
        ...
    }
});
```

群成员资料 SDK 本地存储说明：
当自己退群、或者被移出群时，本地数据库会继续保留这个群成员资料，只是设置了无效标记，此时依然可以通过 queryTeamMember 查出来该群成员资料，只是 isInTeam 将返回 false 。

### <span id="查询高级群资料">查询高级群资料</span>

除了管理员邀请，用户也可以主动申请加入高级群。用户可以通过群号查询高级群信息：

```java
NIMClient.getService(TeamService.class).searchTeam(teamId)
	.setCallback(new RequestCallback<Team>() { ... });
```

### <span id="指定群成员强制推送">指定群成员强制推送</span>

群消息支持设置强制推送。强制推送优先级最高，

1、打开强推开关，即使关闭消息提醒，依然能收到通知栏提醒，但是声音和震动跟随本机设置。

2、打开强推开关，即使设置了 PC/Web 端在线时不推送，依然能收到通知栏提醒。

3、无论是否打开强推开关，设置了强推通知文本内容并且位于强推列表中的成员（若强推列表为 null，则表示本群所有成员），收到的消息提醒均显示强推通知文本。

4、强推列表可以配置需要强制推送的群成员，若为 null ，则表示该群所有群成员均收到强制推送消息。

5、打开强推开关，也打开免打扰设置。有通知栏提醒，但是没有声音和震动。

```
IMMessage message = MessageBuilder.createTextMessage(teamId, SessionTypeEnum.Team, content);
MemberPushOption option = new MemberPushOption();
option.setForcePush(true); // 打开强推
option.setForcePushContent(forcePushContent); // 设置强推通知文本内容
ArrayList<String> memberAccounts = new ArrayList<>(); 
memberAccounts.add("uid");
option.setForcePushList(memberAccounts); // 强推需要通知的帐号列表，null表示所有群成员
message.setMemberPushOption(option); // 设置强推
NIMClient.getService(MsgService.class).sendMessage(message, false);
```

## <span id="聊天室">聊天室</span>

聊天室模型特点：

- 进入聊天室时必须建立新的连接，退出聊天室或者被踢会断开连接，在聊天室中掉线会有自动重连，开发者需要监听聊天室连接状态来做出正确的界面表现。
- 支持聊天人数无上限。
- 聊天室只允许用户手动进入，无法进行邀请。
- 支持同时进入多个聊天室，会建立多个连接。
- 不支持多端进入同一个聊天室，后进入的客户端会将前一个踢掉。
- 断开聊天室连接后，服务器不会再推送该聊天室的消息给此用户。
- 聊天室成员分固定成员（固定成员有四种类型，分别是创建者,管理员,普通用户,受限用户。禁言用户和黑名单用户都属于受限用户。）和游客两种类型。

### <span id="进入聊天室">进入聊天室</span>

进入聊天室必填字段 roomId 。

进入聊天室可选字段：

1\. 用户扩展字段 extension ，进入聊天室后展示用户信息的扩展字段，长度限制4K 。设置后在获取聊天室成员信息 `ChatRoomMember` 时可以查询到此扩展字段；此外，在收到聊天室消息时，可以从 `ChatRoomMessage#ChatRoomMessageExtension#getSenderExtension` 中获取消息发送者的用户信息扩展字段。

2\. 通知的扩展字段 notifyExtension ，进入聊天室通知消息扩展字段，长度限制1K（进入聊天室后，聊天室成员都会收到一条通知消息）。设置后，在收到聊天室通知消息时的 `ChatRoomNotificationAttachment` 中可以查询到此扩展字段。

```java
EnterChatRoomData data = new EnterChatRoomData(roomId);
        NIMClient.getService(ChatRoomService.class).enterChatRoom(data)
                .setCallback(new RequestCallback<EnterChatRoomResultData>() {...});
```

如果聊天室不存在，那么上述接口将回调 onFailed ，错误码404（即 roomId 不存在）
> 注意：当进入聊天室后，再发生掉线问题时，SDK会自动进行重连，无需开发者再次调用进入聊天室接口。
> 注意：进入聊天室前，必须先成功登录 IM，否则会登录失败，并上报错误码1000。

SDK 有聊天室断线重连机制，会在网络恢复后做重连并自动登录，如果自动登录失败，则会有在线状态变更通知，见[监听聊天室在线状态](#监听聊天室在线状态)。
如果为在线状态变更为 UNLOGIN 则表示自动登录失败（例如账号被群主拉黑，聊天室状态异常等），那么在状态变更观察者回调中可以调用 SDK 接口获取服务器返回的失败原因，并在界面上做相应的处理（比如退出聊天室）。

进入聊天室错误码主要有：
414:参数错误
404:聊天室不存在
403:无权限
500:服务器内部错误 
13001:IM主连接状态异常 
13002:聊天室状态异常 
13003:黑名单用户禁止进入聊天室 

```java
/**
* 获取进入聊天室失败的错误码
* 如果是手动登录，在 enterChatRoom 的回调函数中已有错误码。
* 如果是断网重连，在自动登录失败时，即监听到在线状态变更为 UNLOGIN 时，可以采用此接口查看具体自动登录失败的原因，如果是13001，13002，13003，403，404，414错误，此时应该调用离开聊天室接口。
*/
int errorCode = NIMClient.getService(ChatRoomService.class).getEnterErrorCode(roomId));
```

### <span id="离开聊天室">离开聊天室</span>

离开聊天室，会断开聊天室对应的链接，并不再收到该聊天室的任何消息。如果用户要离开聊天室，可以手动调用离开聊天室接口，该接口没有回调。

```java
NIMClient.getService(ChatRoomService.class).exitChatRoom(roomId);
```
如果聊天室被解散，会收到被踢出的通知。

### <span id="发送消息">发送消息</span>

先通过 `ChatRoomMessageBuilder` 提供的接口创建消息对象，然后调用 `ChatRoomService` 的 `sendMessage` 接口发送出去即可。

聊天室消息 `ChatRoomMessage` 继承自 `IMMessage` 。新增了两个方法：分别是设置和获取聊天室消息扩展属性。

下面给出发送文本类型消息的示例代码：

```java
// 创建文本消息
ChatRoomMessage message = ChatRoomMessageBuilder.createChatRoomTextMessage(
	sessionId, // 聊天室id
	content // 文本内容
	);

// 发送消息。如果需要关心发送结果，可设置回调函数。发送完成时，会收到回调。如果失败，会有具体的错误码。
NIMClient.getService(ChatRoomService.class).sendMessage(message);
```
其它类型的消息发送方式与会话消息类似，请参考[发送消息](#发送消息) 一节。

### <span id="接收消息">接收消息</span>

通过添加消息接收观察者，在有新消息到达时，第三方 APP 就可以接收到通知。代码如下：

```java
Observer<List<ChatRoomMessage>> incomingChatRoomMsg = new Observer<List<ChatRoomMessage>>() {
        @Override
        public void onEvent(List<ChatRoomMessage> messages) {
            // 处理新收到的消息
        }
    };

NIMClient.getService(ChatRoomServiceObserver.class)
	.observeReceiveMessage(incomingChatRoomMsg, register);
```

### <span id="聊天室通知消息">聊天室通知消息</span>

聊天室通知消息是聊天室消息的一种（消息类型为 Notification）。即聊天室消息为 `ChatRoomMessage`， 其中附件类型为 `ChatRoomNotificationAttachment` ，附件类型中的 type 字段来标识聊天室通知消息的类型。目前支持的类型见 `NotificationType` 。

### <span id="获取历史消息">获取历史消息</span>

聊天室支持获取云端消息记录的功能。以 startTime（单位毫秒） 为时间戳，拉取 limit 条消息。拉取到的消息中也包含成员操作的通知消息。

```java
/**
 * 获取历史消息
 *
 * @param roomId    聊天室id
 * @param startTime 时间戳，单位毫秒
 * @param limit     可拉取的消息数量
 * @return InvocationFuture 可以设置回调函数。回调中返回历史消息列表
 */
NIMClient.getService(ChatRoomService.class).pullMessageHistory(roomId, 0, 10)
	.setCallback(new RequestCallbackWrapper<List<IMMessage>>() {
	@Override
	public void onResult(int code, List<IMMessage> messages, Throwable exception) {
		if (messages != null) {
			messageListPanel.onLoadHistoryMessage(messages);
		}
	}
});
```

### <span id="获取聊天室基本信息">获取聊天室基本信息</span>

此接口可以向服务器获取聊天室信息。SDK 不提供聊天室信息的缓存，开发者可根据需要自己做缓存。聊天室基本信息的扩展字段需要在服务器端设置，客户端 SDK 只提供查询。

```java
NIMClient.getService(ChatRoomService.class)
	.fetchRoomInfo(roomId).setCallback(new RequestCallback<ChatRoomInfo>() { ... });
```

### <span id="获取聊天室成员">获取聊天室成员</span>

#### <span id="获取成员信息">获取成员信息</span>

该接口可以获取固定成员信息、仅在线的固定成员信息及游客信息 `ChatRoomMember`，其中的扩展字段由该用户进入聊天室时填写。

固定成员有四种类型，分别是创建者,管理员,普通用户,受限用户。禁言用户和黑名单用户都属于受限用户。

拉取固定成员时，无论成员在线不在线，都会返回。拉取游客时，只返回在线的成员。

```java
/**
 * 获取聊天室成员信息
 *
 * @param roomId          聊天室id
 * @param memberQueryType 成员查询类型。见{@link MemberQueryType}
 * @param time            查询固定成员列表用ChatRoomMember.getUpdateTime,
 *                        查询游客列表用ChatRoomMember.getEnterTime，
 *                        填0会使用当前服务器最新时间开始查询，即第一页，单位毫秒
 * @param limit           条数限制
 * @return InvocationFuture 可以设置回调函数。回调中返回成员信息列表
 */
NIMClient.getService(ChatRoomService.class)
	.fetchRoomMembers(roomId, memberQueryType, time, limit)
	.setCallback(new RequestCallbackWrapper<List<ChatRoomMember>>() {
	@Override
	public void onResult(int code, List<ChatRoomMember> result, Throwable exception) { ... });
```

#### <span id="批量获取成员信息">批量获取成员信息</span>

通过用户 id 批量获取指定成员在聊天室中的信息。

```java
List<String> accounts = new ArrayList<>();
accounts.add(account);

/**
 * 根据用户id获取聊天室成员信息
 *
 * @param roomId   聊天室id
 * @param accounts 成员帐号列表
 * @return InvocationFuture 可以设置回调函数。回调中返回成员信息列表
 */
NIMClient.getService(ChatRoomService.class)
	.fetchRoomMembersByIds(roomId, accounts)
	.setCallback(new RequestCallbackWrapper<List<ChatRoomMember>>() { ... });
```

### <span id="监听聊天室在线状态">监听聊天室在线状态</span>

进入聊天室成功后，SDK 会负责维护与服务器的长连接以及断线重连等工作。当用户在线状态发生改变时，会发出通知。登录过程也有状态回调。此外，网络连接上之后，SDK 会负责聊天室的自动登录。开发者可以通过加入以下代码监听聊天室在线状态改变：

```java
NIMClient.getService(ChatRoomServiceObserver.class)
	.observeOnlineStatus(onlineStatus, register);

Observer<ChatRoomStatusChangeData> onlineStatus
	= new Observer<ChatRoomStatusChangeData>() {
        @Override
        public void onEvent(ChatRoomStatusChangeData data) {
            if (data.status == StatusCode.UNLOGIN) {      
                int errorCode = NIMClient.getService(ChatRoomService.class).getEnterErrorCode(roomId));
                // 如果遇到错误码13001，13002，13003，403，404，414，表示无法进入聊天室，此时应该调用离开聊天室接口。
            } 
            ...
        }
    };
```

### <span id="成员操作">成员操作</span>

#### <span id="拉黑">拉黑</span>

- 加入黑名单时，会收到类型为 `ChatRoomMemberBlackAdd` 聊天室通知消息。
- 移出黑名单时，会收到类型为 `ChatRoomMemberBlackRemove` 的聊天室通知消息。

```java
MemberOption option = new MemberOption(roomId, account);
/**
 * 添加/移出聊天室黑名单
 *
 * @param isAdd        true:添加, false:移出
 * @param memberOption 请求参数，包含聊天室id，帐号id以及可选的扩展字段
 * @return InvocationFuture 可以设置回调函数。回调中返回成员信息
 */
NIMClient.getService(ChatRoomService.class)
	.markChatRoomBlackList(!chatRoomMember.isInBlackList(), option)
	.setCallback(new RequestCallback<ChatRoomMember>() { ... });
```

#### <span id="禁言">禁言</span>

- 添加到禁言名单时, 会收到类型为 `ChatRoomMemberMuteAdd` 的聊天室通知消息。
- 取消禁言时, 会收到类型为 `ChatRoomMemberMuteRemove` 的聊天室通知消息。

取消禁言之后，恢复为原来的身份。原来是游客，取消禁言后，变为游客。原来是普通成员，取消禁言后，变为普通成员。

```java
MemberOption option = new MemberOption(roomId, account);

/**
 * 添加到禁言名单/取消禁言
 *
 * @param isAdd        true:添加, false:取消
 * @param memberOption 请求参数，包含聊天室id，帐号id以及可选的扩展字段
 * @return InvocationFuture 可以设置回调函数。回调中返回成员信息
 */
NIMClient.getService(ChatRoomService.class)
	.markChatRoomMutedList(!chatRoomMember.isMuted(), option)
	.setCallback(new RequestCallback<ChatRoomMember>() { ... });
```

#### <span id="临时禁言">临时禁言</span>

- 设置临时禁言时, 会收到类型为 `ChatRoomMemberTempMuteAdd` 的聊天室通知消息。
- 取消临时禁言时, 会收到类型为 `ChatRoomMemberTempMuteRemove` 的聊天室通知消息。

聊天室支持设置临时禁言，禁言时长时间到了，自动取消禁言。设置临时禁言成功后的通知消息中，包含的时长是禁言剩余时长。若设置禁言时长为0，表示取消临时禁言。若第一次设置的禁言时长还没结束，又设置第二次临时禁言，以第二次设置的时间开始计时。

```
/**
* 设置聊天室成员临时禁言
* @param needNotify 是否需要发送广播通知，true：通知，false：不通知
* @param duration  禁言时长,单位秒
* @param memberOption 请求参数，包含聊天室id，帐号id以及可选的扩展字段
* @return InvocationFuture 可以设置回调函数。如果出错，会有具体的错误代码。
*/
NIMClient.getService(ChatRoomService.class)
	.markChatRoomTempMute(needNotify, Long.parseLong(content), new MemberOption(roomId, account))
	.setCallback(new RequestCallback<Void>() {...});
```

#### <span id="设置管理员">设置管理员</span>

- 设为管理员时, 会收到类型为 `ChatRoomManagerAdd` 的聊天室通知消息。
- 取消管理员时, 会收到类型为 `ChatRoomManagerRemove` 的聊天室通知消息。

如果游客被设为管理员，再被取消管理员，该成员不在变为游客，而成为普通成员。

```java
/**
 * 设为管理员/取消管理员
 *
 * @param isAdd        true:设为, false:取消
 * @param memberOption 请求参数，包含聊天室id，帐号id以及可选的扩展字段
 * @return InvocationFuture 可以设置回调函数。回调中返回成员信息
 */
NIMClient.getService(ChatRoomService.class)
	.markChatRoomManager(!isAdmin, new MemberOption(roomId, member.getAccount()))
	.setCallback(new RequestCallback<ChatRoomMember>() { ... });
```

#### <span id="设置普通成员">设置普通成员</span>

即将游客变为固定成员中的普通成员身份。

- 设为普通成员时, 会收到类型为 `ChatRoomCommonAdd` 的聊天室通知消息。
- 移除普通成员时, 会收到类型为 `ChatRoomCommonRemove` 的聊天室通知消息。

```java
 /**
 * 设为/取消聊天室普通成员
 * @param isAdd         true:设为, false:取消
 * @param memberOption  请求参数，包含聊天室id，帐号id以及可选的扩展字段
 * @return InvocationFuture 可以设置回调函数。回调中返回成员信息
 */
NIMClient.getService(ChatRoomService.class)
	.markNormalMember(!isNormal, new MemberOption(roomId, member.getAccount()))
	.setCallback(new RequestCallback<ChatRoomMember>() { ... });
```

#### <span id="踢出聊天室">踢出聊天室</span>

踢出成员。仅管理员可以踢；如目标是管理员仅创建者可以踢。

可以添加被踢通知扩展字段，这个字段会放到被踢通知的扩展字段中。通知扩展字段最长1K；扩展字段需要传入 Map<String,Object>， SDK 会负责转成Json String。

当有人被踢出聊天室时，会收到类型为 `ChatRoomMemberKicked` 的聊天室通知消息。

```java
Map<String, Object> reason = new HashMap<>();
reason.put("reason", "kick");
NIMClient.getService(ChatRoomService.class)
	.kickMember(roomId, chatRoomMember.getAccount(), reason)
	.setCallback(new RequestCallback<Void>() { ... });
```

### <span id="监听被踢出聊天室">监听被踢出聊天室</span>

当用户被主播或者管理员踢出聊天室、聊天室被关闭（被解散），会收到通知。注意：收到被踢出通知后，不需要再调用退出聊天室接口，SDK 会负责聊天室的退出工作。可以在踢出通知中做相关缓存的清理工作和界面操作。开发者可以通过加入以下代码监听是否被踢出聊天室:

```java
NIMClient.getService(ChatRoomServiceObserver.class)
	.observeKickOutEvent(kickOutObserver, register);

Observer<ChatRoomKickOutEvent> kickOutObserver = new Observer<ChatRoomKickOutEvent>() {
        @Override
        public void onEvent(ChatRoomKickOutEvent chatRoomKickOutEvent) {
            // 提示被踢出的原因（聊天室已解散、被管理员踢出、被其他端踢出等）
            // 清空缓存数据
        }
    };
```

### <span id="更新聊天室信息">更新聊天室信息</span>

支持更新聊天室信息。可以设置更新是否通知，若设置通知，聊天室内会收到类型为`NotificationType#ChatRoomInfoUpdated`的消息。

```
/**
* 更新聊天室信息
*
* @param roomId             聊天室id
* @param chatRoomUpdateInfo 可更新的聊天室信息
* @param needNotify         是否通知
* @param notifyExtension    更新聊天室信息扩展字段，这个字段会放到更新聊天室信息通知的扩展字段中
* @return InvocationFuture 可以设置回调函数。更新聊天室信息完成之后才会调用，如果出错，会有具体的错误代码。
*/
NIMClient.getService(ChatRoomService.class)
	.updateRoomInfo(roomId, chatRoomUpdateInfo, needNotify, notifyExtension)
	.setCallback(new RequestCallback<Void>() { ... });
```

### <span id="更新本人聊天室成员信息">更新本人聊天室成员信息</span>

支持更新本人聊天室成员信息。目前只支持昵称，头像和扩展字段的更新。可以设置更新是否通知，若设置通知，聊天室内会收到类型为`NotificationType#ChatRoomMyRoomRoleUpdated`的消息。

```
/**
* 更新本人在聊天室内的信息
*
* @param roomId               聊天室id
* @param chatRoomMemberUpdate 可更新的本人角色信息
* @param needNotify           是否通知
* @param notifyExtension      更新聊天室信息扩展字段，这个字段会放到更新聊天室信息通知的扩展字段中
* @return InvocationFuture     可以设置回调函数。更新信息完成之后才会调用，如果出错，会有具体的错误代码。
*/
NIMClient.getService(ChatRoomService.class)
	.updateMyRoomRole(roomId, chatRoomMemberUpdate, needNotify, notifyExtension)
	.setCallback(new RequestCallback<Void>() { ... });
```

### <span id="队列服务">队列服务</span>

聊天室提供队列服务，针对直播连麦场景使用。

#### <span id="加入或更新队列元素">加入或更新队列元素</span>

> 注意：key是唯一键

当 key 不存在时候，调用该接口，表示加入新元素。
当 key 存在的时候，调用该接口，表示修改对应 key 的 value。

```
/**
* 聊天室队列服务：加入或者更新队列元素
* @param roomId 聊天室id
* @param key    新元素（或待更新元素）的唯一键
* @param value  新元素（待待更新元素）的内容
* @return InvocationFuture 可以设置回调函数。回调中返回操作成功或者失败具体的错误码。
*/
NIMClient.getService(ChatRoomService.class)
	.updateQueue(roomId, key, value)
    .setCallback(new RequestCallback<Void>() { ... });
```

#### <span id="取出队列元素">取出队列元素</span>

> 注意：key是唯一键

key 若为 null， 则表示取出头元素。若不为 null， 则表示取出指定元素。

```
/**
* 聊天室队列服务：取出队列头部或者指定元素
* @param roomId 聊天室id
* @param key    需要取出的元素的唯一键。若为 null，则表示取出队头元素
* @return InvocationFuture 可以设置回调函数。如果元素存在或者队列不为空，
*                          那么回调中返回元素的键值对，否则返回失败错误码。
*/
NIMClient.getService(ChatRoomService.class)
	.pollQueue(roomId, key)
	.setCallback(new RequestCallback<Entry<String, String>>() { ... });
```

#### <span id="获取所有队列元素">获取所有队列元素</span>

```
/**
* 聊天室队列服务：列出所有队列元素
* @param roomId 聊天室id
* @return InvocationFuture 可设置回调函数。回调中返回所有排列的队列元素键值对。
*/
NIMClient.getService(ChatRoomService.class)
	.fetchQueue(roomId)
	.setCallback(new RequestCallback<List<Entry<String, String>>>() { ... });
```

#### <span id="删除队列">删除队列</span>

```
/**
* 聊天室队列服务：删除队列
* @param roomId 聊天室id
* @return InvocationFuture 可设置回调函数。回调中返回操作成功或者失败具体的错误码。
*/
NIMClient.getService(ChatRoomService.class)
	.dropQueue(roomId)
	.setCallback(new RequestCallback<Void>() { ... });
```

## <span id="系统通知">系统通知</span>

系统通知是网易云信系统内建的消息/通知，其对应的数据结构为 `SystemMessage`。由网易云信服务器推送给用户的通知类消息，用于网易云信系统类的事件通知。现在主要包括群变动的相关通知，例如入群申请，入群邀请等，如果第三方应用还托管了好友关系，好友的添加、删除也是这个类型的通知。系统通知由 SDK 负责接收和存储，并提供较简单的未读数管理。

### <span id="监听系统通知">监听系统通知</span>

开发者可通过 `SystemMessageObserver` 监听系统通知，包括系统通知的到达事件和未读数的变化。

- 监听系统通知的到达事件：

```java
NIMClient.getService(SystemMessageObserver.class)
	.observeReceiveSystemMsg(new Observer<SystemMessage>() {
            @Override
            public void onEvent(SystemMessage message) {
            }
        }, register);
```

- 监听未读数变化：

```java
NIMClient.getService(SystemMessageObserver.class)
    .observeUnreadCountChange(sysMsgUnreadCountChangedObserver, register);

private Observer<Integer> sysMsgUnreadCountChangedObserver = new Observer<Integer>() {
        @Override
        public void onEvent(Integer unreadCount) {
            ...
        }
    };
```

### <span id="管理系统通知">管理系统通知</span>

- 查询系统通知列表

```java
List<SystemMessage> temps = NIMClient.getService(SystemMessageService.class)
	.querySystemMessagesBlock(offset, limit); // 参数offset为当前已经查了offset条，limit为要继续查询limit条。
```
- 根据类型查询系统通知列表

需要传入系统消息类型 `SystemMessageType` 集合。

```java
List<SystemMessageType> types = new ArrayList<>();
types.add(SystemMessageType.AddFriend);

// 只查询“添加好友”类型的系统通知
List<SystemMessage> temps = NIMClient.getService(SystemMessageService.class)
    .querySystemMessageByType(types, loadOffset, LOAD_MESSAGE_COUNT);
```

- 设置系统通知状态

系统通知状态枚举见 `SystemMessageStatus`，目前除了提供了未处理、已通过、已拒绝、已忽略、已过期这五种状态之外，提供了五个自定义扩展类型，供第三方开发者使用。
在用户处理过系统通知之后，调用此函数更新系统通知状态。

```java
SystemMessageStatus status = SystemMessageStatus.expired;
NIMClient.getService(SystemMessageService.class)
     .setSystemMessageStatus(message.getMessageId(), status);
```

- 删除一条系统通知

```java
NIMClient.getService(SystemMessageService.class)
	.deleteSystemMessage(message.getMessageId());
```

- 删除所有系统通知

```java
NIMClient.getService(SystemMessageService.class).clearSystemMessages();
```

- 删除指定类型的系统通知

```java
List<SystemMessageType> types = new ArrayList<>();
types.add(SystemMessageType.AddFriend);

// 只删除“添加好友”类型的系统通知
NIMClient.getService(SystemMessageService.class).clearSystemMessagesByType(types);
```

- 查询系统通知未读数总和

`SystemMessage` 中属性 unread 用来标志该条系统通知是否未读，该函数将返回所有未读的系统通知总数。

```java
int unread = NIMClient.getService(SystemMessageService.class)
	.querySystemMessageUnreadCountBlock();
```

- 查询指定类型的系统通知未读数总和

```java
List<SystemMessageType> types = new ArrayList<>();
types.add(SystemMessageType.AddFriend);

// 查询“添加好友”类型的系统通知未读数总和
int unread = NIMClient.getService(SystemMessageService.class)
	.querySystemMessageUnreadCountByType(types);
```

- 设置单条系统通知为已读

```java
NIMClient.getService(SystemMessageService.class).setSystemMessageRead(messageId);
```

- 将所有系统通知设为已读

该函数调用后系统通知未读数将为零。

```java
// 进入过系统通知列表后，可调用此函数将未读数值为0
NIMClient.getService(SystemMessageService.class).resetSystemMessageUnreadCount();
```

- 将指定类型的系统通知设为已读接口

```java
List<SystemMessageType> types = new ArrayList<>();
types.add(SystemMessageType.AddFriend);

// 将“添加好友”类型的系统通知设为已读
NIMClient.getService(SystemMessageService.class).resetSystemMessageUnreadCount();
```

## <span id="自定义通知">自定义通知</span>

系统通知属于网易云信的体系内，如果第三方 APP 需要自己的系统通知，可使用自定义通知，其数据结构为 `CustomNotification`。

自定义通知提供的灵活性包括：
- 消息格式由第三方 APP 自己定义，只要内容是 `String` 就可以了。
- 第三方 APP 的客户端和服务器均可以发送自定义通知。
- 接收对象可以是个人，也可以是群组。
- 可设置通知的到达级别：保证必达，或是通知接收者只有当前在线才能收到。
- 如果需要向 iOS 用户推送，可自定义 iOS 的推送内容，可以自定义推送属性。

注意：自定义通知和自定义消息的不同之处在于，自定义消息归属于网易云信的消息体系内，适用于会话中，由 SDK 存储在消息数据库中，与网易云信的其他内建消息类型一同展现给用户。而自定义通知主要用于第三方的一些事件状态通知，网易云信不存储，也不解释这些通知，网易云信仅仅负责替第三方传递和通知这些事件，起到透传的作用。

### <span id="发送自定义通知">发送自定义通知</span>

通过 SDK 提供的接口，第三方 APP 可以在客户端向其他用户或者群组发送自定义通知。SDK 能发送的自定义通知主要分为两种。

一种是只有接收方当前在线才会收到，如果发送方发送时，指定的接收者不在线，这条通知将会被丢弃。在 demo 中，我们以此实现了"正在输入"这种状态的通知。

```java
// 构造自定义通知，指定接收者
CustomNotification notification = new CustomNotification();
notification.setSessionId(receiverId);
notification.setSessionType(sessionType);

// 构建通知的具体内容。为了可扩展性，这里采用 json 格式，以 "id" 作为类型区分。
// 这里以类型 “1” 作为“正在输入”的状态通知。
JSONObject json = new JSONObject();
json.put("id", "1");
notification.setContent(json.toString());

// 发送自定义通知
NIMClient.getService(MsgService.class).sendCustomNotification(notification);
```

另外一种保证接收方一定会收到，如果接收方当前在线，会立即收到，如果当前不在线，则在下次登录后立即收到。如果接收方上次登录是 iOS 版本，还会收到 APNS 推送通知。

下面做了一个简单的示例：

```java
// 构造自定义通知，指定接收者
CustomNotification notification = new CustomNotification();
notification.setSessionId(receiverId);
notification.setSessionType(sessionType);

// 构建通知的具体内容。为了可扩展性，这里采用 json 格式，以 "id" 作为类型区分。
JSONObject json = new JSONObject();
json.put("id", "2");
JSONObject data = new JSONObject();
data.put("body", "the_content_for_display");
data.put("url", "url_of_the_game_or_anything_else");
json.put("data", data);
notification.setContent(json.toString());

// 设置该消息需要保证送达
notification.setSendToOnlineUserOnly(false);

// 设置 APNS 的推送文本
notification.setApnsText("the_content_for_apns");

// 自定义推送属性
Map<String,Object> pushPayload = new HashMap<>();
pushPayload.put("key1", "payload 1");
pushPayload.put("key2", 2015);
notification.setPushPayload(pushPayload);

// 发送自定义通知
NIMClient.getService(MsgService.class).sendCustomNotification(notification);
```

发送自定义通知时还可以设置通知配置选项 `CustomNotificationConfig`，目前支持的配置选项有：

1\. enablePush ：该通知是否进行推送（消息提醒）。默认为 true 。

2\. enablePushNick ：该通知是否需要推送昵称（针对iOS客户端有效），如果为true，那么对方收到通知后，iOS端将不显示推送昵称。默认为 false 。

3\. enableUnreadCount ：该通知是否要计入未读数，如果为true，那么对方收到通知后，可以通过读取此配置项决定自己业务的未读计数变更。默认为 true 。

示例如下：

```java
CustomNotification command = new CustomNotification();
command.setSessionId(account);
command.setSessionType(sessionType);
CustomNotificationConfig config = new CustomNotificationConfig();
config.enablePush = false; // 不推送
config.enableUnreadCount = false;
command.setConfig(config);
...
NIMClient.getService(MsgService.class).sendCustomNotification(command);
```

### <span id="接收自定义通知">接收自定义通知</span>

上层有两种方式接收自定义通知，一是通过添加通知接收观察者的，二是通过广播的方式接收。SDK 从版本 1.4.0 开始，推荐使用第一种方式接收。从这个版本起，收到消息后就会激活 UI 主进程，并通知到已注册的观察者。只要在主进程的入口添加自定义通知的观察者，就能收到该通知。

添加自定义通知的接收观察者代码如下：

```java
// 如果有自定义通知是作用于全局的，不依赖某个特定的 Activity，那么这段代码应该在 Application 的 onCreate 中就调用
NIMClient.getService(MsgServiceObserve.class).observeCustomNotification(new Observer<CustomNotification>() {
    @Override
    public void onEvent(CustomNotification message) {
        // 在这里处理自定义通知。
    }
}, register);
```

如果使用广播接收者的方式，首先需要在 AndroidManifest.xml 文件中声明一个接收器：

```xml
<!-- 声明自定义通知的广播接收器，第三方 APP 集成时，action 中的 com.netease.nim.demo 请替换为自己的包名 -->
<!-- 需要权限声明 <uses-permission android:name="com.netease.nim.demo.permission.RECEIVE_MSG"/> -->
<receiver
    android:name=".receiver.CustomNotificationReceiver"
    android:enabled="true"
    android:exported="false">
    <intent-filter>
        <action android:name="com.netease.nim.demo.ACTION.RECEIVE_CUSTOM_NOTIFICATION"/>
    </intent-filter>
</receiver>
```

然后实现这个接收器：

```java
public class CustomNotificationReceiver extends BroadcastReceiver {

	@Override
	public void onReceive(Context context, Intent intent) {
		String action = context.getPackageName() + NimIntent.ACTION_RECEIVE_CUSTOM_NOTIFICATION;
		if (action.equals(intent.getAction())) {
			// 从 intent 中取出自定义通知， intent 中只包含了一个 CustomNotification 对象
			CustomNotification notification = (CustomNotification)
			intent.getSerializableExtra(NimIntent.EXTRA_BROADCAST_MSG);
			// 第三方 APP 在此处理自定义通知：存储，处理，展示给用户等
			Log.i("demo", "receive custom notification: " + notification.getContent()
			 + " from :" + notification.getSessionId() + "/" + notification.getSessionType());
		}
	}
}
```

## <span id="用户关系托管">用户关系托管</span>

网易云信提供了好友关系的托管，好友资料（用户资料）由第三方 APP 自行管理或者选择网易云信用户资料托管，见[用户资料托管](#用户资料托管)。

### <span id="好友关系">好友关系</span>

#### 添加好友

目前添加好友有两种验证类型（见 `VerifyType`）：直接添加为好友和发起好友验证请求。添加好友时需要构造 `AddFriendData`，需要填入包括对方帐号，好友验证类型及附言（可选）。代码示例如下：

```java
final VerifyType verifyType = VerifyType.VERIFY_REQUEST; // 发起好友验证请求
String msg = "好友请求附言";
NIMClient.getService(FriendService.class).addFriend(new AddFriendData(account, verifyType, msg))
	.setCallback(new RequestCallback<Void>() { ... });
```
添加好友请求发出后，对方会收到一条 `SystemMessage`,可以通过 SystemMessageObserver 的observeReceiveSystemMsg 函数来监听系统通知，通过 getAttachObject 函数可以获取添加好友的通知 `AddFriendNotify`，通知的事件类型见 `AddFriendNotify.Event`

```java
if (message.getType() == SystemMessageType.AddFriend) {
    AddFriendNotify attachData = (AddFriendNotify) message.getAttachObject();
    if (attachData != null) {
    	// 针对不同的事件做处理
        if (attachData.getEvent() == AddFriendNotify.Event.RECV_ADD_FRIEND_DIRECT) {
            // 对方直接添加你为好友
        } else if (attachData.getEvent() == AddFriendNotify.Event.RECV_AGREE_ADD_FRIEND) {
            // 对方通过了你的好友验证请求
        } else if (attachData.getEvent() == AddFriendNotify.Event.RECV_REJECT_ADD_FRIEND) {
    	    // 对方拒绝了你的好友验证请求
        } else if (attachData.getEvent() == AddFriendNotify.Event.RECV_ADD_FRIEND_VERIFY_REQUEST) {
            // 对方请求添加好友，一般场景会让用户选择同意或拒绝对方的好友请求。
            // 通过message.getContent()获取好友验证请求的附言
        }
    }
}
```
#### 通过/拒绝对方好友请求

收到好友的验证请求的系统通知后，可以通过或者拒绝，对方会收到一条系统通知，通知的事件类型为 AddFriendNotify.Event.RECV_AGREE_ADD_FRIEND 或者 AddFriendNotify.Event.RECV_REJECT_ADD_FRIEND

```java
NIMClient.getService(FriendService.class).ackAddFriendRequest(account, true); // 通过对方的好友请求
```

#### 删除好友

删除好友后，将自动解除双方的好友关系，双方的好友列表中均不存在对方。删除好友后，双方依然可以聊天。

```java
NIMClient.getService(FriendService.class).deleteFriend(account)
	.setCallback(new RequestCallback<Void>() { ... });
```

#### 监听好友关系的变化

第三方 APP 应在 APP 启动后监听好友关系的变化，当主动添加好友成功、被添加为好友、主动删除好友成功、被对方解好友关系、好友关系更新、登录同步好友关系数据时都会收到通知：

```java
NIMClient.getService(FriendServiceObserve.class).observeFriendChangedNotify(friendChangedNotifyObserver, true);
private Observer<FriendChangedNotify> friendChangedNotifyObserver = new Observer<FriendChangedNotify>() {
    @Override
    public void onEvent(FriendChangedNotify friendChangedNotify) {
        List<Friend> addedOrUpdatedFriends = friendChangedNotify.getAddedOrUpdatedFriends(); // 新增的好友
        List<String> deletedFriendAccounts = friendChangedNotify.getDeletedFriends(); // 删除好友或者被解除好友
        ...
    }
};
```

#### 获取好友列表

该方法是同步方法，返回我的好友帐号集合，可以根据帐号来获取对应的用户资料来构建自己的通讯录,见[构建通讯录](#构建通讯录)。

```java
List<String> friends = NIMClient.getService(FriendService.class).getFriendAccounts();
```

#### 根据用户账号获取好友关系

```java
Friend friend = NIMClient.getService(FriendService.class).getFriendByAccount("account");
```

#### 判断用户是否为我的好友

```java
boolean isMyFriend = NIMClient.getService(FriendService.class).isMyFriend(account);
```

#### 更新好友关系

目前支持更新好友的备注名和好友关系扩展字段，见 `FriendFieldEnum`。

注意：备注名最长128个字符；扩展字段需要传入 Map<String,Object>， SDK 会负责转成Json String，最大长度256字符。

```java
// 更新备注名
Map<FriendFieldEnum, Object> map = new HashMap<>();
map.put(FriendFieldEnum.ALIAS, content);
NIMClient.getService(FriendService.class).updateFriendFields(data, map)
	.setCallback(callback);

// 更新扩展字段
Map<FriendFieldEnum, Object> map = new HashMap<>();
Map<String, Object> exts = new HashMap<>();
exts.put("ext", "ext");
map.put(FriendFieldEnum.EXTENSION, exts);
NIMClient.getService(FriendService.class).updateFriendFields(data, map)
	.setCallback(callback);
```

### <span id="黑名单">黑名单</span>

将用户加入黑名单后，将不在收到对方发来的任何消息或者请求。例如：A用户将B用户加入黑名单，B用户发送的消息，A用户将收不到。A用户发送的消息，B用户依然可以看到。

#### 加入黑名单

```java
NIMClient.getService(FriendService.class).addToBlackList(account)
    .setCallback(new RequestCallback<Void>() { ... });

```
#### 移出黑名单

```java
NIMClient.getService(FriendService.class).removeFromBlackList(user.getAccount())
	.setCallback(new RequestCallback<Void>() { ... });
```

#### 监听黑名单变化

```java
NIMClient.getService(FriendServiceObserve.class)
    .observeBlackListChangedNotify(blackListChangedNotifyObserver, true);

private Observer<BlackListChangedNotify> blackListChangedNotifyObserver
    = new Observer<BlackListChangedNotify>() {
    @Override
    public void onEvent(BlackListChangedNotify blackListChangedNotify) {
        List<String> addedAccounts = blackListChangedNotify.getAddedAccounts();  // 被拉黑的账号集合
        List<String> removedAccounts = blackListChangedNotify.getRemovedAccounts(); // 被移出黑名单的账号集合
        ...
    }
};
```

#### 获取黑名单列表

```java
List<String> accounts = NIMClient.getService(FriendService.class).getBlackList();
```

#### 判断用户是否被拉进黑名单

```java
boolean black = NIMClient.getService(FriendService.class).isInBlackList(account);
```

### <span id="消息提醒">消息提醒</span>

网易云信支持对用户设置或关闭消息提醒（静音），关闭后，收到该用户发来的消息时，不再进行通知栏消息提醒。个人用户的消息提醒设置支持漫游。

#### 设置/关闭消息提醒

```java
NIMClient.getService(FriendService.class).setMessageNotify(account, checkState)
	.setCallback(new RequestCallback<Void>() {
		@Override
		public void onSuccess(Void param) {
			if (checkState) {
				Toast.makeText(UserProfileActivity.this, "开启消息提醒", Toast.LENGTH_SHORT).show();
			} else {
				Toast.makeText(UserProfileActivity.this, "关闭消息提醒", Toast.LENGTH_SHORT).show();
			}
});
```
#### 判断用户是否需要消息提醒

```java
boolean notice = NIMClient.getService(FriendService.class).isNeedMessageNotify(account);
```
#### 获取所有静音帐号（不进行消息提醒的用户）

```java
List<String> accounts = NIMClient.getService(FriendService.class).getMuteList();
```
#### 监听消息提醒变更通知

同一个账号在其他端登录时发生了消息提醒变更，会有下面回调通知：

```java
NIMClient.getService(FriendServiceObserve.class).observeMuteListChangedNotify(
    new Observer<MuteListChangedNotify>() {
        @Override
        public void onEvent(MuteListChangedNotify notify) {
            // notify.isMute() 是否被静音
        }
    }, register);
```
## <span id="用户资料托管">用户资料托管</span>

网易云信提供了用户资料托管( `NimUserInfo` )，第三方 APP 可以使用也可以自行实现，但必须实现 `UserInfo` 接口。

### <span id="获取本地用户资料">获取本地用户资料</span>

#### 从本地数据库中批量获取用户资料

通过用户帐号集合，从本地数据库批量获取用户资料列表。代码示例如下：

```java
List<NimUserInfo> users = NIMClient.getService(UserService.class).getUserInfoList(accounts);
```

通过用户账号，从本地数据库获取用户资料。代码示例如下：

```java
NimUserInfo user = NIMClient.getService(UserService.class).getUserInfo(account);
```

#### 从本地数据库中获取所有用户资料

获取本地数据库中所有的用户资料，一般适合在登录后构建用户资料缓存时使用，代码示例如下：

```java
List<NimUserInfo> users = NIMClient.getService(UserService.class).getAllUserInfo();
```

#### <span id="构建通讯录">构建通讯录</span>

如果使用网易云信用户关系、用户资料托管，构建通讯录，先获取我所有好友帐号，再根据帐号去获取对应的用户资料，代码示例如下:

```java
List<String> accounts = NIMClient.getService(FriendService.class).getFriendAccounts(); // 获取所有好友帐号
List<NimUserInfo> users = NIMClient.getService(UserService.class).getUserInfoList(accounts); // 获取所有好友用户资料
```

### <span id="获取服务器用户资料">获取服务器用户资料</span>

从服务器获取用户资料，一般在本地用户资料不存在时调用，获取后 SDK 会负责更新本地数据库。代码示例如下：

```java
NIMClient.getService(UserService.class).fetchUserInfo(accounts)
    .setCallback(new RequestCallback<List<UserInfo>>() { ... });
```
此接口可以批量从服务器获取用户资料，从用户体验和流量成本考虑，不建议应用频繁调用此接口。对于用户数据实时性要求不高的页面，应尽量调用读取本地缓存接口。

### <span id="编辑用户资料">编辑用户资料</span>

#### 更新用户本人资料

传入参数 `Map<UserInfoFieldEnum, Object>` 更新用户本人资料，key 为字段，value 为对应的值。具体字段见 `UserInfoFieldEnum`，包括：昵称，性别，头像 URL，签名，手机，邮箱，生日以及扩展字段等。代码示例如下：

```java
Map<UserInfoFieldEnum, Object> fields = new HashMap<>(1);
fields.put(UserInfoFieldEnum.Name, "new name");
NIMClient.getService(UserService.class).updateUserInfo(fields)
    .setCallback(new RequestCallbackWrapper<Void>() { ... });
```
SDK对部分字段进行格式校验：
- 邮箱：必须为合法邮箱
- 手机号：必须为合法手机号 如13588888888、+(86)-13055555555
- 生日：必须为"yyyy-MM-dd"格式

更新头像可选方案：
- 先将头像图片上传至网易云信云存储上（见 `NosService` ) ，上传成功后可以得到 url 。
- 更新个人资料的头像字段，保存 url 。

此外，开发者也可以自行存储头像，仅将 url 更新到个人资料上。

#### 监听用户资料变更

用户资料除自己之外，不保证其他用户资料实时更新。其他用户数据更新时机为：

- 调用 fetchUserInfo 方法刷新用户
- 收到此用户发来消息（如果消息发送者有资料变更，SDK 会负责更新并通知）
- 程序再次启动，此时会同步好友信息

由于用户资料变更需要跨进程异步调用，开发者最好能在第三方 APP 中做好用户资料缓存，查询用户资料时都从本地缓存中访问。在用户资料有变化时，SDK 会告诉注册的观察者，此时，第三方 APP 可更新缓存，并刷新界面。 代码示例如下：

```java
// 注册/注销观察者
NIMClient.getService(UserServiceObserve.class).observeUserInfoUpdate(userInfoUpdateObserver, register);
// 用户资料变更观察者
private Observer<List<UserInfo>> userInfoUpdateObserver = new Observer<List<UserInfo>>() {
	@Override
	public void onEvent(List<UserInfo> users) {
	...
	}
};
```

## <span id="语音视频通话">语音视频通话</span>

网易云信提供基于网络的语音、视频聊天功能。支持通话中音视频设备的控制，并支持音视频切换。

***注意: 语音视频通话需要 Android 4.0 及以上版本的系统。***

### <span id="语音视频通话配置">语音视频通话配置</span>

使用音视频功能，需要在 `AndroidManifest.xml` 文件中配置接收器。

```xml
<!-- 申明本地电话状态（通话状态）的广播接收器，第三方APP集成时音视频模块时，如果需要网络通话与本地电话互斥，请加上此接收器 -->
<receiver android:name="com.netease.nimlib.receiver.IncomingCallReceiver">
    <intent-filter>
        <action android:name="android.intent.action.PHONE_STATE"/>
    </intent-filter>
</receiver>
```

### <span id="双人语音视频通话流程">双人语音视频通话流程</span>

#### 发起通话（主叫方）

会话类型参数 `AVChatTypeEnum` 主要分为语音通话和视频通话。

会话可选参数 `AVChatOptionalConfig` 包含了视频质量控制、服务器录制以及一些其它可选参数，可以根据自己的需求在通话前选择性的设置。

会话通知参数 `AVChatNotifyOption` 包含iOS的通知配置以及可自定义的扩展消息。

```java
AVChatManager.getInstance().call(account, callType, configs, notifyOption，new AVChatCallback<AVChatData>() { ... });
```

#### 监听来电（被叫方）

一般是在 APP 启动时注册来电监听，例如在 Application 的 onCreate 里添加。当监听到来电时，会返回来电信息 `AVChatData`，其中包含呼叫方式（音频或者视频）、来电帐号。

```java
private void enableAVChat() {
    registerAVChatIncomingCallObserver(true);
}


private void registerAVChatIncomingCallObserver(boolean register) {
    AVChatManager.getInstance().observeIncomingCall(new Observer<AVChatData>() {
        @Override
        public void onEvent(AVChatData chatData) {
            AVChatActivity.launch(DemoCache.getContext(), chatData);
        }
    }, register);
}
```

#### 监听该帐号其他端回应（被叫方）

如果自己的帐号有其他端在线（PC、Web），来电会被其他端做了回应，那么移动端会收到一条通知。因此，移动端在收到来电后需要监听 PC 端对主叫方的响应。

```java
AVChatManager.getInstance().observeOnlineAckNotification(onlineAckObserver, register);
Observer<AVChatOnlineAckEvent> onlineAckObserver = new Observer<AVChatOnlineAckEvent>() {
        @Override
        public void onEvent(AVChatOnlineAckEvent ackInfo) {
            if (ackInfo.getClientType() != ClientType.Android) {
                String client; // 做回应的客户端
                switch (ackInfo.getClientType()) {
					...
                    case ClientType.Windows:
                        client = "Windows";
                        break;
                    default:
                        break;
                }
                // your code
                avChatUI.closeSessions();
            }
        }
    };
```

#### 监听主叫方挂断（被叫方）

详见[监听对方挂断](#监听对方挂断)节

#### 同意接听（被叫方）

当监听到来电后启动通话界面，被叫方可以选择接听或者拒绝。当选择接听时，可以传入相关的可选参数 `AVChatOptionalConfig`，SDK 会自动开启音视频设备，建立通话连接。

在某些特殊情况下，有可能音视频启动失败，此时会回调 onFailed ，错误码-1表示初始化引擎失败。

注意：由于音视频引擎析构需要时间，请尽可能保证上一次通话挂断到本次电话接听时间间隔在2秒以上，否则有可能在接听时出现初始化引擎失败（code = -1），此问题后期会进行优化。

```java
AVChatManager.getInstance().accept(config, new AVChatCallback<Void>() { ... });
```

#### 拒绝接听（被叫方）

```java
AVChatManager.getInstance().hangUp(new AVChatCallback<Void>() { ... });
```

#### 监听被叫方回应（主叫方）

主叫方在发起呼叫成功后需要监听被叫方的回应，监听接口 `observeCalleeAckNotification`，回调返回 `AVChatCalleeAckEvent`，其中包含被叫方的回应结果：
- 对方拒绝接听;
- 对方已经有来电了，此时会返回忙;
- 对方同意接听，此时 SDK 会自动开启音视频设备，建立通话连接，然后双方就可以进行语音视频通话了。

```java
AVChatManager.getInstance().observeCalleeAckNotification(callAckObserver, register);
Observer<AVChatCalleeAckEvent> callAckObserver = new Observer<AVChatCalleeAckEvent>() {
        @Override
        public void onEvent(AVChatCalleeAckEvent ackInfo) {
        	if (ackInfo.getEvent() == AVChatEventType.CALLEE_ACK_BUSY) {
                // 对方正在忙
            } else if (ackInfo.getEvent() == AVChatEventType.CALLEE_ACK_REJECT) {
                // 对方拒绝接听
            } else if (ackInfo.getEvent() == AVChatEventType.CALLEE_ACK_AGREE) {
            	// 对方同意接听
                if (ackInfo.isDeviceReady()) {
                    // 设备初始化成功，开始通话
                } else {
                    // 设备初始化失败，无法进行通话
                }
            }
        }
    };
```

#### <span id="监听对方挂断">监听对方挂断（主叫方、被叫方）</span>

当被叫方收到来电时（在通话建立之前）需要监听主叫方挂断通知，当双方通话建立之后，都需要监听对方挂断通知来结束本次通话。

```java
AVChatManager.getInstance().observeHangUpNotification(callHangupObserver, register);
Observer<AVChatCommonEvent> callHangupObserver = new Observer<AVChatCommonEvent>() {
        @Override
        public void onEvent(AVChatCommonEvent hangUpInfo) {
            // 结束通话
        }
    };
```

#### <span id="请求音视频切换">请求音视频切换</span>

双方通话建立之后，就可以发起音视频切换请求。发送音频到视频的切换请求，对方需要同意才能切换成功；发送视频到音频的切换请求，对方默认同意，自动切换。

```java
// 请求音频切换到视频
AVChatManager.getInstance().requestSwitchToVideo( new AVChatCallback<Void>() { ...});

// 请求视频切换到音频
AVChatManager.getInstance().requestSwitchToAudio(new AVChatCallback<Void>() { ... });
```

#### <span id="音视频切换请求的回应">音视频切换请求的回应</span>

一方发送音视频切换请求之后，对方会收到通知，见一下小节。当收到对方音频切换到视频的请求时，用户可以选择同意或者拒绝。对方将收到应答结果的通知，见下一小节。

```java
// 同意音频切换到视频
AVChatManager.getInstance().ackSwitchToVideo(true, new AVChatCallback<Void>() { ... });

// 拒绝音频切换到视频
AVChatManager.getInstance().ackSwitchToVideo(false, null);
```

#### 监听通话控制通知

双方通话建立之后，需要监听通话控制通知。

```java
AVChatManager.getInstance().observeControlNotification(callControlObserver, register);

Observer<AVChatControlEvent> callControlObserver = new Observer<AVChatControlEvent>() {
        @Override
        public void onEvent(AVChatControlEvent event) {
            handleCallControl(event);
        }
    };

private void handleCallControl(AVChatControlEvent event) {
        switch (event.getControlCommand()) {
            case SWITCH_AUDIO_TO_VIDEO:
                // 对方请求切换音频到视频
                break;
            case SWITCH_AUDIO_TO_VIDEO_AGREE:
                // 对方同意切换音频到视频
                break;
            case SWITCH_AUDIO_TO_VIDEO_REJECT:
                // 对方拒绝切换音频到视频
                break;
            case SWITCH_VIDEO_TO_AUDIO:
                // 对方请求视频切换到音频
                break;
            case NOTIFY_VIDEO_OFF:
                // 对方关闭视频的通知
                break;
            case NOTIFY_VIDEO_ON:
                // 对方开启视频的通知
                break;
            default:
                break;
        }
    }
```

#### 监听呼叫或接听超时通知

主叫方在拨打网络通话时，超过 45 秒被叫方还未接听来电，则自动挂断。被叫方超过 45 秒未接听来听，也会自动挂断，在通话过程中网络超时 30 秒自动挂断。

```java
AVChatManager.getInstance().observeTimeoutNotification(timeoutObserver, register);
Observer<AVChatTimeOutEvent> timeoutObserver = new Observer<AVChatTimeOutEvent>() {
	@Override
	public void onEvent(AVChatTimeOutEvent event) {
			// 超时类型
		}
	}
};
```

#### 监听网络通话发起，接听或正在进行时有本地来电的通知

网络通话发起或者正在接通时，需要监听是否有本地来电（用户接通本地来电）。若有本地来电，网络通话自动拒绝或者挂断。

```java
AVChatManager.getInstance().observeAutoHangUpForLocalPhone(autoHangUpForLocalPhoneObserver, register);

/** 参数为自动挂断的原因：
* 1 作为被叫方：网络通话有来电还未接通，此时有本地来电，那么拒绝网络来电
* 2 作为主叫方：正在发起网络通话时有本地来电，那么挂断网络呼叫
* 3 双方正在进行网络通话，当有本地来电，用户接听时，挂断网络通话
* 4 如果发起网络通话，无论是否建立连接，用户又拨打本地电话，那么网络通话挂断
*/
Observer<Integer> autoHangUpForLocalPhoneObserver = new Observer<Integer>() {
	@Override
	public void onEvent(Integer integer) {
		// 结束通话
	}
};
```

#### 结束通话

发起结束通话的一方，调用 `hangUp` 接口。另外一方，需要注册 `observeHangUpNotification` 监听挂断通知，收到通知后，做相应处理，代码示例见[监听对方挂断（主叫方、被叫方）](#监听对方挂断) 。

```java
AVChatManager.getInstance().hangUp(new AVChatCallback<Void>() {}
```


### <span id="多人语音视频通话流程">多人语音视频通话流程</span>

#### 创建多人会话房间

通过一个房间名 `roomName` 来创建多人会话房间。

可以传入一个扩展字段 `extraMessage`。 后续加入房间的用户会收到这个扩展字段。

```java
AVChatManager.getInstance().createRoom(roomName, extraMessage, new AVChatCallback<AVChatChannelInfo>() {}
```


#### 加入多人会话房间

通过一个房间名 `roomName` 来加入一个已经创建好的多人会话房间。

加入房间时需要指定自己的会话类型 `AVChatTypeEnum`。 主要为音频通话和视频通话两种。

会话可选参数 `AVChatOptionalConfig` 包含了视频质量控制、服务器录制以及一些其它可选参数，可以根据自己的需求选择性设置。


```java
AVChatManager.getInstance().joinRoom(roomName, callType, config, new AVChatCallback<AVChatData>() {}
```

#### 离开多人会话房间

离开一个已经加入的多人会话房间。


```java
AVChatManager.getInstance().leaveRoom(new AVChatCallback<Void>() {}
```


### <span id="通话状态监听">通话状态监听</span>

实现 `AVChatStateObserver` 监听通话过程中状态变化。被叫方同意来电请求后，SDK 自动进行音视频服务器连接，并返回相应信息供上层应用使用。

```java
public class AVChatActivity implements AVChatStateObserver {
	AVChatManager.getInstance().observeAVChatState(this, register);
}
```

#### 当前音视频服务器连接回调

首先返回服务器连接是否成功的回调 `onJoinedChannel`，并通过返回的 result code 做相应的处理。

参数 `code` 返回加入频道是否成功。常见错误码参考 <code>JoinChannelCode</code>
参数 `filePath` `fileName`  在开启服务器录制的情况下返回录制文件的保存路径。

```java
@Override
public void onJoinedChannel(int code, String filePath, String fileName) { }
```

#### 加入当前音视频频道用户帐号回调

其他用户音视频服务器连接成功后，会回调 `onUserJoined`，可以获取当前通话的用户帐号。

```java
@Override
public void onUserJoined(String account) {}
```

#### 当前用户离开频道回调

通话过程中，若有用户离开，则会回调 `onUserLeave`。

```java
// @param event   －1,用户超时离开  0,正常退出
@Override
public void onUserLeave(String account, int event) {}
```

#### 自己成功离开频道回调

```java
@Override
public void onLeaveChannel() {}
```

#### 版本协议不兼容回调

若语音视频通话双方软件版本不兼容，则会回调 `onProtocolIncompatible`。

```java
// @param status 0 自己版本过低  1 对方版本过低
@Override
public void onProtocolIncompatible(int status) {}
```

#### 服务器断开回调

通话过程中，服务器断开，会回调 `onDisconnectServer`。

```java
@Override
public void onDisconnectServer() {}
```

#### 当前通话网络状况回调

通话过程中网络状态发生变化，会回调 `onNetworkQuality`。

```java
// @param value 0~3 ,the less the better; 0 : best; 3 : worst
@Override
public void onNetworkQuality(String account, int value) {}
```

#### 音视频连接成功建立回调

音视频连接建立，会回调 `onCallEstablished`。音频切换到正在通话的界面，并开始计时等处理。视频则通过用户账户取出 SurfaceView 并添加到相应的 layout 上显示图像。

```java
@Override
public void onCallEstablished() {
	if (state == AVChatTypeEnum.AUDIO.getValue()) {
		aVChatUIManager.onCallStateChange(CallStateEnum.AUDIO);
	} else {
		aVChatUIManager.initSurfaceView(aVChatUIManager.getVideoAccount());
		aVChatUIManager.onCallStateChange(CallStateEnum.VIDEO);
	}
	isCallEstablished = true;
}
```

#### 音视频设备状态通知

音视频设备状态发生改变时，会回调 `onDeviceEvent`。

```java
@Override
public void onDeviceEvent(String account, int code, String desc) {}

```

#### 打开音视频设备出错回调

用户执行截图后会回调 `onTakeSnapshotResult`。

```java
@Override
public void onTakeSnapshotResult(String account, boolean success, String file) {}

```

#### 本地网络类型发生改变回调

本地客户端网络类型发生改变时回调，会通知当前网络类型。

```java
@Override
public void onConnectionTypeChanged(int netType) {}

```

#### 本地音视频录制结束回调

当本地录制音视频发送数据结束时回调，会通知录制文件路径以及结束原因。

```java
@Override
public void onLocalRecordEnd(String[] files, int event) {}

```


#### 用户第一帧画面通知

当用户第一帧视频画面绘制前通知。

```java
@Override
public void onFirstVideoFrameAvailable(String account) {}

```

#### 用户第一帧画面绘制后通知

当用户第一帧视频画面绘制后通知。

```java
@Override
public void onFirstVideoFrameRendered(String user) {}

```

#### 用户视频画面分辨率改变通知

当用户视频画面的分辨率改变时通知。

```java
@Override
public void onVideoFrameResolutionChanged(String user, int width, int height, int rotate) {}

```

#### 用户视频帧率汇报

实时汇报用户的视频绘制帧率。

```java
@Override
public void onVideoFpsReported(String account, int fps) {}

```

#### 采集视频数据回调

当用户开始外部视频处理后,采集到的视频数据通过次回调通知。 用户可以对视频数据做相应的美颜等不同的处理。 需要通过<code>setParameters</code>开启视频数据处理。

```java
@Override
public int onVideoFrameFilter(AVChatVideoFrame frame) {}
```  

#### 采集语音数据回调

当用户开始外部语音处理后,采集到的语音数据通过次回调通知。 用户可以对语音数据做相应的变声等不同的处理。需要通过<code>setParameters</code>开启语音数据处理。

```java
@Override
public int onAudioFrameFilter(AVChatAudioFrame frame) {}
```  

#### 语音播放设备变化通知

当用户切换扬声器或者耳机的插拔等操作时, 语音的播放设备都会发生变化通知。 语音设备参考 <code>AVChatAudioDevice</code>

```java
@Override
public void onAudioOutputDeviceChanged(int device) {}
``` 

#### 语音正在说话用户声音强度通知

正在说话用户的语音强度回调，包括自己和其他用户的声音强度。如果一个用户没有说话,或者说话声音小没有被参加到混音,那么这个用户的信息不会在回调中出现。

```java
@Override
void onReportSpeaker(Map<String, Integer> speakers, int mixedEnergy) {}
``` 


### <span id="通话中的设备控制">通话中的设备控制</span>

通话进行中，可以进行设备静音，扬声器，摄像头切换，开关摄像头和切换通话模式的设置。

#### 通话中实时设置参数

在通话过程中, 可以实时设置部分参数。用户可以通过这些参数进行软硬件编解码切换，清晰度切换等。

```java
AVChatParameters params = new AVChatParameters();
params.setBoolean(AVChatParameters.KEY_VIDEO_FPS_REPORTED, false);
AVChatManager.getInstance().setParameters(params);
```

#### 通话中实时获取参数

在通话过程中, 可以实时获取部分参数。

```java
AVChatParameters params = new AVChatParameters();
params.requestKey(AVChatParameters.KEY_VIDEO_FPS_REPORTED);
AVChatManager.getInstance().getParameters(params);
```

#### 设置视频绘制画布

通过用的<code>account</code>设置用户的视频画布，同时还可以制定画布是否镜像处理，以及相应的缩放模式。

对于交换用户画布的操作，需要先把当前用户的画布解除，通过此接口传入<code>null</code>即可解除，然后再设置新的画布。

```java
AVChatManager.getInstance().setupVideoRender(String account, AVChatVideoRender render, boolean mirror, int scalingType);
```

#### 设置本地语音流静音

将设置本地发送语音是否静音。**注意：不影响本地语音数据发送。**

```java
AVChatManager.getInstance().muteLocalAudio(true);
```

#### 设置远端用户语音流静音

将设置是否播放其他用户的语音数据。**注意：不影响远端语音数据接收。**

```java
AVChatManager.getInstance().muteRemoteAudio(account, true);
```

#### 设置本地视频流静音

设置本地视频数据是否发送。**注意：不影响本地视频数据采集绘制。**

```java
AVChatManager.getInstance().muteLocalVideo(true);
```

#### 设置远端用户视频流静音

将设置是否绘制远端用户的视频数据。**注意：不影响远端视频数据接收。**

```java
AVChatManager.getInstance().muteRemoteVideo(account, true);
```


#### 设置扬声器

```java
// 设置扬声器是否开启
AVChatManager.getInstance().setSpeaker(!AVChatManager.getInstance().speakerEnabled());
```

#### 切换摄像头

```java
// 切换摄像头（主要用于前置和后置摄像头切换）
AVChatManager.getInstance().switchCamera();
```


#### 切换通话模式

具体见[请求音视频切换](#请求音视频切换) 和 [音视频切换请求的回应](#音视频切换请求的回应)


#### 客户端本地录制发送的音视频数据

通话进行中，可以录制自己发送的音频和视频数据, 文件将以MP4格式保存在客户端本地。程序卸载时录制的本地文件也会随程序一并删除。

客户端本地录制接口，通过返回值判断是否调用成功。仅录制发送的音频和视频文件，前后摄像头切换时录制文件可能存在多个。

```java
// 开始录制
AVChatManager.getInstance().startLocalRecord(null);
```

客户端停止本地录制接口, 停止录制后将会通过回调函数返回结果。

```java
// 停止录制
AVChatManager.getInstance().stopLocalRecord(null);

```

录制结束后会通过网络通话状态通知告知。

```java
/**
 * 本地录制结束时通知
 * @param files  录制文件，由于不同的摄像头参数不一样可能会存在多个文件。
 * @param event  录制结束原因. 0,正常结束 1,异常结束(存储空间不足等)
*/
void onRecordEnd(String[] files, int event);
```


#### 视频画面截图

截取指定用户的视频画面， 截图结果将会通过 `onTakeSnapshotResult` 回调通知。

```java
AVChatManager.getInstance().takeSnapshot(account);
```


#### 多人模式观众角色设置

是否打开观众角色, 设置观众角色后所有的语音和视频数据的采集和发送会关闭，仅允许接收和播放远端其他用户的数据。

```java
AVChatManager.getInstance().enableAudienceRole(true);
```


### <span id="网络通话其它接口">网络通话其它接口</span>

#### 权限检查
在Android 6.0 及以上系统中提供网络通话权限检查，需要保证所有权限获取后再进行网络通话。

```java
//返回缺失的权限
AVChatManager.getInstance().checkPermission(context);
```



## <span id="实时会话（白板）">实时会话（白板）</span>

网易云信提供数据通道（TCP/UDP/语音通道)来满足实时会话的需求，如在线白板教学(参考demo)、文件传输等场景，其中 TCP/UDP 通道，可以同时存在多个，语音通道全局只能有一个。

### <span id="实时会话配置">实时会话配置</span>

使用白板会话功能，需要在 `AndroidManifest.xml` 文件中声明一个接收器。

```java
<!-- 申明白板会话的广播接收器，第三方APP集成时，action中的com.netease.nim.demo请替换为自己的包名 -->
<receiver
    android:name="com.netease.nimlib.receiver.RTSBroadcastReceiver"
    android:enabled="true"
    android:exported="false">
    <intent-filter>
        <action android:name="com.netease.nim.demo.ACTION.RECEIVE_RTS_NOTIFICATION"/>
    </intent-filter>
</receiver>
```

### <span id="实时会话主流程">实时会话主流程</span>

#### 发起会话/创建数据通道（主叫方）

目前我们提供两种数据通道：TCP 通道和语音通道，后续会支持 UDP 通道。通道类型见  `RTSTunType`,可选参数见  `RTSOptions`（包含推送内容、发起方附带给其他参与者的内容、是否录制通道数据等）
一个会话可以同时创建多个通道，但全局只能有一个语音通道。发起会话时，RTSNotifyOption 中可进行iOS端推送通知自定义配置，同时也包含一个可自定的扩展字段。
例如：

```java

List<RTSTunType> types = new ArrayList<>(1);
types.add(RTSTunType.AUDIO);
types.add(RTSTunType.TCP);

String pushContent = account + "发起一个会话";
String extra = "extra_data";
RTSOptions options = new RTSOptions().setRecordAudioTun(true).setRecordTCPTun(true);
RTSNotifyOption notifyOption = new RTSNotifyOption();

sessionId = RTSManager.getInstance().start(account, types, options, notifyOption, new RTSCallback<RTSData>() { ... });
if (sessionId == null) {
    // 发起会话失败,音频通道同时只能有一个会话开启
    onFinish();
}
```
start 接口返回的为会话 ID，开发者务必保存起来，下面调用该会话的相关接口都需要传入此 sessionId。注意，若返回 null，表示发起失败，原因是语音通道同时只能有一个会话开启。

#### 监听会话请求/数据通道请求（被叫方）

一般是在 APP 启动时监听会话（新通道）请求，例如在 Application 的 onCreate 里添加。当监听到新通道请求时，会返回基本信息 `RTSData`，其中包括通道类型列表、发起方帐号。

```java
private void registerRTSIncomingCallObserver(boolean register) {
    RTSManager.getInstance().observeIncomingSession(new Observer<RTSData>() {
    	@Override
        public void onEvent(RTSData rtsData) {
        	// 启动会话界面
        }
    },register);
}
```

#### 监听该帐号其他端回应（被叫方）

如果自己的帐号有其他端在线（例如 PC 端），其他端做了回应（接受或者拒绝），那么移动端会收到一条通知。因此，移动端在收到会话请求后需要监听 PC 端对发起方的响应。

```java
RTSManager.getInstance().observeOnlineAckNotification(sessionId, onlineAckObserver, register);
Observer<RTSOnlineAckEvent> onlineAckObserver = new Observer<RTSOnlineAckEvent>() {
    @Override
    public void onEvent(RTSOnlineAckEvent rtsOnlineAckEvent) {
        if (rtsOnlineAckEvent.getClientType() != ClientType.Android) {
            String client = null;
            switch (rtsOnlineAckEvent.getClientType()) {
                case ClientType.Web:
                    client = "Web";
                    break;
                case ClientType.Windows:
                    client = "Windows";
                    break;
                default:
                    break;
            }
            // your code
            onFinish();
        }
    };
```

#### 监听主叫方结束会话（被叫方）

详见[监听对方结束会话](#监听对方结束会话)节

#### 接受会话（被叫方）

当监听到会话请求后，被叫方可以选择接受或者拒绝，可以传入可选参数：是否录制通道数据。当选择接受时，SDK 会自动开启相关的通道，建立与对方的连接。

```java
RTSOptions options = new RTSOptions().setRecordAudioTun(true).setRecordTCPTun(true);
RTSManager.getInstance().accept(sessionId, options, new RTSCallback<Boolean>() { ... });
```
如果回调 onFailed 返回 -1，表示本地正在使用语音通道，不能同时存在两个语音通道。

#### 拒绝会话（被叫方）

```java
RTSManager.getInstance().close(sessionId, new RTSCallback<Void>() { ... });
```

#### 监听被叫方回应（主叫方）

主叫方在发起会话成功后需要监听被叫方的回应，监听接口 `observeCalleeAckNotification`，回调返回 `RTSCalleeAckEvent`，其中包含被叫方的回应结果：
- 对方同意，此时 SDK 会自动开启相应的通道，建立通道连接。
- 对方拒绝，结束会话。

```java
RTSManager.getInstance().observeCalleeAckNotification(sessionId, calleeAckEventObserver, register);
private Observer<RTSCalleeAckEvent> calleeAckEventObserver = new Observer<RTSCalleeAckEvent>() {
    @Override
    public void onEvent(RTSCalleeAckEvent rtsCalleeAckEvent) {
        if (rtsCalleeAckEvent.getEvent() == RTSEventType.CALLEE_ACK_AGREE) {
            // 判断SDK自动开启通道是否成功
            if (!rtsCalleeAckEvent.isTunReady()) {
                return;
            }
            // 进入会话界面
        } else if (rtsCalleeAckEvent.getEvent() == RTSEventType.CALLEE_ACK_REJECT) {
            // 被拒绝，结束会话
        }
    }
};
```

#### <span id="监听对方结束会话">监听对方结束会话（主叫方、被叫方）</span>

当被叫方收到会话请求时需要监听主叫方结束会话的通知；当双方会话建立之后，需要监听对方结束会话的通知。

```java
RTSManager.getInstance().observeHangUpNotification(sessionId, endSessionObserver, register);
private Observer<RTSCommonEvent> endSessionObserver = new Observer<RTSCommonEvent>() {
    @Override
    public void onEvent(RTSCommonEvent rtsCommonEvent) {
        // 结束会话
    }
};
```

#### <span id="发送控制信息">发送控制信息 </span>

双方会话建立之后，就可以相互发送控制信息了。

```java
RTSManager.getInstance().sendControlCommand(sessionId, content, new RTSCallback<Void>() { ... });
```

#### 监听会话控制通知

双方会话建立之后，需要监听会话控制通知。

```java
RTSManager.getInstance().observeControlNotification(sessionId, controlObserver, register);

Observer<RTSControlEvent> controlObserver = new Observer<RTSControlEvent>() {
    @Override
    public void onEvent(RTSControlEvent rtsControlEvent) {
        // your code
    }
};
```

#### 监听（发起）创建新通道或接受新通道超时通知

主叫方在（发起会话）创建通道时，超过 40 秒被叫方还未接受，则自动挂断。被叫方超过 40 秒接受会话，也会自动挂断。

```java
Observer<RTSTimeOutEvent> timeoutObserver = new Observer<RTSTimeOutEvent>() {
    @Override
    public void onEvent(RTSTimeOutEvent rtsTimeOutEvent) {
        // 超时，结束会话
    }
};
```

#### 监听数据通道的状态

发起会话（对方接受后），或者接受了会话请求后，需要立即注册对数据通道状态的监听。

```java
RTSManager.getInstance().observeChannelState(sessionId, channelStateObserver, register);

RTSChannelStateObserver channelStateObserver = new RTSChannelStateObserver() {

    @Override
    public void onConnectResult(RTSTunType tunType, long channelId, int code) {
        // 与服务器连接结果通知，成功返回 200
    }

    @Override
    public void onRecordInfo(RTSTunType tunType, String url, String name) {
        // 服务器返回通道数据录制的地址和名称（一般在登录服务器成功后返回）
    }

    @Override
    public void onChannelEstablished(RTSTunType tunType) {
    	// 双方通道连接建立(对方用户已加入)
    }

    @Override
    public void onDisconnectServer(RTSTunType tunType) {
    	// 与服务器断开连接
    }

    @Override
    public void onError(RTSTunType tunType, int code) {
    	// 通道发生错误
    }

    @Override
    public void onNetworkStatusChange(RTSTunType tunType, int value) {
        // 网络信号强弱
    }
};
```

#### 结束会话

发起结束通话的一方，调用 `close`  接口。另外一方，需要注册  `observeHangUpNotification`  监听挂断通知，收到通知后，做相应处理，代码示例见[监听对方结束会话（主叫方、被叫方）](#监听对方结束会话) 。

```java
RTSManager.getInstance().close(sessionId, new RTSCallback<Void>() { ... });
```

### <span id="数据收发">数据收发</span>

数据通道建立之后就可以进行数据的收发。

#### 发送数据

发送数据，需要构造  `RTSTunData`, 需要指定会话 ID，通道类型，对方帐号，数据（字节数组）及数据的长度。如果需要发送数据到所有用户，对方帐号填 null。

```java
RTSTunData channelData = new RTSTunData(sessionId, RTSTunType.TCP, toAccount, data.getBytes("UTF-8"), data.getBytes().length);
RTSManager.getInstance().sendData(channelData);
```

#### 接收数据

数据通道建立完之后，就可以监听对方发送来的数据。

```java
RTSManager.getInstance().observeReceiveData(sessionId, receiveDataObserver, register);
Observer<RTSTunData> receiveDataObserver = new Observer<RTSTunData>() {
    @Override
    public void onEvent(RTSTunData rtsTunData) {
        String data = "[parse bytes error]";
        try {
            data = new String(rtsTunData.getData(), 0, rtsTunData.getLength(), "UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }

        TransactionCenter.getInstance().onReceive(sessionId, data);
    }
};
```

### <span id="音频设备控制">音频设备控制</span>

在语音通道建立成功之后，可以对音频设备进行操作，目前开放的操作接口如下：

#### 静音开关

开启静音后，对方将收不到语音。

```java
RTSManager.getInstance().setMute(sessionId, true);
```

#### 扬声器开关

语音默认在听筒播放，开启扬声器后，由听筒切换到扬声器。

```java
RTSManager.getInstance().setSpeaker(sessionId, true);
```

## <span id="容易混淆的概念">容易混淆的概念</span>

### <span id="通知类消息">通知类消息</span>

- 概念

属于会话中的一种消息，其对应的数据结构为 `IMMessage`，消息类型为 MsgTypeEnum.notification，有在线、离线、漫游。继承 NotificationAttachment，目前用于（已操作完成的）群通知事件，不计入消息未读数。包含 MemberChangeAttachment , UpdateTeamAttachment , LeaveTeamAttachment 和 DismissAttachment 。没有通知栏提醒（如有需要，第三方自行实现）。

- 使用场景

一般位于聊天界面的中间。例如，群名称更新，某某某退出了群聊等。

### <span id="自定义消息">自定义消息</span>

- 概念

属于会话中的一种消息，其对应的数据结构为 `IMMessage`，消息类型为 MsgTypeEnum.custom，主要提供给第三方开发者定制消息使用，有在线、离线、漫游、通知栏提醒（文案需要自行定制）。主要是通过 IMMessage的 setAttachment 来实现。

- 使用场景

一般与普通文本，语音消息相同，位于聊天界面的左右两侧。例如，猜拳、贴图、阅后即焚均可以采用自定义消息来实现。

### <span id="Tip消息">Tip消息</span>

- 概念

属于会话中的一种消息，其对应的数据结构为 `IMMessage`，消息类型为 MsgTypeEnum.tip，是自定义消息的简化，有在线、离线、漫游、通知栏提醒（文案需要自行定制），但是区别于自定义消息，Tip 消息不支持 setAttachment。

- 使用场景

一般用于自定义的通知提醒，位于聊天界面的中间，例如进入会话时出现的欢迎消息，或是会话过程中命中敏感词后的提示消息等场景，当然也可以用自定义消息实现，只是相对复杂一些。

### <span id="系统通知">系统通知</span>

- 概念

属于云信内建的系统通知，其对应的数据结构为 `SystemMessage`， 由网易云信服务器推送给用户的通知类消息，用于网易云信系统类的事件通知。现在主要包括群变动的相关通知，例如入群申请，入群邀请等，如果第三方应用还托管了好友关系，好友的添加、删除也是这个类型的通知。系统通知由 SDK 负责接收和存储，并提供较简单的未读数管理。只有在线和离线，没有漫游。没有通知栏提醒（如有需要，第三方自行实现）。

- 使用场景

通常在验证消息列表中展现。

### <span id="自定义通知">自定义通知</span>

- 概念

提供给第三方自定义的全局的通知类型，其对应的数据结构为 `CustomNotification`。只有在线和离线，没有漫游，没有通知栏提醒（第三方自行实现）。

自定义通知和自定义消息的不同之处在于，自定义消息归属于会话中的消息体系内，由 SDK 存储在消息数据库中，与网易云信的其他内建消息类型一同展现给用户。而自定义通知主要用于第三方的一些事件状态通知，网易云信不存储，也不解析这些通知，网易云信仅仅负责替第三方传递和通知这些事件，起到透传的作用。

- 使用场景

聊天双方处于P2P聊天界面时，显示的“正在输入通知”。

## <span id="API 文档">API 文档</span>
* [在线文档](http://dev.netease.im/doc/android/index.html "target=_blank")

