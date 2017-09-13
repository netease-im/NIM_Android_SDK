## <span id="概要介绍"> 概要介绍</span>
网易云通信 SDK 为移动应用提供一个完善的 IM 系统开发框架，屏蔽掉 IM 系统的复杂细节，对外提供较为简洁的 API 接口，方便第三方应用快速集成 IM 功能。IM SDK 提供的能力如下：

- 网络连接管理
- 登录认证服务
- 消息传输服务
- 消息存储服务
- 消息检索服务
- 群组管理/聊天室/会话服务
- 用户资料、好友关系托管服务
- 智能对话机器人服务

### <span id="总体接口介绍">总体接口介绍</span>

网易云通信 SDK 提供了两类接口供开发者调用：一类是第三方 APP 主动发起请求，第二类是第三方 APP 作为观察者监听事件和变化。第一类接口名均以 **Service** 结尾，例如 AuthService ，第二类接口名均以 **ServiceObserver** 结尾，例如 AuthServiceObserver，个别太长的类名则可能直接以 **Observer** 结尾，比如 SystemMessageObserver。

SDK 提供三种接口返回值：基本数据类型（同步接口），InvocationFuture（异步接口） 和 AbortableFuture（异步接口）。异步接口基本上都是从主进程发起调用，然后在后台进程执行，最后再将结果返回给主进程。

| SDK 接口返回值      |     说明 |
| :-------- | :-------- |
| 基本数据类型 | 同步接口 |
| InvocationFuture | 异步接口 |
| AbortableFuture | 异步接口，耗时很长或者传输大量数据时使用，可用 abort() 方法，中断请求。<br>例如上传下载、登录等 |

异步接口可设置回调函数，提供两种方式：RequestCallback 和 RequestCallbackWrapper。

| 异步接口回调函数 | 说明 |
| :-------- | :--------|
| RequestCallback | 需要实现3个接口：<br> 成功 onSuccess, 失败 onFailed, 异常 onException|
| RequestCallbackWrapper | 需要实现 onResult。封装了成功，失败和异常的3个接口，在参数上进行区分|

SDK 提供的接口主要按照业务进行分类，大致说明如下：

| SDK接口 | 说明 |
| :-------- | :--------|
| AuthService | 用户认证服务接口，提供登录注销接口。 |
| AuthServiceObserver | 用户认证服务观察者接口。 |
| MsgService | 消息服务接口，用于发送消息，管理消息记录等。<br>同时还提供了发送自定义通知的接口。 |
| MsgServiceObserve | 接收消息，消息状态变化等观察者接口。 |
| LuceneService | 聊天消息全文检索接口。 |
| TeamService | 群组服务接口，用于发送群组消息，管理群组和群成员资料等。 |
| TeamServiceObserve | 群组和群成员资料变化观察者。 |
| SystemMessageService | 系统通知观察者。 |
| FriendService | 好友关系托管接口，目前支持添加、删除好友、<br>获取好友列表、黑名单、设置消息提醒。 |
| FriendServiceObserve | 好友关系变更、黑名单变更通知观察者。 |
| UserService | 用户资料托管接口，提供获取用户资料、修改个人资料等。 |
| UserServiceObserve | 用户资料托管接口，提供获取用户资料、修改个人资料等。 |
| AVChatManager | 语音视频通话接口。 |
| RTSManager | 实时会话接口。 |
| NosService | 网易云存储服务，提供文件上传和下载。 |
| NosServiceObserve | 网易云存储传输进度观察者接口。 |
| MixPushService | 第三方推送接口，提供第三方推送服务。 |
| EventSubscribeService | 事件订阅服务接口，提供事件订阅等服务 |
| EventSubscribeServiceObserver | 事件状态变更观察者接口。 |
| RedPacketService | 红包接口。提供获取红包sdk token等功能。 |
| RobotService | 机器人操作相关接口，提供获取机器人、<br>获取机器人信息、判断是否是机器人等功能。 |
| RobotServiceObserve | 机器人数据变更观察者接口。 |
| SettingsService | 系统设置接口。提供多端推送、免打扰配置 |
| SettingsServiceObserver | 系统设置变更观察者接口。 |

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

## <span id="集成方式">集成方式</span>

IM SDK 是网易云通信其他能力（实时语音视频、互动白板等）的基础，本节讲述 IM SDK 的集成步骤也将其他能力 SDK 的集成步骤融合起来，开发者可以根据实际业务需要选择接入的类库。

网易云通信 IM SDK 支持两种方式集成。

1\. 通过 Gradle 集成 SDK （推荐）

2\. 通过类库配置集成 SDK

网易云通信 Android SDK 2.5.0 以上强烈推荐通过 Gradle 集成 SDK。

> 注意：
> 1\. IM SDK 最低要求 Android 4.0， 其中网络音视频通话和白板最低要求 Android 4.1。
>
> 2\. 从 3.2 版本开始 jni 库支持 64位 系统

### <span id="通过 Gradle 集成 SDK">通过 Gradle 集成 SDK</span>

首先，在整个工程的 build.gradle 文件中，配置 repositories，使用 jcenter 或者 maven ，二选一即可，如下：

```groovy
allprojects {
    repositories {
        jcenter() // 或者 mavenCentral()
    }
}
```

第二步，在主工程的 build.gradle 文件中，添加 dependencies。根据自己项目的需求，添加不同的依赖即可。注意：版本号必须一致，这里以 3.3.0 版本为例：

```groovy

android {
   defaultConfig {
       ndk {
           //设置支持的SO库架构
           abiFilters "armeabi-v7a", "x86","arm64-v8a","x86_64"
        }
   }
}

dependencies {
    compile fileTree(dir: 'libs', include: '*.jar')
    // 添加依赖。注意，版本号必须一致。
    // 基础功能 (必需)
    compile 'com.netease.nimlib:basesdk:3.3.0'
    // 音视频需要
    compile 'com.netease.nimlib:avchat:3.3.0'
    // 聊天室需要
    compile 'com.netease.nimlib:chatroom:3.3.0'
    // 实时会话服务需要
    compile 'com.netease.nimlib:rts:3.3.0'
    // 全文检索服务需要
    compile 'com.netease.nimlib:lucene:3.3.0'
}
```

> 再次注意：依赖包的版本号必须一致。

### <span id="通过类库配置集成 SDK">通过类库配置集成 SDK</span>

首先到下载页面进行下载 Android SDK。开发者可以根据实际需求，配置类库。

以下介绍以 Android SDK v2.5 及以上版本为例，Android SDK v2.5 以下的配置，请咨询技术支持。

网易云通信 Android SDK v2.5 及以上分为两种 SDK 包下载，第一种包含全部功能：IM + 聊天室 + 实时音视频 + 教学白板。第二种包含部分功能，包含：IM + 聊天室。

SDK 包的 libs 文件夹中，包含了网易云通信的 jar 文件，各 jni 库文件夹以及 SDK 依赖的第三方库。

第一种，包含全部功能的 SDK 包。如果需用网易云通信 SDK 提供的所有功能，将这些文件拷贝到你的工程的 libs 目录下，即可完成配置。列表如下：

```
libs
├── arm64-v8a
│   ├── libne_audio.so （高清语音录制功能必须）
│   └── libcosine.so （后台保活需要）
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
├── x86_64
│   ├── libne_audio.so
│   └── libcosine.so
│   ├── libnrtc_engine.so
│   └── libnrtc_network.so
│   └── librts_network.so
│
├── nim-basesdk-3.3.0.jar （基础功能）
├── nim-chatroom-3.3.0.jar （聊天室需要）
├── nim-rts-3.3.0.jar （实时会话、文档转码需要）
├── nim-avchat-3.3.0.jar （音视频需要）
├── nim-lucene-3.3.0.jar （全文检索需要）
├── nrtc-sdk.jar（音视频需要）
└── cosinesdk.jar (Android 后台保活需要)
```

第二种，只包含 IM 基础功能和聊天室功能的 SDK 包。如果只需要 IM 基础功能和聊天室功能，只需要将下面这些文件拷贝到你的工程的 libs 目录下，即可完成配置。列表如下：

```
libs
├── arm64-v8a
│   ├── libne_audio.so （高清语音录制功能必须）
│   └── libcosine.so （Android 后台保活需要）
├── armeabi-v7a
│   ├── libne_audio.so
│   └── libcosine.so
├── x86_64
│   ├── libne_audio.so
│   └── libcosine.so
├── x86
│   ├── libne_audio.so
│   └── libcosine.so
├── nim-basesdk-3.3.0.jar （基础功能）
├── nim-chatroom-3.3.0.jar （聊天室需要）
└── cosinesdk.jar (Android 后台保活需要)
```

以上文件列表中，jar 文件版本号可能会不同，子目录中的文件是 SDK 所依赖的各个 CPU 架构的 so 库。

> 按需配置 jar 包： 如果不需要聊天室功能，可以在IM和聊天室的基础包中，去掉 nim-chatroom-3.3.0.jar。 如果只需要 IM 基础功能和 音视频功能，可以在 IM 和聊天室的基础包中，去掉 nim-chatroom-3.3.0.jar，so 库需要加上 libnrtc*.so，还需加上 nim-avchat-3.3.0.jar 和 nrtc-sdk.jar； 如果不需要安卓保活功能，可以去掉 libcosine.so 和 cosinesdk.jar ，以及 assets 文件夹中的 cosine 文件夹( AndroidManifest.xml 文件中相关的安卓保活的配置需要删去)。 如果不需要全文检索功能，可以去掉 nim-lucene-3.3.0.jar（该包有 1M+ 大小，如果没有用到消息全文检索功能，建议去掉）。

如果你使用的 IDE 是 Android Studio，要将 jni 库按照 IDEA 工程目录的结构，放置在对应的目录中（一般为 src/main/jniLibs）。或者在 build.gradle 中配置好 jniLibs 的 sourceSets（可参考 demo 的 build.gradle）。

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

        <!-- 声明网易云通信后台服务，如需保持后台推送，使用独立进程效果会更好。 -->
        <service
            android:name="com.netease.nimlib.service.NimService"
            android:process=":core"/>

       <!-- 运行后台辅助服务 -->
        <service
            android:name="com.netease.nimlib.service.NimService$Aux"
            android:process=":core"/>

        <!-- 声明网易云通信后台辅助服务 -->
        <service
            android:name="com.netease.nimlib.job.NIMJobService"
            android:exported="true"
            android:permission="android.permission.BIND_JOB_SERVICE"
            android:process=":core"/>

        <!-- 网易云通信SDK的监视系统启动和网络变化的广播接收器，用户开机自启动以及网络变化时候重新登录，
            保持和 NimService 同一进程 -->
        <receiver android:name="com.netease.nimlib.service.NimReceiver"
            android:process=":core"
            android:exported="false">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
            </intent-filter>
        </receiver>

        <!-- 网易云通信进程间通信 Receiver -->
        <receiver android:name="com.netease.nimlib.service.ResponseReceiver"/>

        <!-- 网易云通信进程间通信service -->
        <service android:name="com.netease.nimlib.service.ResponseService"/>

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
-keep class com.netease.** {*;}
#如果你使用全文检索插件，需要加入
-dontwarn org.apache.lucene.**
-keep class org.apache.lucene.** {*;}
```

## <span id="初始化">初始化</span>

在你的程序的 Application 的 `onCreate` 中，加入网易云通信 SDK 的初始化代码：

```java
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

> 特别提醒：SDK 的初始化方法必须在主进程中调用，在非主进程中初始化无效。请在主进程中调用 SDK XXXService 提供的方法，在主进程中注册 XXXServiceObserver
的观察者（有事件变更，会回调给主进程的主线程）。如果你的模块运行在非主进程，请自行实现主进程与非主进程的通信（Binder/AIDL/BroadcastReceiver等IPC）将主进程回调或监听返回的数据传递给非主进程。

