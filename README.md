# 网易云信安卓SDK开发指南

## <span id="SDK 概述">SDK 概述</span>

网易云信 SDK 为移动应用提供一个完善的 IM 系统开发框架，屏蔽掉 IM 系统的复杂细节，对外提供较为简洁的 API 接口，方便第三方应用快速集成 IM 功能。SDK 提供的能力如下：

- 网络连接管理
- 登录认证服务
- 消息传输服务
- 群组管理/消息服务
- 消息存储服务
- 实时语音视频通话

## <span id="开发准备">开发准备</span>

首先从[网易云信官网](http://netease.im/base.html?page=download  "target=_blank")下载 Android SDK。开发者可以根据实际需求，配置类库。

### <span id="类库配置">类库配置</span>

 SDK 包的libs文件夹中，包含了网易云信的 jar 文件，各 jni 库文件夹以及 SDK 依赖的第三方库，列表如下：
 
```
libs
├── armeabi
│   ├── libne_audio.so （高清语音录制功能必须）
│   └── libcosine.so （Android 后台保活功能必须）
│   └── libnio.so （音视频、实时会话服务需要）
│   ├── librtc_engine.so （音视频需要）
│   └── librtc_network.so （音视频需要）
│   └── librts_network.so （实时会话服务需要）
├── armeabi-v7a
│   ├── libne_audio.so
│   └── libcosine.so
│   └── libnio.so
│   ├── librtc_engine.so
│   └── librtc_network.so
│   └── librts_network.so
├── x86
│   ├── libne_audio.so
│   └── libcosine.so
│   └── libnio.so
│   ├── librtc_engine.so
│   └── librtc_network.so
│   └── librts_network.so
├── nim-sdk-1.0.0.jar
└── netty-4.0.23-for-yx.final.jar
```

将这些文件拷贝到你的工程的 libs 目录下，即可完成配置。

以上文件列表中，nim-sdk-1.0.0.jar (版本号可能会不同)为网易云信 SDK，子目录中的文件是 SDK 所依赖的各个 CPU 架构的 so 库。

> 注意：如果你只需要 SDK 的基础功能（不含音视频及实时会话服务），则 so 库只需要 libne_audio.so 和 libcosine.so 两个，没有 libnio.so、librtc\*.so、librts\*.so；如果需要音视频功能，so 库需要加上 libnio.so及 librtc\*.so；如果需要实时会话服务，so 库需要加上 libnio.so、librts\*.so。

如果你的 APP 的 libs 里面只包含 armeabi 一个文件夹，为了保证在 arm-v7a 上有较好的性能，以及兼容各个平台，可将各目录下的 so 文件名改为原文件名加上"_{arch_of_cpu}"，然后统一放到 armeabi 目录下，SDK 也会加载到正确版本的so库。改名后的目录结构如下：

```
libs
├── armeabi
│   ├── libne_audio_armeabi.so
│   ├── libcosine_armeabi.so
│   ├── libnio_armeabi.so
│   ├── librtc_engine_armeabi.so
│   ├── librtc_network_armeabi.so
│   ├── librts_network_armeabi.so
│   ├── libne_audio_armeabi-v7a.so
│   ├── libcosine_armeabi-v7a.so
│   ├── libnio_armeabi-v7a.so
│   ├── librtc_engine_armeabi-v7a.so
│   └── librtc_network_armeabi-v7a.so
│   ├── librts_network_armeabi-v7a.so
├── x86(如果不考虑支持x86，可不包含)
│   ├── libne_audio_x86.so
│   ├── libcosine_x86.so
│   ├── libnio_x86.so
│   ├── librtc_engine_x86.so
│   └── librtc_network_x86.so
│   ├── librts_network_x86.so
├── nim-sdk-1.0.0.jar
└── netty-4.0.23-for-yx.final.jar
```

网易云信 SDK 的网络连接还依赖于 netty 框架，在上面的文件夹中，我们也包含了 netty 的 4.0.23 版本的 jar 包，这个 jar 包我们有一些修改，主要是增加 Android 兼容性，去掉了一些依赖反射的调用，以及性能上的一些优化。如果你的 APP 也依赖于 netty，你可以使用这个修改后的版本，也可以使用官方的原始版本。

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
```

### <span id="总体接口介绍">总体接口介绍</span>

网易云信 SDK 提供了两类接口供开发者调用：一类是第三方 APP 主动发起请求，第二类是第三方 APP 作为观察者监听事件和变化。第一类接口名均以 **Service** 结尾，例如 `AuthService` ，第二类接口名均以 **ServiceObserver** 结尾，例如 `AuthServiceObserver`，个别太长的类名则可能直接以 **Observer** 结尾，比如 `SystemMessageObserver`。

可以通过 `NIMClient` 的 `getService` 接口获取到各个服务实例，例如：通过 `NIMClient.getService(AuthService.class)` 获取到 `AuthService` 服务实例，通过 `NIMClient.getService(AuthServiceObserver.class)` 获取到 `AuthServiceObserver` 观察者接口。

SDK 由于需要保持后台运行，典型场景下会在独立进程中运行，第一类接口基本上都是从主进程发起调用，然后在后台进程执行，最后再将结果返回给主进程。因此，如无特殊说明，所有接口均为异步调用，开发者无需担心调用 SDK 接口阻塞 UI 的问题。如果调用的接口需要一些后期操作，包括结果回调，取消调用等，此类接口会返回一个 `InvocationFuture` 对象。如果开发者关心调用的结果，可在此返回的接口中设置回调函数。
接口回调接口为 `RequestCallback` ，有3个接口需要实现：onSuccess, onFailed, onException，分别对应成功，失败，以及出现异常。另外还提供了一个更简洁的接口 `RequestCallbackWrapper`，作为 `RequestCallback` 的包裹，封装了上面的 3 个接口，然后将他们合并成一个 onResult 接口，在参数上做不同结果的区分。

还有一类接口，耗时会很长，可能需要传输大量数据，或者要发起网络连接，比如上传下载，登录等。对于这类接口，返回值会是一个 `AbortableFuture` 对象，该接口继承自 `InvocationFuture`，增加了一个 `abort()` 方法，可取消之前的请求。

观察者接口的方法名都是以 observe 开头，并包含一个 register 参数，该值为`true`时，为注册观察者，为`false`时，注销观察者。开发者要在不需要观察者时，主动注销，以免造成资源泄露。

> 注意：除开 `NIMClient.init` 接口外，其他 SDK 暴露的接口都只能在 UI 进程调用。如果 APP 包含远程 service，该 APP 的 Application 的 onCreate 会多次调用。因此，如果需要在 onCreate 中调用除 init 接口外的其他接口，应先判断当前所属进程，并只有在当前是 UI 进程时才调用。判断代码如下：

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
- `TeamService`: 群组服务接口，用于发送群组消息，管理群组和群成员资料等。
- `TeamServiceObserve`: 群组和群成员资料变化观察者。
- `SystemMessageService`: 管理系统通知接口。
- `SystemMessageObserve`: 系统通知观察者。
- `FriendService`: 好友关系托管接口，目前支持添加、删除好友、获取好友列表、黑名单、设置消息提醒。
- `FriendServiceObserve`: 好友关系变更、黑名单变更通知观察者。
- `UserServie`: 用户资料托管接口，提供获取用户资料、修改个人资料等。
- `UserServieObserve`: 用户资料托管接口，提供获取用户资料、修改个人资料等。
- `AVChatManager`: 语音视频通话接口。
- `RTSManager`: 实时会话接口。
- `NosService`: 网易云存储服务，提供文件上传和下载。

### <span id="SDK 数据目录结构">SDK 数据目录结构</span>

当收到多媒体消息后，SDK 会负责下载这些多媒体文件，同时 SDK 还要记录一些 log，因此 SDK 需要一个数据缓存目录。该目录由第三方 APP 通过 `SDKOptions` 传入，默认为 “/{外卡根目录}/{app\_package\_name}/nim/”。如果你的 APP 需要清除缓存功能，可扫描该目录下的文件，按照你们的规则清理即可。
缓存目录下面包含如下子目录：
- log: 包含一个文件：nim\_sdk.log，默认路径为“/{外卡根目录}/{app\_package\_name}/nim/nim\_sdk.log”，大小一般不超过 8M
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
		NIMClient.init(this, loginInfo(), options()); // SDK初始化（启动后台服务，若已经存在用户登录信息，SDK 将完成自动登录）
		// ... your codes
	}

	// 如果返回值为 null，则全部使用默认参数。
	private SDKOptions options() {
		SDKOptions options = new SDKOptions();

	    // 如果将新消息通知提醒托管给 SDK 完成，需要添加以下配置。否则无需设置。
	    StatusBarNotificationConfig config = new StatusBarNotificationConfig();
	    config.notificationEntrance = WelcomeActivity.class;
	    config.notificationSmallIconId = R.drawable.ic_stat_notify_msg;
	    options.statusBarNotificationConfig = config;

	    // 配置保存图片，文件，log 等数据的目录
	    // 如果 options 中没有设置这个值，SDK 会使用下面代码示例中的位置作为 SDK 的数据目录。
	    // 该目录目前包含 log, file, image, audio, video, thumb 这6个目录。
	    // 如果第三方 APP 需要缓存清理功能， 清理这个目录下面个子目录的内容即可。
	    String sdkPath = Environment.getExternalStorageDirectory() + "/" + getPackageName() + "/nim";
	    options.sdkStorageRootPath = sdkPath;

	    // 配置是否需要预下载附件缩略图，默认为 true
	    options.preloadAttach = true;

	    // 配置附件缩略图的尺寸大小，该值一般应根据屏幕尺寸来确定， 默认值为 Screen.width / 2
	    options.thumbnailSize = ${Screen.width} / 2;

        // 用户资料提供者, 目前主要用于提供用户资料，用于新消息通知栏中显示消息来源的头像和昵称
        options.userInfoProvider = new UserInfoProvider() {
             @Override
             public UserInfo getUserInfo(String account) {
                 return null;
             }

			 @Override
        	 public Bitmap getAvatarForMessageNotifier(String account) {
             	 return null;
        	 }

        	 @Override
        	 public int getDefaultIconResId() {
            	 return R.drawable.avatar_def;
        	 }

             @Override
             public Bitmap getTeamIcon(String tid) {
                 return null;
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

### <span id="登录">登录</span>

#### <span id="手动登录">手动登录</span>

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

#### <span id="自动登录">自动登录</span>

如果上次登录已经存在用户登录信息，那么在初始化 SDK 时传入 `LoginInfo`，SDK 后台会自动登录，并在登录发起前即打开相关数据库，供上层调用。开发者此时无需再手动调用登录接口，可以跳过登录界面直接进入主界面。

进入主界面后，可以通过监听用户状态（每次注册用户状态监听都会立即回调通知当前的用户状态），或者主动获取当前用户状态，来判断自动登录是否成功。

在初始化 SDK 时自动登录示例： 

```java
public class NimApplication extends Application {

	public void onCreate() {
		// ... your codes
		NIMClient.init(this, loginInfo(), options()); // SDK初始化（启动后台服务，若已经存在用户登录信息，SDK 将完成自动登录）
		// ... your codes
	}
	
	private LoginInfo getLoginInfo() {
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

### <span id="监听用户状态">监听用户状态</span>

登录成功后，SDK 会负责维护与服务器的长连接以及断线重连等工作。当用户状态发生改变时，会发出通知。开发者可以通过加入以下代码监听用户状态改变：

```java
NIMClient.getService(AuthServiceObserver.class).observeUserStatus(
	new Observer<StatusCode> () {
		public void onEvent(StatusCode status) {
			Log.i("tag", "User status changed to: " + status);
		}
}, true);
```

开发者也可以主动获取当前用户状态：

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

### <span id="登出">登出</span>

如果用户手动登出，不再接收消息和提醒，开发者可以调用 logout 方法，该方法没有回调。

```java
NIMClient.getService(AuthService.class).logout();
```

## <span id="基础消息功能">基础消息功能</span>

### <span id="消息功能概述">消息功能概述</span>

SDK 提供一套完善的消息传输管理服务，包括收发消息，存储消息，上传下载附件，管理最近联系人等。
SDK 原生支持发送文本，语音，图片，视频和地理位置等 5 种类型消息，同时支持用户发送自定义的消息类型。
网易云信消息对象均为 `IMMessage`，不同消息类型以 `MsgTypeEnum` 作区分，消息内容根据类型不同也不一样。文本消息最为简单，消息内容就是 `content` 字符串。其他消息类型均带有一个消息附件对象 `MsgAttachment`，该对象在传输时一般序列化为 json 格式字符串。内建的消息附件主要有以下几种：
- LocationAttachment： 位置消息附件对象类型。
- FileAttachment:  文件消息附件对象类型
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
    duration // 音频持续时间
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
```

发送消息后，还需要持续关注消息的发送进度，以更新发送界面。发送消息的接口可以设置回调函数，并在发送完成时回调，通知上层消息发送结果，如果出错，会有具体错误码。但是如果该回调函数没有发送状态变化通知和进度通知，开发者需要通过另一个观察者接口来完成。这样，开发者在任意界面都能接收到消息状态的改变。示例代码如下：

```java
// 监听消息发送状态的变化通知
NIMClient.getService(MsgServiceObserver.class).observeMsgStatus(
	new Observer<IMMessage> {
		public void onEvent(IMMessage message) {
			// 参数为有状态发生改变的消息对象，其 msgStatus 和 attachStatus 均为最新状态。
			// 发送消息和接收消息的状态监听均可以通过此接口完成。
		}
	}
}, true);
	
// 如果发送的多媒体文件消息，还需要监听文件的上传进度。
NIMClient.getService(MsgServiceObserver.class).observerAttachProgress(
	new Observer(AttachProgress progress) {
		public void onEvent(AttachProgress progress) {
			// 参数为附件的传输进度，可根据 progress 中的 uuid 查找具体的消息对象，更新 UI
			// 上传附件和下载附件的进度监听均可以通过此接口完成。
		}
	}, true);
```

此外，如果第三方APP想保存消息到本地，可以调用 MsgService#saveMessageToLocal ，该接口保存消息到本地数据库，但不发送到服务器端。
该接口将消息保存到数据库后，如果需要通知到UI，可将参数 notify 设置为 true ，此时会触发 MsgServiceObserve#observeReceiveMessage 通知。

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
如果接收到消息是带文件附件的多媒体消息，SDK 默认会在后台自动下载附件：如果是语音消息，直接下载文件，如果是图片或视频消息，下载缩略图文件。开发者可在 `SDKOptions` 中关闭自动下载，并在用户翻阅到对应消息，再通过以下代码手动下载。如果自动下载或手动下载失败，也可以通过这段代码重新下载：

```java
// 下载附件，参数1位消息对象，参数2为是下载缩略图还是下载原图。
// 因为下载的文件可能会很大，这个接口返回类型为 AbortableFuture ，允许用户中途取消下载。
AbortableFuture future = NIMClient.getService(MsgService.class).downloadAttachment(message, true);
```

### <span id="最近会话">最近会话</span>

最近会话，也可称作会话列表或者最近联系人列表，它记录了与用户最近有过会话的联系人信息，包括联系人帐号，联系人类型，最近一条消息的时间，消息状态，消息缩略，未读条数等信息。
最近会话列表由 SDK 维护并提供查询、监听变化的接口，只要与某个用户或者群组有产生聊天（自己发送消息或者收到消息）， SDK 会自动更新最近会话列表并通知上层，开发者无需手动更新。
某些场景下，开发者可能需要手动向最近会话列表中插入一条会话项（即插入一个最近联系人），例如：在创建完高级群时，需要在最近会话列表中显示该群的会话项。由创建高级群完成时并不会收到任何消息， SDK 并不会立即更新最近会话，此时要满足需求，可以在创建群成功的回调中，插入一条本地消息， 即调用 MsgService#saveMessageToLocal。

获取最近联系人列表：

```java
 NIMClient.getService(MsgService.class).queryRecentContacts()
	 .setCallback(new RequestCallbackWrapper<List<RecentContact>>() {
	   @Override
       public void onResult(int code, List<RecentContact> recents, Throwable exception) {
            // recents参数即为最近联系人列表（最近会话列表）
       }
    });
```

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

获取会话未读数总数，总两种方法：

- 通过接口直接获取

```java
int unreadNum = NIMClient.getService(MsgService.class).getTotalUnreadCount();
```

- 对最近联系人列表中的每个最近联系人的未读数进行求和：

```java
int unreadNum = 0;
for (RecentContact r : items) {
    unreadNum += r.getUnreadCount();
}
```

如果用户在开始聊天时，开发者都调用了 `setChattingAccount` 接口，SDK会自动管理消息的未读条数。当收到新消息时，自动增加未读数，在 `setChattingAccount` 时，自动将未读数清零。如果第三方 APP 需要不进入聊天窗口，就能主动将未读数清零，可以通过调用如下接口来实现：

```java
// 触发 MsgServiceObserve#observeRecentContact(Observer, boolean) 通知，
// 通知中的 RecentContact 对象的未读数为0
NIMClient.getService(MsgService.class).clearUnreadCount(account, sessionType);
```

如果需要在最近联系人列表界面显示当前消息状态，还需要增加消息状态监听，操作见[发送消息](#发送消息) 一节。

### <span id="自定义消息">自定义消息</span>

除了内建消息类型以外，SDK 还支持收发自定义消息类型。SDK 不负责定义和解析自定义消息的具体内容，解释工作由开发者完成。SDK 会将自定义消息存入消息数据库，会和内建消息一并展现在消息记录中。
为了使用更加方便，自定义消息采用附件的方式展示给开发者。体现在 `IMMessage` 类中，自定义消息的内容会被解析为 `MsgAttachment` 对象，由于 SDK 并不知道自定义消息的格式，第三方 APP 需要注册一个自定义消息解析器。当第三方 APP 调用查询查询消息历史的接口，或者是 SDK 收到消息通知第三方 APP 时，就能将自定义消息内容转换为一个 `MsgAttachment` 对象，然后就可以同操作图片消息等带附件消息一样，操作自定义消息了。
为了增加自定义消息的灵活性，SDK 还提供了对其生命周期和推送参数的一些配置选项，开发者可以自主设定一条自定义消息是否要保存云端消息记录，是否要漫游，发送时是否要多端同步等。详见 `CustomMessageConfig` 的 API 文档。

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

在 demo 中，我们提供了一个用自定义消息实现的“剪刀石头布”的游戏，下面以此为例，详细解析其实现步骤。

首先，我们先定义一个自定义消息附件的基类，负责解析你的自定义消息的公用字段，比如类型等。还可以定义一些公共接口，用于一些便利性的调用。

```java
// 先定义一个自定义消息附件的基类，负责解析你的自定义消息的公用字段，比如类型等等。
public abstract class CustomAttachment implements MsgAttachment {

	// 自定义消息附件的类型，根据该字段区分不同的自定义消息
    protected int type;

    CustomAttachment(int type) {
        this.type = type;
    }

	// 解析公用字段，然后将具体的附件内容分发给具体的子类去解析。
    public void fromJson(JSONObject json) {
        type = json.getInteger("type");
        JSONObject data = json.getJSONObject("data");
        if (data != null) {
            parseData(data);
        }
    }

	// 实现 MsgAttachment 的接口，封装公用字段，然后调用子类的封装函数。
    @Override
    public String toJson(boolean send) {
        JSONObject object = new JSONObject();
        object.put("type", type);
        JSONObject data = packData();
        if (data != null) {
            object.put("data", data);
        }

        return object.toJSONString();
    }

    // 子类的解析和封装接口。子类仅处理自己的具体数据，避免污染公共字段。
    protected abstract void parseData(JSONObject data);
    protected abstract JSONObject packData();
}
```

然后，继承这个基类，实现“剪刀石头布”的附件类型。

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
            switch (type) {
                case CustomAttachmentType.Guess:
                    attachment = new GuessAttachment();
                    break;
                default:
                    attachment = new DefaultCustomAttachment();
                    break;
            }

            if (attachment != null) {
                attachment.fromJson(object);
            }
        } catch (Exception e) {

        }

        return attachment;
    }
}
```

最后，将该附件解析器注册到 SDK 中。为了保证生成历史消息时能够正确解析自定义附件，注册一般应放在 Application 的 onCreate 中完成。

```java
NIMClient.getService(MsgService.class).registerCustomAttachmentParser(new CustomAttachParser());
```

### <span id="消息提醒">消息提醒</span>

SDK 内置了新消息状态栏提醒功能，开发者可以在 `SDKOptions` 中开启或关闭，以及设置铃声/振动提醒，免打扰时段等选项。各选项配置可参见[在线API文档](http://dev.netease.im/doc/android/index.html) 中 StatusBarNotificationConfig。同时，SDK 也提供了更新消息提醒配置的接口：

```java
// 开启/关闭通知栏消息提醒
NIMClient.toggleNotification(enable);

// 更新消息提醒设置
 NIMClient.updateStatusBarNotificationConfig(config);
```

针对不同的消息格式，状态栏显示不同的提醒内容。默认提醒内容为：
- 文本消息：文本消息内容。
- 文件消息：{说话者}发来一条文件消息
- 图片消息：{说话者}发来一条图片消息
- 语音消息：{说话者}发来一条语音消息
- 视频消息：{说话者}发来一条视频消息
- 位置消息：{说话者}分享了一个地理位置
- 通知消息：{说话者}: 通知消息
- 自定义消息：{说话者}: 自定义消息

除文本消息外，开发者可以通过  `NimStrings`  类修改这些默认提醒内容。
如果开发者需要为每条消息指定具体的提醒内容，而不是显示默认内容，开发者可以调用 `IMMessage` 的 `setContent` 接口。对于文本消息，该接口会同时修改消息内容和提醒内容，对于其他格式消息，该接口仅修改提醒内容。
如果接收方是 iOS 客户端，消息推送的内容遵从相同的规则：如果设置了 `setContent` 字段，则使用设置的字符串作为推送内容，否则使用默认提醒内容。

如果用户正在与某一个人聊天，当这个人的消息到达时，是不应该有状态栏提醒的，如果用户停留在最近联系人列表界面，收到消息也不应该有提醒，因此，为了内置的新消息提醒功能正确工作，开发者需要在进入进出聊天界面以及最近联系人列表界面时，通知 SDK。接口如下：

```java
// 进入聊天界面
NIMClient.getService(MsgService.class).setChattingAccount(
	account, 
	sessionType
	);
// 进入最近联系人列表界面
NIMClient.getService(MsgService.class).setChattingAccount(
	MsgService.MSG_CHATTING_ACCOUNT_ALL, 
	SessionTypeEnum.None
	);
// 退出聊天界面或最近联系人列表界面
NIMClient.getService(MsgService.class).setChattingAccount(
	MsgService.MSG_CHATTING_ACCOUNT_NONE, 
	SessionTypeEnum.None
	);
```

如果 SDK 内建的消息提醒不能满足你的需求，你可以关闭内建消息提醒，然后自己去实现。收到新消息时，上层有两种方式得到通知，然后给出通知栏提醒：

- 添加消息接收观察者，在观察者的 `onEvent` 中实现状态栏提醒。注册注销方式详见[接收消息](#接收消息)一节。注意：只有 SDK 1.4.0 及以上版本才能使用该方式，1.4.0 以下的版本使用此方式有可能会漏掉通知。
- SDK 收到新消息后，还会发送一个广播，在广播接收者的 `onReceive` 中实现自己的状态栏提醒。该广播的 ACTION 为 APP 的包名加上 `NimIntent#ACTION_RECEIVE_MSG`，例如 demo 的为："com.netease.nim.demo.ACTION.RECEIVE\_MSG"。使用这种方式时，开发者需要在 AndroidManifest.xml 文件中注册此广播的接收者，并添加对应的广播权限。

## <span id="历史记录">历史记录</span>

### <span id="本地记录">本地记录</span>

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
                .setCallback(new RequestCallbackWrapper<List<IMMessage>>() {});

```

删除消息记录：

```java
// 删除单条消息
NIMClient.getService(MsgService.class).deleteChattingHistory(message);
// 删除与某个聊天对象的全部消息记录
NIMClient.getService(MsgService.class).clearChattingHistory(account, sessionType);
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
NIMClient.getService(MsgService.class).pullMessageHistory( anchor, limit, persist);
```

## <span id="语音录制及回放">语音录制及回放</span>

### <span id="录制">录制</span>

网易云信 SDK 提供了一套录制高清语音的接口，用于采集，编码，存储高清语音数据，并提供过程回调，供开发者进行自由的界面展现。
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
普通群没有权限操作，适用于快速创建多人会话的场景。每个普通群只有一个管理员。管理员可以对群进行增减员操作，普通成员只能对群进行增员操作。在添加新成员的时候，并不需要经过对方同意。
- 高级群
高级群在权限上有更多的限制，权限分为群主、管理员、以及群成员。在添加成员的时候需要对方接受邀请。
- 群操作权限对比

| 权限   | 普通群             | 高级群     |
| :--| :---------------| :---------------|
| 邀请成员 | 群主、普通成员 | 群主、管理员 |
| 踢出成员 | 群主 | 群主、管理员(管理员之间无法互相踢) |
| 解散群 | / | 群主 |
| 退群 | 群主、普通成员 |  管理员、普通成员 |
| 处理入群申请 | / | 群主、管理员 |
| 更改他人群昵称 | / | 群主、管理员 |
| 更改群名称 | 群主、普通成员 | 群主、管理员 |
| 更改群公告 | / | 群主、管理员 |
| 更改群介绍 | / | 群主、管理员 |
| 更新验证方式 | / | 群主、管理员 |
| 添加(删除)管理员 | / | 群主 |
| 移交群主 | / | 群主 |

### <span id="群聊消息">群聊消息</span>

群聊消息收发和管理和单人聊天完全相同，仅在 `SessionTypeEnum` 上做了区分，详见[基础消息功能](#基础消息功能)节。

### <span id="关闭群聊消息提醒">关闭群聊消息提醒</span>

群聊消息提醒可以单独打开或关闭，关闭提醒之后，用户仍然会收到这个群的消息。如果开发者使用的是云信内建的消息提醒，用户收到新消息后不会再用通知栏提醒，如果用户使用的 iOS 客户端，则他将收不到该群聊消息的 APNS 推送。如果开发者自行实现状态栏提醒，可通过 `Team` 的 `mute` 接口获取提醒配置，并决定是不是要显示通知。
开发者可通过调用一下接口打开或关闭群聊消息提醒：

```java
NIMClient.getService(TeamService.class).muteTeam(teamId, mute);
```

### <span id="获取群组">获取群组</span>

SDK 提供了两个获取自己加入的所有群的列表的接口，一个是获取所有群（包括高级群和普通群）的接口，另一个是根据类型获取列表的接口。开发者可根据实际产品需求选择使用。
获取所有群：

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

普通群所有人都可以拉人入群，高级群仅管理员和拥有者可以邀请人入群，接口均为：

```java
NIMClient.getService(TeamService.class).addMembers(teamId, accounts)
	.setCallback(new RequestCallback<Void>() { ... });
```

普通群可直接将用户拉入群聊。
高级群不能直接拉入，被邀请的人会收到一条系统通知 (`SystemMessage`)，类型为 `SystemMessageType#TeamInvite`。用户接受该邀请后，才真正入群。
用户入群（普通群被拉人，高级群接受邀请）后，在收到之后的第一条消息时，群内所有成员（包括入群者）会收到一条入群消息，附件类型为 `MemberChangeAttachment`。

### <span id="踢人出群">踢人出群</span>

普通群仅拥有者可以踢人，高级群拥有者和管理员可以踢人，且管理员不能踢拥有者和其他管理员。

```java
NIMClient.getService(TeamService.class).removeMember(teamId, account)
	.setCallback(new RequestCallback<Void>() { ... });
```

踢人后，群内所有成员(包括被踢者)会收到一条消息类型为 `notification` 的 `IMMessage`，附件类型为 `MemberChangeAttachment`。

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

普通群所有人均可以修改群名，高级群仅拥有者和管理员可修改群名及其他群资料：

```java
// 每次仅修改群的一个属性，可修改的属性包括：群名，介绍，公告，验证类型等。
NIMClient.getService(TeamService.class).updateTeam(teamId, TeamFieldEnum, value)
	.setCallback(new RequestCallback<Void>() {...});
```
当然，SDK 也支持批量更新群组资料，可一次性更新多个字段的值，见 `TeamService#updateTeamFields`。

修改后，群内所有成员会收到一条消息类型为 `notification` 的 `IMMessage`，带有一个消息附件，类型为 `UpdateTeamAttachment`。如果注册了群组资料变化观察者，观察者也会收到通知，见[监听群组资料变化](#监听群组资料变化)。

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

群被解散后，群内所有成员都会收到一条消息类型为 `notification` 的 `IMMessage`，带有一个消息附件，类型为 `DismissAttachment`，原群主为该消息的 fromAccount。如果注册了群组被移除的观察者，这个观察者会受到通知。

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
}
/**
 * 拥有者撤销管理员权限 <br>
 * @param teamId 群ID
 * @param managers 待撤销的管理员的帐号列表
 * @return InvocationFuture 可以设置回调函数，如果成功，参数为被撤销的群成员列表(权限已被降为Normal)。
 */
 NIMClient.getService(TeamService.class)
	.removeManagers(teamId, managers)
	.setCallback(new RequestCallback<List<TeamMember>>() {
}
```

操作完成后，群内所有成员都会收到一条消息类型为 `notification` 的 `IMMessage`，附件类型为 `MemberChangeAttachment`。

### <span id="获取群组成员">获取群组成员</span>

由于群组成员数据比较大，且除开进入群组成员列表界面外，其他地方均不需要群组成员列表的数据，因此 SDK 不会在登录时同步群组成员数据，而是按照按需获取的原则，当上层主动调用获取指定群的群组成员列表时，才判断是否需要同步。获取群组成员的示例代码如下：

```java
// 该操作有可能只是从本地数据库读取缓存数据，也有可能会从服务器同步新的数据，
// 因此耗时可能会比较长。
NIMClient.getService(TeamService.class).queryMemberList(teamId)
	.setCallback(new RequestCallback<List<TeamMember>>() {
            @Override
            public void onSuccess(List<TeamMember> members) {
                updateTeamMember(members);
            }
        });
```

### <span id="查询高级群资料">查询高级群资料</span>

除了管理员邀请，用户也可以主动申请加入高级群。用户可以通过群号查询高级群信息：

```java
NIMClient.getService(TeamService.class).searchTeam(teamId)
	.setCallback(new RequestCallback<Team>() { ... });
```

## <span id="系统通知">系统通知</span>

### <span id="内置">内置</span>

系统通知是网易云信系统内建的消息/通知，其对应的数据结构为 
`SystemMessage`。由网易云信服务器推送给用户的通知类消息，用于网易云信系统类的事件通知。现在主要包括群变动的相关通知，例如入群申请，入群邀请等，如果第三方应用还托管了好友关系，好友的添加、删除也是这个类型的通知。系统通知由 SDK 负责接收和存储，并提供较简单的未读数管理。

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
注意：系统通知目前不提供按照类型查询，如果无法满足您的需求，可以在查询出所有类型的系统通知后自行分类型存储。

```java
List<SystemMessage> temps = NIMClient.getService(SystemMessageService.class)
	.querySystemMessagesBlock(offset, limit); // 参数offset为当前已经查了offset条，limit为要继续查询limit条。
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

- 清空所有系统通知

```java
NIMClient.getService(SystemMessageService.class).clearSystemMessages();
```

- 查询系统通知未读数
`SystemMessage` 中属性 unread 用来标志该条系统通知是否未读，该函数将返回所有未读的系统通知总数。

```java
int unread = NIMClient.getService(SystemMessageService.class)
	.querySystemMessageUnreadCountBlock();
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

### <span id="自定义">自定义</span>

系统通知属于网易云信的体系内，如果第三方 APP 需要自己的系统通知，可使用自定义通知，其数据结构为 `CustomNotification`。

自定义通知提供的灵活性包括：
- 消息格式由第三方 APP 自己定义，只要内容是 `String` 就可以了。
- 第三方 APP 的客户端和服务器均可以发送自定义通知。
- 接收对象可以是个人，也可以是群组。
- 可设置通知的到达级别：保证必达，或是通知接收者只有当前在线才能收到。
- 如果需要向 iOS 用户推送，可自定义 iOS 的推送内容。

注意：自定义通知和自定义消息的不同之处在于，自定义消息归属于网易云信的消息体系内，适用于会话中，由 SDK 存储在消息数据库中，与网易云信的其他内建消息类型一同展现给用户。而自定义通知主要用于第三方的一些事件状态通知，网易云信不存储，也不解释这些通知，网易云信仅仅负责替第三方传递和通知这些事件，起到透传的作用。

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

// 发送自定义通知
NIMClient.getService(MsgService.class).sendCustomNotification(notification);
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
注意：当前版本，好友关系托管仅提供添加、删除及获取我的好友列表功能，暂不支持扩展。

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

删除好友后，将自动解除双方的好友关系，双方的好友列表中均不存在对方。

```java
NIMClient.getService(FriendService.class).deleteFriend(account)
	.setCallback(new RequestCallback<Void>() { ... }
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

#### 判断用户是否为我的好友

```java
boolean isMyFriend = NIMClient.getService(FriendService.class).isMyFriend(account);
```

### <span id="黑名单">黑名单</span>

将用户加入黑名单后，将不在收到对方发来的任何消息或者请求。

#### 加入黑名单

```java
NIMClient.getService(FriendService.class).addToBlackList(account)
    .setCallback(new RequestCallback<Void>() { ... }

```
#### 移出黑名单

```java
NIMClient.getService(FriendService.class).removeFromBlackList(user.getAccount())
	.setCallback(new RequestCallback<Void>() { ... }
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

网易云信支持对用户设置或关闭消息提醒，关闭后，收到该用户发来的消息时，不再进行通知栏消息提醒。

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


## <span id="用户资料托管">用户资料托管</span>

网易云信提供了用户资料托管( `NimUserInfo` )，第三方 APP 可以使用也可以自行实现，但必须实现 `UserInfo` 接口。

### <span id="获取本地用户资料">获取本地用户资料</span>

#### 从本地数据库中获取用户资料

网易云信在登录之后，会同步好友的用户资料数据并保存在本地数据库中。可以通过用户帐号，从本地数据库获取用户资料，该接口为同步接口。代码示例如下：

```java
NimUserInfo user = NIMClient.getService(UserService.class).getUserInfo(account);
```

#### 从本地数据库中批量获取用户资料

通过用户帐号集合，从本地数据库批量获取用户资料列表。代码示例如下：

```java
List<NimUserInfo> users = NIMClient.getService(UserService.class).getUserInfo(accounts);
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
List<NimUserInfo> users = NIMClient.getService(UserService.class).getUserInfo(accounts); // 获取所有好友用户资料
```

### <span id="获取服务器用户资料">获取服务器用户资料</span>

从服务器获取用户资料，一般在本地用户资料不存在时调用，获取后 SDK 会负责更新本地数据库。代码示例如下：

```java
NIMClient.getService(UserService.class).fetchUserInfo(accounts)
    .setCallback(new RequestCallback<List<UserInfo>>() {...});
```
此接口可以批量从服务器获取用户资料，从用户体验和流量成本考虑，不建议应用频繁调用此接口。对于用户数据实时性要求不高的页面，应尽量调用读取本地缓存接口。

### <span id="编辑用户资料">编辑用户资料</span>

#### 更新用户本人资料

传入参数 `Map<UserInfoFieldEnum, Object>` 更新用户本人资料，key 为字段，value 为对应的值。具体字段见 `UserInfoFieldEnum`，包括：昵称，性别，头像 URL，签名，手机，邮箱，生日以及扩展字段等。代码示例如下：

```java
Map<UserInfoFieldEnum, Object> fields = new HashMap<>(1);
fields.put(UserInfoFieldEnum.Name, "new name");
NIMClient.getService(UserService.class).updateUserInfo(fields)
    .setCallback(new RequestCallbackWrapper<Void>() {...});
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

网易云信提供基于网络的点对点的语音、视频聊天功能。支持通话中音视频设备的控制，并支持音视频切换。

### <span id="语音视频通话配置">语音视频通话配置</span>

使用音视频功能，需要在 `AndroidManifest.xml` 文件中配置接收器。

```xml
<!-- 申明实时音视频来电通知的广播接收器，第三方APP集成时，
    action中的com.netease.nim.demo请替换为自己的包名 -->
<receiver
    android:name="com.netease.nimlib.receiver.AVChatBroadcastReceiver"
    android:enabled="true"
    android:exported="false">
    <intent-filter>
        <action android:name="com.netease.nim.demo.ACTION.RECEIVE_AVCHAT_CALL_NOTIFICATION"/>
    </intent-filter>
</receiver>

<!-- 申明本地电话状态（通话状态）的广播接收器，第三方APP集成时音视频模块时，如果需要网络通话与本地电话互斥，请加上此接收器 -->
<receiver android:name="com.netease.nimlib.receiver.IncomingCallReceiver">
    <intent-filter>
        <action android:name="android.intent.action.PHONE_STATE"/>
    </intent-filter>
</receiver>
```

### <span id="语音视频通话流程">语音视频通话流程</span>

#### 发起通话（主叫方）

主叫方可以发起语音或者视频通话，通话类型见 `AVChatTypeEnum`，如果发起的是视频通话，需要传入 `VideoChatParam`，其中包含视频采集用的 SurfaceView（一般只需要在界面布局里放置一个 1×1 的 SurfaceView）及视频旋转角度，如果发起的是语音通话，该参数填 null。

```java
AVChatManager.getInstance().call(account, callType, videoParam, new AVChatCallback<AVChatData>() {}
```

#### 监听来电（被叫方）

一般是在 APP 启动时注册来电监听，例如在 Application 的 onCreate 里添加，这里可以进行铃声相关的配置，见 `AVChatRingerConfig` 类，当监听到来电时，会返回来电信息 `AVChatData`，其中包含呼叫方式（音频或者视频）、来电帐号。

```java
private void enableAVChat() {
    setupAVChat();
    registerAVChatIncomingCallObserver(true);
}

private void setupAVChat() {
    AVChatRingerConfig config = new AVChatRingerConfig();
    config.res_connecting = R.raw.avchat_connecting;
    config.res_no_response = R.raw.avchat_no_response;
    config.res_peer_busy = R.raw.avchat_peer_busy;
    config.res_peer_reject = R.raw.avchat_peer_reject;
    config.res_ring = R.raw.avchat_ring;
    AVChatManager.getInstance().setRingerConfig(config); // 铃声配置
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

当监听到来电后启动通话界面，被叫方可以选择接听或者拒绝。当选择接听时，如果是视频通话，那么同样需要传入 `VideoChatParam`，SDK 会自动开启音视频设备，建立通话连接。
在某些特殊情况下，有可能音视频启动失败，此时会回调 onFailed ，错误码-1表示初始化引擎失败。
注意：由于音视频引擎析构需要时间，请尽可能保证上一次通话挂断到本次电话接听时间间隔在2秒以上，否则有可能在接听时出现初始化引擎失败（code = -1），此问题后期会进行优化。

```java
AVChatManager.getInstance().accept(videoParam, new AVChatCallback<Void>() {
}
```

#### 拒绝接听（被叫方）

```java
AVChatManager.getInstance().hangUp(new AVChatCallback<Void>() {
}
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

#### <span id="发送通话控制信息">发送通话控制信息 </span>

双方通话建立之后，就可以相互发送通话控制信息了。

```java
// 开启本地视频
AVChatManager.getInstance().toggleLocalVideo(true, null);

// 关闭本地视频
AVChatManager.getInstance().toggleLocalVideo(false, null);
```

#### <span id="请求音视频切换">请求音视频切换</span>

双方通话建立之后，就可以发起音视频切换请求。发送音频到视频的切换请求，对方需要同意才能切换成功；发送视频到音频的切换请求，对方默认同意，自动切换。

```java
// 请求音频切换到视频，需要传入VideoChatParam
AVChatManager.getInstance().requestSwitchToVideo(videoParam, new AVChatCallback<Void>() {}

// 请求视频切换到音频
AVChatManager.getInstance().requestSwitchToAudio(new AVChatCallback<Void>() {}
```

#### <span id="音视频切换请求的回应">音视频切换请求的回应</span>

一方发送音视频切换请求之后，对方会收到通知，见一下小节。当收到对方音频切换到视频的请求时，用户可以选择同意或者拒绝。对方将收到应答结果的通知，见下一小节。

```java
// 同意音频切换到视频
AVChatManager.getInstance().ackSwitchToVideo(true, videoParam, new AVChatCallback<Void>() {}

// 拒绝音频切换到视频
AVChatManager.getInstance().ackSwitchToVideo(false, videoParam, null);
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

### <span id="当前通话信息">当前通话信息</span>

实现 `AVChatStateObserver` 监听通话过程中状态变化。被叫方同意来电请求后，SDK 自动进行音视频服务器连接，并返回相应信息供上层应用使用。

```java
public class AVChatActivity implements AVChatStateObserver {
	AVChatManager.getInstance().observeAVChatState(this, register);
} 
```

#### 当前音视频服务器连接回调

首先返回服务器连接是否成功的回调 `onConnectedServer`，并通过返回的 result code 做相应的处理。

```java
@Override
public void onConnectedServer(int res) {
   handleWithConnectServerResult(res);
}

/**
* @param auth_result
* 200 连接成功
* 101 连接超时
* 417 无效频道
* 401 验证失败
* 400 格式不对
*/
protected void handleWithConnectServerResult(int auth_result) {}
```

#### 加入当前音视频频道用户帐号回调

音视频服务器连接成功后，会回调 `onUserJoin`，可以获取当前通话的用户帐号。

```java
@Override
public void onUserJoin(String account) {}
```

#### 当前用户离开频道回调

通话过程中，若有用户离开，则会回调 `onUserLeave`。

```java
// @param event   －1,用户超时离开  0,正常退出
@Override
public void onUserLeave(String account, int event) {}
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

通话过程中网络状态发生变化，会回调 `onNetworkStatusChange`。

```java
// @param value 0~3 ,the less the better; 0 : best; 3 : worst
@Override
public void onNetworkStatusChange(int value) {}
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

#### 打开音视频设备出错回调

音视频设备打开错误，会回调 `onOpenDeviceError`。

```java
/**
* 1：音频录制错误
* 2：音频播放错误
* 4：视频采集错误
* 8：视频渲染错误
*/
@Override
public void onOpenDeviceError(int code) {}

```

#### 本地及远程摄像头图像获取

发起视频请求时，需要传递采集点 SurfaceView 给 SDK。SDK 会自动采集图像。界面需要显示图像时，通过用户帐号获取采集好的图像 SurfaceView。

```java
AVChatManager.getInstance().getSurfaceRender(account);
```

### <span id="通话中的设备控制">通话中的设备控制</span>

通话进行中，可以进行设备静音，扬声器，摄像头切换，开关摄像头和切换通话模式的设置。

#### 设置静音

```java
if(!AVChatManager.getInstance().isMute()) { // isMute是否处于静音状态
	// 关闭音频
	AVChatManager.getInstance().setMute(true);
} else {
	// 打开音频
	AVChatManager.getInstance().setMute(false);
}
```

#### 设置扬声器

```java
// 设置扬声器是否开启
AVChatManager.getInstance().setSpeaker(!AVChatManager.getInstance().speakerEnabled());
```

#### 切换摄像头

```java
// 切换摄像头（主要用于前置和后置摄像头切换）
AVChatManager.getInstance().toggleCamera();
```

#### 关闭/开启摄像头

具体见[发送通话控制信息](#发送通话控制信息)

#### 切换通话模式

具体见[请求音视频切换](#请求音视频切换) 和 [音视频切换请求的回应](#音视频切换请求的回应)

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
一个会话可以同时创建多个通道，但全局只能有一个语音通道。发起会话时，RTSOptions 中，参数 pushContent  填入要推送到 iOS 端的文本（不需要 填 null），参数 extra 填写发起方附带给其他参与者的内容,开发者可以封装自己的数据。
例如：

```java

List<RTSTunType> types = new ArrayList<>(1);
types.add(RTSTunType.AUDIO);
types.add(RTSTunType.TCP);

String pushContent = account + "发起一个会话";
String extra = "extra_data";
RTSOptions options = new RTSOptions().setPushContent(pushContent).setExtra(extra).setRecordAudioTun(true)
            .setRecordTCPTun(true);

sessionId = RTSManager.getInstance().start(account, types, options, new RTSCallback<RTSData>() { ... }
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
RTSManager.getInstance().accept(sessionId, options, new RTSCallback<Boolean>() { ... }
```
如果回调 onFailed 返回 -1，表示本地正在使用语音通道，不能同时存在两个语音通道。

#### 拒绝会话（被叫方）

```java
RTSManager.getInstance().close(sessionId, new RTSCallback<Void>() { ... }
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
RTSManager.getInstance().sendControlCommand(sessionId, content, new RTSCallback<Void>() { ... }
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
    public void onConnectResult(RTSTunType tunType, int code) {
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
RTSManager.getInstance().close(sessionId, new RTSCallback<Void>() { ... }
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

## <span id="API 文档"> API 文档 </span>
* [在线文档](http://dev.netease.im/doc/android/index.html "target=_blank")