[网易云通信 Andorid SDK 断网重连机制及登录返回码说明 ](http://bbs.netease.im/read-tid-231)

## <span id="登录登出">登录登出</span>

云信中登录分为手动登录和自动登录两种模式，他们的区别主要在于

1\. SDK 是否会接管登录失败后的处理

2\. 服务器是否会验证当前登录设备的安全性

第一点，SDK 认为手动登录即是终端用户发起登录的流程，那么在登录失败后（如网络情况不佳，密码错误）SDK 会触发相应回调，并停止重连操作，等待用户再次发起登录操作。而自动登录则会在登录失败后仍旧尝试重连，直到登录成功为止（密码错误等情况除外）。

第二点，服务器为了保障当前用户的安全性，在登录时会根据当前登录是否是自动登录进行检查设备唯一性的检查，如果当前登录为自动登录，且当前登录设备不是上一次登录设备，则会自动禁止其登录，以保证安全性。当然你也可以设置自动登录为强制登录，以忽略安全性检查。这种做法比较适合非 IM 产品延迟加载通讯模块的场景：通过服务器下发的正确云信 ID 直接进行自动登录，无需关心任何登录出错的情况，简单方便。

具体说明可阅读 [帐号集成与登录](/docs/product/IM即时通讯/产品介绍/帐号集成与登录)

### <span id="手动登录">手动登录</span>

- API 介绍

一般 APP 在首次登录、切换帐号登录、注销重登时需要手动登录，开发者需要调用 `AuthService` 提供的 `login` 接口主动发起登录请求。该接口返回类型为 `AbortableFuture`，允许用户在后面取消登录操作。如果服务器一直没有响应，30 秒后 RequestCallback 的 onFailed 会被调用，参数为 408 （网络连接超时）。

- API 原型

```
/**
 * 登录接口。sdk会自动连接服务器，传递用户信息，返回登录结果。<br>
 * 该操作中途可取消。如果因为网络比较差，或其他原因导致服务器迟迟没有返回，用户也没有主动取消，
 * 在45秒后AbortableFuture的onFailed会被调用到。
 *
 * @param info 登录的用户信息
 * @return AbortableFuture
 */
public AbortableFuture<LoginInfo> login(LoginInfo info);
```

- 参数说明

| LoginInfo 参数 | 说明  |
| :-------- | :--------|
| account | 用户帐号 |
| token | 登录 token |
| appKey（可选） | 当前应用的 appKey，一个 appKey 对应一个账号体系。<br>如果不填，则优先使用 SDKOptions 中配置的 appKey，<br>如果没有则使用 AndroidManifest 中配置的appKey |

- 示例

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

说明： 登录成功后，可以将用户登录信息 LoginInfo 信息保存到本地，下次启动 App 时，读取本地保存的 LoginInfo 进行自动登录。

### <span id="自动登录">自动登录</span>

- API 介绍

如果上次登录已经存在用户登录信息，那么在初始化 SDK 时传入 `LoginInfo`，SDK 后台会自动登录，并在登录发起前即打开相关账号的数据库，供上层调用。开发者此时无需再手动调用登录接口，可以跳过登录界面直接进入主界面。

进入主界面后，可以通过监听用户在线状态（每次注册用户在线状态监听都会立即回调通知当前的用户在线状态），或者主动获取当前用户在线状态，来判断自动登录是否成功。

- API 原型

```
// 在初始化SDK的时候，传入 loginInfo()， 其中包含用户信息，用以自动登录
NIMClient.init(this, loginInfo(), options());
```

- 示例

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
说明：在自动登录过程中，如果没有网络或者网络断开或者与网易云通信服务器建立连接失败，会上报在线状态 NET_BROKEN，表示当前网络不可用，当网络恢复的时候，会触发断网自动重连；如果连接建立成功但登录超时，会上报在线状态 UNLOGIN，并触发自动重连，无需上层手动调用登录接口。

> 特别提醒: 在自动登录成功前，调用服务器相关请求接口（由于与网易云通信服务器连接尚未建立成功，会导致发包超时）会报408错误。但可以调用本地数据库相关接口获取本地数据（自动登录的情况下会自动打开相关账号的数据库）。自动登录过程中也会有用户在线状态回调。

### <span id="登出">登出</span>

- API 介绍

如果用户手动登出，不再接收消息和提醒，开发者可以调用 logout 方法，该方法没有回调。

- API 原型

```
/**
 * 注销接口
 */
public void logout();
```

- 示例

```java
NIMClient.getService(AuthService.class).logout();
```

> 注意: 登出操作，不要放在 Activity(Fragment) 的 onDestroy 方法中。

### <span id="特殊错误码介绍"> 特殊错误码介绍</span>

| 错误码 | 说明 | 在线状态 |
| :--------: | :-------- | :-------- |
| 302 | token 错误或者账号不存在都会导致 302 错误码。<br>这种情况一般发生于用户在其他设备上修改了密码。 | PWD_ERROR |
| 408 | 1、连接建立成功，SDK 发出登录请求后网易云通信服务器一直<br>没有响应，那么 30s 后将导致登录超时。<br>2、登录成功之前，调用服务器相关请求接口<br>（由于与网易云通信服务器连接尚未建立成功，会导致发包超时） | UNLOGIN |
| 415 | 网络断开或者与网易云通信服务器建立连接失败 | NET_BROKEN |
| 416 | 请求过频错误，<br>为了防止开发者错误的调用导致服务器压力过大， SDK 做了频控限制，<br>并有一分钟的惩罚时间，过了惩罚时间后接口可以再次正常调用。 | UNLOGIN |
| 417 | 一般由一端登录导致自动登录失败导致。<br>这种情况发生于非强制登录模式下已有一端在线而当前设备进行自动登录(设置为只允许一端同时登录时)，<br>出于安全方面的考虑，云信服务器判定当前端需要重新手动输入用户密码进行登录，故拒绝登录。 | KICKOUT |
| 1000 | 登录成功之前，调用本地数据库相关接口（手动登录的情况下数据库未打开） | UNLOGIN |

### <span id="多端登录和互踢">多端登录和互踢</span>

#### 多端登录

- API 介绍

登录成功后，可以注册多端登录状态观察者。当有其他端登录或者注销时，会通过此接口通知到UI。登录成功后，如果有其他端登录着（在线），也会发出通知。返回的 `OnlineClient` 能够获取当前同时在线的客户端类型和操作系统。

- API 原型

```
/**
 * 注册/注销多端登录状态观察者。当有其他端登录或者注销时，会通过此接口通知到UI。
 * 登录成功后，如果有其他端登录着，也会发出通知。
 *
 * @param observer 观察者，参数为同时登录的其他端信息。
 *                 如果有其他端注销，参数为剩余的在线端。如果没有剩余在线端了，参数为null。
 * @param register true为注册，false为注销
 */
public void observeOtherClients(Observer<List<OnlineClient>> observer, boolean register);
```
- 参数说明

| 参数 | 说明 |
| :-------- | :------ |
| observer   |  观察者，参数为同时登录的其他端信息。<br>如果有其他端注销，参数为剩余的在线端。<br>如果没有剩余在线端了，参数为 null。  |
| register   |  是否注册观察者，注册为 true， 注销为 false  |

OnlineClient 接口说明：

| 返回值 | 方法 | 说明 |
| :-------- | :--------| :------ |
| String | getOs() | 客户端的操作系统信息 |
| int | getClientType() | 客户端类型 |
| long | getLoginTime() | 登录时间 |
| String | getClientIp() | 客户端 IP |

- 示例

```java
Observer<List<OnlineClient>> clientsObserver = new Observer<List<OnlineClient>>() {
        @Override
        public void onEvent(List<OnlineClient> onlineClients) {
            if (onlineClients == null || onlineClients.size() == 0) {
                OnlineClient client = onlineClients.get(0);
                switch (client.getClientType()) {
                    case ClientType.Windows:
                    // PC端
	                    break；
                    case ClientType.MAC:
                    // MAC端
                        break;
                    case ClientType.Web:
                    // Web端
                        break;
                    case ClientType.iOS:
                    // IOS端
	                    break；
                    case ClientType.Android:
                    // Android端
                        break;
                    default:
                        break;
            }
        }
    };

NIMClient.getService(AuthServiceObserver.class).observeOtherClients(clientsObserver, true);
```

#### 互踢

- API 介绍

网易云通信内置多端登录互踢策略为：移动端( Android 、 iOS )互踢，桌面端( PC 、 Mac 、 Web )互踢，移动端和桌面端共存（ 可以采用 kickOtherClient 主动踢下共存的其他端）。

- API 原型

```java
/**
 * 踢掉多端同时在线的其他端
 * @param client 被踢端信息
 * @return InvocationFuture 可设置回调函数，监听操作结果。
 */
public InvocationFuture<Void> kickOtherClient(OnlineClient client);
```

- 示例

```
NIMClient.getService(AuthService.class).kickOtherClient(client).setCallback(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void param) {
        // 踢出其他端成功
    }

    @Override
    public void onFailed(int code) {
		// 踢出其他端失败，返回失败code
    }

    @Override
    public void onException(Throwable exception) {
		// 踢出其他端错误
    }
});
```

说明：当被其他端踢掉，可以通过在线状态观察者来接收监听消息，参考监听用户在线状态章节

如果当前的互踢策略无法满足业务需求的话，可以联系我们取消内置互踢，根据多端登录的回调和当前的设备列表，判断本设备是否需要被踢出。如果需要踢出，直接调用登出接口并在界面上给出相关提示即可。

### <span id="登录状态">登录状态</span>

#### 监听用户在线状态

- API 介绍

登录成功后，SDK 会负责维护与服务器的长连接以及断线重连等工作。当用户在线状态发生改变时，会发出通知。此外，自动登录过程中也会有状态回调。

- API 原型

```java
/**
 * 注册/注销在线状态变化观察者。
 * 注册后，Observer的onEvent方法会被立即调用一次，告知观察者当前状态。
 *
 * @param observer 观察者, 参数为当前状态
 * @param register true为注册，false为注销
 */
public void observeOnlineStatus(Observer<StatusCode> observer, boolean register);
```

- 参数说明

| StatusCode属性    | 说明 |
| :-------- | :--------|
| INVALID | 未定义 |
| UNLOGIN | 未登录/登录失败 |
| NET_BROKEN | 网络连接已断开 |
| CONNECTING | 正在连接服务器 |
| LOGINING | 正在登录中 |
| SYNCING | 正在同步数据 |
| LOGINED | 已成功登录 |
| KICKOUT | 被其他端的登录踢掉 |
| KICK_BY_OTHER_CLIENT | 被同时在线的其他端主动踢掉 |
| FORBIDDEN | 被服务器禁止登录 |
| VER_ERROR | 客户端版本错误 |
| PWD_ERROR | 用户名或密码错误 |


- 示例

```java
NIMClient.getService(AuthServiceObserver.class).observeOnlineStatus(
	new Observer<StatusCode> () {
		public void onEvent(StatusCode status) {
			if (code.wontAutoLogin()) {
                // 被踢出、账号被禁用、密码错误等情况，自动登录失败，需要返回到登录界面进行重新登录操作
            }
		}
}, true);
```

被踢出的情况说明：

1\. 当用户在线时被踢出，会立刻收到被踢出的状态变更通知；

2\. 当用户离线后在其他设备成功登录，又在本设备重新自动登录时，也会收到被踢出的状态变更通知。

####  手动获取用户在线状态

- API 介绍

开发者可以主动获取当前用户在线状态，建议使用监听的方式获取用户在线状态。

- API 原型

```
/**
 * 获取当前用户状态
 *
 * @return 当前状态
 */
public static StatusCode getStatus();
```

- 示例

```java
StatusCode status = NIMClient.getStatus();
```

#### 监听数据同步状态

- API 介绍

登录成功后，SDK 会立即同步数据（用户资料、用户关系、群资料、离线消息、漫游消息等），同步开始和同步完成都会发出通知。

- API 原型

```
/**
 * 注册/注销登录后同步数据过程通知
 *
 * @param observer 观察者，参数为同步数据的过程状态（开始/结束）
 * @param register true为注册，false为注销
 */
public void observeLoginSyncDataStatus(Observer<LoginSyncStatus> observer, boolean register);

```

- 参数说明

| LoginSyncStatus属性 | 说明 |
| :-------- | :--------|
| NO_BEGIN | 未开始 |
| BEGIN_SYNC | 开始同步（正在同步）。<br>同步开始时，SDK 数据库中的数据可能还是旧数据。<br>（如果是首次登录，那么 SDK 数据库中还没有数据，<br>重新登录时 SDK 数据库中还是上一次退出时保存的数据）<br>在同步过程中，SDK 数据的更新会通过相应的监听接口发出数据变更通知。 |
| SYNC_COMPLETED | 同步完成 。SDK 数据库已完成更新|

- 示例

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

一般来说， App 开发者在登录完成后可以开始构建数据缓存：
- 登录完成后立即从 SDK 读取数据构建缓存，此时加载到的可能是旧数据；
- 在 Application 的 onCreate 中注册 XXXServiceObserver 来监听数据变化，那么在同步过程中， App 会收到数据更新通知，此时直接更新缓存。当同步完成时，缓存也就构建完成了。

### <span id="离线查看数据">离线查看数据</span>

- API 介绍

对于一些弱 IM 场景，需要在登录成功前或者未登录状态下访问指定账号的数据（聊天记录、好友资料等）。 SDK 提供两种方案：

1\. 使用自动登录。在登录成功前，可以访问 SDK 服务来读取本地数据（但不能发送数据）。

2\. 使用 AuthService#openLocalCache 接口打开本地数据，这是个同步方法，打开后即可读取 SDK 数据库中的记录。可以通过注销来切换账号查看本地数据。

- API 原型

```
/**
 * 离线时打开本地数据
 * 适用场景：在手动登录没有成功前（可能由于网络问题，登录时间较长），可以访问SDK本地数据。
 * 此外，不调用本接口，采用自动登录也能达到同样的效果。
 *
 * @return 是否成功打开SDK本地数据
 */
public boolean openLocalCache(String account);
```

- 示例

```java
boolean res = NIMClient.getService(AuthService.class).openLocalCache(account);
```

### <span id="断线重连机制">断线重连机制</span>

SDK 提供三种断线重连的策略（重新建立与网易云通信服务器的连接并重新登录）：

1\. 当网络由连通变为断开时，SDK 会启动立即上报网络断开的状态，并启动重连定时器，采用特定的策略并根据当前网络状态进行重连（如果 App 处于后台，重连时间间隔会较长）。

2\. SDK会监听设备的网络连接状况，当监听到手机断网重连上网络的通知后，会立即进行重连并登录。

3\. 应用长时间处于后台（后台进程可能活着但网络连接被系统切断）后切回到前台（恢复网络连通），SDK 监测到当前处于未登录状态，会在短时间内进行重连。

## <span id="消息收发">消息收发</span>

### <span id="消息功能概述">消息功能概述</span>

SDK 提供一套完善的消息传输管理服务，包括收发消息，存储消息，上传下载附件，管理最近联系人等。

SDK 原生支持发送文本，语音，图片，视频，文件，地理位置，提醒，智能对话机器人等 8 种类型消息，同时支持用户发送自定义的消息类型。

消息功能具体介绍可参考产品介绍的 [基础消息功能](/docs/product/IM即时通讯/产品介绍/基础消息功能)

网易云通信消息对象均为 `IMMessage`，不同消息类型以 `MsgTypeEnum` 作区分。

- IMMessage 部分重要参数说明

| 返回值      |  IMMessage接口 |   说明   |
| :-------- | :--------| :-------------------------------- |
| String | getUuid() | 获取消息的uuid, 该域在生成消息时即会填上 |
| String | getSessionId() | 获取聊天对象的Id（好友帐号，群ID等） |
| SessionTypeEnum | getSessionType() | 获取会话类型，<br>SessionTypeEnum.P2P 为单聊类型，<br>SessionTypeEnum.Team 为群聊类型，<br>SessionTypeEnum.System 为系统消息，<br>SessionTypeEnum.ChatRoom 为聊天室消息 |
| String | getFromNick() | 获取消息发送者的昵称 |
| MsgTypeEnum | getMsgType() | 获取消息类型。MsgTypeEnum的类型，请见下表 |
| MsgStatusEnum | getStatus() | 获取消息接收/发送状态 |
| MsgDirectionEnum | getDirect() | 获取消息方向，<br>MsgDirectionEnum.Out 表示发出去的消息，<br>MsgDirectionEnum.In 表示接收到的消息 |
| String | getContent() | 获取消息具体内容，<br>当消息类型 为MsgTypeEnum.text 时，该域为消息内容。<br>当为其他消息类型时，该域为可选项，<br>如果设置，将作为 iOS 的 apns 推送文本以及 android 内置消息推送的显示文本<br>（ 1.7.0及以上版本建议使用 pushContent ） |
| long | getTime() | 获取消息时间，单位为 ms |
| String | getFromAccount() | 获取该条消息发送方的帐号 |
| void | setAttachment<br>(MsgAttachment<br> attachment) | 设置消息附件对象。详细见 MsgAttachment 参数说明 |
| MsgAttachment | getAttachment() | 获取消息附件对象。仅当 getMsgType() 返回为非 text 时有效 |
| AttachStatusEnum | getAttachStatus() | 获取消息附件接收/发送状态，<br>AttachStatusEnum.def 表示默认状态，未开始。<br>AttachStatusEnum.transferring 表示正在传输。<br>AttachStatusEnum.transferred 表示传输成功。<br>AttachStatusEnum.fail 表示传输失败 |
| CustomMessageConfig | getConfig() | 获取消息配置。具体见CustomMessageConfig参数说明 |
| Map<String, Object> | getRemoteExtension() | 获取扩展字段（该字段会发送到其他端） |
| void | setRemoteExtension<br>(Map<br> remoteExtension) | 设置扩展字段<br>（该字段会发送到其他端），<br>最大长度 1024 字节。<br>开发者需要保证此 Map 能够转换为 JsonObject |
| Map<String, Object> | getLocalExtension() | 获取本地扩展字段（仅本地有效） |
| void | setLocalExtension<br>(Map<br> localExtension) | 设置本地扩展字段<br>（该字段仅在本地使用有效，不会发送给其他端），<br>最大长度 1024 字节。<br>开发者需要保证此 Map 能够转换为 JsonObject |
| String | getPushContent() | 获取自定义推送文案 |
| void | setPushContent(String pushContent) | 设置自定义推送文案<br>（ 1.7.0 及以上版本建议使用此字段，不要使用 setContent 来设置推送文案）, <br>最大长度 200 字节 |
| Map<String, Object> | getPushPayload() | 获取第三方自定义的推送属性 |
| void | setPushPayload<br>(Map<br> pushPayload) | 设置第三方自定义的推送属性， <br>开发者需要保证此 Map 能够转换为 JsonObject，<br>最大长度 2048 字节 |
| MemberPushOption | getMemberPushOption() | 获取指定成员推送选项，详细介绍见指定推送设置详解章节 |
| boolean | isRemoteRead() | 判断自己发送的消息对方是否已读，<br>true：对方已读；false：对方未读 |
| int | getFromClientType() | 获取消息发送方类型，与 ClientType 类比较 |
| NIMAntiSpamOption | getNIMAntiSpamOption() | 获取易盾反垃圾配置项，详细介绍见反垃圾设置详解章节 |


- MsgTypeEnum 参数说明

| MsgTypeEnum参数      |     说明 |
| :-------- | :--------|
| undef | 未知消息 |
| text | 文本消息 |
| image | 图片消息 |
| audio | 音频消息 |
| video | 视频消息 |
| location | 位置消息 |
| file | 文件消息 |
| avchat | 音视频通话消息 |
| notification | 通知消息 |
| tip | 提醒类型消息 |
| robot | 机器人消息 |
| custom | 第三方 App 自定义消息 |

消息内容根据类型不同也不一样。文本消息最为简单，消息内容就是 `content` 字符串。其他消息类型均带有一个消息附件对象 `MsgAttachment`，该对象在传输时一般序列化为 json 格式字符串。内建的消息附件主要有以下几种：

| 内建消息附件      |     说明 |
| :-------- | :--------|
| LocationAttachment | 位置消息附件对象类型。 |
| FileAttachment | 文件消息附件对象类型（继承该附件， SDK 在发送消息时将自动上传文件）。 |
| ImageAttachment | 图片消息附件对象类型。 |
| AudioAttachment | 音频消息附件对象类型。 |
| VideoAttachment | 视频消息附件对象类型。 |
| RobotAttachment | 机器人消息附件对象类型。 |
| NotificationAttachment | 通知类消息附件基类，目前主要用于群类各种操作事件的通知。<br>该类型事件仅由服务器生成和发送，客户端不会生成该类型的消息。 |

### <span id="消息发送">消息发送</span>

- API 介绍

发送原生支持的消息类型流程比较简单，先通过 `MessageBuilder` 提供的接口创建消息对象，然后调用 `MsgService` 的 `sendMessage` 接口发送出去即可。

- API 原型

```
/**
 * 发送消息。
 * @param msg    带发送的消息体，由{@link MessageBuilder}构造
 * @param resend 如果是发送失败后重发，标记为true，否则填false
 * @return InvocationFuture 可以设置回调函数。消息发送完成后才会调用，如果出错，会有具体的错误代码。
 */
public InvocationFuture<Void> sendMessage(IMMessage msg, boolean resend);
```

#### <span id="文本消息">文本消息</span>

首先创建文本消息，调用 sendMessage 方法发送。

文本消息创建原型：

```
/**
 * 创建一条普通文本消息
 *
 * @param sessionId   聊天对象ID
 * @param sessionType 会话类型
 * @param text        文本消息内容
 * @return IMMessage 生成的消息对象
 */
public static IMMessage createTextMessage(String sessionId, SessionTypeEnum sessionType, String text);
```

- 参数说明

| 参数 | 说明 |
| :-------- | :--------|
| sessionId | 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID |
| sessionType | 聊天类型，SessionTypeEnum.P2P 为单聊类型，SessionTypeEnum.Team 为群聊类型 |
| text | 文本消息内容 |

- 示例

```
// 该帐号为示例，请先注册
String account = "testAccount";
// 以单聊类型为例
SessionTypeEnum sessionType = SessionTypeEnum.P2P;
String text = "this is an example";
// 创建一个文本消息
IMMessage textMessage = MessageBuilder.createTextMessage(account, sessionType, text);

// 发送给对方
NIMClient.getService(MsgService.class).sendMessage(textMessage, false);
```

#### <span id="图片消息">图片消息</span>

图片消息创建原型

```
**
 * 创建一条图片消息
 *
 * @param sessionId   聊天对象ID
 * @param sessionType 会话类型
 * @param file        图片文件
 * @param displayName 图片文件的显示名，可不同于文件名
 * @return IMMessage 生成的消息对象
 */
public static IMMessage createImageMessage(String sessionId, SessionTypeEnum sessionType, File file, String displayName);

```
- 参数说明

| 参数 | 说明 |
| :-------- | :--------|
| sessionId | 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID |
| sessionType | 聊天类型，SessionTypeEnum.P2P 为单聊类型，SessionTypeEnum.Team 为群聊类型 |
| file | 图片文件对象 |
| displayName | 图片文件的显示名，可不同于文件名，可设置为 null |

- 示例

```
// 该帐号为示例，请先注册
String account = "testAccount";
// 以单聊类型为例
SessionTypeEnum sessionType = SessionTypeEnum.P2P;
// 示例图片，需要开发者在相应目录下有图片
File file = new File("/sdcard/test.jpg");
// 创建一个图片消息
IMMessage message = MessageBuilder.createImageMessage(account, sessionType, file, file.getName());

// 发送给对方
NIMClient.getService(MsgService.class).sendMessage(message, false);
```

#### <span id="音频消息">音频消息</span>

音频消息创建原型：

```
/**
 * 创建一条音频消息
 *
 * @param sessionId   聊天对象ID
 * @param sessionType 会话类型
 * @param file        音频文件对象
 * @param duration    音频文件持续时间，单位是ms
 * @return IMMessage 生成的消息对象
 */
public static IMMessage createAudioMessage(String sessionId, SessionTypeEnum sessionType, File file, long duration);
```

- 参数说明

| 参数 | 说明 |
| :-------- | :--------|
| sessionId | 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID |
| sessionType | 聊天类型，SessionTypeEnum.P2P 为单聊类型，SessionTypeEnum.Team 为群聊类型 |
| file | 音频文件对象 |
| duration | 音频文件持续时间，单位是 ms |

- 示例

```
// 该帐号为示例，请先注册
String account = "testAccount";
// 以单聊类型为例
SessionTypeEnum sessionType = SessionTypeEnum.P2P;
// 示例音频，需要开发者在相应目录下有文件
File audioFile = new File("/sdcard/testAudio.mp3");
// 音频时长，时间为示例
long audiolength = 2000;
// 创建音频消息
IMMessage audioMessage = MessageBuilder.createAudioMessage(account, sessionType, audioFile, audioLength);

// 发送给对方
NIMClient.getService(MsgService.class).sendMessage(audioMessage, false);
```

#### <span id="视频消息">视频消息</span>

视频消息创建原型：

```
/**
 * 创建一条视频消息
 *
 * @param sessionId   聊天对象ID
 * @param sessionType 会话类型
 * @param file        视频文件对象
 * @param duration    视频文件持续时间
 * @param width       视频宽度
 * @param height      视频高度
 * @param displayName 视频文件显示名，可以为空
 * @return 视频消息
 */
public static IMMessage createVideoMessage(String sessionId, SessionTypeEnum sessionType, File file, long duration, int width, int height, String displayName);
```
- 参数说明

| 参数 | 说明 |
| :-------- | :--------|
| sessionId | 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID |
| sessionType | 聊天类型，SessionTypeEnum.P2P 为单聊类型，SessionTypeEnum.Team 为群聊类型 |
| file | 视频文件对象 |
| duration | 视频文件持续时间，单位 ms |
| width | 视频宽度 |
| height | 视频高度 |
| displayName | 视频文件显示名，可为 null |

- 示例

```
// 该帐号为示例，请先注册
String account = "testAccount";
// 以单聊类型为例
SessionTypeEnum sessionType = SessionTypeEnum.P2P;
// 示例视频，需要开发者在相应目录下有文件
File file = new File("/sdcard/testVideo.mp4");
// 获取视频mediaPlayer
MediaPlayer mediaPlayer;
try {
    mediaPlayer = MediaPlayer.create(context, Uri.fromFile(file));
} catch (Exception e) {
    e.printStackTrace();
}
// 视频文件持续时间
long duration = mediaPlayer == null ? 0 : mediaPlayer.getDuration();
// 视频高度
int height = mediaPlayer == null ? 0 : mediaPlayer.getVideoHeight();
// 视频宽度
int width = mediaPlayer == null ? 0 : mediaPlayer.getVideoWidth();
// 创建视频消息
IMMessage message = MessageBuilder.createVideoMessage(account, sessionType, file, duration, width, height, null);

// 发送给对方
NIMClient.getService(MsgService.class).sendMessage(message, false);
```
#### <span id="文件消息">文件消息</span>

文件消息创建原型:

```
/**
 * 创建一条文件消息
 *
 * @param sessionId   聊天对象ID
 * @param sessionType 会话类型
 * @param file        文件
 * @param displayName 文件的显示名，可不同于文件名
 * @return IMMessage 生成的消息对象
 */
public static IMMessage createFileMessage(String sessionId, SessionTypeEnum sessionType, File file, String displayName);
```

- 参数说明

| 参数      |     说明 |
| :-------- | :-------- |
| sessionId | 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID |
| sessionType | 聊天类型，SessionTypeEnum.P2P 为单聊类型，SessionTypeEnum.Team 为群聊类型 |
| file | 文件 |
| displayName | 文件的显示名，可不同于文件名 |

- 示例

```
// 该帐号为示例，请先注册
String account = "testAccount";
// 以单聊类型为例
SessionTypeEnum sessionType = SessionTypeEnum.P2P;
// 示例文件，需要开发者在相应目录下有文件
File file = new File("/sdcard/test.txt");
// 创建文件消息
IMMessage message = MessageBuilder.createFileMessage(account, sessionType, file, file.getName());

// 发送给对方
NIMClient.getService(MsgService.class).sendMessage(message, false);
```

#### <span id="地理位置消息">地理位置消息</span>

地理位置消息创建原型：

```
/**
 * 创建一条地理位置信息
 *
 * @param sessionId   聊天对象ID
 * @param sessionType 会话类型
 * @param lat         纬度
 * @param lng         经度
 * @param addr        地理位置描述信息
 * @return IMMessage 生成的消息对象
 */
public static IMMessage createLocationMessage(String sessionId, SessionTypeEnum sessionType, double lat, double lng, String addr);
```
- 参数说明

| 参数      |     说明 |
| :-------- | :-------- |
| sessionId | 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID |
| sessionType | 聊天类型，SessionTypeEnum.P2P 为单聊类型，SessionTypeEnum.Team 为群聊类型 |
| lat | 纬度 |
| lng | 经度 |
| addr | 地理位置描述信息 |

- 示例

```
// 该帐号为示例，请先注册
String account = "testAccount";
// 以单聊类型为例
SessionTypeEnum sessionType = SessionTypeEnum.P2P;
// 纬度
double lat = 30.3;
// 经度
double lng = 120.2;
// 地理位置描述信息
String addr = "杭州";
// 创建地理位置信息
IMMessage message = MessageBuilder.createLocationMessage(account, sessionType, lat, lng, addr);

// 发送给对方
NIMClient.getService(MsgService.class).sendMessage(message, false);
```

#### <span id="机器人提问消息">机器人提问消息</span>

机器人提问消息是指，普通用户发送给机器人的消息（也称作机器人上行消息）。机器人提问消息创建原型：

```
/**
 * @param sessionId    聊天对象ID
 * @param sessionType  会话类型
 * @param robotAccount @机器人账号
 * @param text         消息显示的文案，一般基于content加上@机器人的标签作为消息显示的文案。
 * @param type         机器人消息类型，参考{@link com.netease.nimlib.sdk.robot.model.RobotMsgType}
 * @param content      消息内容，如果消息类型是{@link com.netease.nimlib.sdk.robot.model.RobotMsgType#TEXT}，必须传入说话内容
 * @param target       如果消息类型是{@link com.netease.nimlib.sdk.robot.model.RobotMsgType#LINK}, 必须传入跳转目标
 * @param params       如果消息类型是{@link com.netease.nimlib.sdk.robot.model.RobotMsgType#LINK}时，可能需要传入参数
 * @return 机器人消息
 */
public static IMMessage createRobotMessage(String sessionId, SessionTypeEnum sessionType, String robotAccount, String text, String type, String content, String target, String params);
```

- 参数说明

| 参数 | 说明 |
| :-------- | :--------|
| sessionId | 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID |
| sessionType | 聊天类型，SessionTypeEnum.P2P 为单聊类型，SessionTypeEnum.Team 为群聊类型 |
| robotAccount | 机器人帐号 |
| text | 消息显示的文案，一般基于 content 加上机器人的标签作为消息显示的文案。 |
| type | 机器人消息类型 RobotMsgType。<br>RobotMsgType.WELCOME 表示欢迎消息，<br>RobotMsgType.TEXT 表示文本消息，<br>RobotMsgType.LINK 表示模块跳转消息|
| content | 消息内容，如果机器人消息类型是 RobotMsgType.TEXT，必须传入说话内容 |
| target | 跳转目标， 如果机器人消息类型是 RobotMsgType.LINK，必须传入跳转目标 |
| params | 跳转参数，如果消息类型是 RobotMsgType.LINK时，可能需要传入参数 |

- 示例

```
// 该帐号为示例，请先注册
String account = "testAccount";
// 以单聊类型为例
SessionTypeEnum sessionType = SessionTypeEnum.P2P;
// 机器人帐号
String robotAccount = "robotAccount";
// 消息文案
String text = "这是消息文案";
// 机器人消息类型, 文本消息类型
String type = RobotMsgType.TEXT;
// 消息内容
String content = "这是消息内容";
// 创建机器人消息
IMMessage message = MessageBuilder.createRobotMessage(account, sessionType, robotAccount, text, type, content, null, null);

// 发送给对方
NIMClient.getService(MsgService.class).sendMessage(message, false);
```
#### <span id="机器人跳转消息">机器人跳转消息</span>

机器人跳转消息的原型和参数说明，参考机器人提问消息章节。机器人消息类型变为 RobotMsgType.LINK，同时注意，必须传入跳转目标。

- 示例

```
// 该帐号为示例，请先注册
String account = "testAccount";
// 以单聊类型为例
SessionTypeEnum sessionType = SessionTypeEnum.P2P;
// 机器人帐号
String robotAccount = "robotAccount";
// 消息文案
String text = "这是消息文案";
// 机器人消息类型, 跳转消息类型
String type = RobotMsgType.LINK;
// 消息内容
String content = "这是消息内容";
// view是界面元素
RobotLinkView robotLinkView = (RobotLinkView) view;
LinkElement element = robotLinkView.getElement();
// 跳转目标
String target;
// 跳转参数
String params;
if (element != null) {
	target = element.getTarget();
	params = element.getParams();
}

// 创建机器人消息
IMMessage message = MessageBuilder.createRobotMessage(account, sessionType, robotAccount, text, type, content, target, params);

// 发送给对方
NIMClient.getService(MsgService.class).sendMessage(message, false);
```

#### <span id="通知消息">通知消息</span>

- 介绍

通知消息属于会话中的一种消息，其对应的数据结构为 IMMessage，消息类型为 MsgTypeEnum.notification。目前用于（已操作完成的）群通知和聊天室通知事件。通知消息由服务器下发，客户端无法生成通知消息。

- 独享属性列表

通知消息的附件类型基类为 NotificationAttachment，通过 NotificationAttachment#getType() 可以得到通知消息的类型 NotificationType，可根据 NotificationType 区分不同的通知消息，具体类型如下：

| NotificationType属性 | 说明 |
| :-------- | :--------|
| undefined | 未定义类型 |
| InviteMember | TEAM：邀请群成员，用于讨论组中，讨论组可直接拉人入群 |
| KickMember | TEAM：移除群成员 |
| LeaveTeam | TEAM：有成员离开群 |
| UpdateTeam | TEAM：群资料更新 |
| DismissTeam | TEAM：群被解散 |
| PassTeamApply | TEAM：管理员通过用户入群申请 |
| TransferOwner | TEAM：群组拥有者权限转移通知 |
| AddTeamManager | TEAM：新增管理员通知 |
| RemoveTeamManager | TEAM：撤销管理员通知 |
| AcceptInvite | TEAM：用户接受入群邀请 |
| MuteTeamMember | TEAM：群成员禁言/解禁 |
| ChatRoomMemberIn | ChatRoom：成员进入聊天室 |
| ChatRoomMemberExit | ChatRoom：成员离开聊天室 |
| ChatRoomMemberBlackAdd | ChatRoom：成员被加黑 |
| ChatRoomMemberBlackRemove | ChatRoom：成员被取消黑名单 |
| ChatRoomMemberMuteAdd | ChatRoom：成员被设置禁言 |
| ChatRoomMemberMuteRemove | ChatRoom：成员被取消禁言 |
| ChatRoomManagerAdd | ChatRoom：设置为管理员 |
| ChatRoomManagerRemove | ChatRoom：取消管理员 |
| ChatRoomCommonAdd | ChatRoom：成员设定为固定成员 |
| ChatRoomCommonRemove | ChatRoom：成员取消固定成员 |
| ChatRoomClose | ChatRoom：聊天室被关闭了 |
| ChatRoomInfoUpdated | ChatRoom：聊天室信息被更新了 |
| ChatRoomMemberKicked | ChatRoom：成员被踢了 |
| ChatRoomMemberTempMuteAdd | ChatRoom: 新增临时禁言 |
| ChatRoomMemberTempMuteRemove | ChatRoom: 主动解除临时禁言 |
| ChatRoomMyRoomRoleUpdated | ChatRoom: 成员主动更新了聊天室内的角色信息(仅指nick/avator/ext) |
| ChatRoomQueueChange | ChatRoom: 麦序队列中有变更 |
| ChatRoomRoomMuted | ChatRoom: 聊天室被禁言了,只有管理员可以发言,其他人都处于禁言状态 |
| ChatRoomRoomDeMuted | ChatRoom: 聊天室解除全体禁言状态 |

通知消息不仅可以判断通知类型，还携带不同的参数信息，由基类 NotificationAttachment 派生出针对不同通知消息类型的附件。

| 通知消息附件类型     |     说明 |
| :-------- | :--------|
| MemberChangeAttachment | 群成员变动的通知消息实体。包括群成员变化和管理权限变化 |
| MuteMemberAttachment | MemberChangeAttachment 的子类，<br>群成员被禁言或者被取消禁言的通知消息实体 |
| UpdateTeamAttachment | 更新群资料的通知消息实体 |
| LeaveTeamAttachment | 成员离开讨论组的通知消息实体 |
| DismissAttachment | 解散群的通知消息实体 |
| ChatRoomNotificationAttachment | 聊天室通知消息实体 |
| ChatRoomRoomMemberInAttachment | ChatRoomNotificationAttachment 的子类, <br>进入聊天室通知消息实体|
| ChatRoomTempMuteRemoveAttachment | ChatRoomNotificationAttachment 的子类, <br>解除聊天室禁言通知消息实体|
| ChatRoomQueueChangeAttachment | ChatRoomNotificationAttachment 的子类, <br>聊天室队列变更通知消息实体|
| ChatRoomTempMuteAddAttachment | ChatRoomNotificationAttachment 的子类, <br>添加聊天室禁言通知消息实体 |

- 特别说明

1\. 通知类消息有在线、离线、漫游。
2\. 不计入消息未读数。
3\. 没有通知栏提醒（如有需要，第三方自行实现）。

#### <span id="提示消息">提示消息</span>

- API 介绍

提示消息（又叫做 Tip 消息）主要用于会话内的通知提醒，可以看做是自定义消息的简化，有独立的消息类型 MsgTypeEnum.tip 。
区别于自定义消息，Tip 消息暂不支持 setAttachment，如果要使用 Attachment 请使用自定义消息。
Tip 消息使用场景例如：进入会话时出现的欢迎消息，或是会话过程中命中敏感词后的提示消息等场景，当然也可以用自定义消息实现，只是相对复杂一些。

- API 原型

```
/**
 * 创建一条提示消息
 *
 * @param sessionId   聊天对象ID
 * @param sessionType 会话类型
 * @return IMMessage 生成的消息对象
 */
public static IMMessage createTipMessage(String sessionId, SessionTypeEnum sessionType);
```

- 示例

```
// 向群里插入一条Tip消息，使得该群能立即出现在最近联系人列表（会话列表）中，满足部分开发者需求。
Map<String, Object> content = new HashMap<>(1);
content.put("content", "成功创建高级群");
// 创建tip消息，teamId需要开发者已经存在的team的teamId
IMMessage msg = MessageBuilder.createTipMessage(teamId, SessionTypeEnum.Team);
msg.setRemoteExtension(content);
// 自定义消息配置选项
CustomMessageConfig config = new CustomMessageConfig();
// 消息不计入未读
config.enableUnreadCount = false;
msg.setConfig(config);
// 消息发送状态设置为success
msg.setStatus(MsgStatusEnum.success);
// 保存消息到本地数据库，但不发送到服务器
NIMClient.getService(MsgService.class).saveMessageToLocal(msg, true);

```

#### <span id="自定义消息">自定义消息</span>

- API 介绍

除了内建消息类型以外，SDK 还支持收发自定义消息类型。SDK 不负责定义和解析自定义消息的具体内容，解释工作由开发者完成。SDK 会将自定义消息存入消息数据库，会和内建消息一并展现在消息记录中。

为了使用更加方便，自定义消息采用附件的方式展示给开发者。体现在 `IMMessage` 类中，自定义消息的内容会被解析为 `MsgAttachment` 对象，由于 SDK 并不知道自定义消息的格式，第三方 App 需要注册一个自定义消息解析器。当第三方 App 调用查询查询消息历史的接口，或者是 SDK 收到消息通知第三方 App 时，就能将自定义消息内容转换为一个 `MsgAttachment` 对象，然后就可以同操作图片消息等带附件消息一样，操作自定义消息了。

与 IMMessage 一致，为了增加自定义消息的灵活性，SDK 还提供了对其生命周期和推送参数的一些配置选项，开发者可以自主设定一条自定义消息是否要保存云端消息记录，是否要漫游，发送时是否要多端同步等的配置选项 `CustomMessageConfig`。

- API 原型

创建自定义消息原型

```

/**
 * 创建一条APP自定义类型消息, 同时提供描述字段，可用于推送以及状态栏消息提醒的展示。
 *
 * @param sessionId   聊天对象ID
 * @param sessionType 会话类型
 * @param content     消息简要描述，可通过IMMessage#getContent()获取，主要用于用户推送展示。
 * @param attachment  消息附件对象
 * @param config      自定义消息配置
 * @return 自定义消息
 */
public static IMMessage createCustomMessage(String sessionId, SessionTypeEnum sessionType, String content, MsgAttachment attachment, CustomMessageConfig config);
```

- 示例一（剪刀石头布）

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

- 示例二 （阅后即焚）

若需要发送文件类型消息，例如图片等，可以参考阅后即焚的实现。具体实现步骤如下：

第一步，定义一个自定义的附件类型，并继承 `FileAttachment`。注意，成员变量都要实现 Serializable。

```java
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

为方便开发者参考，我们提供实现自定义消息的示例：
[Android自定义消息(类似微信骰子功能) ](http://bbs.netease.im/read-tid-257)

### <span id="消息特殊设置"> 消息特殊设置 </span>

####  <span id="消息扩展字段">消息扩展字段 <span>

扩展字段分为服务器扩展字段（ RemoteExtension ）和本地扩展字段（ LocalExtension ），最大长度 1024 字节。对于服务器扩展字段，该字段会发送到其他端，而本地扩展字段仅在本地有效。对于这两种扩展字段， SDK 都会存储在数据库中。示例如下：

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

3\. 1.8.0 版本后消息提醒支持定制，开发者可以自行定制@消息在通知栏提醒时需要展示的文案。

#### <span id="消息属性设置详解">消息属性设置详解</span>

发送消息时可以设置消息配置选项 `CustomMessageConfig`，主要用于设定消息的声明周期，是否需要推送，是否需要计入未读数等。

- 参数说明

自定义消息配置 CustomMessageConfig 属性

```
/**
 * 该消息是否要保存到服务器，如果为false，通过 MsgService#pullMessageHistory(IMMessage, int, boolean)拉取的结果将不包含该条消息。
 * 默认为true。
 */
public boolean enableHistory = true;

/**
 * 该消息是否需要漫游。如果为false，一旦某一个客户端收取过该条消息，其他端将不会再漫游到该条消息。
 * 默认为true
 */
public boolean enableRoaming = true;

/**
 * 多端同时登录时，发送一条自定义消息后，是否要同步到其他同时登录的客户端。
 * 默认为true
 */
public boolean enableSelfSync = true;

/**
 * 该消息是否要消息提醒，如果为true，那么对方收到消息后，系统通知栏会有提醒。
 * 默认为true
 */
public boolean enablePush = true;

/**
 * 该消息是否需要推送昵称（针对iOS客户端有效），如果为true，那么对方收到消息后，iOS端将显示推送昵称。
 * 默认为true
 */
public boolean enablePushNick = true;

/**
 * 该消息是否要计入未读数，如果为true，那么对方收到消息后，最近联系人列表中未读数加1。
 * 默认为true
 */
public boolean enableUnreadCount = true;

/**
 * 该消息是否支持路由，如果为true，默认按照app的路由开关（如果有配置抄送地址则将抄送该消息）
 * 默认为true
 */
public boolean enableRoute = true;
```

- 示例

```
// 该帐号为示例，请先注册
String account = "testAccount";
// 以单聊类型为例
SessionTypeEnum sessionType = SessionTypeEnum.P2P;
String text = "this is an example";
// 创建一个文本消息
IMMessage textMessage = MessageBuilder.createTextMessage(account, sessionType, text);

// 消息的配置选项
CustomMessageConfig config = new CustomMessageConfig();
// 该消息不保存到服务器
config.enableHistory = false;
// 该消息不漫游
config.enableRoaming = false;
// 该消息不同步
config.enableSelfSync = false;
textMessage.setConfig(config);

// 发送给对方
NIMClient.getService(MsgService.class).sendMessage(textMessage, false);
```

#### <span id="反垃圾设置详解">反垃圾设置详解</span>

支持设置易盾反垃圾功能，开启该功能后服务端会将消息抄送到易盾做反垃圾处理。Android 需要对即将发送的消息调用 `IMMessage` 的 `setNIMAntiSpamOption` 方法，传递参数为 `NIMAntiSpamOption` 对象，开发者需要构造这个 `NIMAntiSpamOption` 对象。

- 参数说明

NIMAntiSpamOption 参数说明

|  NIMAntiSpamOption参数  |  说明 |
| :-------- | :-------- |
| enable | boolean，是否过易盾反垃圾 |
| content | String，自定义的反垃圾字段 |

content 必须传递 json 对象，长度不超过 5000 字节，格式如下

```json
{
    "type": 1, //1:文本，2：图片，3视频
    "data": "" //文本内容or图片地址or视频地址
}
```

- 示例

```
// 该帐号为示例，请先注册
String account = "testAccount";
// 以单聊类型为例
SessionTypeEnum sessionType = SessionTypeEnum.P2P;
String text = "这是反垃圾文本";
// 创建一个文本消息
IMMessage textMessage = MessageBuilder.createTextMessage(account, sessionType, text);

// 构造反垃圾对象
NIMAntiSpamOption antiSpamOption = new NIMAntiSpamOption();
antiSpamOption.enable = true;
JSONObject json = new JSONObject();
json.put("type", 1);
json.put("data", "反垃圾");
antiSpamOption.content = json.toString();
textMessage.setNIMAntiSpamOption(antiSpamOption);

// 发送给对方
NIMClient.getService(MsgService.class).sendMessage(textMessage, false);

```

### <span id="消息接收">消息接收</span>

#### 基本概念介绍

- API 介绍

通过添加消息接收观察者 `ChatRoomServiceObserver#observeReceiveMessage`，在有新消息到达时，第三方 APP 就可以接收到通知。该代码的典型场景为消息对话界面，在界面 `onCreate` 里注册消息接收观察者，在 `onDestroy` 中注销观察者。在收到消息时，判断是否是当前聊天对象的消息，如果是，加入到列表中显示。

- API 原型

```java
/**
 * 注册/注销消息接收观察者。
 *     通知的消息列表中的消息不一定全是接收的消息，也有可能是自己发出去，比如其他端发的消息漫游过来，
 *     或者调用MsgService#saveMessageToLocal(IMMessage, boolean)后，notify参数设置为true，通知出来的消息。
 * @param observer 观察者， 参数为收到的消息列表，消息列表中的消息均保证来自同一个聊天对象。
 * @param register true为注册，false为注销
 */
public void observeReceiveMessage(Observer<List<IMMessage>> observer, boolean register);

```

- 示例

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

#### 文本消息接收
文本消息的接收比较简单，通过注册监听消息接口，接收消息即可，具体示例见上文。

#### 多媒体消息接收

多媒体消息包括语音消息，图片消息，视频消息等带有文件附件的消息。通过 `MsgServiceObserve#observeReceiveMessage` 接收到消息时，需要下载文件附件，包含自动下载附件和手动下载附件两种方式，默认为自动下载。下载完成之后，才可以通过 IMMessage 获取到相应的文件路径。可以通过监听消息状态，来判断附件是否下载完成。

1\. 附件参数说明

多媒体附件的基类是 FileAttachment，它的子类有原生消息类型中的：视频消息附件 VideoAttachment，图片消息附件 ImageAttachment， 音频消息附件 AudioAttachment，以及自定义消息的阅后即焚消息附件 SnapChatAttachment。

以 FileAttachment 为例，说明附件参数：

| 返回值 | FileAttachment 接口      |     说明 |
| :-------- | :-------- | :--------|
| String | getPath() | 获取文件本地路径，若文件不存在，返回 null。 <br>语音消息的文件路径，在附件下载完成之后可以获取。<br>图片或视频的文件路径，需要手动下载之后获取，收到消息时，SDK 只默认自动下载缩略图文件 |
| String | getPathForSave() | 获取用于保存该文件的位置 |
| String | getThumbPath() | 获取缩略图文件的本地路径，若文件不存在，返回 null。<br>当消息状态显示缩略图下载完成之后，可获取缩略图文件路径 |
| String | getThumbPathForSave() | 获取用于保存缩略图文件的位置 |
| void | setPath(String path) | 设置文件路径 |
| long | getSize() | 获取文件大小，单位为byte |
| void | setSize(long size) | 设置文件大小，单位为 byte |
| String | getMd5() | 获取文件内容 MD5 |
| void | setMd5(String md5) | 设置文件内容 MD5 |
| String | getUrl() | 获取文件在服务器上的下载 url。若文件还未上传，返回 null |
| void | setUrl(String url) | 设置文件在服务器上下载 url |
| String | getExtension() | 获取文件后缀名 |
| void | setExtension(String extension) | 设置文件后缀名 |
| String | getFileName() | 获取文件名 |
| String | getDisplayName() | 获取文件的显示名。可以和文件名不同，仅用于界面展示 |
| String | setDisplayName() | 设置文件显示名 |

2\. 自动下载附件

如果接收到多媒体消息，SDK 默认会在后台自动下载附件。如果是语音消息，直接下载文件，如果是图片或视频消息，下载缩略图文件。

3\. 手动下载附件

开发者可在 Application 初始化，配置 SDKOptions 时，将 SDKOptions.preloadAttach 设置为 false 来关闭自动下载，并在用户翻阅到对应消息，再通过以下代码手动下载。

- 原型

```
/**
 * 正常情况收到消息后附件会自动下载。如果下载失败，可调用该接口重新下载
 *
 * @param msg   附件所在的消息体
 * @param thumb 下载缩略图还是原文件。为true时，仅下载缩略图。
 *              该参数仅对图片和视频类消息有效
 * @return AbortableFuture 调用跟踪。可设置回调函数，可中止下载操作
 */
public AbortableFuture<Void> downloadAttachment(IMMessage msg, boolean thumb);
```

- 参数说明

| 参数 | 说明 |
| :-------- | :--------|
| msg | 附件所在的消息体 |
| thumb | 下载缩略图还是下载原文件。为 true 时，仅下载缩略图。该参数仅对图片和视频类消息有效 |

- 示例

```java
// 下载之前判断一下是否已经下载。若重复下载，会报错误码414。（以SnapChatAttachment为例）
private boolean isOriginImageHasDownloaded(final IMMessage message) {
    if (message.getAttachStatus() == AttachStatusEnum.transferred &&
        !TextUtils.isEmpty(((SnapChatAttachment) message.getAttachment()).getPath())) {
        return true;
    }
    return false;
}
// 下载附件，参数1为消息对象，参数2为是下载缩略图还是下载原图。
// 因为下载的文件可能会很大，这个接口返回类型为 AbortableFuture ，允许用户中途取消下载。
AbortableFuture future = NIMClient.getService(MsgService.class).downloadAttachment(message, true);
```

> 说明：错误码414可能是重复下载，或者下载参数错误。

4\. 监听消息状态

通过监听消息状态的变化，来查看消息是否下载完成。这个接口可以监听消息接收或发送状态 MsgStatusEnum 和 消息附件接收或发送状态 AttachStatusEnum 的变化。当状态更改为 AttachStatusEnum.transferred 表示附件下载成功。

- 原型

```
/**
 * 注册/注销消息状态变化观察者
 * @param observer 观察者， 参数为改变的消息体，更改的状态可能包含status和attachStatus
 * @param register true为注册，false为注销
 */
public void observeMsgStatus(Observer<IMMessage> observer, boolean register);
```

- 参数说明

MsgStatusEnum 属性说明

| MsgStatusEnum 属性说明 | 说明 |
| :-------- | :--------|
| draft | 草稿 |
| sending | 正在发送中 |
| success | 发送成功 |
| fail | 发送失败 |
| read | 消息已读，<br>发送消息时表示对方已看过该消息，<br>接收消息时表示自己已读过，<br>一般仅用于音频消息 |
| unread | 未读状态 |

AttachStatusEnum 属性说明

| AttachStatusEnum 属性说明 | 说明 |
| :-------- | :--------|
| def | 默认状态，未开始 |
| transferring | 正在传输 |
| transferred | 传输成功, 附件下载成功标志 |
| fail | 传输失败 |

- 示例

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

### <span id="消息重发">消息重发</span>

消息发送失败之后，可以重发消息。消息重发和消息发送是同一个接口，只是第二个重发参数的设置不同，当 resend 参数为 true 时，表示重发消息。具体示例请参考消息发送。

- API 原型

```
/**
 * 发送消息。<br>
 * @param msg    带发送的消息体，由{@link MessageBuilder}构造
 * @param resend 如果是发送失败后重发，标记为true，否则填false
 * @return InvocationFuture 可以设置回调函数。消息发送完成后才会调用，如果出错，会有具体的错误代码。
 */
public InvocationFuture<Void> sendMessage(IMMessage msg, boolean resend);
```

### <span id="已读回执">已读回执</span>

- API 介绍

网易云通信提供点对点消息的已读回执。注意：此功能仅对 P2P 消息中有效。

在会话界面中调用发送已读回执的接口并传入最后一条消息，即表示这之前的消息都已读，对端将收到此回执。

#### 发送已读回执

发送消息已读回执的一般场景：

1\. 进入 P2P 聊天界面（如果没有收到新的消息，反复进入调用发送已读回执接口， SDK 会自动过滤，只会发送一次给网易云通信服务器）。

2\. 处于聊天界面中，收到当前会话新消息时。

- API 原型

```java
/**
* 发送消息已读回执
* @param sessionId 会话ID（聊天对象账号）
* @param message 已读的消息(一般是当前接收的最后一条消息）
*/
NIMClient.getService(MsgService.class).sendMessageReceipt(sessionId, message);
```

- 示例

```
// 该帐号为示例，请先注册
String account = "testAccount";

// message为会话中已读的最后一条消息
NIMClient.getService(MsgService.class).sendMessageReceipt(account, message);

```

#### 监听已读回执

监听到已读回执，根据 IMMessage 中的 `isRemoteRead()` 方法来判断该条消息是否已读，并刷新界面。

一般场景：一个会话显示一个已读回执（比该已读回执对应的消息时间早的消息都是已读），那么刷新界面时，可以倒序查找第一条 isRemoteRead 接口返回 true 的消息打上已读回执标记。注意：当删除带有已读回执标记的消息时，也应该刷新界面。

- API 原型

```
/**
 * 注册/注销消息已读回执观察者
 * @param observer 观察者， 参数为已读回执集合。
 * @param register true为注册，false为注销
 */
public void observeMessageReceipt(Observer<List<MessageReceipt>> observer, boolean register);
```

- 参数说明

| 返回值 | MessageReceipt接口 | 说明 |
| :-------- | :--------| :--------|
| String | getSessionId() | 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID |
| long | getTime() | 该会话最后一条已读消息的时间，比该时间早的消息都视为已读 |

- 示例

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

### <span id="消息过滤">消息过滤</span>

- API 介绍

支持单聊和群聊的通知类型消息过滤，支持音视频类型消息过滤。

通知消息是指 IMMessage#getMsgType 为 MsgTypeEnum#notification。 SDK 在 2.4.0 版本后支持上层指定过滤器决定是否要将通知消息，音视频消息存入 SDK 数据库（并通知上层收到该消息）。请注意，注册过滤器的时机，建议放在 Application 的 onCreate 中， SDK 初始化之后。

- API 原型

```
/**
 * 注册通知消息过滤器
 *
 * @param filter 上层实现的通知消息过滤器，决定是否过滤通知消息（不存储到数据库中），传 null 表示注销（取消）通知消息过滤器
 */
void registerIMMessageFilter(IMMessageFilter filter);
```

- 参数说明

| 返回值 | IMMessageFilter接口 | 说明 |
| :-------- | :--------| :------ |
| boolean | shouldIgnore(IMMessage message) |  是否过滤通知消息。<br>返回 true 表示过滤（那么 SDK 将不存储此消息，上层也不会收到此消息），<br>默认 false 即不过滤（默认存储到 db 并通知上层） |

- 示例

SDK 过滤群头像变更通知和过滤音视频类型消息。

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

### <span id="消息转发">消息转发</span>

- API 介绍

网易云通信支持消息转发功能，不支持通知消息和音视频消息以及机器人消息的转发，其他消息类型均支持。

首先，通过 `MessageBuilder` 创建一个待转发的消息，参数为想转发的消息，转发目标的聊天对象id， 转发目标的会话类型。然后，通过 `MsgService#sendMessage` 接口，将消息发送出去。

- API 原型

```
/**
 * 创建一条待转发的消息
 *
 * @param message     要转发的消息
 * @param sessionId   聊天对象ID
 * @param sessionType 会话类型
 * @return 待转发的消息
 */
public static IMMessage createForwardMessage(IMMessage message, String sessionId, SessionTypeEnum sessionType);
```

- 参数说明

| 参数      |     说明 |
| :-------- | :--------|
| message    |   要转发的消息 |
| sessionId | 聊天对象的 ID，如果是单聊，为用户帐号，如果是群聊，为群组 ID |
| sessionType | 聊天类型，SessionTypeEnum.P2P 为单聊类型，SessionTypeEnum.Team 为群聊类型 |

- 示例

```
// 该帐号为示例，请先注册
String account = "testAccount";
// 以单聊类型为例
SessionTypeEnum sessionType = SessionTypeEnum.P2P;
// forwardMessage为待转发的消息, 一般由上下文获得
IMMessage message = MessageBuilder.createForwardMessage(forwardMessage, account, sessionType);

// 发送给对方
NIMClient.getService(MsgService.class).sendMessage(textMessage, false);
```

### <span id="消息撤回">消息撤回</span>

#### <span id="发起消息撤回">发起消息撤回</span>

网易云通信支持发送消息的撤回。超过限定时间（默认时间为 2 分钟）的撤回会失败，并返回错误码508。消息撤回后，未读数不发生变化，通知栏提醒的文案变为：撤回了一条消息。

几种不能撤回的情况：

1\. 消息为空，不能撤回
2\. 消息没有发送成功，不能撤回
3\. 消息发起者不是自己，不能撤回
4\. 会话 sessionId 是自己，不能撤回

- API 原型

```
/**
 *  消息撤回
 *
 * @param message 待撤回的消息
 * @return InvocationFuture 可设置回调函数，监听发送结果。
 */
public InvocationFuture<Void> revokeMessage(IMMessage message);
```

- 示例

```
// message表示待撤回的消息
NIMClient.getService(MsgService.class).revokeMessage(message).setCallback(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void param) {
        // 撤回消息成功，可执行界面相关操作
    }

    @Override
    public void onFailed(int code) {
        if (code == ResponseCode.RES_OVERDUE) {
           // 发送时间超过2分钟的消息，不能被撤回
        } else {
           // 其他错误code
        }
    }

    @Override
    public void onException(Throwable exception) {

    }
});
```

#### <span id="监听消息撤回">监听消息撤回</span>

消息被撤回，会收到消息撤回通知。当开发者收到消息撤回通知，可以在界面上做相应的消息删除等操作。

- API 原型

```
/**
 * 注册/注销消息撤回的观察者
 * @param observer 观察者，参数为被撤回的消息。
 * @param register true为注册，false为注销
 */
public void observeRevokeMessage(Observer<IMMessage> observer, boolean register);
```

- 示例

```
Observer<IMMessage> revokeMessageObserver = new Observer<IMMessage>() {
    @Override
    public void onEvent(IMMessage message) {
       // 监听到消息撤回的通知，可以在界面做相应的操作
    }
};

NIMClient.getService(MsgServiceObserve.class).observeRevokeMessage(revokeMessageObserver, true);
```


## <span id="消息全文检索">消息全文检索</span>

网易云通信 SDK 2.7.0 加入基于 Lucene 的全文检索插件，支持聊天消息的全文检索，目前开放的查询接口适用于两类需求（与微信类似）：

- 需求1：

检索所有会话，返回所有匹配关键字的会话、每个会话匹配关键字的消息条数。如果会话只有一条消息匹配，那么直接返回该消息内容，返回的会话数量可以指定，也可以一次性列出所有会话。会话间的排序规则，按照每个会话最近一条匹配消息的时间倒序排列。需要支持多个关键字查询，采用空格隔开，关键字之间是 AND 关系。

- 需求2：

检索单个会话，返回所有匹配关键字的消息，并高亮显示被击中的关键字，可以跳转到该消息的上下文。

网易云通信 SDK 目前主要针对上述两种需求提供查询服务，只要集成了全文检索插件，SDK 会自动同步所有聊天记录到全文检索索引中。

注意：全文检索插件最低需要 Android 14 (Android 4.0)，APP 构建时 targetSdkVersion 强烈建议指向 19 及以上。

在 2.8.0 版本我们已经对依赖的 Lucene 源码做了大幅度的精简，约 1M+ 的大小，对于大的工程，仍然可能造成方法数 65535 的限制，此时可能需要 Multi dex 的支持。

### <span id="插件集成">插件集成</span>

有两种方式，选其一即可：

方式一： libs 中引入 nim-lucene-2.8.0.jar。

方式二： 在 build.gradle 中集成：

```groovy
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

## <span id="群组功能">群组功能</span>

### <span id="群组功能概述">群组功能概述</span>

网易云通信 SDK 提供了普通群 (`TeamTypeEnum#Normal`)，以及高级群 (`TeamTypeEnum#Advanced`)两种形式的群聊功能。高级群拥有更多的权限操作，两种群聊形式在共有操作上保持了接口一致。在群组中，当前会话的 ID 就是群组的 ID。

- 普通群

开发手册中所提及的普通群都等同于 Demo 中的讨论组。普通群没有权限操作，适用于快速创建多人会话的场景。每个普通群只有一个管理员。管理员可以对普通群进行增减员操作，普通成员只能对普通群进行增员操作。在添加新成员的时候，并不需要经过对方同意。

- 高级群

高级群在权限上有更多的限制，权限分为群主、管理员、以及群成员。2.4.0之前版本在添加成员的时候需要对方接受邀请；2.4.0版本之后，可以设定被邀请模式（是否需要对方同意）。高级群可以覆盖所有普通群的能力，建议开发者创建时选用高级群。

### <span id="群组消息">群组消息</span>

#### <span id="群聊消息">群聊消息</span>

群聊消息收发和管理和单人聊天完全相同，仅在 `SessionTypeEnum` 上做了区分，详见[消息收发](#消息收发)节。

#### <span id="关闭群聊消息提醒">关闭群聊消息提醒</span>

群聊消息提醒可以单独打开或关闭，关闭提醒之后，用户仍然会收到这个群的消息。如果开发者使用的是网易云通信内建的消息提醒，用户收到新消息后不会再用通知栏提醒，如果用户使用的 iOS 客户端，则他将收不到该群聊消息的 APNS 推送。如果开发者自行实现状态栏提醒，可通过 `Team` 的 `mute` 接口获取提醒配置，并决定是不是要显示通知。群聊消息提醒设置可以漫游。

- API 原型

```
/**
 * 打开/关闭指定群的消息提醒。关闭消息提醒后，该群收到新消息后将不会有通知栏提醒
 *
 * @param teamId 群组ID
 * @param mute   若为true：关闭消息提醒，false：打开消息提醒
 * @return InvocationFuture 可以设置回调函数，监听操作结果
 */
InvocationFuture<Void> muteTeam(String teamId, boolean mute);
```

- 参数说明

|参数|说明|
|-|-|
|teamId |群组ID|
|mute   |若为 true：关闭消息提醒，false：打开消息提醒|

- 示例

```
// 以关闭消息提醒为例
NIMClient.getService(TeamService.class).muteTeam(teamId, true).setCallback(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void param) {
        // 关闭群聊消息提醒成功
    }

    @Override
    public void onFailed(int code) {
        // 关闭群聊消息提醒失败
    }

    @Override
    public void onException(Throwable exception) {
		// 错误
    }
});
```

#### <span id="指定群成员强制推送">指定群成员强制推送</span>

群消息支持设置强制推送。强制推送优先级最高，

1、打开强推开关，即使关闭消息提醒，依然能收到通知栏提醒，但是声音和震动跟随本机设置。

2、打开强推开关，即使设置了 桌面端在线时不推送，依然能收到通知栏提醒。

3、无论是否打开强推开关，设置了强推通知文本内容并且位于强推列表中的成员（若强推列表为 null，则表示本群所有成员），收到的消息提醒均显示强推通知文本。

4、强推列表可以配置需要强制推送的群成员，若为 null ，则表示该群所有群成员均收到强制推送消息。

5、打开强推开关，也打开免打扰设置。有通知栏提醒，但是没有声音和震动。

- 参数说明

MemberPushOption 接口说明

| 返回值 | MemberPushOption接口 | 说明 |
| :-------- | :--------| :------ |
| List<String> | getForcePushList() | 返回强制推送的账号列表 |
| String | getForcePushContent() | 返回强制推送的文案 |
| boolean | isForcePush() | 返回是否强制推送 |
| void | setForcePush(boolean forcePush) | 设置是否强推。<br>针对 forcePushList 里的帐号，<br>false 为不强推，true 为强推，默认为 true。<br>对于 forcePushList 中的用户，推送文案为 forcePushContent；<br>对于不在 forcePushList 中的用户，推送文案为 pushContent；<br>针对在 forcePushList 中的账号：<br>isForcePush 为true 时，推送文案中不会包含发送者 nick，直接为 forcePushContent；<br>isForcePush 为 false 时，推送文案中目前包含了发送者 nick（暂定），即为 fromNick：forcePushContent |
| void | setForcePushList(List<String> forcePushList) | 设置强推列表，<br>填 null 表示强推给该会话所有成员，<br>不为 null 时最大上限账号为100个 |
| void | setForcePushContent(String forcePushContent) | 强推文案（可扩展为区别与普通推送的推送文案） |

- 示例

```
// 该帐号为示例，请先注册
String account = "testAccount";
// 群聊才有强推消息
SessionTypeEnum sessionType = SessionTypeEnum.Team;
String text = "指定推送消息";
// 创建一个文本消息
IMMessage textMessage = MessageBuilder.createTextMessage(account, sessionType, text);

// 配置指定成员推送
MemberPushOption memberPushOption = new MemberPushOption();
// 开启强制推送
memberPushOption.setForcePush(true);
// 设置强推文案
memberPushOption.setForcePushContent(textMessage.getContent());
List<String> pushList = new ArrayList<>();
pushList.add("account1");
pushList.add("account2");
// 设置指定推送列表
memberPushOption.setForcePushList(pushList);
textMessage.setMemberPushOption(memberPushOption);

// 发送给对方
NIMClient.getService(MsgService.class).sendMessage(textMessage, false);
```

### <span id="群组管理">群组管理</span>

#### <span id="获取群组">获取群组</span>

SDK 提供了两个获取自己加入的所有群的列表的接口，一个是获取所有群（包括高级群和普通群）的接口，另一个是根据类型获取列表的接口。开发者可根据实际产品需求选择使用。

1\. 获取我加入的所有群组

- API 原型

异步版本:

```
/**
 * 获取自己加入的群的列表
 *
 * @return InvocationFuture 可以设置回调函数，如果成功，参数为自己加入的群的列表
 */
InvocationFuture<List<Team>> queryTeamList();
```
同步版本

```
/**
 * 获取自己加入的群的列表(同步版本)
 *
 * @return 自己加入的群的列表
 */
public List<Team> queryTeamListBlock();
```
- 参数说明

Team函数说明：

|返回值|Team函数|说明|
|-|-|-|
|String|getAnnouncement()| 获取群组公告|
|long |	getCreateTime() | 获取群组的创建时间
|String	|getCreator()|获取创建群组的用户帐号|
|String|	getExtension()|获取群组扩展配置。|
|String	|getExtServer()|获取服务器设置的扩展配置。|
|String	|getIcon()|获取群头像|
|String	|getId()|获取群组ID|
|String	|getIntroduce()|获取群组简介|
|int	|getMemberCount()|获取群组的总成员数|
|int	|getMemberLimit()|获取群组的成员人数上限|
|String	|getName()|获取群组名称|
|TeamBeInvite<br>ModeEnum	|getTeamBeInviteMode()|获取群被邀请模式：被邀请人的同意方式|
|TeamExtension<br>UpdateModeEnum	|getTeamExtension<br>UpdateMode()|获取群资料扩展字段修改模式：谁可以修改群自定义属性(扩展字段)|
|TeamInviteModeEnum	|getTeamInviteMode()|获取群邀请模式：谁可以邀请他人入群|
|TeamUpdateModeEnum	|getTeamUpdateMode()|获取群资料修改模式：谁可以修改群资料|
|TeamTypeEnum	|getType()|获取群组类型|
|VerifyTypeEnum	|getVerifyType()|获取申请加入群组时的验证类型|
|boolean	|isAllMute()|是否群全员禁言|
|boolean	|isMyTeam()|获取自己是否在这个群里|
|boolean	|mute()|获取这个群收到新消息时要不要提醒的设置。|
|void	|setExtension(extension)|设置群组扩展配置。|

- 示例

异步请求示例：

```
NIMClient.getService(TeamService.class).queryTeamList().setCallback(new RequestCallback<List<Team>>() {
    @Override
    public void onSuccess(List<Team> teams) {
        // 获取成功，teams为加入的所有群组
    }

    @Override
    public void onFailed(int i) {
	     // 获取失败，具体错误码见i参数
    }

    @Override
    public void onException(Throwable throwable) {
        // 获取异常
    }
});
```

同步请求示例：

```
// teams为加入的所有群组
List<Team> teams = NIMClient.getService(TeamService.class).queryTeamListBlock();
```

> 注意：这里获取的是所有我加入的群列表（退群、被移除群后，将不在返回列表中），如果需要自己退群或者被移出群的群资料，请使用下面的 queryTeam 接口。

2\. 根据类型获取我加入的群组

- API 原型

异步版本:

```
/**
 * 获取自己加入的指定类型群（讨论组/高级群）列表
 *
 * @param type 群类型
 * @return 可以设置回调函数，如果成功，参数为加入的指定类型群列表
 */
InvocationFuture<List<Team>> queryTeamListByType(TeamTypeEnum type);
```

同步版本:

```
/**
 * 获取自己加入的指定类型群（讨论组/高级群）列表（同步版本）
 *
 * @param type 群类型
 * @return 加入的指定类型群列表
 */
public List<Team> queryTeamListByTypeBlock(TeamTypeEnum type);
```

- 参数说明

|TeamTypeEnum属性|说明|
|:--|:--|
|Advanced|高级群，有完善的权限管理功能|
|Normal|讨论组，仅具有基本的权限管理功能，所有人都能加入，<br>仅拥有者可以踢人|

- 示例

异步版本示例：

```
// 以讨论组为例
NIMClient.getService(TeamService.class).queryTeamListByType(TeamTypeEnum.Normal).setCallback(new RequestCallback<List<Team>>() {
    @Override
    public void onSuccess(List<Team> teams) {
      // 成功
    }

    @Override
    public void onFailed(int i) {
      // 失败
    }

    @Override
    public void onException(Throwable throwable) {
        // 错误
    }
});
```

同步版本示例:

```
// 以高级群为例
List<Team> teams = NIMClient.getService(TeamService.class).queryTeamListByTypeBlock(TeamTypeEnum.Advanced)
```

#### <span id="创建群组">创建群组</span>

网易云通信群组分为两类：普通群和高级群，两种群组的消息功能都是相同的，区别在于管理功能。普通群所有人都可以拉人入群，除群主外，其他人都不能踢人；固定群则拥有完善的成员权限体系及管理功能。创建群的接口相同，传入不同的类型参数即可。

- API 原型
```
/**
 * 创建一个群组
 *
 * @return InvocationFuture 可以设置回调函数，如果成功，参数为创建的群组资料
 */
InvocationFuture<Team> createTeam(Map<TeamFieldEnum, Serializable> fields, TeamTypeEnum type, String postscript, List<String> members);
```

- 参数说明

|参数|说明|
|:---|:---|
|fields | 群组预设资料, key为数据字段，value对应的值，<br>该值类型必须和field中定义的fieldType一致。|
|type |要创建的群组类型|
|postscript | 邀请入群的附言。如果是创建临时群，该参数无效|
|members | 邀请加入的成员帐号列表|

- 示例

```
// 群组类型
TeamTypeEnum type = TeamTypeEnum.Advanced;
// 创建时可以预设群组的一些相关属性，如果是普通群，仅群名有效。
// fields 中，key 为数据字段，value 对对应的值，该值类型必须和 field 中定义的 fieldType 一致
HashMap<TeamFieldEnum, Serializable> fields = new HashMap<TeamFieldEnum, Serializable>();
fields.put(TeamFieldEnum.Name, teamName);
fields.put(TeamFieldEnum.Introduce, teamIntroduce);
fields.put(TeamFieldEnum.VerifyType, verifyType);
NIMClient.getService(TeamService.class).createTeam(fields, type, "", accounts)
     .setCallback(new RequestCallback<Team> { ... });
```

#### <span id="申请加入群组">申请加入群组</span>

- API 原型

```
/**
 * 用户申请加入群。
 *
 * @param tid        申请加入的群ID
 * @param postscript 申请附言
 * @return InvocationFuture 可以设置回调函数，如果成功，参数为群资料
 */
InvocationFuture<Team> applyJoinTeam(String tid, String postscript);
```
- 参数说明

|参数|说明|
|:---|:---|
|tid | 申请加入的群ID|
|postscript | 申请附言|

- 示例

```
NIMClient.getService(TeamService.class).applyJoinTeam(team.getId(), null).setCallback(new RequestCallback<Team>() {
    @Override
    public void onSuccess(Team team) {
        // 申请成功, 等待验证入群
    }

    @Override
    public void onFailed(int code) {
        // 申请失败
    }

    @Override
    public void onException(Throwable exception) {
		// error
    }
});
```

- 特殊错误码说明

|错误码|说明|
|:---|:---|
|808|申请已发出|
|809|已经在群里|

#### <span id="验证入群申请">验证入群申请</span>

用户发出申请后，所有管理员都会收到一条系统通知，类型为 `SystemMessageType#TeamApply`，具体参数解释请参考系统通知章节。管理员可选择同意或拒绝。

1\. 同意入群申请

仅管理员和拥有者有此权限。如果同意入群申请，群内所有成员(包括申请者)都会收到一条消息类型为 `notification` 的通知消息，附件类型为 `MemberChangeAttachment`。具体参数说明见消息收发章节。

- API 原型

```
/**
 * 通过用户的入群申请
 * 仅管理员和拥有者有此权限
 *
 * @return InvocationFuture 可以设置回调函数
 */
InvocationFuture<Void> passApply(String teamId, String account);
```
- 参数说明

|参数|说明|
|:---|:---|
|teamId | 群ID|
|account | 申请入群的用户ID|

- 示例

```
// teamId为申请加入的群组id， account为申请入群的用户id
NIMClient.getService(TeamService.class).passApply(teamId, account).setCallback(...);
```

2\. 拒绝入群申请

仅管理员和拥有者有此权限。如果拒绝入群申请，申请者会收到一条系统通知，类型为 `SystemMessageType#RejectTeamApply`。具体参数说明见系统通知章节。

- API 原型

```
/**
 * 拒绝用户的入群申请    
 * 仅管理员和拥有者有此权限
 *
 * @return InvocationFuture 可以设置回调函数
 */
InvocationFuture<Void> rejectApply(String teamId, String account, String reason);
```

- 参数说明

|参数|说明|
|:---|:---|
|teamId | 群ID|
|account | 申请入群的用户ID|
|reason | 拒绝理由，选填|

- 示例

```
// teamId为申请加入的群组id， account为申请入群的用户id
NIMClient.getService(TeamService.class).rejectApply(teamId, account, "您已被拒绝").setCallback(...);
```

> 说明：任意一管理员操作后，其他管理员再操作都会失败。

#### 邀请加入群组

普通群所有人都可以拉人入群，SDK 2.4.0之前版本高级群仅管理员和拥有者可以邀请人入群， SDK 2.4.0及以后版本高级群在创建时可以设置群邀请模式，支持仅管理员或者所有人均可拉人入群。

- API 原型

```
/**
 * 添加成员
 *
 * @return InvocationFuture 可以设置回调函数，监听操作结果
 */
InvocationFuture<Void> addMembers(String teamId, List<String> accounts);
```

- 参数说明

|参数|说明|
|:---|:---|
|teamId | 群组ID|
|accounts | 待加入的群成员帐号列表|

- 示例

```
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

- 特殊错误码说明

|错误码|说明|
|:---|:---|
|810|高级群邀请人的情况下，发出邀请成功了，但是还需要对方同意|

说明：被邀请的人会收到一条系统通知 (SystemMessage)，类型为 `SystemMessageType#TeamInvite`。用户接受该邀请后，才真正入群。可以通过 SystemMessage#getAttachObject 获取服务器设置的扩展字段，类型为TeamInviteNotify。

用户入群（普通群被拉人，高级群接受邀请）后，在收到之后的第一条消息时，群内所有成员（包括入群者）会收到一条入群消息，附件类型为 MemberChangeAttachment。若通知类型为`NotificationType#InviteMember`，可以通过 MemberChangeAttachment#getExtension    获取服务器设置的扩展字段。

#### 验证入群邀请

收到入群邀请后，用户可在系统通知中看到该邀请，并选择接受或拒绝。接受邀请后，用户真正入群。如果拒绝邀请，邀请该用户的管理员会收到一条系统通知，类型为 `SystemMessageType#DeclineTeamInvite`。

1\. 接受入群邀请

- API 原型

```
/**
 * 接受别人的入群邀请
 *
 * @return InvocationFuture 可以设置回调函数
 */
InvocationFuture<Void> acceptInvite(String teamId, String inviter);
```

- 参数说明

|参数|说明|
|:---|:---|
|teamId | 群ID|
|inviter | 邀请我的用户帐号|

- 示例

```
NIMClient.getService(TeamService.class).acceptInvite(teamId, inviter).setCallback(...);
```

2\. 拒绝入群邀请

- API 原型

```
/**
 * 拒绝别人的入群邀请通知
 *
 * @return InvocationFuture 可以设置回调函数
 */
InvocationFuture<Void> declineInvite(String teamId, String inviter, String reason);
```

- 参数说明

|参数|说明|
|:---|:---|
|teamId | 群ID|
|inviter | 邀请我的用户帐号|
|reason | 拒绝理由，选填|

- 示例

```
NIMClient.getService(TeamService.class).declineInvite(teamId, inviter, "").setCallback(callback);
```

#### <span id="踢人出群">踢人出群</span>

普通群仅拥有者可以踢人，高级群拥有者和管理员可以踢人，且管理员不能踢拥有者和其他管理员。踢人后，群内所有成员(包括被踢者)会收到一条消息类型为 notification 的 IMMessage，类型为 `NotificationType#KickMember`, 附件类型为 `MemberChangeAttachment`。可以通过 MemberChangeAttachment#getExtension 获取服务器设置的扩展字段。

- API 原型

```
/**
 * 移除成员，只有创建者有此权限
 *
 * @return InvocationFuture 可以设置回调函数，监听操作结果
 */
InvocationFuture<Void> removeMember(String teamId, String member);
```

- 参数说明

|参数|说明|
|:---|:---|
|teamId | 群ID|
|member | 被踢出的成员帐号|

- 示例

```
// teamId表示群ID，account表示被踢出的成员帐号
NIMClient.getService(TeamService.class).removeMember(teamId, account).setCallback(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void param) {
       // 成功
    }

    @Override
    public void onFailed(int code) {
        // 失败
    }

    @Override
    public void onException(Throwable exception) {
        // 错误
    }
});
```

#### <span id="主动退群">主动退群</span>

普通群群主可以退群，若退群，该群没有群主。高级群除群主外，其他用户均可以主动退群。退群后，群内所有成员(包括退出者)会收到一条消息类型为 notification 的 IMMessage，附件类型为 `MemberChangeAttachment`。

- API 原型

```
/**
 * 退出群
 *
 * @param teamId 群ID
 * @return InvocationFuture 可以设置回调函数，监听操作结果
 */
InvocationFuture<Void> quitTeam(String teamId);
```

- 示例

```
NIMClient.getService(TeamService.class).quitTeam(teamId).setCallback(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void param) {
       // 退群成功
    }

    @Override
    public void onFailed(int code) {
        // 退群失败
    }

    @Override
    public void onException(Throwable exception) {
        // 错误
    }
});
```

#### <span id="转让群组">转让群组</span>

高级群拥有者可以将群的拥有者权限转给群内的其他成员，转移后，被转让者变为新的拥有者，原拥有者变为普通成员。原拥有者还可以选择在转让的同时，直接退出该群。

- API 原型

```
/**
 * 拥有者将群的拥有者权限转给另外一个人，转移后，另外一个人成为拥有者。
 * 原拥有者变成普通成员。若参数quit为true，原拥有者直接退出该群。
 *
 * @return InvocationFuture 可以设置回调函数，如果成功，视参数quit值：
 * quit为false：参数仅包含原拥有着和当前拥有者的(即操作者和account)，权限已被更新。
 * quit为true: 参数为空。
 */
InvocationFuture<List<TeamMember>> transferTeam(String tid, String account, boolean quit);
```

- 参数说明

|参数|说明|
|:---|:---|
|tid | 群ID|
|account | 新任拥有者的用户帐号|
|quit | 转移时是否要同时退出该群|

|返回值 | 说明|
|:---|:---|
|InvocationFuture |可以设置回调函数，如果成功，视参数quit值：<br>quit为false：参数仅包含原拥有着和当前拥有者的(即操作者和account)，权限已被更新。<br>quit为true: 参数为空|

|返回值|TeamMember属性 | 说明|
|:---|:---|:---|
|String	|getAccount()| 群成员帐号|
|Map|getExtension()|获取扩展字段|
|long	|getJoinTime()|获取群成员入群时间|
|String	|getTeamNick()|获取该用户在这个群内的群昵称|
|String	|getTid()|获取所在群ID|
|TeamMemberType	|getType()|群成员类型|
|boolean	|isInTeam()|是否该用户是否在这个群中|
|boolean	|isMute()|是否被禁言|

- 示例

```
// false表示群主转让后不退群
NIMClient.getService(TeamService.class).transferTeam(teamId, account, false)
.setCallback(new RequestCallback<List<TeamMember>>() {
    @Override
    public void onSuccess(List<TeamMember> members) {
        // 群转移成功
    }

    @Override
    public void onFailed(int code) {
        // 群转移失败
    }

    @Override
    public void onException(Throwable exception) {
		// 错误
    }
});
```

#### <span id="解散群组">解散群组</span>

仅群主有这个权限。

- API 原型

```java
/**
 * 解散群，只有群主有此权限
 *
 * @param teamId 群ID
 * @return InvocationFuture 可以设置回调函数，监听操作结果
 */
InvocationFuture<Void> dismissTeam(String teamId);
```

- 示例

```
NIMClient.getService(TeamService.class).dismissTeam(teamId).setCallback(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void param) {
        // 解散群成功
    }

    @Override
    public void onFailed(int code) {
        // 解散群失败
    }

    @Override
    public void onException(Throwable exception) {
        // 错误
    }
});
```

#### <span id="获取群组成员">获取群组成员</span>

由于群组成员数据比较大，且除了进入群组成员列表界面外，其他地方均不需要群组成员列表的数据，因此 SDK 不会在登录时同步群组成员数据，而是按照按需获取的原则，当上层主动调用获取指定群的群组成员列表时，才判断是否需要同步。

群成员资料 SDK 本地存储说明：
当自己退群、或者被移出群时，本地数据库会继续保留这个群成员资料，只是设置了无效标记，此时依然可以通过 queryTeamMember 查出来该群成员资料，只是 isInTeam 将返回 false 。


1\. 获取群组所有成员

- API 原型

```
/**
 * 获取指定群的成员信息列表. <br>
 * 该操作有可能只是从本地数据库读取缓存数据，也有可能会从服务器同步新的数据, 因此耗时可能会比较长。
 *
 * @param teamId 群ID
 * @return InvocationFuture 可以设置回调函数，如果成功，参数为群的成员信息列表
 */
InvocationFuture<List<TeamMember>> queryMemberList(String teamId);
```

- 示例

```
NIMClient.getService(TeamService.class).queryMemberList(teamId).setCallback(new RequestCallbackWrapper<List<TeamMember>>() {
            @Override
            public void onResult(int code, final List<TeamMember> members, Throwable exception) {
                ...
            }
        });
```

2\. 根据群 ID 和成员帐号获取群成员

如果本地群成员资料已过期， SDK 会去服务器获取最新的。

- API 原型

异步版本：

```
/**
 * 查询群成员资料。如果本地群成员资料已过期会去服务器获取最新的。
 *
 * @param teamId  群ID
 * @param account 群成员帐号
 * @return InvocationFuture 可以设置回调函数，如果成功，参数为该群成员信息
 */
InvocationFuture<TeamMember> queryTeamMember(String teamId, String account);
```

同步版本（仅查询本地群资料）：

```
/**
 * 查询群成员资料。（同步版本）仅查询本地群资料
 *
 * @param teamId  群ID
 * @param account 群成员帐号
 * @return 群成员信息
 */
TeamMember queryTeamMemberBlock(String teamId, String account);
```

- 示例

异步版本示例:

```
NIMClient.getService(TeamService.class).queryTeamMember(teamId, account).setCallback(new RequestCallbackWrapper<TeamMember>() {
    @Override
    public void onResult(int code, TeamMember member, Throwable exception) {
        ...
    }
});
```

同步版本示例：

```
TeamMember member = NIMClient.getService(TeamService.class).queryTeamMemberBlock(teamId, account);
```

#### <span id="提升管理员">提升管理员</span>

高级群中，群主可以增加管理员。操作完成后，群内所有成员都会收到一条消息类型为 notification 的 IMMessage，附件类型为 `MemberChangeAttachment`。

- API 原型

```
/**
 * 拥有者添加管理员 
 * 仅拥有者有此权限
 *
 * @param teamId   群ID
 * @param accounts 待提升为管理员的用户帐号列表
 * @return InvocationFuture 可以设置回调函数,如果成功，参数为新增的群管理员列表
 */
InvocationFuture<List<TeamMember>> addManagers(String teamId, List<String> accounts);
```

- 参数说明

|参数|说明|
|:---|:---|
|teamId | 群ID|
|accounts | 待提升为管理员的用户帐号列表|

|返回值|说明|
|:---|:---|
|InvocationFuture |可以设置回调函数，如果成功，参数为被撤销的群成员列表<br>(权限已被降为Normal)。|

- 示例

```
// teamId 操作的群id， accountList为待提升为管理员的用户帐号列表
NIMClient.getService(TeamService.class).addManagers(teamId, accountList).setCallback(new RequestCallback<List<TeamMember>>() {
    @Override
    public void onSuccess(List<TeamMember> managers) {
        // 添加群管理员成功
    }

    @Override
    public void onFailed(int code) {
        // 添加群管理员失败
    }

    @Override
    public void onException(Throwable exception) {
        // 错误
});
```

#### <span id="移除管理员">移除管理员</span>

高级群中，群主可以移除管理员。操作完成后，群内所有成员都会收到一条消息类型为 notification 的 IMMessage，附件类型为 `MemberChangeAttachment`。

- API 原型

```
/**
 * 拥有者撤销管理员权限 
 * 仅拥有者有此权限
 *
 * @param teamId   群ID
 * @param managers 待撤销的管理员的帐号列表
 * @return InvocationFuture 可以设置回调函数，如果成功，参数为被撤销的群成员列表(权限已被降为Normal)。
 */
InvocationFuture<List<TeamMember>> removeManagers(String teamId, List<String> managers);
```

- 参数说明

|参数|说明|
|:---|:---|
|teamId | 群ID|
|managers | 待撤销的管理员的帐号列表|

|返回值|说明|
|:---|:---|
|InvocationFuture |可以设置回调函数，如果成功，参数为被撤销的群成员列表<br>(权限已被降为Normal)。|

- 示例

```
// teamid为群id， accountList为待撤销的管理员的账号列表
NIMClient.getService(TeamService.class).removeManagers(teamId, accountList).setCallback(new RequestCallback<List<TeamMember>>() {
    @Override
    public void onSuccess(List<TeamMember> members) {
        // 移除群管理员成功
    }

    @Override
    public void onFailed(int code) {
        // 移除群管理员失败
    }

    @Override
    public void onException(Throwable exception) {
        // 错误
    }
});
```

#### <span id="群成员禁言">群成员禁言</span>

支持管理员和群主对普通成员的禁言、解除禁言操作。

- API 原型

```
/**
 * 禁言、解除禁言
 *
 * @return InvocationFuture 可以设置回调函数，监听操作结果
 */
InvocationFuture<Void> muteTeamMember(String teamId, String account, boolean mute);
```

- 参数说明

|参数|说明|
|:---|:---|
|teamId | 群组ID|
|account | 被禁言、被解除禁言的账号|
|mute | true表示禁言，false表示解除禁言|

- 示例

```
// 以禁言为例
NIMClient.getService(TeamService.class).muteTeamMember(teamId, account, true).setCallback(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void param) {
        // 成功
    }

    @Override
    public void onFailed(int code) {
        // 失败
    }

    @Override
    public void onException(Throwable exception) {
		// 错误
    }
});
```

#### <span id="查询被禁言群成员">查询被禁言群成员</span>

该操作只返回被禁言的用户

- API 原型

```
/**
 * 查询被禁言群成员列表
 * 该操作，只返回调用TeamService#muteTeamMember(String, String, boolean) 禁言的用户。
 *
 * @param teamId    群ID
 * @return 群成员信息列表
 */
List<TeamMember> queryMutedTeamMembers(String teamId);
```

- 示例

```
// members表示被禁言的成员列表
List<TeamMember> members = NIMClient.getService(TeamService.class).queryMutedTeamMembers(teamId);

```

### <span id="群组资料管理"> 群组资料管理</span>

#### <span id="查询群组资料">查询群组资料</span>

1\. 从本地数据库中查询群资料

如果本地没有群组资料，则去服务器查询。如果自己不在这个群中，该接口返回的可能是过期资料，如需最新的，请调用 `searchTeam` 接口去服务器查询。 此外 queryTeam 接口也有同步版本： queryTeamBlock 。

- API 原型

异步版本：

```
/**
 * 查询群资料，如果本地没有群组资料，则去服务器查询。
 * 如果自己不在这个群中，该接口返回的可能是过期资料，如需最新的，请调用{@link #searchTeam(String)}接口
 *
 * @param teamId 群ID
 * @return InvocationFuture 可以设置回调函数，如果成功，参数为群资料
 */
InvocationFuture<Team> queryTeam(String teamId);
```

同步版本：

```
/**
 * 查询群资料（仅查询本地，不会去服务器请求)
 * 如果自己不在这个群中，该接口返回的可能是过期资料，如需最新的，请调用{@link #searchTeam(String)}接口
 *
 * @param teamId 群ID
 * @return 群资料
 */
Team queryTeamBlock(String teamId);
```

> 注意: 同步版本的查询群资料，仅查询本地，不会去服务器请求

- 示例

异步接口示例:

```
NIMClient.getService(TeamService.class).queryTeam(teamId).setCallback(new RequestCallbackWrapper<Team>() {
    @Override
    public void onResult(int code, Team t, Throwable exception) {
        if (code == ResponseCode.RES_SUCCESS) {
            // 成功
        } else {
            // 失败，错误码见code
        }

        if (exception != null) {
            // error
        }
    }
});
```

同步接口示例：

```
// teamId 为想要查询的群组 ID
Team team = NIMClient.getService(TeamService.class).queryTeamBlock(teamId);
```

2\. 从服务器上查询群资料

用户可以直接从服务器上查询群资料

- API 原型

```
/**
 * 从服务器上查询群资料信息
 *
 * @param teamId 群ID
 * @return
 */
InvocationFuture<Team> searchTeam(String teamId);
```

- 示例

```
// teamId为想要查询的群组ID
NIMClient.getService(TeamService.class).searchTeam(teamId).setCallback(new RequestCallback<Team>() {
    @Override
    public void onSuccess(Team team) {
        // 查询成功，获得群组资料
    }

    @Override
    public void onFailed(int code) {
       // 失败
    }

    @Override
    public void onException(Throwable exception) {
       // 错误
    }
});
```

- 特殊错误码说明

| 错误码      |     说明 |
| :-------- | :--------|
| 803    |   查询的群id不存在 |

群资料 SDK 本地存储说明：

1\. 解散群、退群或者被移出群时，本地数据库会继续保留这个群资料，只是设置了无效标记，此时依然可以通过 queryTeam 查出来该群资料，只是 isMyTeam 返回 false 。 如果用户手动清空全部本地数据，下次登录同步时，服务器不会将无效的群资料同步过来，将无法取得已退出群的群资料。

2\. 群解散后，通过 `searchTeam` 接口从服务器查询将返回 null 。

#### <span id="编辑群组资料">编辑群组资料</span>

1\. 单个群属性更新

每次仅修改群的一个属性，可修改的属性查看  `TeamFieldEnum` ，注意： 其中的 `Ext_Server` 群服务器扩展字段，仅限服务端修改。

更新的value值，对应的数据类型，同样查看 `TeamFieldEnum` 。例如：群名 Name(ITeamService.TinfoTag.NAME, String.class)，表示 value 值的数据类型为 String。验证类型 VerifyType(ITeamService.TinfoTag.JOIN_MODE, VerifyTypeEnum.class)，表示 value 值的数据类型为VerifyTypeEnum。

|TeamFieldEnum属性|说明| 数据类型|
|:---|:---|:---|
|AllMute|群禁言（群全员禁言），该字段只读，使用“群资料更新”接口更新该字段无效。|TeamAllMuteModeEnum|
|Announcement|群公告|String|
|BeInviteMode|群被邀请模式：被邀请人的同意方式|TeamBeInviteModeEnum|
|Ext_Server|群扩展字段（仅服务端能够修改）|String|
|Extension|群扩展字段（客户端自定义信息）|String|
|ICON|群头像|String|
|Introduce|群简介|String|
|InviteMode|群邀请模式：谁可以邀请他人入群|TeamInviteModeEnum|
|Name|群名|String|
|TeamExtensionUpdateMode|群资料扩展字段修改模式：<br>谁可以修改群自定义属性(扩展字段)|TeamExtensionUpdateModeEnum|
|TeamUpdateMode|群资料修改模式：谁可以修改群资料|TeamUpdateModeEnum|
|undefined|未定义的域||
|VerifyType|申请加入群组的验证模式|VerifyTypeEnum|

修改后，群内所有成员会收到一条消息类型为 notification 的 IMMessage，带有一个消息附件，类型为 `UpdateTeamAttachment`。如果注册了群组资料变化观察者，观察者也会收到通知，见[监听群组资料变化](#监听群组资料变化)。


- API 原型

```
/**
 * 更新群组资料
 *
 * @return InvocationFuture 可以设置回调函数，监听操作结果
 */
InvocationFuture<Void> updateTeam(String teamId, TeamFieldEnum field, Serializable value);
```

- 参数说明

|参数|说明|
|:---|:---|
|teamId | 群ID|
|field | 待更新的域|
|value | 待更新的域的新值 该值类型必须和field中定义的fieldType一致|

- 示例

以群公告为例

```
String announcement = "这是更新的群公告";
NIMClient.getService(TeamService.class).updateTeam(teamId, TeamFieldEnum.Announcement, announcement).setCallback(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void param) {
        // 成功
    }

    @Override
    public void onFailed(int code) {
        // 失败
    }

    @Override
    public void onException(Throwable exception) {
        // 错误
    }
});
```

> 注意: 更新群头像请注意: 需要先调用 `NosService#upload` 方法，将头像图片成功上传到 nos。再将 nos 的 url 地址更新到群资料。

2\. 批量更新群组资料

- API 原型

```
/**
 * 批量更新群组资料，可一次性更新多个字段的值。
 *
 * @return InvocationFuture 可以设置回调函数，监听操作结果
 */
InvocationFuture<Void> updateTeamFields(String teamId, Map<TeamFieldEnum, Serializable> fields);
```

- 参数说明

|参数|说明|
|:---|:---|
|teamId | 群ID|
|fields | 待更新的所有字段的新的资料，其中值类型必须和field中定义的fieldType一致|

- 示例

```
// 以名字，介绍，公告为例
Map<TeamFieldEnum, Serializable> fieldsMap = new HashMap<>();
fieldsMap.put(TeamFieldEnum.Name, "新的名称");
fieldsMap.put(TeamFieldEnum.Introduce, "新的介绍");
fieldsMap.put(TeamFieldEnum.Announcement, "新的公告");

NIMClient.getService(TeamService.class).updateTeamFields(teamId,
        fieldsMap).setCallback(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
        // 成功
    }

    @Override
    public void onFailed(int i) {
        // 失败
    }

    @Override
    public void onException(Throwable throwable) {
		// 错误
    }
});
```

#### <span id="监听群组资料变化">监听群组资料变化</span>

由于获取群组资料需要跨进程异步调用，开发者最好能在第三方 APP 中做好群组资料缓存，查询群组资料时都从本地缓存中访问。在群组资料有变化时，SDK 会告诉注册的观察者，此时，第三方 APP 可更新缓存，并刷新界面。

1\. 监听群资料变化

- API 原型

```
/**
 * 群资料变动观察者通知。新建群和群更新的通知都通过该接口传递
 * @param observer 观察者, 参数为有更新的群资料列表
 * @param register true为注册，false为注销
 */
public void observeTeamUpdate(Observer<List<Team>> observer, boolean register);
```

- 参数说明

|返回值|Team接口|说明|
|:---|:---|:---|
|String	|getAnnouncement()|获取群组公告|
|long	|getCreateTime()|获取群组的创建时间|
|String	|getCreator()|获取创建群组的用户帐号|
|String	|getExtension()|获取群组扩展配置|
|String	|getExtServer()|获取服务器设置的扩展配置|
|String	|getIcon()|获取群头像|
|String	|getId()|获取群组ID|
|String	|getIntroduce()|获取群组简介|
|int	|getMemberCount()|获取群组的总成员数|
|int	|getMemberLimit()|获取群组的成员人数上限|
|String	|getName()|获取群组名称|
|TeamBeInviteModeEnum	|getTeamBeInviteMode()|获取群被邀请模式：被邀请人的同意方式|
|TeamExtensionUpdateModeEnum	|getTeamExtensionUpdateMode()|获取群资料扩展字段修改模式：谁可以修改群自定义属性(扩展字段)|
|TeamInviteModeEnum	|getTeamInviteMode()|获取群邀请模式：谁可以邀请他人入群|
|TeamUpdateModeEnum	|getTeamUpdateMode()|获取群资料修改模式：谁可以修改群资料|
|TeamTypeEnum	|getType()|获取群组类型|
|VerifyTypeEnum	|getVerifyType()|获取申请加入群组时的验证类型|
|boolean	|isAllMute()|是否群全员禁言|
|boolean	|isMyTeam()|获取自己是否在这个群里|
|boolean	|mute()|获取这个群收到新消息时要不要提醒的设置|
|void	|setExtension(String extension)|设置群组扩展配置|

- 示例

```
// 创建群组资料变动观察者
Observer<List<Team>> teamUpdateObserver = new Observer<List<Team>>() {
    @Override
    public void onEvent(List<Team> teams) {
    }
};
// 注册/注销观察者
NIMClient.getService(TeamServiceObserver.class).observeTeamUpdate(teamUpdateObserver, register);
```

2\. 监听移除群的变化

移除成功后，Team#isMyTeam 返回 false。

- API 原型

```
/**
 * 移除群的观察者通知。自己退群，群被解散，自己被踢出群时，会收到该通知
 * @param observer 观察者, 参数为被移除的群资料，此时群的isMyTeam接口返回false
 * @param register true为注册，false为注销
 */
public void observeTeamRemove(Observer<Team> observer, boolean register);
```

- 示例

```
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

#### <span id="修改群成员群昵称">修改群成员群昵称</span>

普通群不支持修改成员的群昵称。

对于高级群，群主和管理员修改群内其他成员的群昵称，仅群主和管理员拥有权限。

群主可以修改所有人的群昵称。管理员只能修改普通群成员的群昵称。

- API 原型

```
/**
 * 群组管理员修改群内其他成员的群昵称。
 * 仅群管理员和拥有者有此权限
 *
 * @param teamId  所在群组ID
 * @param account 要修改的群成员帐号
 * @param nick    新的群昵称
 * @return InvocationFuture 可以设置回调函数，监听操作结果
 */
InvocationFuture<Void> updateMemberNick(String teamId, String account, String nick);
```

- 示例

```
// teamId表示所在群组ID，account表示要修改的群成员帐号，nickname表示新的群昵称
NIMClient.getService(TeamService.class).updateMemberNick(teamId, account, nickname).setCallback(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void param) {
        // 成功
    }

    @Override
    public void onFailed(int code) {
        // 失败
    }

    @Override
    public void onException(Throwable exception) {
        // 错误
    }
});
```
#### <span id="修改自己的群昵称">修改自己的群昵称</span>

普通群不支持修改自己的群昵称。

- API 原型

```
/**
 * 修改自己的群昵称
 *
 * @param teamId 所在群组ID
 * @param nick   新的群昵称
 * @return InvocationFuture 可以设置回调函数，监听操作结果
 */
InvocationFuture<Void> updateMyTeamNick(String teamId, String nick);
```

- 示例

```
// test为修改后自己的群昵称
NIMClient.getService(TeamService.class).updateMyTeamNick(teamId, "test")
```

#### <span id="修改自己的群成员扩展字段">修改自己的群成员扩展字段</span>

修改后，群成员会收到群成员资料变更通知

- API 原型

```
/**
 * 修改自己的群成员扩展字段（自定义属性）
 *
 * @param teamId    所在群组ID
 * @param extension 新的扩展字段（自定义属性）
 * @return InvocationFuture 可以设置回调函数，监听操作结果
 */
InvocationFuture<Void> updateMyMemberExtension(String teamId, Map<String, Object> extension);
```

- 示例

```
Map<String, Object> extension = new HashMap<>();
extension.put("ext","newExt");
NIMClient.getService(TeamService.class).updateMyMemberExtension(teamId, ext)
```

#### <span id="监听群成员资料变化">监听群成员资料变化</span>

由于获取群成员资料需要跨进程异步调用，开发者最好能在第三方 APP 中做好群成员资料缓存，查询群成员资料时都从本地缓存中访问。在群成员资料有变化时，SDK 会告诉注册的观察者，此时，第三方 APP 可更新缓存，并刷新界面。

1\. 监听群成员资料变化

- API 原型

```

/**
 * 群成员资料变化观察者通知。
 *     上层APP如果管理了群成员资料的缓存，可通过此接口更新缓存。
 * @param observer 观察者, 参数为有更新的群成员资料列表
 * @param register true为注册，false为注销
 */
public void observeMemberUpdate(Observer<List<TeamMember>> observer, boolean register);
```

- 示例

```
// 群成员资料变化观察者通知。群组添加新成员，成员资料变化会收到该通知。
// 返回的参数为有更新的群成员资料列表。
Observer<List<TeamMember>> memberUpdateObserver = new Observer<List<TeamMember>>() {
	@Override
	public void onEvent(List<TeamMember> members) {
	}
};
// 注册/注销观察者
NIMClient.getService(TeamServiceObserver.class).observeMemberUpdate(memberUpdateObserver, register);
```

2\. 监听移除群成员的变化

- API 原型
    
```
/**
 * 移除群成员的观察者通知。
 * @param observer 观察者, 参数为被移除的群成员
 * @param register true为注册，false为注销
 */
public void observeMemberRemove(Observer<TeamMember> observer, boolean register);
```

- 示例

```
// 移除群成员的观察者通知。
private Observer<TeamMember> memberRemoveObserver = new Observer<TeamMember>() {
	@Override
	public void onEvent(TeamMember member) {
	}
};
// 注册/注销观察者
NIMClient.getService(TeamServiceObserver.class).observeMemberRemove(memberRemoveObserver, register);
```

## <span id="聊天室">聊天室</span>

### <span id="聊天室功能概述">聊天室功能概述</span>

聊天室模型特点：

- 进入聊天室时必须建立新的连接，退出聊天室或者被踢会断开连接，在聊天室中掉线会有自动重连，开发者需要监听聊天室连接状态来做出正确的界面表现。
- 支持聊天人数无上限。
- 聊天室只允许用户手动进入，无法进行邀请。
- 支持同时进入多个聊天室，会建立多个连接。
- 不支持多端进入同一个聊天室，后进入的客户端会将前一个踢掉。
- 断开聊天室连接后，服务器不会再推送该聊天室的消息给此用户。
- 聊天室成员分固定成员（固定成员有四种类型，分别是创建者,管理员,普通用户,受限用户。禁言用户和黑名单用户都属于受限用户。）和游客两种类型。

### <span id="进入聊天室">进入聊天室</span>

- API 原型

```
/**
 * 进入聊天室
 *
 * @param roomData   聊天室基本数据（聊天室ID必填）
 * @param retryCount 如果进入失败，重试次数
 * @return InvocationFuture 可以设置回调函数。回调中返回聊天室基本信息
 */
AbortableFuture<EnterChatRoomResultData> enterChatRoomEx(EnterChatRoomData roomData, int retryCount);

```

- 参数说明

|参数|说明|
|:---|:---|
|roomData | 聊天室基本数据（聊天室 ID 必填）|
|retryCount | 如果进入失败，重试次数|

EnterChatRoomData部分参数说明：

|返回值|EnterChatRoomData参数|说明|
|:---|:---|:---|
|String|getRoomId()|获取聊天室 ID|
|void|setRoomId(String roomId)|设置聊天室 ID|
|void|setNick(String nick)|设置聊天室展示的昵称(可选字段),<br>如果不填则直接使用 NimUserInfo 的数据|
|void|setAvatar(String avatar)|设置聊天室展示的头像|
|void|setExtension(Map <br>extension)|设置进入聊天室后展示用户信息的扩展字段，长度限制 4k。<br>设置后在获取聊天室成员信息 ChatRoomMember 时可以查询到此扩展字段；<br>此外，在收到聊天室消息时，可以从 ChatRoomMessage#ChatRoomMessageExtension#getSenderExtension <br>中获取消息发送者的用户信息扩展字段。<br>若没有设置，则这个字段为 null。|
|void|setNotifyExtension<br>(Map <br>notifyExtension)|设置聊天室通知消息扩展字段，长度限制 1k。<br>设置后，在收到聊天室通知消息时的<br> ChatRoomNotificationAttachment 中可以查询到此扩展字段。|

- 示例

```
// roomId 表示聊天室ID
EnterChatRoomData data = new EnterChatRoomData(roomId);
// 以登录一次不重试为例
NIMClient.getService(ChatRoomService.class).enterChatRoomEx(data, 1).setCallback(new RequestCallback<EnterChatRoomResultData>() {
    @Override
    public void onSuccess(EnterChatRoomResultData result) {
        // 登录成功
    }

    @Override
    public void onFailed(int code) {
        // 登录失败
    }

    @Override
    public void onException(Throwable exception) {
        // 错误
    }
});
```

> 注意：当进入聊天室后，再发生掉线问题时，SDK会自动进行重连，无需开发者再次调用进入聊天室接口。
> 注意：进入聊天室前，必须先成功登录 IM，否则会登录失败，并上报错误码1000。

### <span id="获取进入聊天室失败错误码">获取进入聊天室失败错误码</span>

SDK 有聊天室断线重连机制，会在网络恢复后做重连并自动登录，如果自动登录失败，则会有在线状态变更通知，见[监听聊天室在线状态](#监听聊天室在线状态)。
如果为在线状态变更为 UNLOGIN 则表示自动登录失败（例如账号被群主拉黑，聊天室状态异常等），那么在状态变更观察者回调中可以调用 SDK 接口获取服务器返回的失败原因，并在界面上做相应的处理（比如退出聊天室）。

- API 原型

```
/**
 * 获取进入聊天室失败的错误码
 * 如果是手动登录，在enterChatRoom的回调函数中已有错误码。
 * 如果是断网重连，在自动登录失败时，即监听到在线状态变更为UNLOGIN时，可以采用此接口查看具体自动登录失败的原因。
 *
 * @param roomId 聊天室ID
 * @return 聊天室断网重连、自动登录失败的错误码
 */
int getEnterErrorCode(String roomId);
```

- 参数说明

|错误码|说明|
|:---|:---|
|414|参数错误|
|404|聊天室不存在|
|403|无权限|
|500|服务器内部错误|
|13001|IM主连接状态异常|
|13002|聊天室状态异常|
|13003|黑名单用户禁止进入聊天室|

- 示例

```
// code表示进入聊天室失败的错误码
int code = NIMClient.getService(ChatRoomService.class).getEnterErrorCode(roomId);
```

### <span id="离开聊天室">离开聊天室</span>

离开聊天室，会断开聊天室对应的链接，并不再收到该聊天室的任何消息。如果用户要离开聊天室，可以手动调用离开聊天室接口，该接口没有回调。

- API 原型

```
/**
 * 离开聊天室
 *
 * @param roomId 聊天室ID
 */
void exitChatRoom(String roomId);
```

- 示例

```
NIMClient.getService(ChatRoomService.class).exitChatRoom(roomId);
```
### <span id="聊天室消息收发">聊天室消息收发</span>

#### <span id="发送消息">发送消息</span>

先通过 ChatRoomMessageBuilder 提供的接口创建消息对象，然后调用 ChatRoomService 的 sendMessage 接口发送出去即可。

- ChatRoomMessageBuilder 接口说明

|返回值|ChatRoomMessageBuilder 接口|说明|
|:---|:---|:---|
|static ChatRoomMessage	|createChatRoomAudioMessage(String roomId, File file, long duration)|创建一条音频消息|
|static ChatRoomMessage	|createChatRoomCustomMessage(String roomId, MsgAttachment attachment)|创建自定义消息|
|static ChatRoomMessage	|createChatRoomFileMessage(String roomId, File file, String displayName)|创建一条文件消息|
|static ChatRoomMessage	|createChatRoomImageMessage(String roomId, File file, String displayName)|创建一条图片消息|
|static ChatRoomMessage	|createChatRoomLocationMessage(String roomId, double lat, double lng, String addr)|创建一条地理位置信息|
|static ChatRoomMessage	|createChatRoomTextMessage(String roomId, String text)|创建普通文本消息|
|static ChatRoomMessage	|createChatRoomVideoMessage(String roomId, File file, long duration, int width, int height, String displayName)|创建一条视频消息|
|static ChatRoomMessage	|createEmptyChatRoomMessage(String roomId, long time)|创建一条空消息，仅设置了房间ID以及时间点，用于记录查询|
|static ChatRoomMessage	|createTipMessage(String roomId)|创建一条提醒消息|

聊天室消息 ChatRoomMessage 继承自 IMMessage。新增了两个方法：分别是设置和获取聊天室消息扩展属性。

- ChatRoomMessage 接口说明

|返回值|ChatRoomMessage 接口|说明|
|:---|:---|:---|
|CustomChatRoomMessageConfig	|getChatRoomConfig()|获取聊天室消息配置|
|ChatRoomMessageExtension	|getChatRoomMessageExtension()|获取聊天室消息扩展属性|
|void	|setChatRoomConfig<br>(CustomChatRoomMessageConfig config)|设置聊天室消息配置|

1\. 文本消息发送

以文本消息发送为例，其它类型的消息发送方式与会话消息类似。

- API 原型

```
/**
 * 创建普通文本消息
 *
 * @param roomId 聊天室ID
 * @param text   文本消息内容
 * @return ChatRoomMessage 生成的聊天室消息对象
 */
public static ChatRoomMessage createChatRoomTextMessage(String roomId, String text);

/**
 * 发送消息
 *
 * @param msg    带发送的消息体，由{@link ChatRoomMessageBuilder}构造。
 * @param resend 如果是发送失败后重发，标记为true，否则填false。
 * @return InvocationFuture 可以设置回调函数。消息发送完成后才会调用，如果出错，会有具体的错误代码。
 */
InvocationFuture<Void> sendMessage(ChatRoomMessage msg, boolean resend);
```

- 示例

```
// 示例用roomId
String roomId = "50";
// 文本消息内容
String text = "这是聊天室文本消息";
// 创建聊天室文本消息
ChatRoomMessage message = ChatRoomMessageBuilder.createChatRoomTextMessage(roomId, text);
// 将文本消息发送出去
NIMClient.getService(ChatRoomService.class).sendMessage(message, false)
        .setCallback(new RequestCallback<Void>() {
            @Override
            public void onSuccess(Void param) {
	            // 成功
            }

            @Override
            public void onFailed(int code) {
                // 失败
            }

            @Override
            public void onException(Throwable exception) {
               // 错误
            }
        });
```

- 特殊错误码

|错误码|说明|
|:---|:---|
|13004|在禁言列表中，不允许发言|
|13006|聊天室处于整体禁言状态,只有管理员能发言|

#### <span id="发送消息配置选项">发送消息配置选项</span>

发送聊天室消息时，可以设置配置选项 `CustomChatRoomMessageConfig` ，主要用于设置是否存入历史记录等。

|CustomChatRoomMessageConfig 属性|说明|
|:---|:---|
|skipHistory|该消息是否要保存到服务器，<br>如果为true，通过ChatRoomService#pullMessageHistory 拉取的结果将不包含该条消息。<br>默认为false。|

- 示例

```
// 示例用roomId
String roomId = "50";
// 文本消息内容
String text = "这是聊天室文本消息";
// 创建聊天室文本消息
ChatRoomMessage message = ChatRoomMessageBuilder.createChatRoomTextMessage(roomId, text);
// 配置不存历史记录
CustomChatRoomMessageConfig config = new CustomChatRoomMessageConfig();
config.skipHistory = true;
message.setChatRoomConfig(config);
// 发送聊天室消息
NIMClient.getService(ChatRoomService.class).sendMessage(message, false);
```

#### <span id="接收消息">接收消息</span>

通过添加消息接收观察者，在有新消息到达时，第三方 APP 就可以接收到通知。

- API 原型

```
/**
 * 注册/注销消息接收观察者。
 *
 * @param observer 观察者，参数为收到的消息集合
 * @param register true为注册，false为注销
 */
public void observeReceiveMessage(Observer<List<ChatRoomMessage>> observer, boolean register);

```

- 示例

```
Observer<List<ChatRoomMessage>> incomingChatRoomMsg = new Observer<List<ChatRoomMessage>>() {
        @Override
        public void onEvent(List<ChatRoomMessage> messages) {
            // 处理新收到的消息
        }
    };

NIMClient.getService(ChatRoomServiceObserver.class)
	.observeReceiveMessage(incomingChatRoomMsg, register);
```

### <span id="聊天室管理">聊天室管理</span>

#### <span id="获取聊天室信息">获取聊天室信息</span>

此接口可以向服务器获取聊天室信息。SDK 不提供聊天室信息的缓存，开发者可根据需要自己做缓存。聊天室基本信息的扩展字段需要在服务器端设置，客户端 SDK 只提供查询。

- API 原型

```
/**
 * 获取当前聊天室信息
 *
 * @param roomId 聊天室id
 * @return InvocationFuture 可以设置回调函数。回调中返回聊天室的信息
 */
InvocationFuture<ChatRoomInfo> fetchRoomInfo(String roomId);
```

- 参数说明

|返回值|ChatRoomInfo 接口|说明|
|:---|:---|:---|
|String	|getAnnouncement()|获取聊天室公告|
|String	|getBroadcastUrl()|获取聊天室拉流地址|
|String	|getCreator()|获取聊天室创建者帐号|
|Map 	|getExtension()|获取聊天室扩展字段|
|String	|getName()|获取聊天室名称|
|int	|getOnlineUserCount()|获取当前在线用户数量|
|String	|getRoomId()|获取聊天室id|
|boolean	|isMute()|获取当前聊天室禁言状态|
|boolean	|isValid()|获取聊天室有效标记|
|void	|setAnnouncement(String announcement)|设置聊天室公告|
|void	|setBroadcastUrl(String broadcastUrl)|设置聊天室直播拉流地址|
|void	|setCreator(String creator)|设置聊天室创建者|
|void	|setExtension(Map extension)|设置扩展字段|
|void	|setMute(int mute)|设置当前聊天室禁言状态|
|void	|setName(String name)|设置聊天室名称|
|void	|setOnlineUserCount(int onlineUserCount)|设置当前在线用户数量|
|void	|setRoomId(String roomId)|设置聊天室id|
|void	|setValidFlag(int validFlag)|设置聊天室有效标记|

- 示例

```
// roomId为聊天室ID
NIMClient.getService(ChatRoomService.class).fetchRoomInfo(roomId).setCallback(new RequestCallback<ChatRoomInfo>() {
    @Override
    public void onSuccess(ChatRoomInfo param) {
        // 成功
    }

    @Override
    public void onFailed(int code) {
        // 失败
    }

    @Override
    public void onException(Throwable exception) {
        // 错误
    }
});
```

#### <span id="修改聊天室信息">修改聊天室信息</span>

支持修改聊天室信息，只有创建者和管理员拥有权限修改聊天室信息。可以设置修改是否通知，若设置通知，聊天室内会收到类型为`NotificationType#ChatRoomInfoUpdated`的消息。

- API 原型

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
InvocationFuture<Void> updateRoomInfo(String roomId, ChatRoomUpdateInfo chatRoomUpdateInfo, boolean needNotify, Map<String, Object> notifyExtension);
```

- 参数说明

|参数|说明|
|:---|:---|
|roomId | 聊天室id|
|chatRoomUpdateInfo | 可更新的聊天室信息|
|needNotify | 是否通知|
|notifyExtension | 更新聊天室信息扩展字段，这个字段会放到更新聊天室信息通知的扩展字段中|

ChatRoomUpdateInfo 接口说明

|返回值|ChatRoomUpdateInfo 接口|说明|
|:---|:---|:---|
|String	|getAnnouncement() |获取聊天室公告|
|String	|getBroadcastUrl() |获取视频直播拉流地址|
|Map	|getExtension() |获取第三方扩展字段|
|String	|getName() |获取聊天室名称|
|void	|setAnnouncement(String announcement) |设置聊天室公告|
|void	|setBroadcastUrl(String broadcastUrl) |设置视频直播拉流地址|
|void	|setExtension(Map extension) |设置第三方扩展字段,长度 4k|
|void	|setName(String name) |设置聊天室名称|

- 示例

```
// 以更新聊天室信息为例
ChatRoomUpdateInfo updateInfo = new ChatRoomUpdateInfo();
updateInfo.setName("新聊天室名称");

NIMClient.getService(ChatRoomService.class)
        .updateRoomInfo(roomId, updateInfo, false, "")
        .setCallback(new RequestCallback<Void>() {
            @Override
            public void onSuccess(Void aVoid) {
                // 成功
            }

            @Override
            public void onFailed(int i) {
                // 失败
            }

            @Override
            public void onException(Throwable throwable) {
				// 错误
            }
        });
```

#### <span id="获取聊天室成员">获取聊天室成员</span>

1\.  根据成员类型获取聊天室成员

- API 介绍

该接口可以获取固定成员信息、仅在线的固定成员信息及游客信息 ChatRoomMember，其中的扩展字段由该用户进入聊天室时填写。

固定成员有四种类型，分别是创建者,管理员,普通用户,受限用户。禁言用户和黑名单用户都属于受限用户。

拉取固定成员时，无论成员在线不在线，都会返回。拉取游客时，只返回在线的成员。

- API 原型

```
/**
 * 获取聊天室成员信息
 *
 * @param roomId          聊天室id
 * @param memberQueryType 成员查询类型。
 * @param time            固定成员列表用updateTime,
 *                        游客列表用进入enterTime，
 *                        填0会使用当前服务器最新时间开始查询，即第一页，单位毫秒
 * @param limit           条数限制
 * @return InvocationFuture 可以设置回调函数。回调中返回成员信息列表
 */
InvocationFuture<List<ChatRoomMember>> fetchRoomMembers(String roomId, MemberQueryType memberQueryType, long time, int limit);

```

- 参数说明

|参数|说明|
|:---|:---|
|roomId | 聊天室id|
|memberQueryType | 成员查询类型。见 MemberQueryType|
|time | 固定成员列表用 updateTime, 游客列表用进入 enterTime， <br>填 0 会使用当前服务器最新时间开始查询，即第一页，单位毫秒|
|limit | 条数限制|

ChatRoomMember  接口说明

|返回值|ChatRoomMember 接口|说明|
|:---|:---|:---|
|String	|getAccount()|获取成员帐号|
|String	|getAvatar()|获取头像。可从 NimUserInfo 中取 avatar，可以由用户进聊天室时提交|
|long	|getEnterTime()|获取进入聊天室时间。对于离线成员该字段为空|
|Map	|getExtension()|获取扩展字段。长度限制 4k，可以由用户进聊天室时提交|
|int	|getMemberLevel()|获取成员级别。大于等于 0 表示用户开发者可以自定义的级别|
|MemberType	|getMemberType()|获取成员类型。成员类型：主要分为游客和非游客|
|String	|getNick()|获取昵称。可从 NimUserInfo 中取，也可以由用户进聊天室时提交|
|String	|getRoomId()|获取聊天室 id|
|long	|getTempMuteDuration()|获取临时禁言解除时长，单位秒|
|long	|getUpdateTime()|获取固定成员的记录更新时间，用于固定成员列表的排列查询|
|boolean	|isInBlackList()|判断用户是否在黑名单中|
|boolean	|isMuted()|判断用户是否被禁言|
|boolean	|isOnline()|判断用户是否处于在线状态 仅特殊成员才可能离线，对游客/匿名用户而言只能是在线|
|boolean	|isTempMuted()|判断用户是否被临时禁言|
|boolean	|isValid()|判断是否有效|
|void	|setAccount(String account)|设置用户帐号|
|void	|setAvatar(String avatar)|设置成员头像|
|void	|setEnterTime(long enterTime)|设置进入聊天室时间|
|void	|setExtension(Map extension)|设置扩展字段|
|void	|setInBlackList(boolean inBlackList)|设置是否在黑名单中|
|void	|setMemberLevel(int memberLevel)|设置成员等级|
|void	|setMemberType(MemberType type)|设置成员类型|
|void	|setMuted(boolean muted)|设置是否禁言|
|void	|setNick(String nick)|设置成员昵称|
|void	|setOnline(boolean online)|设置在线状态|
|void	|setRoomId(String roomId)|设置聊天室id|
|void	|setTempMuted(boolean tempMuted)|设置是否临时禁言|
|void	|setTempMuteDuration(long tempMuteDuration)|设置临时禁言解除时长|
|void	|setUpdateTime(long updateTime)|设置固定成员的更新时间|
|void	|setValid(boolean valid)|设置是否有效|

MemberQueryType 属性说明

|MemberQueryType 属性|说明|
|:---|:---|
|GUEST|非固定成员 <br>(又称临时成员,只有在线时才能在列表中看到,数量无上限)|
|NORMAL|固定成员<br>（包括创建者,管理员,普通等级用户,受限用户(禁言+黑名单),<br>即使非在线也可以在列表中看到,有数量限制 ）|
|ONLINE_NORMAL|仅在线的固定成员|
|UNKNOWN|未知|

- 示例

```
// 以游客为例，从最新时间开始，查询10条
NIMClient.getService(ChatRoomService.class).fetchRoomMembers(roomId, MemberQueryType.GUEST, 0, 10).setCallback(new RequestCallbackWrapper<List<ChatRoomMember>>() {
    @Override
    public void onResult(int code, List<ChatRoomMember> result, Throwable exception) {
       ...
    }
});
```

2\. 根据用户 id 获取聊天室成员信息

- API 介绍

通过用户 id 批量获取指定成员在聊天室中的信息。

- API 原型

```
/**
 * 根据用户id获取聊天室成员信息
 *
 * @param roomId   聊天室id
 * @param accounts 成员帐号列表
 * @return InvocationFuture 可以设置回调函数。回调中返回成员信息列表
 */
InvocationFuture<List<ChatRoomMember>> fetchRoomMembersByIds(String roomId, List<String> accounts);

```

- 示例

```
List<String> accounts = new ArrayList<>();
accounts.add(account);

NIMClient.getService(ChatRoomService.class)
	.fetchRoomMembersByIds(roomId, accounts)
	.setCallback(new RequestCallbackWrapper<List<ChatRoomMember>>() { ... });
```

#### <span id="监听聊天室在线状态">监听聊天室在线状态</span>

- API 介绍

进入聊天室成功后，SDK 会负责维护与服务器的长连接以及断线重连等工作。当用户在线状态发生改变时，会发出通知。登录过程也有状态回调。此外，网络连接上之后，SDK 会负责聊天室的自动登录。

- API 原型

```
/**
 * 注册/注销聊天室在线状态/登录状态观察者
 *
 * @param observer 观察者, 参数为聊天室ID和聊天室状态（未进入、连接中、进入中、已进入）
 * @param register true为注册，false为注销
 */
public void observeOnlineStatus(Observer<ChatRoomStatusChangeData> observer, boolean register);
```

- 参数说明

|ChatRoomStatusChangeData 属性|说明|
|:---|:---|
|roomId | 聊天室 ID|
|status | 状态码，参考 StatusCode|

- 示例

```
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

#### <span id="修改自己的聊天室成员信息">修改自己的聊天室成员信息</span>

- API 介绍

支持更新本人聊天室成员信息。目前只支持昵称，头像和扩展字段的更新。可以设置更新是否通知，若设置通知，聊天室内会收到类型为`NotificationType#ChatRoomMyRoomRoleUpdated`的消息。

- API 原型

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
InvocationFuture<Void> updateMyRoomRole(String roomId, ChatRoomMemberUpdate chatRoomMemberUpdate, boolean needNotify, Map<String, Object> notifyExtension);
```

- 参数说明

|参数|说明|
|:---|:---|
|roomId | 聊天室 id|
|chatRoomMemberUpdate | 可更新的本人角色信息|
|needNotify | 是否通知|
|notifyExtension | 更新聊天室信息扩展字段，这个字段会放到更新聊天室信息通知的扩展字段中|

ChatRoomMemberUpdate 接口说明

|返回值|ChatRoomMemberUpdate 接口|说明|
|:---|:---|:---|
|String	|getAvatar() |获取聊天室内的头像|
|Map	|getExtension() |获取扩展字段|
|String	|getNick() |获取聊天室内的昵称|
|boolean	|isNeedSave() |更新的信息是否需要做持久化，只对固定成员生效，默认 false，v3.8.0增加|
|void	|setAvatar(String avatar) |设置聊天室内的头像|
|void	|setExtension(Map extension) |设置扩展字段，长度限制 2048|
|void	|setNeedSave(boolean needSave) |设置是否需要持久化，只对固定成员生效，v3.8.0增加|
|void	|setNick(String nick) |设置聊天室内的昵称|

- 示例

```
// 以更新自己的昵称为例
ChatRoomMemberUpdate update = new ChatRoomMemberUpdate();
update.setNick("我的新昵称");

NIMClient.getService(ChatRoomService.class).updateMyRoomRole(roomId, update, false, "").setCallback(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void param) {
        // 成功
    }

    @Override
    public void onFailed(int code) {
        // 失败
    }

    @Override
    public void onException(Throwable exception) {
		// 错误
    }
});
```

### <span id="聊天室权限管理">聊天室权限管理</span>

#### <span id="设置为管理员">设置为管理员</span>

- API 介绍

设为管理员时, 会收到类型为 `ChatRoomManagerAdd` 的聊天室通知消息。

取消管理员时, 会收到类型为 `ChatRoomManagerRemove` 的聊天室通知消息。

如果游客被设为管理员，再被取消管理员，该成员不在变为游客，而成为普通成员。

- API 原型

```

/**
 * 设为/取消聊天室管理员
 *
 * @param isAdd        true:设为, false:取消
 * @param memberOption 请求参数，包含聊天室id，帐号id以及可选的扩展字段
 * @return InvocationFuture 可以设置回调函数。回调中返回成员信息
 */
InvocationFuture<ChatRoomMember> markChatRoomManager(boolean isAdd, MemberOption memberOption);
```

- 参数说明

|返回值|MemberOption 接口|说明|
|:---|:---|:---|
|String	|getAccount() |获取成员帐号|
|Map	|getNotifyExtension() |获取通知事件中开发者定义的扩展字段|
|String	|getRoomId() |获取聊天室id|
|void	|setAccount(String account) |设置成员帐号|
|void	|setNotifyExtension(Map notifyExtension) |设置通知事件中的扩展字段|
|void	|setRoomId(String roomId) |设置聊天室 id|

- 示例

```
// 以设置为管理员为例
NIMClient.getService(ChatRoomService.class)
        .markChatRoomManager(true, new MemberOption(roomId, account))
        .setCallback(new RequestCallback<ChatRoomMember>() {
            @Override
            public void onSuccess(ChatRoomMember param) {
                // 成功
            }

            @Override
            public void onFailed(int code) {
                // 失败

            @Override
            public void onException(Throwable exception) {
				// 错误
            }
        });
```
#### <span id="设置为普通成员">设置为普通成员 </span>

- API 介绍

即将游客变为固定成员中的普通成员身份。

 设为普通成员时, 会收到类型为 `ChatRoomCommonAdd` 的聊天室通知消息。

移除普通成员时, 会收到类型为 `ChatRoomCommonRemove` 的聊天室通知消息。

- API 原型

```
/**
 * 设为/取消聊天室普通成员
 *
 * @param isAdd        true:设为, false:取消
 * @param memberOption 请求参数，包含聊天室id，帐号id以及可选的扩展字段
 * @return InvocationFuture 可以设置回调函数。回调中返回成员信息
 */
InvocationFuture<ChatRoomMember> markNormalMember(boolean isAdd, MemberOption memberOption);
```

- 示例

```
// 以设为普通成员为例
NIMClient.getService(ChatRoomService.class).markNormalMember(true, new MemberOption(roomId, account)))
        .setCallback(new RequestCallback<ChatRoomMember>() {
            @Override
            public void onSuccess(ChatRoomMember param) {
               // 成功
            }

            @Override
            public void onFailed(int code) {
				// 失败
            }

            @Override
            public void onException(Throwable exception) {
				// 错误
            }
        });
```
#### <span id="设置为拉黑用户">设置为拉黑用户 </span>

- API 介绍

加入黑名单时，会收到类型为 `ChatRoomMemberBlackAdd` 聊天室通知消息。

移出黑名单时，会收到类型为 `ChatRoomMemberBlackRemove` 的聊天室通知消息。

- API 原型

```
/**
 * 添加/移出聊天室黑名单
 *
 * @param isAdd        true:添加, false:移出
 * @param memberOption 请求参数，包含聊天室id，帐号id以及可选的扩展字段
 * @return InvocationFuture 可以设置回调函数。回调中返回成员信息
 */
InvocationFuture<ChatRoomMember> markChatRoomBlackList(boolean isAdd, MemberOption memberOption);
```

- 示例

```
// 以添加到黑名单为例
MemberOption option = new MemberOption(roomId, account);
        NIMClient.getService(ChatRoomService.class).markChatRoomBlackList(true, option).setCallback(new RequestCallback<ChatRoomMember>() {
                    @Override
                    public void onSuccess(ChatRoomMember param) {
                        // 成功
                    }

                    @Override
                    public void onFailed(int code) {
                        // 失败
                    }

                    @Override
                    public void onException(Throwable exception) {
						// 错误
                    }
                });
```

#### <span id="设置为禁言用户">设置为禁言用户 </span>

1\. 永久禁言

- API 介绍

添加到禁言名单时, 会收到类型为 `ChatRoomMemberMuteAdd` 的聊天室通知消息。

取消禁言时, 会收到类型为 `ChatRoomMemberMuteRemove` 的聊天室通知消息。

取消禁言之后，恢复为原来的身份。原来是游客，取消禁言后，变为游客。原来是普通成员，取消禁言后，变为普通成员。

- API 原型

```
/**
 * 添加到禁言名单/取消禁言
 *
 * @param isAdd        true:添加, false:取消
 * @param memberOption 请求参数，包含聊天室id，帐号id以及可选的扩展字段
 * @return InvocationFuture 可以设置回调函数。回调中返回成员信息
 */
InvocationFuture<ChatRoomMember> markChatRoomMutedList(boolean isAdd, MemberOption memberOption);

```

- 示例

```
// 以添加到禁言名单为例
MemberOption option = new MemberOption(roomId, account);
NIMClient.getService(ChatRoomService.class).markChatRoomMutedList(true, option)
        .setCallback(new RequestCallback<ChatRoomMember>() {
            @Override
            public void onSuccess(ChatRoomMember param) {
                // 成功
            }

            @Override
            public void onFailed(int code) {
                // 失败
            }

            @Override
            public void onException(Throwable exception) {
				// 错误
            }
        });
```

2\. 临时禁言

- API 介绍

设置临时禁言时, 会收到类型为 `ChatRoomMemberTempMuteAdd` 的聊天室通知消息。

取消临时禁言时, 会收到类型为 `ChatRoomMemberTempMuteRemove` 的聊天室通知消息。

聊天室支持设置临时禁言，禁言时长时间到了，自动取消禁言。设置临时禁言成功后的通知消息中，包含的时长是禁言剩余时长。若设置禁言时长为0，表示取消临时禁言。若第一次设置的禁言时长还没结束，又设置第二次临时禁言，以第二次设置的时间开始计时。

- API 原型

```
/**
 * 设置聊天室成员临时禁言
 *
 * @return InvocationFuture 可以设置回调函数。如果出错，会有具体的错误代码。
 */
InvocationFuture<Void> markChatRoomTempMute(boolean needNotify, long duration, MemberOption memberOption);
```

- 参数说明

|参数|说明|
|:---|:---|
|needNotify | 是否需要发送广播通知，true：通知，false：不通知|
|duration | 禁言时长,单位秒|
|memberOption | 请求参数，包含聊天室 id，帐号 id 以及可选的扩展字段|

- 示例

```
// 以发送广播通知，并且临时禁言10秒为例
MemberOption option = new MemberOption(roomId, account);
NIMClient.getService(ChatRoomService.class).markChatRoomTempMute(true, 10, option)
        .setCallback(new RequestCallback<Void>() {
            @Override
            public void onSuccess(Void param) {
                // 成功
            }

            @Override
            public void onFailed(int code) {
                // 失败
            }

            @Override
            public void onException(Throwable exception) {
				// 错误
            }
        });
```

#### <span id="踢出成员">踢出成员 </span>

- API 介绍

踢出成员。仅管理员可以踢；如目标是管理员仅创建者可以踢。

可以添加被踢通知扩展字段，这个字段会放到被踢通知的扩展字段中。通知扩展字段最长 1K；扩展字段需要传入 Map<String,Object>， SDK 会负责转成 Json String。

当有人被踢出聊天室时，会收到类型为 `ChatRoomMemberKicked` 的聊天室通知消息。

- API 原型

```
/**
 * 踢掉特定成员
 *
 * @return InvocationFuture 可以设置回调函数。踢掉成员完成之后才会调用，如果出错，会有具体的错误代码。
 */
InvocationFuture<Void> kickMember(String roomId, String account, Map<String, Object> notifyExtension);
```

- 参数说明

|参数|说明|
|:---|:---|
|roomId | 聊天室 id|
|account | 踢出成员帐号。仅管理员可以踢；如目标是管理员仅创建者可以踢|
|notifyExtension | 被踢通知扩展字段，这个字段会放到被踢通知的扩展字段中|

- 示例

```
Map<String, Object> reason = new HashMap<>();
reason.put("reason", "kick");
NIMClient.getService(ChatRoomService.class)
	.kickMember(roomId, account, reason)
	.setCallback(new RequestCallback<Void>() { ... });
```

#### <span id="监听被踢出聊天室">监听被踢出聊天室 </span>

当用户被主播或者管理员踢出聊天室、聊天室被关闭（被解散），会收到通知。注意：收到被踢出通知后，不需要再调用退出聊天室接口，SDK 会负责聊天室的退出工作。可以在踢出通知中做相关缓存的清理工作和界面操作。

- API 原型

```
/**
 * 注册/注销被踢出聊天室观察者。
 *
 * @param observer 观察者, 参数为被踢出的事件（可以获取被踢出的聊天室ID和被踢出的原因）
 * @param register true为注册，false为注销
 */
public void observeKickOutEvent(Observer<ChatRoomKickOutEvent> observer, boolean register);
```

- 参数说明

ChatRoomKickOutEvent.ChatRoomKickOutReason 属性说明

|ChatRoomKickOutEvent.ChatRoomKickOutReason  属性|说明|
|:---|:---|
|BE_BLACKLISTED|被加黑了|
|CHAT_ROOM_INVALID|聊天室已经被解散|
|ILLEGAL_STAT|当前连接状态异常|
|KICK_OUT_BY_CONFLICT_LOGIN|被其他端踢出|
|KICK_OUT_BY_MANAGER|被管理员踢出|
|UNKNOWN|未知（出错情况）|

- 示例

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

### <span id="查询聊天室消息历史">查询聊天室消息历史</span>

聊天室支持获取云端消息记录的功能，进入聊天室时，不会下发该聊天室的历史消息，开发者应主动查询历史消息。

1\. 获取历史消息

- API 介绍

以 startTime（单位毫秒） 为时间戳，往前拉取 limit 条消息。拉取到的消息中也包含成员操作的通知消息。

- API 原型

```
/**
 * 获取历史消息，默认从给定时间点往前查询，排序为时间逆序
 *
 * @return InvocationFuture 可以设置回调函数。回调中返回历史消息列表
 */
InvocationFuture<List<ChatRoomMessage>> pullMessageHistory(String roomId, long startTime, int limit);
```

- 参数说明

|参数|说明|
|:---|:---|
|roomId | 聊天室 id|
|startTime | 时间戳，单位毫秒|
|limit | 可拉取的消息数量|


- 示例

```
long startTime = System.currentTimeMillis();
// 从现在开始，查询200条消息
NIMClient.getService(ChatRoomService.class).pullMessageHistory(roomId, startTime, 200)
```

2\. 根据消息方向获取历史消息

- API 介绍

以 startTime（单位毫秒） 为时间戳，选择查询方向往前或者往后拉取 limit 条消息。拉取到的消息中也包含成员操作的通知消息。

查询结果排序如查询方向有关，若方向往前，则结果排序按时间逆序，反之则结果排序按时间顺序。

- API 原型

```
/**
 * 获取历史消息,可选择给定时间往前或者往后查询，若方向往前，则结果排序按时间逆序，反之则结果排序按时间顺序。
 *
 * @return InvocationFuture 可以设置回调函数。回调中返回历史消息列表
 */
InvocationFuture<List<ChatRoomMessage>> pullMessageHistoryEx(String roomId, long startTime, int limit, QueryDirectionEnum direction);
```

- 参数说明

|参数|说明|
|:---|:---|
|roomId | 聊天室 id|
|startTime | 时间戳，单位毫秒|
|limit | 可拉取的消息数量|
|direction | 查询方向|

QueryDirectionEnum 属性说明

|QueryDirectionEnum 属性|说明|
|:---|:---|
|QUERY_OLD|查询比锚点时间更早的消息|
|QUERY_NEW|查询比锚点时间更晚的消息|

- 示例

```
long startTime = System.currentTimeMillis();
// 从现在开始，往 QueryDirectionEnum.QUERY_OLD 查询200条
NIMClient.getService(ChatRoomService.class).pullMessageHistoryEx(roomId, startTime, 200, QueryDirectionEnum.QUERY_OLD);
```

### <span id="聊天室通知消息">聊天室通知消息</span>

聊天室通知消息是聊天室消息的一种（消息类型为 Notification）。即聊天室消息为 ChatRoomMessage， 其中附件类型为 ChatRoomNotificationAttachment ，附件类型中的 type 字段来标识聊天室通知消息的类型。目前支持的类型见 NotificationType 。参考 [通知消息](#通知消息) 章节。

### <span id="队列服务">队列服务</span>

聊天室提供队列服务，针对直播连麦场景使用。

#### <span id="加入或更新队列元素">加入或更新队列元素</span>

- API 介绍


> 注意：key是唯一键

当 key 不存在时候，调用该接口，表示加入新元素。
当 key 存在的时候，调用该接口，表示修改对应 key 的 value。

- API 原型

```

/**
 * 聊天室队列服务：加入或者更新队列元素
 *
 * @return InvocationFuture 可以设置回调函数。回调中返回操作成功或者失败具体的错误码。
 */
InvocationFuture<Void> updateQueue(String roomId, String key, String value);
```

- 参数说明

|参数|说明|
|:---|:---|
|roomId | 聊天室 id|
|key | 新元素（或待更新元素）的唯一键|
|value | 新元素（待待更新元素）的内容|

- 示例

```
NIMClient.getService(ChatRoomService.class)
        .updateQueue(roomId, "this is key", "this is value")
        .setCallback(new RequestCallback<Void>() {
            @Override
            public void onSuccess(Void param) {
                // 成功
            }

            @Override
            public void onFailed(int code) {
                // 失败
            }

            @Override
            public void onException(Throwable exception) {
				// 错误
            }
        });
```

#### <span id="取出队列元素">取出队列元素</span>

- API 介绍

> 注意：key是唯一键

key 若为 null， 则表示取出头元素。若不为 null， 则表示取出指定元素。

- API 原型

```
/**
 * 聊天室队列服务：取出队列头部或者指定元素
 *
 * @return InvocationFuture 可以设置回调函数。如果元素存在或者队列不为空，那么回调中返回元素的键值对，否则返回失败错误码。
 */
InvocationFuture<Entry<String, String>> pollQueue(String roomId, String key);
```

- 参数说明

|参数|说明|
|:---|:---|
|roomId | 聊天室 id|
|key | 需要取出的元素的唯一键。若为 null，则表示取出队头元素|

- 示例

```
NIMClient.getService(ChatRoomService.class).pollQueue(roomId, "this is key"
        .setCallback(new RequestCallback<Entry<String, String>>() {
            @Override
            public void onSuccess(Entry<String, String> param) {
                // 成功
            }

            @Override
            public void onFailed(int code) {
                // 错误
            }

            @Override
            public void onException(Throwable exception) {
				// 失败
            }
        });
```

#### <span id="获取所有队列元素">获取所有队列元素</span>

- API 原型

```
/**
 * 聊天室队列服务：排序列出所有队列元素
 *
 * @param roomId 聊天室id
 * @return InvocationFuture 可设置回调函数。回调中返回所有排列的队列元素键值对。
 */
InvocationFuture<List<Entry<String, String>>> fetchQueue(String roomId);
```

- 示例

```
NIMClient.getService(ChatRoomService.class).fetchQueue(roomId).setCallback(new RequestCallback<List<Entry<String, String>>>() {
    @Override
    public void onSuccess(List<Entry<String, String>> param) {
        // 成功
    }

    @Override
    public void onFailed(int code) {
        // 失败
    }

    @Override
    public void onException(Throwable exception) {
		// 错误
    }
});
```

#### <span id="删除队列">删除队列</span>

- API 原型

```
/**
 * 聊天室队列服务：删除队列
 *
 * @param roomId 聊天室id
 * @return InvocationFuture 可设置回调函数。回调中返回操作成功或者失败具体的错误码。
 */
InvocationFuture<Void> dropQueue(String roomId);
```

- 示例

```
NIMClient.getService(ChatRoomService.class).dropQueue(roomId).setCallback(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void param) {
        // 成功
    }

    @Override
    public void onFailed(int code) {
        // 失败
    }

    @Override
    public void onException(Throwable exception) {
		// 错误
    }
});
```

## <span id="用户资料托管">用户资料托管</span>

网易云通信提供了用户资料托管，包含生日，Email，性别，手机号码，签名和扩展字段的管理，参考 `NimUserInfo` 说明，第三方 APP 可以使用也可以自行实现，但必须实现 `UserInfoProvider.UserInfo` 接口。

NimUserInfo 接口说明：

|返回值|NimUserInfo 接口|说明|
|:---|:---|:---|
|String	|getBirthday()|获取生日|
|String	|getEmail()|获取 Email|
|String	|getExtension()|获取扩展字段|
|Map |getExtensionMap()|获取扩展字段，返回 Map 格式|
|GenderEnum	|getGenderEnum()|获取性别|
|String	|getMobile()|获取手机号码|
|String	|getSignature()|获取签名|

UserInfoProvider.UserInfo 接口说明：

|返回值|UserInfoProvider.UserInfo 接口|说明|
|:---|:---|:---|
|String	|getAccount()|返回用户帐号|
|String	|getAvatar()|返回用户头像地址|
|String	|getName()|返回用户名|

### <span id="获取本地用户资料">获取本地用户资料</span>

#### <span id="从本地数据库中批量获取用户资料">从本地数据库中批量获取用户资料</span>

- API 介绍

通过用户帐号集合，从本地数据库批量获取用户资料列表。

- API 原型

```
/**
 * 从本地数据库中批量获取用户资料（同步接口）
 *
 * @param accounts 要获取的用户帐号集合
 * @return 用户资料列表
 */
List<NimUserInfo> getUserInfoList(List<String> accounts);
```

- 参数说明

|参数|说明|
|:---|:---|
|accounts|要获取的用户帐号集合|

- 示例

```java
List<NimUserInfo> users = NIMClient.getService(UserService.class).getUserInfoList(accounts);
```

#### <span id="从本地数据库中获取用户资料">从本地数据库中获取用户资料</span>

- API 介绍

通过用户账号，从本地数据库获取用户资料。

- API 原型

```
/**
 * 从本地数据库中获取用户资料（同步接口）
 *
 * @param account 要获取的用户账号
 * @return 用户资料
 */
NimUserInfo getUserInfo(String account);
```

- 示例

```java
NimUserInfo user = NIMClient.getService(UserService.class).getUserInfo(account);
```

#### <span id="从本地数据库中获取所有用户资料">从本地数据库中获取所有用户资料</span>

- API 介绍

获取本地数据库中所有的用户资料，一般适合在登录后构建用户资料缓存时使用。

- API 原型

```
/**
 * 获取本地数据库中所有用户资料
 *
 * @return 所有用户资料列表
 */
List<NimUserInfo> getAllUserInfo();
```

- 示例

```java
List<NimUserInfo> users = NIMClient.getService(UserService.class).getAllUserInfo();
```

#### <span id="构建通讯录">构建通讯录</span>

如果使用网易云通信用户关系、用户资料托管，构建通讯录，先获取我所有好友帐号，再根据帐号去获取对应的用户资料。

- API 原型

```
/**
 * 获取我所有的好友帐号
 *
 * @return 好友帐号集合
 */
List<String> getFriendAccounts();
```

- 示例

```java
List<String> accounts = NIMClient.getService(FriendService.class).getFriendAccounts(); // 获取所有好友帐号
List<NimUserInfo> users = NIMClient.getService(UserService.class).getUserInfoList(accounts); // 获取所有好友用户资料
```

### <span id="获取服务器用户资料">获取服务器用户资料</span>

- API 介绍

从服务器获取用户资料，一般在本地用户资料不存在时调用，获取后 SDK 会负责更新本地数据库。每次最多获取150个用户，如果量大，上层请自行分批获取。

- API 原型

```
/**
 * 从服务器获取用户资料（每次最多获取150个用户，如果量大，上层请自行分批获取）
 *
 * @param accounts 要获取的用户帐号
 * @return InvocationFuture 可以设置回调函数。在用户资料存入数据库后就会回调。
 */
InvocationFuture<List<NimUserInfo>> fetchUserInfo(List<String> accounts);
```

- 示例

```java
NIMClient.getService(UserService.class).fetchUserInfo(accounts)
    .setCallback(new RequestCallback<List<UserInfo>>() { ... });
```

说明：此接口可以批量从服务器获取用户资料，从用户体验和流量成本考虑，不建议应用频繁调用此接口。对于用户数据实时性要求不高的页面，应尽量调用读取本地缓存接口。

### <span id="编辑用户资料">编辑用户资料</span>

#### <span id="更新用户本人资料">更新用户本人资料</span>

- API 介绍

传入参数 `Map<UserInfoFieldEnum, Object>` 更新用户本人资料，key 为字段，value 为对应的值。具体字段见 `UserInfoFieldEnum`。

UserInfoFieldEnum 属性说明：

|参数|说明|
|:---|:---|
|AVATAR|头像URL|
|BIRTHDAY|生日|
|EMAIL|电子邮箱|
|EXTEND|扩展字段|
|GENDER|性别|
|MOBILE|手机|
|Name|昵称|
|SIGNATURE|签名|
|undefined|未定义的域|

- API 原型

```
/**
 * 更新本人用户资料
 *
 * @param fields 要更新的字段和新值, key为字段，value为对应的值
 * @return @return InvocationFuture 可以设置回调函数。
 */
InvocationFuture<Void> updateUserInfo(Map<UserInfoFieldEnum, Object> fields);

```

- 参数说明

|参数|说明|
|:---|:---|
|fields |要更新的字段和新值, key 为字段，value 为对应的值|

- 示例

```java
Map<UserInfoFieldEnum, Object> fields = new HashMap<>(1);
fields.put(UserInfoFieldEnum.Name, "new name");
NIMClient.getService(UserService.class).updateUserInfo(fields)
    .setCallback(new RequestCallbackWrapper<Void>() { ... });
```

- 更新头像可选方案：

先将头像图片上传至网易云通信云存储上（见 `NosService` ) ，上传成功后可以得到 url 。 更新个人资料的头像字段，保存 url 。

此外，开发者也可以自行存储头像，仅将 url 更新到个人资料上。

- SDK对部分字段进行格式校验：

|字段|说明|
|:---|:---|
|邮箱|必须为合法邮箱|
|手机号|必须为合法手机号 如 13588888888、+(86)-13055555555|
|生日|必须为 "yyyy-MM-dd" 格式|

#### <span id="监听用户资料变更">监听用户资料变更</span>

- API 介绍

用户资料除自己之外，不保证其他用户资料实时更新。其他用户数据更新时机为：

1\. 调用 fetchUserInfo 方法刷新用户

2\. 收到此用户发来消息（如果消息发送者有资料变更，SDK 会负责更新并通知）

3\. 程序再次启动，此时会同步好友信息

由于用户资料变更需要跨进程异步调用，开发者最好能在第三方 APP 中做好用户资料缓存，查询用户资料时都从本地缓存中访问。在用户资料有变化时，SDK 会告诉注册的观察者，此时，第三方 APP 可更新缓存，并刷新界面。

- API 原型

```
/**
 * 用户资料变更观察者通知
 *
 * @param observer 观察者，参数为更新后的用户资料列表
 * @param register true为注册，false为注销
 */
void observeUserInfoUpdate(Observer<List<NimUserInfo>> observer, boolean register);
```

- 示例

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

## <span id="用户关系托管">用户关系托管</span>

网易云通信提供了好友关系的托管，好友资料（用户资料）由第三方 APP 自行管理或者选择网易云通信用户资料托管，见[用户资料托管](#用户资料托管)。

### <span id="好友关系">好友关系</span>

#### <span id="添加好友">添加好友</span>

1\. 发出添加好友请求

- API 介绍

目前添加好友有两种验证类型（见 `VerifyType`）：直接添加为好友和发起好友验证请求。添加好友时需要构造 `AddFriendData`，需要填入包括对方帐号，好友验证类型及附言（可选）。

VerifyType 属性说明：

|VerifyType 属性|说明|
|:---|:---|
|DIRECT_ADD|直接加对方为好友|
|VERIFY_REQUEST|发起好友验证请求|

AddFriendData 接口说明：

|返回值|AddFriendData 接口|说明|
|:---|:---|:---|
|String	|getAccount() |获取帐号|
|String	|getMsg() |获取好友请求附言|
|VerifyType	|getVerifyType() |获取好友请求方式|

- API 原型

```
/**
 * 好友请求
 *
 * @param data 好友请求信息（包括对方帐号、好友请求验证类型、附言）
 * @return InvocationFuture 可以设置回调函数。消息发送完成后才会调用，如果出错，会有具体的错误代码。
 */
InvocationFuture<Void> addFriend(AddFriendData data);
```

- 参数说明

|参数|说明|
|:---|:---|
|data|好友请求信息（包括对方帐号、好友请求验证类型、附言）|

- 示例

```java
final VerifyType verifyType = VerifyType.VERIFY_REQUEST; // 发起好友验证请求
String msg = "好友请求附言";
NIMClient.getService(FriendService.class).addFriend(new AddFriendData(account, verifyType, msg))
	.setCallback(new RequestCallback<Void>() { ... });
```

2\. 监听添加好友请求

- API 介绍

添加好友请求发出后，对方会收到一条 `SystemMessage`,可以通过 SystemMessageObserver 的observeReceiveSystemMsg 函数来监听系统通知，通过 getAttachObject 函数可以获取添加好友的通知 `AddFriendNotify`，通知的事件类型见 `AddFriendNotify.Event`。

AddFriendNotify.Event 属性说明：

|AddFriendNotify.Event 属性|说明|
|:---|:---|
|RECV_ADD_FRIEND_DIRECT|对方直接加你为好友|
|RECV_ADD_FRIEND_VERIFY_REQUEST|对方发起好友验证请求|
|RECV_AGREE_ADD_FRIEND|对方同意加你为好友|
|RECV_REJECT_ADD_FRIEND|对方拒绝加你为好友|

- API 原型

```
/**
 * 注册/注销系统消息接收事件观察者
 * @param observer 观察者， 参数为接收到的系统消息
 * @param register true为注册，false为注销
 */
public void observeReceiveSystemMsg(Observer<SystemMessage> observer, boolean register);
```

- 参数说明

SystemMessage 参数说明，请参考系统通知章节。

- 示例

```java
NIMClient.getService(SystemMessageObserver.class).observeReceiveSystemMsg(systemMessageObserver, register);

Observer<SystemMessage> systemMessageObserver = new Observer<SystemMessage>() {
    @Override
    public void onEvent(SystemMessage systemMessage) {
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
    }
};
```
#### <span id="通过/拒绝对方好友请求">通过/拒绝对方好友请求</span>

- API 介绍

收到好友的验证请求的系统通知后，可以通过或者拒绝，对方会收到一条系统通知，通知的事件类型为 AddFriendNotify.Event.RECV_AGREE_ADD_FRIEND 或者 AddFriendNotify.Event.RECV_REJECT_ADD_FRIEND

- API 原型

```
/**
 * 同意/拒绝好友请求
 *
 * @param account 对方帐号
 * @param agree   true表示同意，false表示拒绝
 * @return InvocationFuture 可以设置回调函数。消息发送完成后才会调用，如果出错，会有具体的错误代码。
 */
InvocationFuture<Void> ackAddFriendRequest(String account, boolean agree);
```

- 参数说明

|参数|说明|
|:---|:---|
|account |对方帐号|
|agree   |true 表示同意，false 表示拒绝|

- 示例

```java
// 以通过对方好友请求为例
NIMClient.getService(FriendService.class).ackAddFriendRequest(account, true).setCallback(...);
```

#### <span id="删除好友">删除好友</span>

- API 介绍

删除好友后，将自动解除双方的好友关系，双方的好友列表中均不存在对方。删除好友后，双方依然可以聊天。

- API 原型

```
/**
 * 删除好友
 *
 * @param account 要解除好友关系的帐号
 * @return InvocationFuture 可以设置回调函数。消息发送完成后才会调用，如果出错，会有具体的错误代码。
 */
InvocationFuture<Void> deleteFriend(String account);
```

- 示例

```java
NIMClient.getService(FriendService.class).deleteFriend(account)
	.setCallback(new RequestCallback<Void>() { ... });
```

#### <span id="监听好友关系的变化">监听好友关系的变化</span>

- API 介绍

第三方 APP 应在 APP 启动后监听好友关系的变化，当主动添加好友成功、被添加为好友、主动删除好友成功、被对方解好友关系、好友关系更新、登录同步好友关系数据时都会收到通知。

- API 原型

```
/**
 * 监听好友关系变化通知
 *
 * @param observer 观察者，参数为收到的好友关系变化通知。
 * @param register true为注册监听，false为取消监听
 */
void observeFriendChangedNotify(Observer<FriendChangedNotify> observer, boolean register);
```

- 参数说明

FriendChangedNotify 接口说明：

|返回值|FriendChangedNotify 接口|说明|
|:---|:---|:---|
|List	|getAddedOrUpdatedFriends()|返回增加或者发生变更的好友关系|
|List	|getDeletedFriends()|返回被删除的的好友关系|

Friend 接口说明：

|返回值|Friend 接口|说明|
|:---|:---|:---|
|String	|getAccount()|获取好友帐号|
|String	|getAlias()|获取好友备注名|
|Map	|getExtension()|获取扩展字段|

- 示例

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

#### <span id="获取好友列表">获取好友列表</span>

- API 介绍

该方法是同步方法，返回我的好友帐号集合，可以根据帐号来获取对应的用户资料来构建自己的通讯录,见[构建通讯录](#构建通讯录)。

- API 原型

```
/**
 * 获取我所有的好友帐号
 *
 * @return 好友帐号集合
 */
List<String> getFriendAccounts();
```

- 示例

```java
List<String> friends = NIMClient.getService(FriendService.class).getFriendAccounts();
```

#### <span id="根据用户账号获取好友关系">根据用户账号获取好友关系</span>

- API 介绍

该方法是同步方法。

- API 原型

```
/**
 * 根据用户账号获取好友关系
 *
 * @param account 用户账号
 * @return 该账号对应的好友关系
 */
Friend getFriendByAccount(String account);
```

- 示例

```java
Friend friend = NIMClient.getService(FriendService.class).getFriendByAccount("account");
```

#### <span id="判断用户是否为我的好友">判断用户是否为我的好友</span>

- API 原型

```
/**
 * 是否为我的好友
 *
 * @param account 对方帐号
 * @return 该帐号是否为我的好友
 */
boolean isMyFriend(String account);
```

- 示例

```java
boolean isMyFriend = NIMClient.getService(FriendService.class).isMyFriend(account);
```

#### <span id="更新好友关系">更新好友关系</span>

- API 介绍

目前支持更新好友的备注名和好友关系扩展字段，见 `FriendFieldEnum`。

FriendFieldEnum 属性说明:

|FriendFieldEnum 属性|说明|
|:---|:---|
|ALIAS|备注名|
|EXTENSION|扩展字段|
|undefined|未定义的域|

- API 原型

```
/**
 * 更新好友关系
 *
 * @param friendAccount 待更新的好友账号
 * @param fields        待更新的所有字段集合，目前支持更新备注名和扩展字段
 *                      注意:备注名最长128个字符，扩展字段需要传入Map，key为String，Value为Object，SDK负责转成Json String，最大长度256字符。
 * @return InvocationFuture 可以设置回调函数。消息发送完成后才会调用，如果出错，会有具体的错误代码。
 */
InvocationFuture<Void> updateFriendFields(String friendAccount, Map<FriendFieldEnum, Object> fields);
```

- 参数说明

|参数|说明|
|:---|:---|
|friendAccount |待更新的好友账号|
|fields        |待更新的所有字段集合，目前支持更新备注名和扩展字段。<br>注意:备注名最长 128 个字符，扩展字段需要传入 Map，key 为 String，Value 为 Object，SDK 负责转成Json String，最大长度 256 字符。|

- 示例

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

#### <span id="加入黑名单">加入黑名单</span>

- API 原型

```
/**
 * 添加用户到黑名单
 *
 * @param account 用户帐号
 * @return InvocationFuture 可以设置回调函数。消息发送完成后才会调用，如果出错，会有具体的错误代码。
 */
InvocationFuture<Void> addToBlackList(String account);
```

- 示例

```java
NIMClient.getService(FriendService.class).addToBlackList(account)
    .setCallback(new RequestCallback<Void>() { ... });
```
#### <span id="移出黑名单">移出黑名单</span>

- API 原型

```
/**
 * 把用户从黑名单中移除
 *
 * @param account 用户帐号
 * @return InvocationFuture 可以设置回调函数。消息发送完成后才会调用，如果出错，会有具体的错误代码。
 */
InvocationFuture<Void> removeFromBlackList(String account);
```

- 示例

```java
NIMClient.getService(FriendService.class).removeFromBlackList(user.getAccount())
	.setCallback(new RequestCallback<Void>() { ... });
```

#### <span id="监听黑名单变化">监听黑名单变化</span>

- API 原型

```
/**
 * 监听黑名单变更通知
 *
 * @param observer 观察者，参数为收到的黑名单变化通知。
 * @param register true为注册监听，false为取消监听
 */
void observeBlackListChangedNotify(Observer<BlackListChangedNotify> observer, boolean register);
```

- 参数说明

BlackListChangedNotify 接口说明:

|返回值|BlackListChangedNotify 接口|说明|
|:---|:---|:---|
|List	|getAddedAccounts()|返回被加入到黑名单的用户账号
|List	|getRemovedAccounts()|返回移出黑名单的用户账号|

- 示例

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

#### <span id="获取黑名单列表">获取黑名单列表</span>

- API 原型

```
/**
 * 返回黑名单中的用户列表
 *
 * @return 所有黑名单帐号集合
 */
List<String> getBlackList();
```

- 示例

```java
List<String> accounts = NIMClient.getService(FriendService.class).getBlackList();
```

#### <span id="判断用户是否被拉进黑名单">判断用户是否被拉进黑名单</span>

- API 原型

```
/**
 * 判断用户是否已被拉黑
 *
 * @param account 用戶帐号
 * @return 该用户是否在黑名单列表中
 */
boolean isInBlackList(String account);
```

- 示例

```java
boolean black = NIMClient.getService(FriendService.class).isInBlackList(account);
```

### <span id="消息提醒">消息提醒</span>

网易云通信支持对用户设置或关闭消息提醒（静音），关闭后，收到该用户发来的消息时，不再进行通知栏消息提醒。个人用户的消息提醒设置支持漫游。

#### <span id="设置/关闭消息提醒">设置/关闭消息提醒</span>

- API 原型

```
/**
 * 设置消息提醒/静音
 *
 * @return InvocationFuture 可以设置回调函数。消息发送完成后才会调用，如果出错，会有具体的错误代码。
 */
InvocationFuture<Void> setMessageNotify(String account, boolean notify);
```

- 参数说明

|参数|说明|
|:---|:---|
|account |要设置消息提醒的帐号|
|notify  |是否提醒该用户发来的消息，false 为静音（不提醒）|

- 示例

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
#### <span id="判断用户是否需要消息提醒">判断用户是否需要消息提醒</span>

- API 原型

```
/**
 * 判断用户是否需要消息提醒/静音
 *
 * @param account 用户帐号
 * @return true表示需要消息提醒；false表示静音
 */
boolean isNeedMessageNotify(String account);
```

- 示例

```java
boolean notice = NIMClient.getService(FriendService.class).isNeedMessageNotify(account);
```
#### <span id="获取所有静音帐号（不进行消息提醒的用户）">获取所有静音帐号（不进行消息提醒的用户）</span>

- API 原型

```
/**
 * 获取所有不需要进行消息提醒的账号列表(静音帐号列表）
 *
 * @return 不需要进行消息提醒的帐号集合
 */
List<String> getMuteList();
```

- 示例

```java
List<String> accounts = NIMClient.getService(FriendService.class).getMuteList();
```
#### <span id="监听消息提醒变更通知">监听消息提醒变更通知</span>

- API 介绍

同一个账号在其他端登录时发生了消息提醒变更，会有下面回调通知：

- API 原型

```
/**
 * 监听静音列表变更通知
 *
 * @param observer 观察者，参数为静音列表变更通知。
 * @param register true为注册监听，false为取消监听
 */
void observeMuteListChangedNotify(Observer<MuteListChangedNotify> observer, boolean register);
```

- 参数说明

MuteListChangedNotify 接口说明:

|返回值|MuteListChangedNotify 接口|说明|
|:---|:---|:---|
|String	|getAccount()|静音发生变化的用户|
|boolean	|isMute()|该用户是否被静音|

- 示例

```java
NIMClient.getService(FriendServiceObserve.class).observeMuteListChangedNotify(
    new Observer<MuteListChangedNotify>() {
        @Override
        public void onEvent(MuteListChangedNotify notify) {
            // notify.isMute() 是否被静音
        }
    }, register);
```

## <span id="最近会话">最近会话</span>

最近会话列表由 SDK 维护并提供查询、监听变化的接口，只要与某个用户或者群组有产生聊天（自己发送消息或者收到消息）， SDK 会自动更新最近会话列表并通知上层，开发者无需手动更新。

最近会话 `RecentContact` ，也可称作会话列表或者最近联系人列表，它记录了与用户最近有过会话的联系人信息，包括联系人帐号、联系人类型、最近一条消息的时间、消息状态、消息内容、未读条数等信息。

RecentContact 接口说明：

|返回值|RecentContact 接口|说明|
|:---|:---|:---|
|MsgAttachment	|getAttachment()|如果最近一条消息是扩展消息类型，获取消息的附件内容|
|String	|getContactId()|获取最近联系人的 ID（好友帐号，群 ID 等）|
|String	|getContent()|获取最近一条消息的缩略内容|
|Map	|getExtension()|获取扩展字段|
|String	|getFromAccount()|获取与该联系人的最后一条消息的发送方的帐号|
|String	|getFromNick()|获取与该联系人的最后一条消息的发送方的昵称|
|MsgStatusEnum	|getMsgStatus()|获取最近一条消息状态|
|MsgTypeEnum	|getMsgType()|获取最近一条消息的消息类型|
|String	|getRecentMessageId()|最近一条消息的消息 ID， 即 IMMessage#getUuid() |
|SessionTypeEnum	|getSessionType()|获取会话类型|
|long	|getTag()|获取标签|
|long	|getTime()|获取最近一条消息的时间，单位为 ms|
|int	|getUnreadCount()|获取该联系人的未读消息条数|
|void	|setExtension(Map extension)|设置扩展字段，可用于做群@等扩展用途|
|void	|setMsgStatus(MsgStatusEnum msgStatus)|设置最近一条消息的状态|
|void	|setTag(long tag)|设置一个标签，用于做联系人置顶、最近会话列表排序等扩展用途。|

某些场景下，开发者可能需要手动向最近会话列表中插入一条会话项（即插入一个最近联系人），例如：在创建完高级群时，需要在最近会话列表中显示该群的会话项。由创建高级群完成时并不会收到任何消息， SDK 并不会立即更新最近会话，此时要满足需求，可以在创建群成功的回调中，插入一条本地消息， 即调用 MsgService#saveMessageToLocal。

> 注意：最近会话是本地的，不会漫游。漫游与消息相关，与最近会话无关。

### <span id="获取最近会话列表">获取最近会话列表</span>

- API 原型

异步版本：

```
/**
 * 查询最近联系人列表数据
 *
 * @return InvocationFuture
 */
public InvocationFuture<List<RecentContact>> queryRecentContacts();

```

同步版本：

```
/**
 * 查询最近联系人列表数据(同步版本)
 *
 * @return InvocationFuture
 */
public List<RecentContact> queryRecentContactsBlock();
```

- 示例

```java
 NIMClient.getService(MsgService.class).queryRecentContacts()
	 .setCallback(new RequestCallbackWrapper<List<RecentContact>>() {
	   @Override
       public void onResult(int code, List<RecentContact> recents, Throwable e) {
            // recents参数即为最近联系人列表（最近会话列表）
       }
    });
```

### <span id="监听最近会话变更">监听最近会话变更</span>

- API 介绍

在收发消息的同时，SDK 会更新对应聊天对象的最近联系人资料。当有消息收发时，SDK 会发出最近联系人更新通知。

- API 原型

```
/**
 * 注册/注销最近联系人列表变化观察者
 * @param observer 观察者，参数为变化的最近联系人列表
 * @param register true为注册，false为注销
 */
public void observeRecentContact(Observer<List<RecentContact>> observer, boolean register);
```

- 示例

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

### <span id="获取会话未读数总数">获取会话未读数总数</span>

1\. 通过接口直接获取

- API 原型

```
/**
 * 获取未读数总数
 */
public int getTotalUnreadCount();
```

- 示例

```java
int unreadNum = NIMClient.getService(MsgService.class).getTotalUnreadCount();
```

2\. 对最近联系人列表中的每个最近联系人的未读数进行求和：

- 示例

```java
int unreadNum = 0;
for (RecentContact r : items) {
    unreadNum += r.getUnreadCount();
}
```

> 说明：多端同时登录时，在其他端进行查看，客户端不会进行未读数清零操作。

### <span id="设置当前会话">设置当前会话</span>

- API 介绍

如果用户在开始聊天时，开发者调用了 `setChattingAccount` 接口，SDK会自动管理消息的未读数。当收到新消息时，自动将未读数清零。如果第三方 APP 需要不进入聊天窗口，就需要将未读数清零，可以通过调用 MsgService#clearUnreadCount 接口来实现。

- API 原型

```
/**
 * 将指定最近联系人的未读数清零(标记已读)。
 * 调用该接口后，会触发{@link MsgServiceObserve#observeRecentContact(Observer, boolean)} 通知
 *
 * @param account     聊天对象帐号
 * @param sessionType 会话类型
 */
public void clearUnreadCount(String account, SessionTypeEnum sessionType);
```

- 参数说明

|参数|说明|
|:---|:---|
|account     |聊天对象帐号|
|sessionType |会话类型|

- 示例

```java
// 触发 MsgServiceObserve#observeRecentContact(Observer, boolean) 通知，
// 通知中的 RecentContact 对象的未读数为0
NIMClient.getService(MsgService.class).clearUnreadCount(account, sessionType);
```

如果需要在最近联系人列表界面显示当前消息状态，还需要增加消息状态监听，操作见[消息收发](#消息收发) 章节的子章节多媒体消息接收中的监听消息状态。

### <span id="移除最近会话列表中的项">移除最近会话列表中的项</span>

- API 介绍

MsgService 提供了两种方法： deleteRecentContact 和 deleteRecentContact2，区别在于后者会触发 MsgServiceObserve#observeRecentContactDeleted 通知。

- API 原型

不触发观察者通知版本:

```
/**
 * 从最近联系人列表中删除一项。
 * 调用这个接口删除数据后，不会引发观察者通知。
 *
 * @param recent 待删除的最近联系人项
 */
public void deleteRecentContact(RecentContact recent);
```

触发观察者通知版本:

```
/**
 * 删除最近联系人记录。
 * 调用该接口后，会触发{@link MsgServiceObserve#observeRecentContactDeleted(Observer, boolean)}通知
 *
 * @param account
 * @param sessionType
 */
public void deleteRecentContact2(String account, SessionTypeEnum sessionType);
```

- 示例

不触发观察者通知版本：

```java
NIMClient.getService(MsgService.class).deleteRecentContact(recent);
```

触发观察者通知版本：

```
// 以P2P类型为例
NIMClient.getService(MsgService.class).deleteRecentContact2(account, SessionTypeEnum.P2P);
```

### <span id="删除指定最近联系人的漫游消息">删除指定最近联系人的漫游消息</span>

- API 介绍

不删除本地消息，但如果在其他端登录，当前时间点该会话已经产生的消息不漫游。

- API 原型

```
/**
 * 删除指定最近联系人的漫游消息。
 * 不删除本地消息，但如果在其他端登录，当前时间点该会话已经产生的消息不漫游。
 *
 * @return InvocationFuture 可设置回调函数，监听删除结果。
 */
public InvocationFuture<Void> deleteRoamingRecentContact(String contactId, SessionTypeEnum sessionTypeEnum);
```

- 参数说明

|参数|说明|
|:---|:---|
|contactId         |最近联系人的 ID（好友帐号，群 ID 等）|
|sessionTypeEnum   |会话类型|

- 示例

```java
NIMClient.getService(MsgService.class)
	.deleteRoamingRecentContact(contactId, sessionTypeEnum)
	.setCallback(new RequestCallback<Void>() { ... });
```

### <span id="最近会话自定义方案">最近会话自定义方案</span>

最近会话自定义 在某些特殊场景下，用户有对最近会话有较强的定制需求，本质上，网易云通信SDK就是提供数据及数据的管理，如果上层业务有自定义的需要，那么要做组合使用（抽象+组合）。
[网易云通信Android最近联系人列表如何自定义 ](http://bbs.netease.im/read-tid-235)


## <span id="历史记录">历史记录</span>

### <span id="本地记录">本地记录</span>

#### <span id="查询消息历史">查询消息历史</span>

- API 介绍

SDK 提供了一个根据锚点查询本地消息历史记录的接口，根据提供的方向 (direct)，查询比 anchor 更老 (QUERY\_OLD) 或更新 (QUERY\_NEW) 的最靠近anchor 的 limit 条数据。调用者可使用 asc 参数指定结果排序规则，结果使用 time 作为排序字段。

当进行首次查询时，锚点可以用使用 `MessageBuilder#createEmptyMessage` 接口生成。查询结果不包含锚点。

- API 原型

```
/**
 * 根据锚点和方向从本地消息数据库中查询消息历史。
 * 根据提供的方向(direct)，查询比anchor更老（QUERY_OLD）或更新（QUERY_NEW）的最靠近anchor的limit条数据。
 * 调用者可使用asc参数指定结果排序规则，结果使用time作为排序字段。
 * 注意：查询结果不包含锚点。
 *
 * @return 调用跟踪，可设置回调函数，接收查询结果
 */
public InvocationFuture<List<IMMessage>> queryMessageListEx(IMMessage anchor, QueryDirectionEnum direction, int limit, boolean asc);
```

- 参数说明

|参数|说明|
|:---|:---|
|anchor    |查询锚点|
|direction |查询方向|
|limit     |查询结果的条数限制|
|asc       |查询结果的排序规则，如果为 true，结果按照时间升序排列，如果为 false，按照时间降序排列|

QueryDirectionEnum 属性说明:

|QueryDirectionEnum 属性|说明|
|:---|:---|
|QUERY_NEW|查询比锚点时间更晚的消息|
|QUERY_OLD|查询比锚点时间更早的消息|

- 示例

```java
// 查询比 anchor时间更早的消息，查询20条，结果按照时间降序排列
NIMClient.getService(MsgService.class).queryMessageListEx(anchor, QueryDirectionEnum.QUERY_OLD,
    20, false).setCallback(new RequestCallbackWrapper<List<IMMessage>>() {
@Override
public void onResult(int code, List<IMMessage> result, Throwable exception) {
   ...
}
});
```

#### <span id="包含起止时间的消息历史查询">包含起止时间的消息历史查询</span>

- API 介绍

该接口使用方法与 `queryMessageListEx` 类似，增加了一个参数 toTime 作为查询方向上的的截止时间，当方向为 QUERY_OLD 时，toTime 应小于 anchor.getTime()；当方向为 QUERY_NEW 时，toTime 应大于 anchor.getTime()。

查询结果的范围由 toTime 和 limit 共同决定，以先到为准。如果从 anchor 到 toTime 之间消息大于 limit 条，返回 limit 条记录，如果小于 limit 条，返回实际条数。

- API 原型

```
/**
 * 根据起始、截止时间点以及查询方向从本地消息数据库中查询消息历史。
 * 根据提供的方向 (direction)，以 anchor 为起始点，往前或往后查询由 anchor 到 toTime 之间靠近 anchor 的 limit 条消息。
 * 查询范围由 toTime 和 limit 共同决定，以先到为准。如果到 toTime 之间消息大于 limit 条，返回 limit 条记录，如果小于 limit 条，返回实际条数。
 * 查询结果排序规则：direction 为 QUERY_OLD 时 按时间降序排列，direction 为 QUERY_NEW 时按照时间升序排列。
 * 注意：查询结果不包含锚点。
 *
 * @return 调用跟踪，可设置回调函数，接收查询结果
 */
public InvocationFuture<List<IMMessage>> queryMessageListExTime(IMMessage anchor, long toTime, QueryDirectionEnum direction, int limit);
```

- 参数说明

|参数|说明|
|:---|:---|
|anchor |查询锚点|
|toTime |查询截止时间，若方向为 QUERY_OLD，toTime 应小于 anchor.getTime()。<br>方向为 QUERY_NEW，toTime 应大于 anchor.getTime()|
|direction |查询方向|
|limit |查询结果的条数限制|

- 示例

```
// 以P2P类型为例，testAccount为测试帐号
IMMessage msg = MessageBuilder.createEmptyMessage("testAccount", SessionTypeEnum.P2P, System.currentTimeMillis());

// 查询anchor往前10s的数据，10条
NIMClient.getService(MsgService.class).queryMessageListExTime(msg, System.currentTimeMillis() - 10000, QueryDirectionEnum.QUERY_OLD, 10).setCallback(...);
```

#### <span id="根据消息 uuid 查询消息">根据消息 uuid 查询消息</span>

- API 原型

异步版本：

```
/**
 * 通过uuid批量获取IMMessage
 *
 * @param uuids 消息的uuid列表
 * @return
 */
public InvocationFuture<List<IMMessage>> queryMessageListByUuid(List<String> uuids);
```

同步版本：

```
/**
 * 通过uuid批量获取IMMessage(同步版本)
 *
 * @param uuids 消息的uuid列表
 * @return
 */
public List<IMMessage> queryMessageListByUuidBlock(List<String> uuids);
```

- 示例

```java
// 通过uuid批量获取IMMessage(异步版本)
List<String> uuids = new ArrayList<>();
uuids.add(message.getUuid());
NIMClient.getService(MsgService.class).queryMessageListByUuid(uuids);

// 通过uuid批量获取IMMessage(同步版本)
NIMClient.getService(MsgService.class).queryMessageListByUuidBlock(uuids);
```

#### <span id="根据消息类型查询消息历史">根据消息类型查询消息历史</span>

- API 介绍

通过消息类型从本地消息数据库中查询消息历史。查询范围由 msgTypeEnum 参数和 anchor 的 sessionId 决定。该接口查询方向为从后往前。以锚点 anchor 作为起始点（不包含锚点），往前查询最多 limit 条消息。

- API 原型

```
/**
 * 通过消息类型从本地消息数据库中查询消息历史。查询范围由msgTypeEnum参数和anchor的sessionId决定
 * 该接口查询方向为从后往前。以锚点anchor作为起始点（不包含锚点），往前查询最多limit条消息。
 *
 */
public InvocationFuture<List<IMMessage>> queryMessageListByType(MsgTypeEnum msgTypeEnum, IMMessage anchor, int limit);
```

- 参数说明

|参数|说明|
|:---|:---|
|msgTypeEnum |消息类型|
|anchor      |搜索的消息锚点|
|limit       |搜索结果的条数限制|

- 示例

```java
NIMClient.getService(MsgService.class).queryMessageListByType(msgTypeEnum, anchor, limit);
```

#### <span id="插入本地消息">插入本地消息</span>

保存消息到本地数据库，但不发送到服务器端。有两种接口可供调用。

1\. 不可设置保存时间

- API 原型

```
/**
 * 保存消息到本地数据库，但不发送到服务器端。用于第三方APP保存本地提醒一类的消息。
 * 该接口将消息保存到数据库后，如果需要通知到UI，可将notify设置为true，此时会触发{@link MsgServiceObserve#observeReceiveMessage(Observer, boolean)}通知。
 *
 * @param msg    待保存的消息对象
 * @param notify 是否要通知UI更新界面
 * @return InvocationFuture 可以设置回调函数。在消息存入数据库后,就会回调。
 */
public InvocationFuture<Void> saveMessageToLocal(IMMessage msg, boolean notify);
```

- 参数说明

|参数|说明|
|:---|:---|
|msg    |待保存的消息对象|
|notify |是否要通知UI更新界面|

- 示例

```
// 向群里插入一条Tip消息，使得该群能立即出现在最近联系人列表（会话列表）中，满足部分开发者需求。
Map<String, Object> content = new HashMap<>(1);
content.put("content", "成功创建高级群");
// 创建tip消息，teamId需要开发者已经存在的team的teamId
IMMessage msg = MessageBuilder.createTipMessage(teamId, SessionTypeEnum.Team);
msg.setRemoteExtension(content);
// 自定义消息配置选项
CustomMessageConfig config = new CustomMessageConfig();
// 消息不计入未读
config.enableUnreadCount = false;
msg.setConfig(config);
// 消息发送状态设置为success
msg.setStatus(MsgStatusEnum.success);
// 保存消息到本地数据库，但不发送到服务器
NIMClient.getService(MsgService.class).saveMessageToLocal(msg, true);
```

2\. 可设置保存时间

- API 原型

```
/**
 * 保存消息到本地数据库，但不发送到服务器端。可设置保存消息的时间
 * 用于第三方APP保存本地提醒一类的消息。
 * 该接口将消息保存到数据库后，如果需要通知到UI，可将notify设置为true，此时会触发{@link MsgServiceObserve#observeReceiveMessage(Observer, boolean)}通知。
 *
 * @param msg       待保存的消息对象
 * @param notify    是否要通知UI更新界面
 * @param time      保存消息的时间, 使用{@link IMMessage#getTime()}
 * @return InvocationFuture 可以设置回调函数。在消息存入数据库后,就会回调。
 */
public InvocationFuture<Void> saveMessageToLocalEx(IMMessage msg, boolean notify, long time);
```

- 参数说明

|参数|说明|
|:---|:---|
|msg       |待保存的消息对象|
|notify    |是否要通知UI更新界面|
|time      |保存消息的时间, 使用 IMMessage#getTime() |

- 示例

```
// 创建一条tip消息，并保存到本地数据库，时间设置为当前时间10秒之前
IMMessage message = MessageBuilder.createTipMessage("testAccount", SessionTypeEnum.P2P);
message.setContent("保存一条本地消息");
message.setStatus(MsgStatusEnum.success);
long time = System.currentTimeMillis() - 10000;
NIMClient.getService(MsgService.class).saveMessageToLocalEx(message, true, time);
```

### <span id="云端记录">云端记录</span>

1\. 获取云端记录详细接口

- API 介绍

网易云通信还提供云端消息记录的功能，根据应用选择的套餐类型，保存不同时间长度范围的云端消息记录。SDK 提供了查询云端消息历史的接口，查询以锚点 anchor 作为起始点（不包含锚点），根据 direction 参数，往前或往后查询由锚点到 toTime 之间的最多 limit 条消息。查询范围由 toTime 和 limit 共同决定，以先到为准。如果到 toTime 之间消息大于 limit 条，返回 limit 条记录，如果小于 limit 条，返回实际条数。当已经查询到头时，返回的结果列表的 size 可能会比 limit 小。当进行首次查询时，锚点可以用使用 `MessageBuilder#createEmptyMessage` 接口生成。查询结果不包含锚点。该接口的最后一个参数可用于控制是否要将拉取到的消息记录保存到本地，如果选择保存到了本地，在使用拉取本地消息记录的接口时，也能取到这些数据。

- API 原型

```
 /**
 * 从服务器拉取消息历史记录。以锚点anchor作为起始点（不包含锚点），根据direction参数，往前或往后查询由锚点到toTime之间的最多limit条消息。
 * 查询范围由toTime和limit共同决定，以先到为准。如果到toTime之间消息大于limit条，返回limit条记录，如果小于limit条，返回实际条数。
 * 当已经查询到头时，返回的结果列表的size可能会比limit小
 *
 * @param anchor    起始时间点的消息，不能为null。
 *                  设置自定义锚点时，使用 {@link MessageBuilder#createEmptyMessage(String, SessionTypeEnum, long)} 创建一个空对象即可
 * @return InvocationFuture
 */
public InvocationFuture<List<IMMessage>> pullMessageHistoryEx(IMMessage anchor, long toTime, int limit, QueryDirectionEnum direction, boolean persist);

```

- 参数说明

|参数|说明|
|:---|:---|
|anchor    |起始时间点的消息，不能为 null。<br>设置自定义锚点时，使用  MessageBuilder#createEmptyMessage 创建一个空对象即可|
|toTime    |结束时间点单位毫秒|
|limit     |本次查询的消息条数上限(最多 100 条)|
|direction |查询方向，QUERY_OLD 按结束时间点逆序查询，逆序排列；<br>QUERY_NEW 按起始时间点正序起查，正序排列|
|persist   |通过该接口获取的漫游消息记录，要不要保存到本地消息数据库。|

- 示例

```java
// 服务端拉取历史消息100条
long newTime = System.currentTimeMillis();
long oldTime = newTime - 1000 * 60 * 60; // 一小时前
IMMessage anchor = MessageBuilder.createEmptyMessage("testAccount", SessionTypeEnum.P2P, newTime);
NIMClient.getService(MsgService.class).pullMessageHistoryEx(anchor, oldTime, 100, QueryDirectionEnum.QUERY_OLD, false);
```

2\. 获取云端记录简单接口

- API 介绍

上面是一个全接口，参数比较复杂。如果只需要类似于在聊天窗口下拉刷新的效果，可以使用下面这个简单版本。

- API 原型

```

/**
 * 从服务器拉取消息历史记录。该接口查询方向为从后往前。以锚点anchor作为起始点（不包含锚点），往前查询最多limit条消息。
 * 当已经查询到头时，返回的结果列表的size可能会比limit小
 *
 * @param anchor  查询锚点。
 *                首次查询，使用 {@link MessageBuilder#createEmptyMessage(String, SessionTypeEnum, long)} 创建一个空对象即可
 * @param limit   本次查询的消息条数上限(最多100条)
 * @param persist 通过该接口获取的漫游消息记录，要不要保存到本地消息数据库。
 * @return InvocationFuture
 */
public InvocationFuture<List<IMMessage>> pullMessageHistory(IMMessage anchor, int limit, boolean persist);
```

- 参数说明

|参数|说明|
|:---|:---|
|anchor  |查询锚点。<br>首次查询，使用 MessageBuilder#createEmptyMessage 创建一个空对象即可|
|limit   |本次查询的消息条数上限(最多 100 条)|
|persist |通过该接口获取的漫游消息记录，要不要保存到本地消息数据库。|

- 示例

```java
// 以测试帐号 testAccount，查询100条消息
IMMessage anchor = MessageBuilder.createEmptyMessage("testAccount", SessionTypeEnum.P2P, System.currentTimeMillis());

NIMClient.getService(MsgService.class).pullMessageHistory(anchor, 100, false);
```

### <span id="本地消息搜索">本地消息搜索</span>

#### <span id="根据关键字搜索消息历史">根据关键字搜索消息历史</span>

- API 介绍

SDK 提供了按照关键字搜索聊天记录的功能，可以对指定的聊天对象，输入关键字进行消息内容搜索。查询方向为从后往前。以锚点 anchor 作为起始点开始查询，返回最多 limit 条匹配 keyword 的记录。该接口目前仅搜索文本类型的消息，匹配规则为文本内容包含 keyword，目前仅支持精确匹配，不支持模糊匹配和拼音匹配。

由于 SDK 并不存储用户数据，因此 keyword 不会匹配用户资料。如果调用者希望查询指定用户的说话记录，可提供 fromAccounts 参数。如果提供的 fromAccounts 参数不为空，那么 anchor 对应的会话 (由 sessionId 和 sessionType 决定) 的消息记录中，凡是消息说话者在 fromAccounts 列表中的记录，也会被当做匹配结果，加入搜索结果中。

- API 原型

```
/**
 * 从本地消息数据库搜索消息历史。查询范围由anchor的sessionId和sessionType决定。
 * 该接口查询方向为从后往前。以锚点anchor作为起始点开始查询，返回最多limit条匹配key的记录。
 * 该接口目前仅搜索文本类型的消息，匹配规则为文本内容包含keyword，仅支持精确匹配，不支持模糊匹配和拼音匹配。
 * 由于sdk并不存储用户数据，因此keyword不会匹配用户资料。如果调用者希望查询指定用户的说话记录，可提供fromAccounts参数。
 * 如果提供的fromAccounts参数不为空，那么anchor对应的会话（sessionId和sessionType）的消息记录中，
 * 凡是消息说话者在fromAccounts列表中的记录，也会被当做匹配结果，加入搜索结果中。
 * 注意：搜索结果不包含anchor
 *
 * @return InvocationFuture
 */
public InvocationFuture<List<IMMessage>> searchMessageHistory(String keyword, List<String> fromAccounts, IMMessage anchor, int limit);
```

- 参数说明

|参数|说明|
|:---|:---|
|keyword      |文本消息的搜索关键字|
|fromAccounts |消息说话者帐号列表，如果消息说话在该列表中，那么无需匹配 keyword，<br>对应的消息记录会直接加入搜索结果集中。|
|anchor       |搜索的消息锚点|
|limit        |搜索结果的条数限制|

- 示例

```java
// 搜索『关键字』20条
NIMClient.getService(MsgService.class).searchMessageHistory("关键字", fromAccounts, anchor, 20)
	 .setCallback(new RequestCallbackWrapper<List<IMMessage>>(){ ... });
```

#### <span id="根据时间点搜索消息历史">根据时间点搜索消息历史</span>

- API 介绍

SDK 还提供了按照关键字全局搜索聊天记录的接口：MsgService#searchAllMessageHistory，用法与上述类似。目前暂不提供索引方式的全文检索功能，该全局搜索接口需要开发者评估大数据情况下的性能开销。

根据时间点搜索消息历史。该接口查询方向以某个时间点为基准从后往前，返回最多 limit 条匹配 key 的记录。该接口目前仅搜索文本类型的消息，匹配规则为文本内容包含 keyword，仅支持精确匹配，不支持模糊匹配和拼音匹配。 由于 sdk 并不存储用户数据，因此 keyword 不会匹配用户资料。如果调用者希望查询指定用户的说话记录，可提供 fromAccounts 参数。如果提供的 fromAccounts 参数不为空，那么凡是消息说话者在 fromAccounts 列表中的记录，也会被当做匹配结果，加入搜索结果中。

- API 原型

```
/**
 * 从本地消息数据库搜索消息历史。
 * 该接口查询方向以某个时间点为基准从后往前，返回最多limit条匹配key的记录。
 * 该接口目前仅搜索文本类型的消息，匹配规则为文本内容包含keyword，仅支持精确匹配，不支持模糊匹配和拼音匹配。
 * 由于sdk并不存储用户数据，因此keyword不会匹配用户资料。如果调用者希望查询指定用户的说话记录，可提供fromAccounts参数。
 * 如果提供的fromAccounts参数不为空，那么凡是消息说话者在fromAccounts列表中的记录，也会被当做匹配结果，加入搜索结果中。
 *
 * @return InvocationFuture
 */
public InvocationFuture<List<IMMessage>> searchAllMessageHistory(String keyword, List<String> fromAccounts, long time, int limit);
```

- 参数说明

|参数|说明|
|:---|:---|
|keyword      |文本消息的搜索关键字|
|fromAccounts |消息说话者帐号列表，如果消息说话在该列表中，那么无需匹配 keyword，<br>对应的消息记录会直接加入搜索结果集中。|
|time         |查询范围时间点，比 time 小（从后往前查）|
|limit        |搜索结果的条数限制|

- 示例

```java
NIMClient.getService(MsgService.class).searchAllMessageHistory(keyword, fromAccounts, time, limit)
	.setCallback(new RequestCallbackWrapper<List<IMMessage>>(){ ... });
```

### <span id="删除消息记录">删除消息记录</span>

#### <span id="删除单条消息记录">删除单条消息记录</span>

- API 原型

```
/**
 * 删除一条消息记录
 *
 * @param message 待删除的消息记录
 */
public void deleteChattingHistory(IMMessage message);
```

- 示例

```
// 删除单条消息
NIMClient.getService(MsgService.class).deleteChattingHistory(message);
```

#### <span id="删除与某个对象的全部消息记录">删除与某个对象的全部消息记录</span>

- API 原型

```
/**
 * 清除与指定用户的所有消息记录
 *
 * @param account     用户帐号
 * @param sessionType 聊天类型
 */
public void clearChattingHistory(String account, SessionTypeEnum sessionType);
```

- 示例

```java
// 删除与某个聊天对象的全部消息记录
NIMClient.getService(MsgService.class).clearChattingHistory(account, sessionType);
```

> 注意：当调用 MsgService#clearChattingHistory 接口删除与某个对象的全部聊天记录后，最近会话列表（最近联系人列表）中对应的项不会被移除，但会清空最近的消息内容，包括消息的时间显示（但 RecentContact 的 tag, extension 字段作为开发者的扩展字段，不会被清除。开发者可使用 tag, extension 进行最近会话列表显示定制）。如果需要移除最近会话项，请调用 MsgService#deleteRecentContact 接口。


## <span id="系统通知">系统通知</span>

系统通知是网易云通信系统内建的消息/通知，其对应的数据结构为 `SystemMessage`。由网易云通信服务器推送给用户的通知类消息，用于网易云通信系统类的事件通知。现在主要包括群变动的相关通知，例如入群申请，入群邀请等，如果第三方应用还托管了好友关系，好友的添加、删除也是这个类型的通知。系统通知由 SDK 负责接收和存储，并提供较简单的未读数管理。

### <span id="内置系统通知">内置系统通知</span>

#### <span id="监听系统通知">监听系统通知</span>

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

#### <span id="管理系统通知">管理系统通知</span>

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
    .querySystemMessageByTypeBlock(types, loadOffset, LOAD_MESSAGE_COUNT);
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
NIMClient.getService(SystemMessageService.class).resetSystemMessageUnreadCountByType(types);
```

### <span id="自定义系统通知">自定义系统通知</span>

系统通知属于网易云通信的体系内，如果第三方 APP 需要自己的系统通知，可使用自定义通知，其数据结构为 `CustomNotification`。

自定义通知提供的灵活性包括：
- 消息格式由第三方 APP 自己定义，只要内容是 `String` 就可以了。
- 第三方 APP 的客户端和服务器均可以发送自定义通知。
- 接收对象可以是个人，也可以是群组。
- 可设置通知的到达级别：保证必达，或是通知接收者只有当前在线才能收到。
- 如果需要向 iOS 用户推送，可自定义 iOS 的推送内容，可以自定义推送属性。

注意：自定义通知和自定义消息的不同之处在于，自定义消息归属于网易云通信的消息体系内，适用于会话中，由 SDK 存储在消息数据库中，与网易云通信的其他内建消息类型一同展现给用户。而自定义通知主要用于第三方的一些事件状态通知，网易云通信不存储，也不解释这些通知，网易云通信仅仅负责替第三方传递和通知这些事件，起到透传的作用。

#### <span id="发送自定义通知">发送自定义通知</span>

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

#### <span id="接收自定义通知">接收自定义通知</span>

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

如果使用广播接收者的方式(Android 未来版本会禁止程序后台接收隐式广播，因此不建议开发者使用该方式)，首先需要在 AndroidManifest.xml 文件中声明一个接收器：

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


## <span id="消息提醒">消息提醒</span>

集成网易云通信 Android SDK 的 APP 运行起来时，会有个后台进程（push 进程），该进程保持了与网易云通信 Server 的长连接。只要这个 push 进程活着（网易云通信提供安卓保活机制），就能接收网易云通信 Server 推过来的消息，进行通知栏提醒。

### <span id="消息提醒场景">消息提醒场景</span>

- 需要消息提醒的场景：

1\. APP 在后台时。

2\. 在前台与 A 聊天但收到非 A 的消息时（与 iOS 不一样）。

3\. 在非聊天界面且非最近会话界面时。

- 不需要消息提醒的场景：

1\. 如果用户正在与某一个人聊天，当这个人的消息到达时，是不应该有通知栏提醒的。

2\. 如果用户停留在最近联系人列表界面，收到消息也不应该有通知栏提醒（但会有未读数变更通知）。

网易云通信 SDK 提供内置的消息提醒功能，如需使用，开发者需要在进出聊天界面以及最近联系人列表界面时，通知 SDK。相关接口如下：

```java
// 进入聊天界面，建议放在onResume中
NIMClient.getService(MsgService.class).setChattingAccount(account， sessionType);

// 进入最近联系人列表界面，建议放在onResume中
NIMClient.getService(MsgService.class).setChattingAccount(MsgService.MSG_CHATTING_ACCOUNT_ALL, SessionTypeEnum.None);

// 退出聊天界面或离开最近联系人列表界面，建议放在onPause中
NIMClient.getService(MsgService.class).setChattingAccount(MsgService.MSG_CHATTING_ACCOUNT_NONE, SessionTypeEnum.None);
```

### <span id="内置消息提醒定制">内置消息提醒定制</span>

网易云通信 SDK 提供内置的消息提醒（通知栏提醒）功能，并提供以下四个维度的定制，开启/关闭内置的消息提醒、更新消息提醒配置接口如下：

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
* 通知栏样式展示样式配置（折叠、展开）（3.3版本新增）

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

提示消息：{说话者}: 提示消息

自定义消息：{说话者}: 自定义消息

除文本消息外，开发者可以通过  `NimStrings` 类修改这些默认提醒内容。

#### 接收消息时定制通知栏的头像

网易云通信支持定制通知栏显示的头像(用户头像、群头像)，在 UserInfoProvider 接口下提供方法：

```java
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
实现上述需要的方法，在 `SDKOptions` 中配置 `UserInfoProvider` 实例，在 SDK 初始化时传入 `SDKOptions` 方可生效。

需要注意的是，上述返回头像 Bitmap 的函数，请尽可能从内存缓存里拿头像，如果读取本地头像可能导致 UI 进程阻塞，从而导致通知栏提醒延时弹出。

#### 发送消息时指定消息提醒

发送消息时可以设置消息配置选项 `CustomMessageConfig`，可以设定是否需要推送，是否需要计入未读数等。

`enablePush` ： 该消息是否进行推送（消息提醒）。默认为 true 。

`enableUnreadCount` ：该消息是否要计入未读数，如果为 true ，那么对方收到消息后，最近联系人列表中未读数加1。默认为 true 。

### <span id="自行实现消息提醒">自行实现消息提醒</span>

如果 SDK 内建的消息提醒不能满足你的需求，你可以关闭 SDK 内置的消息提醒，自行实现。添加消息接收观察者，收到新消息时，在观察者的 `onEvent` 中实现状态栏提醒。注册注销方式详见[接收消息](#接收消息)一节。注意：只有 SDK 1.4.0 及以上版本才能使用该方式，1.4.0 以下的版本使用此方式有可能会漏掉通知。

### <span id="桌面端在线配置推送">桌面端在线配置推送</span>

支持配置桌面端（PC/Mac/Web）在线时，移动端是否需要推送。

```java
/**
* 设置桌面端(PC/Mac/WEB)在线时，移动端是否需要推送
* @param isOpen true 桌面端在线时移动端不需推送；false 桌面端在线时移动端需推送
* @return InvocationFuture 可以设置回调函数。成功会返回成功信息，错误会返回相应的错误码。
*/
NIMClient.getService(SettingsService.class)
	.updateMultiportPushConfig(isOpen)
	.setCallback(new RequestCallback<Void>() { ... });
```

支持查询当前配置的推送状态。true 桌面端在线时移动端不需推送；false 桌面端在线时移动端需推送

```java
NIMClient.getService(SettingsService.class).isMultiportPushOpen()
```

## <span id="推送">推送</span>

为了提高消息送达率，网易云通信在 Android 进程保活上做了很多努力，但是在国内 Android 系统的大环境下，厂家的深度定制 ROM 对系统做了越来越严格的限制，想要在所有机型上做到永久保活是不可能实现的。因此，网易云通信 SDK 3.2.0 引进第三方消息推送来增加消息送达率。

鉴于谷歌 GCM 服务无法在国内使用，某些第三方推送平台采用的相互唤醒策略容易被 厂商ROM 或者 XX管家 或者 XX卫士 禁止，小米、华为等推送等采用的系统级长连接则具有明显的优势。统计表明在小米系统上小米推送到达率可以达到 90% 以上。

网易云通信 SDK 在提供端内推送外，还提供端外推送来解决进程在厂商限制下可能无法保活的问题，网易云通信SDK 3.2.0 支持的第三方推送有小米推送，4.0.0 版本新加入了华为推送。网易云通信已经打通推送通道，在网易云通信SDK基础上，开发者可快速接入第三方推送，在支持的设备上，网易云通信 SDK 进程与服务器连接断开之后，联系人发来的消息将通过第三方推送平台推送给用户，从而提高消息达到率。

### <span id="小米推送">小米推送</span>

集成小米推送需要以下几个步骤：

#### 1. 准备工作

- 前往[小米推送官网](http://dev.xiaomi.com/doc/?page_id=1670)注册账号并通过认证，创建应用并获取AppID、AppKey、AppSecret。

- 下载小米推送SDK，将jar包导入到工程中。（网易云通信IM SDK 中并不包含小米推送 SDK，开发者需要自行下载并导入到工程中）

- 前往网易云通信后台管理中心，创建小米推送证书。首先选择需要接入推送的应用，点击证书管理：

![](http://yx-web.nos.netease.com/webdoc/im/android/certificates.jpg)

在Android推送证书一栏选择添加证书，随后在如下图所示的弹出框中填写对应的信息，其中应用包名填写开发者 Android 应用的包名，AppSecret 填写第一步在小米官网创建应用所获取的 AppSecret，请开发者谨记证书名，这个会在 SDK初始化推送的时候使用。

![](http://yx-web.nos.netease.com/webdoc/im/android/xiaomi.jpg)

#### 2. 接入流程

快速接入只需要2步，详细可参考[小米推送Android SDK使用指南](http://dev.xiaomi.com/doc/?p=544#d5e25)。

1\. AndroidManifest.xml声明

首先，在权限配置里面，有2处需要改成开发者自己的 APP 包名，已经配置过的条目则无须添加。

```xml
<!--配置权限，已经配置过的条目则无须添加-->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.GET_TASKS" />
<uses-permission android:name="android.permission.VIBRATE"/>

<!--以下两处com.netease.nim.demo改开发者App的包名-->
<permission android:name="com.netease.nim.demo.permission.MIPUSH_RECEIVE"
            android:protectionLevel="signature" />
<uses-permission android:name="com.netease.nim.demo.permission.MIPUSH_RECEIVE" />
```

下面这些配置直接拷贝到AndroidManifest.xml。

```xml
<!--配置的service和receiver-->
<service
    android:name="com.xiaomi.push.service.XMPushService"
    android:enabled="true"
    android:process=":mixpush"/>
<service
    android:name="com.xiaomi.push.service.XMJobService"
    android:enabled="true"
    android:exported="false"
    android:permission="android.permission.BIND_JOB_SERVICE"
    android:process=":mixpush" />
    <!--注：此service必须在3.0.1版本以后（包括3.0.1版本）加入-->
<service
    android:enabled="true"
    android:exported="true"
    android:name="com.xiaomi.mipush.sdk.PushMessageHandler" />

<service android:enabled="true"
    android:name="com.xiaomi.mipush.sdk.MessageHandleService" />
<!--注：此service必须在2.2.5版本以后（包括2.2.5版本）加入-->
<receiver
    android:exported="true"
    android:name="com.xiaomi.push.service.receivers.NetworkStatusReceiver" >
    <intent-filter>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</receiver>
<receiver
    android:exported="false"
    android:process=":mixpush"
    android:name="com.xiaomi.push.service.receivers.PingReceiver" >
    <intent-filter>
        <action android:name="com.xiaomi.push.PING_TIMER" />
    </intent-filter>
</receiver>
<receiver
    android:name="com.netease.nimlib.mixpush.mi.MiPushReceiver"
    android:exported="true">
    <intent-filter android:priority="0x7fffffff">
        <action android:name="com.xiaomi.mipush.RECEIVE_MESSAGE"/>
        <action android:name="com.xiaomi.mipush.MESSAGE_ARRIVED"/>
        <action android:name="com.xiaomi.mipush.ERROR"/>
    </intent-filter>
</receiver>
```

2\. 配置`appID`，`appkey`，证书

请在 App 启动逻辑中，在 `NIMClient.init()` 之前，调用如下方法在客户端配置小米推送的证书名称 certificate（在网易云通信后台管理系统中配置），`appID` ，`appkey`。可参照网易云通信 Demo。

```java
// 此处 certificate 请传入为开发者配置好的小米证书名称
NIMPushClient.registerMiPush(context, certificate, appID, appKey);
```

如果构建 APP 时有使用代码混淆，需要在proguard.cfg中加入
```
-dontwarn com.xiaomi.push.**
-keep class com.xiaomi.** {*;}
```

至此，小米推送接入完成，现在可以在小米设备上进行消息推送的测试。

#### 3. 小米推送兼容性

若开发者自身业务体系中，也需要接入小米推送，则需要考虑开发者自身业务体系的小米推送与网易云通信消息的小米推送兼容
需要做好以下2点，其余配置、逻辑都不需要修改。

- 对于小米推送，为了接收推送消息，小米SDK 要求开发者自定义一个继承自 `PushMessageReceiver`类的 `BroadcastReceiver` ，并注册到 `AndroidManifest.xml`。由于小米的特殊处理，同时注册多个继承自 `PushMessageReceiver` 的 `BroadcastReceiver` 会存在收不到消息的情况，要保证开发者自身业务体系的小米推送与网易云通信消息的小米推送兼容，开发者需要将自身的小米推送广播接收器，从继承 `PushMessageReceiver` 改为继承 `MiPushMessageReceiver`。开发者自行处理自身体系的推送消息，网易云通信不做处理。

```java
/**
 * 以下这些方法运行在非 UI 线程中, 与小米SDK PushMessageReceiver 方法一一对应。
 * 开发者如果自身也需要接入小米推送，则应将继承 PushMessageReceiver 改为继承 MiPushMessageReceiver
 */

public class MiPushMessageReceiver extends BroadcastReceiver{

    @Override
    public final void onReceive(Context context, Intent intent) {
    }

    public void onReceivePassThroughMessage(Context context, MiPushMessage message) {
    }

    public void onNotificationMessageClicked(Context context, MiPushMessage message) {
    }

    public void onNotificationMessageArrived(Context context, MiPushMessage message) {
    }

    public void onReceiveRegisterResult(Context context, MiPushCommandMessage message) {
    }

    public void onCommandResult(Context context, MiPushCommandMessage message) {
    }
}
```

- 在将广播接收器改为继承 `MiPushMessageReceiver` 之后，将该广播在 `AndroidManifest` 中配置如下，开发者只需将广播名称 `Receiver` 替换成自身的广播名。此外，请不要为此 Receiver 配置 `priority`。

```xml
<receiver android:name=".Receiver"
    android:exported="true">
    <intent-filter>
        <action android:name="com.xiaomi.mipush.RECEIVE_MESSAGE"/>
        <action android:name="com.xiaomi.mipush.MESSAGE_ARRIVED"/>
        <action android:name="com.xiaomi.mipush.ERROR"/>
    </intent-filter>
</receiver>
```

> 注意：
>
> - 避免在逻辑中调用注销推送方法，即 ` MiPushClient.unregisterPush()`，即便是需要设置关闭推送。
>
> - 开发者应确保自身业务中推送的流程和逻辑完整，虽然已经调用了`NIMPushClient.registerMiPush()`，仍然要在合适的时机调用 `MiPushClient.registerPush()` 注册推送。由于网易云通信也会在小米设备上调用此方法，因此可能存在 `MiPushMessageReceiver`中 `onReceiveRegisterResult` 多次回调的情形。此外，小米 SDK 在 5s内重复调用 `MiPushClient.registerPush()` 注册推送会被过滤，因此，请开发者不要在应用生命周期中反复调用注册方法。

### <span id="华为推送">华为推送</span>
华为推送从新版 HMS 包开始采用了和小米推送相似的系统级连接方案，但是依赖于EMUI 版本和 `华为移动服务`版本，经过大量测试表明，EMUI4.1及以上并同时满足`华为移动服务`版本 20401300 以上时比较稳定能收到通知栏推送消息，和 MIUI 效果一致。因此网易云通信 Android
SDK 在同时满足这两个条件的设备上才会启动华为推送服务。

集成华为推送流程和小米类似，，需要以下几个步骤：

#### 1. 准备工作

- 前往[华为推送官网](http://developer.huawei.com/push)注册账号并通过认证，创建应用并获取AppID、AppKey、AppSecret，步骤可参考[华为HMS接入文档](http://developer.huawei.com/consumer/cn/wiki/index.php?title=HMS%E5%BC%80%E5%8F%91%E6%8C%87%E5%AF%BC%E4%B9%A6-%E5%BC%80%E5%8F%91%E5%87%86%E5%A4%87)。

- 下载华为移动服务 HMS 2.4.0.300，将 aar 包导入到工程中，导入方式参考[华为HMS接入文档](http://developer.huawei.com/consumer/cn/wiki/index.php?title=HMS%E5%BC%80%E5%8F%91%E6%8C%87%E5%AF%BC%E4%B9%A6-%E5%BC%80%E5%8F%91%E5%87%86%E5%A4%87)。（与小米推送类似，网易云通信IM SDK 中并不包含华为推送 SDK，开发者需要自行下载并导入到工程中）。

- 前往网易云通信后台管理中心，创建华为推送证书。首先选择需要接入推送的应用，点击证书管理：

![](http://yx-web.nos.netease.com/webdoc/im/android/certificates.jpg)

在Android推送证书一栏选择添加证书，随后在弹出框中填写对应的信息，其中应用包名填写开发者 Android 应用的包名，AppSecret 填写第一步在华为官网创建应用所获取的 AppSecret，请开发者谨记证书名，这个会在 SDK初始化推送的时候使用。这个过程和小米类似。

#### 2. 接入流程

与接入小米推送类似，快速接入华为推送只需要2步，详细可参考[华为推送接入文档](http://developer.huawei.com/consumer/cn/wiki/index.php?title=HMS%E5%BC%80%E5%8F%91%E6%8C%87%E5%AF%BC%E4%B9%A6-PUSH%E6%9C%8D%E5%8A%A1%E6%8E%A5%E5%8F%A3)。

1\. AndroidManifest.xml声明

首先，在权限配置里面，有一处需要填写appid，有1处需要改成开发者自己的 APP 包名。

```xml
<meta-data
    android:name="com.huawei.hms.client.appid"
    <!-- XXXX替换为华为推送平台配置的应用appid -->
    android:value="XXXX" />

<provider
    android:name="com.huawei.hms.update.provider.UpdateProvider"
    <!-- XXXX替换为自己的包名 -->
    android:authorities="XXXX.hms.update.provider"
    android:exported="false"
    android:grantUriPermissions="true"></provider>
```

下面这些配置直接拷贝到AndroidManifest.xml。

```xml
<!-- 云信华为推送消息广播 -->
<receiver android:name="com.netease.nimlib.mixpush.hw.HWPushReceiver">
    <intent-filter android:priority="0x7fffffff">
        <action android:name="com.huawei.android.push.intent.REGISTRATION" />
        <action android:name="com.huawei.android.push.intent.RECEIVE" />
        <action android:name="com.huawei.android.push.intent.CLICK" />
        <action android:name="com.huawei.intent.action.PUSH_STATE" />
    </intent-filter>
</receiver>

<!-- 兼容性广播 -->
<receiver android:name="com.huawei.hms.support.api.push.PushEventReceiver">
    <intent-filter>
        <!-- 接收通道发来的通知栏消息，兼容老版本Push -->
        <action android:name="com.huawei.intent.action.PUSH" />
    </intent-filter>
</receiver>
```

2\. 配置华为推送证书

请在 App 启动逻辑中，在 `NIMClient.init()` 之前，调用如下方法在客户端配置华为推送的证书名称 certificate（在网易云通信后台管理系统中配置），可参照网易云通信 Demo。

```java
// 此处 certificate 请传入开发者自身的华为证书名称
NIMPushClient.registerHWPush(context, certificate);
```

如果构建 APP 时有使用代码混淆，需要在proguard.cfg中加入

```
-ignorewarning
-keepattributes *Annotation*
-keepattributes Exceptions
-keepattributes Signature
# hmscore-support: remote transport
-keep class * extends com.huawei.hms.core.aidl.IMessageEntity { *; }
# hmscore-support: remote transport
-keepclasseswithmembers class * implements com.huawei.hms.support.api.transport.DatagramTransport {
<init>(...); }
# manifest: provider for updates
-keep public class com.huawei.hms.update.provider.UpdateProvider { public *; protected *; }
```

至此，华为推送接入完成，现在可以在满足条件的华为设备上进行消息推送测试。

#### 3. 华为推送兼容性

若开发者自身业务体系中，也需要接入华为推送，则需要考虑开发者自身业务体系的华为推送与网易云通信消息的华为推送兼容
需要做好以下2点，其余配置、逻辑都不需要修改。

- 对于华为推送，为了接收推送消息，华为 SDK 要求开发者自定义一个继承自 `PushReceiver`类的 `BroadcastReceiver` ，并注册到 `AndroidManifest.xml`。由于同时注册多个继承自 `PushReceiver ` 的 `BroadcastReceiver` 会存在收不到消息的情况，要保证开发者自身业务体系的华为推送与网易云通信消息的华为推送兼容，开发者需要将自身的华为推送广播接收器，从继承 `PushReceiver ` 改为继承 `HWPushMessageReceiver`。开发者自行处理自身体系的推送消息，网易云通信不做处理。

```java
/**
 *
 * 以下这些方法运行在非 UI 线程中, 与华为 SDK 的 PushReceiver 方法一一对应。
 * 当开发者自身也接入华为推送，则应将继承 PushReceiver 改为继承 HWPushMessageReceiver，其他不变
 */

public class HWPushMessageReceiver extends BroadcastReceiver {

    @Override
    public final void onReceive(Context context, Intent intent) {
    }

    public void onToken(Context context, String token, Bundle extras) {
    }

    public boolean onPushMsg(Context context, byte[] raw, Bundle bundle) {
        return false;
    }

    public void onEvent(Context context, PushReceiver.Event event, Bundle extras) {
    }

    public void onPushState(Context context, boolean pushState) {
    }
}
```

- 在将广播接收器改为继承 `HWPushMessageReceiver` 之后，将该广播在 `AndroidManifest` 中配置如下，开发者只需将广播名称 `Receiver` 替换成自身的广播名。此外，请不要为此 Receiver 配置 `priority`。

```xml
<receiver android:name=".Receiver">
    <intent-filter>
        <action android:name="com.huawei.android.push.intent.REGISTRATION" />
        <action android:name="com.huawei.android.push.intent.RECEIVE" />
        <action android:name="com.huawei.android.push.intent.CLICK" />
        <action android:name="com.huawei.intent.action.PUSH_STATE" />
    </intent-filter>
</receiver>
```

> 注意：
>
> - 避免在逻辑中调用注销推送方法，即 ` MiPushClient.unregisterPush()`，即便是需要设置关闭推送。
>
> - 开发者应确保自身业务中推送的流程和逻辑完整，虽然已经调用了`NIMPushClient.registerHWPush()`，仍然要在合适的时机注册华为推送。

### <span id="第三方推送自定义接口">第三方推送自定义接口</span>

对于普通的使用场景，上述接口和功能已经可以满足，为了部分特殊场景的扩展性，网易云通信第三方提供了一些扩展接口，开发者可以依靠这些接口实现一些自定义的推送服务。

扩展接口由 `MixPushMessageHandler` 提供

```java
/**
 * 第三方推送消息回调接口，用户如果需要自行处理云信的第三方推送消息，则可实现该接口，并注册到{@link NIMPushClient}
 */

public interface MixPushMessageHandler {

    /**
     * 第三方推送通知栏点击之后的回调方法，（对于华为推送，这个方法并不能保证一定会回调）
     *
     * @param context
     * @param payload IMessage 中的用户设置的自定义pushPayload {@link com.netease.nimlib.sdk.msg.model.IMMessage}
     * @return true 表示开发者自行处理第三方推送通知栏点击事件，SDK将不再处理; false 表示仍然使用SDK提供默认的点击后的跳转
     */
    boolean onNotificationClicked(Context context, Map<String, String> payload);

    /**
     * 华为推送通知清除接口，利用该接口开发者可以自定义清除华为推送通知。
     * 因为华为推送 SDK 没有清除通知栏接口，对于华为推送消息，云信 SDK 默认调用了清除应用所有通知栏的接口。
     * 如果开发者需要自定义，可以使用这个接口做清除处理
     *
     * @return 返回true表示开发者处理了清除工作，云信 SDK 不再处理，返回false 则相反
     */
    boolean cleanHuaWeiNotifications();
}
```

开发者需要实现 `MixPushMessageHandler`，并在 `NIMPushClient.registerMiPush()` 之后调用下面方法注册该接口。

```java
NIMPushClient.registerMixPushMessageHandler(mixPushMessageHandler);
```

下面分别讲述具体接口的功能以及示例。

- onNotificationClicked：自定义处理推送通知点击事件。

当消息通过第三方推送到用户，用户点击通知栏之后便回调 `onNotificationClicked` 方法（对于华为推送，该方法偶尔会失效，点击通知栏之后不能保证一定回调此方法）。网易云通信消息 `IMMessage` 提供了消息提醒定制（可查看[发送消息](#发送消息)这一节），这个功能在第三方推送中也同样支持，自定义的消息提醒包含消息推送文案以及自定义字段。自定义的消息推送文案优先级高于默认的推送文案，而自定义的字段 `payload` 也会跟随第三方推送消息携带过来。依靠此回调方法用户可以处理推送传递下来的`payload`字段。

- cleanHuaWeiNotifications 华为推送清除通知栏接口

因为华为推送 SDK 没有清除通知栏接口，对于华为推送消息，云信 SDK 默认调用了清除应用所有通知栏的接口。如果开发者有些通知栏不希望被清除，可以使用这个接口自定义处理，例如可以先将这些通知缓存，在调用清除所有通知接口之后，再将缓存的通知弹出。

> 注意，由于消息的自定义推送文案以及字段由消息发送端设置，若开发者要使用该功能，请保证各端（IOS, Android, PC, Mac, Web）在发送消息时做到统一。
>
> 此外，对于小米推送，`payload` 内容会通过小米推送消息 extra 字段传输，而 extra 字段可以实现一些自定义的功能，如自定义通知栏铃声的URI、通知栏的点击行为等等（小米推送 extra 字段选项请参照小米推送Android 以及服务端文档）。
> 因此使用 `payload` 能间接达到设置这些自定义推送选项。由于小米SDK extra 字段限制，请各端消息发送 `payload` 字段 key-value 组合不要超过 8 对（若超出可使用嵌套 Map），不要使用 "nim" 作为key，整体 payload 长度不超过1000。
>
> 对于华为推送，因为点击通知栏不一定会有方法回调，所以可能导致点击推送通知栏启动应用无法收到 `payload` 的情况。


### <span id="推送设置">推送设置</span>

#### 1. 网易云通信消息开启、关闭推送

使用 `MixPushService` 提供的 `enable` 方法可以实现。

```java
/**
 * 开启/关闭第三方推送服务
 *
 * @param enable true 开启，SDK 需要与网易云通信服务器做确认；false 关闭，SDK 也需要通知网易云通信服务器。
 * @return InvocationFuture 可以设置回调函数。只有与服务器交互完成后才算成功，如果出错，会有具体的错误代码。
 */
public InvocationFuture<Void> enable(boolean enable);
```

调用示例

```java
NIMClient.getService(MixPushService.class).enable(enable).setCallback(new RequestCallback<Void>(){ ... });
```

在关闭、开启消息提醒的时候，可以调用该接口关闭推送。

#### 2. 设置推送免打扰时间

使用 `MixPushService` 提供的 `setPushNoDisturbConfig` 方法可以实现。

```java
/**
 * 设置推送免打扰时间，时间参数为北京时间的24小时计数 HH:mm，该时间段将不再向用户推送消息
 * SDK 3.2.0 版本以前的用户，为了将用户设置的免打扰配置与push免打扰同步，应该在监听到登陆同步完成后，
 * 调用 setPushNoDisturbConfig 方法。如果开发者不使用新版第三方推送功能，只要不调用该方法，则旧的功能不受影响。
 * 此外，在免打扰设置界面也应该做到同时设置push免打扰
 * @param isOpen 是否开启
 * @param startTime 开始时间 格式 HH:mm
 * @param stopTime 结束时间 格式 HH:mm
 * @return InvocationFuture 可以设置回调函数。成功会返回成功信息，错误会返回相应的错误码。
 */
InvocationFuture<Void> setPushNoDisturbConfig(boolean isOpen, String startTime, String stopTime);
```

在 3.2.0 版本之前，设置的消息免打扰对本地消息提醒有效，若在3.2.0接入了推送，为了确保设置统一，应该在用户登陆同步完成后，将用户设置的免打扰时间同步设置推送免打扰，参考 Demo 中 `MainActivity` 的做法。


### <span id="常见问题">常见问题</span>

1\. 开发者接入小米推送之后，网易云通信只支持对小米设备进行消息推送，其它设备默认不进行推送；对于华为推送，需同时满足设备 EMUI4.1(包括)以上以及华为移动服务 20401300(包括)以上版本会进行消息推送，其他设备默认不进行推送。

2\. 网易云通信在使用小米、华为推送时，使用通知栏而不是透传提醒模式，因为透传模式要求应用在后台存活，这样消息到达率很低。

3\. 第三方推送通知栏展示效果与端内推送展示效果、点击跳转效果存在一些差异，这个问题网易云通信端内推送做了统一，开发者可以在初始化的时候选择启用展开、折叠样式的通知栏。

4\. 第三方推送不支持通知栏自定义：铃声、通知栏样式以及提醒模式，旧版设置只对端内推送生效。

5\. 若用户不需要接入小米、华为推送，则不做任何配置，将不受影响。

6\. 在满足第三方推送的设备上，若网络畅通、账号登陆成功、消息收发正常条件下，若开发者若收不到推送，可从以下几个方面进行排查，
首先检查 AndroidManifest 配置是否正确，包括权限、Service、BroadcastReceiver，网易云通信SDK对 AndroidManifest 做了检查，并有日志记录；其次检查是否关闭了消息提醒以及设置了免打扰等，
如果仍然不能解决，可以求助网易云通信技术支持。


## <span id="事件订阅">事件订阅(在线状态)</span>

3.6.0 版本新增事件订阅、发布机制，IM Demo 基于事件订阅实现了在线状态展示，开发者可以参考 Demo 的实现，根据自己的场景做修改。

事件均为 `Event` 对象，包含事件类型、事件值、有效期、发布者、多端配置等属性，事件不保存，同一账号的同一类型事件，后发布将会覆盖之前发布。当有新事件产生时（服务端产生或者客户端发布），服务端会对所有订阅着下发该事件。

事件类型 1-99999 为网易云通信预留，目前仅使用事件类型 1 为在线状态事件 `NimOnlineStateEvent`，预留事件比普通事件多一个 `nimConfig` 字段信息，该字段为服务端填写，对于在线状态事件，`nimConfig` 字段值为一段 json ，携带各端是否在线的信息。

### <span id="订阅事件">订阅事件</span>

可以订阅指定账号的指定类型的事件，使用 `EventSubscribeService#subscribeEvent` 方法，参数为 `EventSubscribeRequest` 对象，指定订阅的事件类型、事件发布者集合、订阅有效期，以及是否立即同步事件。订阅有效期范围为 60 秒到 30 天，数值单位为秒，是否立即同步事件若设置为true则订阅成功后会立即返回目标事件。

对于订阅有效期，由于多端订阅会覆盖这个时长，所以建议开发者各端订阅时长保持一致。此外，需要注意的是，为了性能考虑，在30秒内对同一账号同一事件订阅，即使设置为立即同步服务的将不会下发目标事件。

```java
NIMClient.getService(EventSubscribeService.class).subscribeEvent(eventSubscribeRequest).setCallback(new RequestCallbackWrapper<List<String>>() {
        @Override
        public void onResult(int code, List<String> result, Throwable exception) {
            if (code == ResponseCode.RES_SUCCESS) {
                if (result != null) {
                    // 部分订阅失败的账号。。。
                    //
                    //
                }
            } else {

            }
        }
    });
```

### <span id="取消订阅事件">取消订阅事件</span>

可以取消订阅指定账号的指定类型的事件，使用 `EventSubscribeService#unSubscribeEvent` 方法，参数为 `EventSubscribeRequest` 对象，指定取消订阅的事件类型、事件发布者集合。

```java
NIMClient.getService(EventSubscribeService.class).unSubscribeEvent(eventSubscribeRequest);
```

### <span id="发布事件">发布事件</span>

目前网易云通信支持客户端发布类型 1 并且值为 10001 的事件，仅用于设置事件的多端配置信息，多端同时在线时设置了该信息后服务端会将这些设置合并后下发，订阅者收到事件之后可以通过 `Event#getConfigByClient` 方法获取该账号的多端信息，基于此，Demo实现的在线状态发布了网络状态以及客户端类型，根据多端的优先级来进行展示。

使用 `EventSubscribeService#publishEvent` 方法，参数为 `Event` 对象。

```java
NIMClient.getService(EventSubscribeService.class).publishEvent(event);
```

### <span id="查询事件订阅">查询事件订阅</span>

支持查询事件订阅，用于查询某种事件的订阅关系，使用 `EventSubscribeService#querySubscribeEvent ` 方法，参数为 `EventSubscribeRequest` 对象，指定查询订阅的事件类型、事件发布者集合，返回`List<EventSubscribeResult>` 集合。

```java
NIMClient.getService(EventSubscribeService.class).querySubscribeEvent(request);
```

### <span id="监听事件">监听事件</span>

通过监听接口监听接收到在线状态事件，开发者必须将此监听生命周期与 Application 生命周期一致。

```java
NIMClient.getService(EventSubscribeServiceObserver.class).observeEventChanged(new Observer<List<Event>>() {
            @Override
            public void onEvent(List<Event> events) {
                // 处理
            }
        }, true);
```

## <span id="语音录制和播放">语音录制和播放</span>

### <span id="录制">录制</span>

网易云通信 SDK 提供了一套录制高清语音的接口 `AudioRecorder` ，用于采集，编码，存储高清语音数据，并提供过程回调，供开发者进行自由的界面展现。
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
	maxDuration, // 最长录音时长，到该长度后，会自动停止录音, 默认120s
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

网易云通信的语音消息格式有 aac 和 amr 两种格式可选，由于 2.x 系统的原生 MediaPlayer 不支持 aac 格式，因此 SDK 也提供了一个 AudioPlayer 来播放网易云通信的语音消息。同时，将 MediaPlayer 的接口进行了一些封装，使得在会话场景下播放语音更加方便。
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
## <span id="智能对话机器人">智能对话机器人</span>

4.0.0 版本新增智能对话机器人功能，智能对话机器人解决方案依托网易IM即时通讯、语音识别、语义理解等服务，为开发者提供人机交互方式API/SDK、语音识别、意图识别、知识库配置、动态接口等功能，可以在应用IM内快速集成场景丰富的智能对话机器人。

区别于开发者业务后台自行设定的机器人，网易波特中配置的机器人，类似网易精灵、小黄鸡这种通过配置知识库，与用户进行交流问答的智能对话机器人。开发者可以在 [网易波特 服务](http://sw.bot.163.com) 开通机器人服务。

机器人和云信账号是有绑定关系的，一个机器人账号对应了一个云信 id ，两者互相独立 ， 云信内部负责维护对应关系。机器人所对应的云信用户不会在线，也不应该和其他正常用户有用户关系，如加好友，拉黑等。

智能对话机器人消息属于云信内置基础消息类型中的一种，详细请参考 [发送消息](#发送消息) 章节。

### <span id="机器人信息">机器人信息</span>

- 获取机器人信息

机器人信息使用 `NimRobotInfo` 表示，继承至 `UserInfo`，登录成功之后机器人信息会同步至本地，开发者使用
RobotServiceObserve#observeRobotChangedNotify 监听机器人增、减变化通知。

```java
private Observer<RobotChangedNotify> robotChangedNotifyObserver = new Observer<RobotChangedNotify>() {
    @Override
    public void onEvent(RobotChangedNotify robotChangedNotify) {
        List<NimRobotInfo> addedOrUpdatedRobots = robotChangedNotify.getAddedOrUpdatedRobots();
        List<String> addedOrUpdateRobotAccounts = new ArrayList<>(addedOrUpdatedRobots.size());
        List<String> deletedRobotAccounts = robotChangedNotify.getDeletedRobots();

        String account;
        for (NimRobotInfo f : addedOrUpdatedRobots) {
            account = f.getAccount();
            robotMap.put(account, f);
            addedOrUpdateRobotAccounts.add(account);
        }

        // 通知机器人变更
        if (!addedOrUpdateRobotAccounts.isEmpty()) {
            // log
            DataCacheManager.Log(addedOrUpdateRobotAccounts, "on add robot", UIKitLogTag.ROBOT_CACHE);
        }

        // 处理被删除的机器人
        if (!deletedRobotAccounts.isEmpty()) {
            // update cache
            for (String a : deletedRobotAccounts) {
                robotMap.remove(a);
            }
        }
    }
};

NIMClient.getService(RobotServiceObserve.class).observeRobotChangedNotify(robotChangedNotifyObserver, register);
```
此外使用 RobotService#getAllRobots 获取本地数据库的所有机器人信息。

```java
List<NimRobotInfo> robots = NIMClient.getService(RobotService.class).getAllRobots();
```

### <span id="机器人消息">机器人消息</span>

- 机器人对话流程

机器人类型消息分为机器人上行、下行消息，分别表示发送给机器人和机器人下发的消息。客户端使用 MessageBuilder#createRobotMessage 构建上行消息并调用消息发送接口，收到机器人下行消息之后通过，机器人消息附件 RobotAttachment 中可以获取机器人回复内容。

```java
RobotAttachment attachment = (RobotAttachment) message.getAttachment();

if (attachment.isRobotSend()) {
    // 下行消息
    RobotResponseContent content =  new RobotResponseContent(attachment.getResponse()));
}
```

- 机器人回复内容

机器人内容格式规范可参考 [机器人消息体模板说明](/docs/product/IM即时通讯/机器人消息体模板说明)。

开发者调用 RobotAttachment#getResponse 方法返回机器人回复内容，再将机器人返回内容解析成为视图展示，UiKit组件提供了解析模板，开发者可以参考 RobotResponseContent、RobotContentLinearLayout 的实现。

```java
RobotResponseContent content = new RobotResponseContent(attachment.getResponse()));
```
RobotContentLinearLayout 负责将 RobotResponseContent 解析为视图。

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

属于网易云通信内建的系统通知，其对应的数据结构为 `SystemMessage`， 由网易云通信服务器推送给用户的通知类消息，用于网易云通信系统类的事件通知。现在主要包括群变动的相关通知，例如入群申请，入群邀请等，如果第三方应用还托管了好友关系，好友的添加、删除也是这个类型的通知。系统通知由 SDK 负责接收和存储，并提供较简单的未读数管理。只有在线和离线，没有漫游。没有通知栏提醒（如有需要，第三方自行实现）。

- 使用场景

通常在验证消息列表中展现。

### <span id="自定义通知">自定义通知</span>

- 概念

提供给第三方自定义的全局的通知类型，其对应的数据结构为 `CustomNotification`。只有在线和离线，没有漫游，没有通知栏提醒（第三方自行实现）。

自定义通知和自定义消息的不同之处在于，自定义消息归属于会话中的消息体系内，由 SDK 存储在消息数据库中，与网易云通信的其他内建消息类型一同展现给用户。而自定义通知主要用于第三方的一些事件状态通知，网易云通信不存储，也不解析这些通知，网易云通信仅仅负责替第三方传递和通知这些事件，起到透传的作用。

- 使用场景

聊天双方处于P2P聊天界面时，显示的“正在输入通知”。

