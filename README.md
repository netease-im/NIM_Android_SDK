# 网易云信安卓SDK开发指南

## 简介

网易云信SDK为移动应用提供一个完善的IM系统开发框架，屏蔽掉IM系统的复杂的细节，对外提供较为简介的API接口，方便第三方应用快速集成IM功能。SDK提供的能力如下：

- 网络连接管理
- 登录认证服务
- 消息传输服务
- 群组管理/消息服务
- 消息存储服务
- 实时音视频通话

## 集成SDK

首先从[网易云信官网](http://netease.im/base.html?page=download  "target=_blank")下载Android SDK。根据实际需求，我们提供了两个版本以供下载：
- 基础版本：包含基础的在线消息，群组功能。
- 实时音视频通话版本：除基础功能外，还包含在线音视频通话功能，SDK体积会大不少，用户可按需使用。

### 类库配置

 sdk包的libs文件夹中，包含了网易云信的jar文件，各jni库文件夹以及sdk依赖的第三方库，列表如下：
 
```
libs
├── armeabi
│   ├── libne_audio.so
│   ├── librtc_engine.so
│   └── librtc_network.so
├── armeabi-v7a
│   ├── libne_audio.so
│   ├── librtc_engine.so
│   └── librtc_network.so
├── x86
│   ├── libne_audio.so
│   ├── librtc_engine.so
│   └── librtc_network.so
├── nim-sdk-1.0.0.jar
└── netty-4.0.23-for-yx.final.jar
```

将这些文件拷贝到你的工程的libs目录下，即可完成配置。

以上文件列表中，nim-sdk-1.0.0.jar(版本号可能会不同)为网易云信SDK，子目录中是SDK所依赖的各个CPU架构的jni库。 如果下载的只是基础版本，则so库只有libne_audio.so一个，没有librtc\*.so。

如果第三方app的libs里面只包含armeabi一个文件夹，为了保证在arm-v7a上有较好的性能，以及兼容各个平台，可将各目录下的so文件名改为原文件名加上"_{arch_of_cpu}"，然后统一放到armeabi目录下，SDK也会加载到正确版本的so库。改名后的目录结构如下：

```
libs
├── armeabi
│   ├── libne_audio_armeabi.so
│   ├── librtc_engine_armeabi.so
│   ├── librtc_network_armeabi.so
│   ├── libne_audio_armeabi-v7a.so
│   ├── librtc_engine_armeabi-v7a.so
│   └── librtc_network_armeabi-v7a.so
├── x86(如果不考虑支持x86，可不包含)
│   ├── libne_audio_x86.so
│   ├── librtc_engine_x86.so
│   └── librtc_network_x86.so
├── nim-sdk-1.0.0.jar
└── netty-4.0.23-for-yx.final.jar
```

网易云信SDK的网络连接还依赖于netty框架，在上面的文件夹中，我们也包含了netty的4.0.23版本的jar包，这个jar包我们有一些修改，主要是增加Android兼容性，去掉了一些依赖反射的调用，以及性能上的一些优化。如果你的app也依赖于netty，你可以使用这个修改后的版本，也可以使用官方的原始版本。

如果你使用的IDE是Android Studio，要将jni库按照IDEA工程目录的结构，放置在对应的目录中（一般为src/main/jniLibs）。或者在build.gradle中配置好jniLibs的sourceSets（可参考demo的build.gradle）。
   
### 权限及组件声明

在`AndroidManifest.xml`中加入以下配置:

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

    <!-- SDK权限申明, 第三方APP接入时，请将com.netease.nim.demo替换为自己的包名 -->
    <!-- 和下面的uses-permission一起加入到你的AndroidManifest文件中。 -->
    <permission
        android:name="com.netease.nim.demo.permission.RECEIVE_MSG"
        android:protectionLevel="signature"/>
    <!-- 接收SDK消息广播权限， 第三方APP接入时，请将com.netease.nim.demo替换为自己的包名 -->
     <uses-permission android:name="com.netease.nim.demo.permission.RECEIVE_MSG"/>

    <application
        ...>
        <!-- app key, 可以在这里设置，也可以在SDKOptions中提供。如果SDKOptions中提供了，取SDKOptions中的值。 -->
        <meta-data
            android:name="com.netease.nim.appKey"
            android:value="key_of_your_app" />

        <!-- NimService组件。 如需保持后台推送，使用独立进程效果会更好。-->
        <service android:name="com.netease.nimlib.service.NimService"
            android:process=":core"/>
        <!-- 运行后台辅助服务 -->
        <service
            android:name="com.netease.nimlib.service.NimService$Aux"
            android:process=":core"/>

        <!-- 状态监听Receiver，保持和NimService同一进程 -->
        <receiver android:name="com.netease.nimlib.service.NimReceiver"
            android:process=":core"
            android:exported="false"
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
            </intent-filter>
        </receiver>

    </application>
</manifest>
```

### 混淆配置

如果你的apk最终会经过代码混淆，请在proguard配置文件中加入以下代码:

```
-dontwarn com.netease.**
-dontwarn io.netty.**
-keep class com.netease.** {*;}
#如果netty使用的官方版本，它中间用到了反射，因此需要keep。如果使用的是我们提供的版本，则不需要keep
-keep class io.netty.** {*;}
```

### <span id="总体接口介绍">总体接口介绍</span>

网易云信SDK提供了两类接口供开发者调用：一类是第三方app主动发起请求，第二类是第三方app作为观察者监听事件和变化。第一类接口名均以**Service**结尾，例如`AuthService`，第二类接口名均以**ServiceObserver**结尾，例如`AuthServiceObserver`，个别太长的类名则可能直接以**Observer**结尾，比如`SystemMessageObserver`。

可以通过`NIMClient`的`getService`接口获取到各个服务实例，例如：通过`NIMClient.getService(AuthService.class)`获取到`AuthService`服务实例，通过`NIMClient.getService(AuthServiceObserver.class)`获取到`AuthServiceObserver`观察者接口。

SDK由于需要保持后台运行，典型场景下会在独立进程中运行，第一类接口基本上都是从主进程发起调用，然后在后台进程执行，最后再将结果返回给主进程。因此，如无特殊说明，所有接口均为异步调用，开发者无需担心调用SDK接口阻塞UI的问题。如果调用的接口需要一些后期操作，包括结果回调，取消调用等，此类接口会返回一个`InvocationFuture`对象。如果开发者关心调用的结果，可在此返回的接口中设置回调函数。
接口回调接口为`RequestCallback`，有3个接口需要实现：onSuccess, onFailed, onException，分别对应成功，失败，以及出现异常。另外还提供了一个更简洁的接口 `RequestCallbackWrapper`，作为`RequestCallback`的包裹，封装了上面的3个接口，然后将他们合并成一个onResult接口，在参数上做不同结果的区分。

还有一类接口，耗时会很长，可能需要传输大量数据，或者要发起网络连接，比如上传下载，登录等。对于这类接口，返回值会是一个`AbortableFuture`对象，该接口继承自`InvocationFuture`，增加了一个`abort()`方法，可取消之前的请求。

观察者接口的方法名都是以observe开头，并包含一个register参数，该值为`true`时，为注册观察者，为`false`时，注销观察者。开发者要在不需要观察者时，主动注销，以免造成资源泄露。

>注意：除开`NIMClient.init`接口外，其他SDK暴露的接口都只能在UI进程调用。如果app包含远程service，该app的Application的onCreate会多次调用。因此，如果需要在onCreate中调用出init接口外的其他接口，应先判断当前所属进程，并只有在当前是UI进程时才调用。判断代码如下：

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

SDK提供的接口主要按照业务进行分类，大致说明如下：
- `AuthService`：用户认证服务接口，提供登录注销接口。
- `AuthServiceObserver`: 用户状态观察者接口。
- `MsgService`: 消息服务接口，用于发送消息，管理消息记录等。同时还提供了发送自定义通知的接口。
- `MsgServiceObserver`: 接收消息，消息状态变化等观察者接口。
- `TeamService`: 群组服务接口，用于发送群组消息，管理群组和群成员资料等。
- `TeamServiceObserver`: 群组和群成员资料变化观察者。
- `SystemMessageService`: 管理系统通知接口。
- `SystemMessageObserver`: 接收系统通知观察者。

### SDK数据目录结构

当收到多媒体消息后，SDK会负责下载这些多媒体文件，同时SDK还要记录一些log，因此SDK需要一个数据缓存目录。该目录由第三方app通过`SDKOptions`传入，默认为“/{外卡根目录}/{app\_package\_name}/nim/”。如果第三方app需要清除缓存功能，可扫描该目录下的文件，按照你们的规则清理即可。
缓存目录下面包含如下子目录：
- log: 包含一个文件：nim\_sdk.log，大小一般不超过8M
- file: 文件消息文件
- image: 图片消息文件
- audio：语音消息文件
- video：视频消息文件
- thumb：图片/视频缩略图文件
	
## 初始化SDK

在你的程序的Application的`onCreate`中，加入网易云信SDK的初始化代码：

```
public class NimApplication extends Application {

	public void onCreate() {
		// ... your codes
		NIMClient.init(this, loginInfo(), options());
		// ... your codes
	}

	// 如果返回值为null，则全部使用默认参数。
	private SDKOptions options() {
		SDKOptions options = new SDKOptions();

	    // 如果将新消息通知提醒托管给SDK完成，需要添加以下配置。否则无需设置。
	    // 其中notificationSmallIconId必须提供
	    StatusBarNotificationConfig config = new StatusBarNotificationConfig();
	    config.notificationEntrance = WelcomeActivity.class;
	    config.notificationSmallIconId = R.drawable.ic_stat_notify_msg;
	    options.statusBarNotificationConfig = config;

	    // 配置保存图片，文件，log等数据的目录
	    // 如果options中没有设置这个值，SDK会使用下面代码示例中的位置作为sdk的数据目录。
	    // 该目录目前包含log, file, image, audio, video, thumb这6个目录。
	    // 如果第三方APP需要缓存清理功能， 清理这个目录下面个子目录的内容即可。
	    String sdkPath = Environment.getExternalStorageDirectory() + "/" + getPackageName() + "/nim";
	    options.sdkStorageRootPath = sdkPath;

	    // 配置是否需要预下载附件缩略图，默认为true
	    options.preloadAttach = true;

	    // 配置附件缩略图的尺寸大小，该值一般应根据屏幕尺寸来确定， 默认值为 Screen.width / 2
	    options.thumbnailSize = ${Screen.width} / 2;

        // 用户信息提供者, 目前主要用于新消息通知栏中，显示消息来源的头像和昵称，此项如果为null，则显示程序图标
        options.userInfoProvider = new UserInfoProvider() {
                     @Override
                     public UserInfo getUserInfo(String account) {
                        return null;
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

再次提醒，如果你的app会启动多个进程，此处无需做特殊处理，在每次Application的`onCreate`调用初始化代码即可。SDK会处理好进程关系，并只在需要的进程中初始化。但是，要注意不要在SDK进程中再调用各种service提供的接口，也不要在sdk进程中做UI相关的初始化动作，以免造成资源浪费。判断当前进程是否是在主进程的代码见[总体接口介绍](#总体接口介绍)。

## 登录

如果已经存在用户登录信息，那么在初始化SDK时传入`LoginInfo`，SDK后台会自动登录，并在登录发起前即打开相关数据库，供上层调用。开发者无需在手动调用登录接口。
如果不存在登录信息，开发者可在合适的位置调用`AuthService`提供的`login`接口主动发起登录请求。该接口返回类型为`AbortableFuture`，允许用户在后面取消登录操作。如果服务器一直没有响应，30秒后RequestCallback的onFailed会被调用，参数为408（网络连接超时）。

```java
public class LoginActivity extends Activity {
	
	public void doLogin() {
		LoginInfo info = new LoginInfo(); // config...
		RequestCallback<LoginInfo> callback = 
			new RequestCallback<LoginInfo>() {
			// overwrite methods
		};
		NIMClient.getService(AuthService.class).login(info)
				.setCallback(callback);
	}
}
```

登录成功后，SDK会负责维护与服务器的长连接以及断线重连等工作。当用户状态发生改变时，会发出通知。开发者可以通过加入以下代码监听用户状态改变：

```java
NIMClient.getService(AuthServiceObserver.class).observeUserStatus(
	new Observer<StatusCode> () {
		public void onEvent(StatusCode status) {
			Log.i("tag", "User status changed to: " + status);
		}
}, true);
```

如果用户手动注销，不再接收消息和提醒，开发者可以调用logout方法，该方法没有回调。

```java
NIMClient.getService(AuthService.class).logout();
```

## <span id="消息服务">消息服务</span>

SDK提供一套完善的消息传输管理服务，包括收发消息，存储消息，上传下载附件，管理最近联系人等。
SDK原生支持发送文本，语音，图片，视频和地理位置等5种类型消息，同时支持用户发送自定义的消息类型。
网易云信消息对象均为`IMMessage`，不同消息类型以`MsgTypeEnum`作区分，消息内容根据类型不同也不一样。文本消息最为简单，消息内容就是`content`字符串。其他消息类型均带有一个消息附件对象`MsgAttachment`，该对象在传输时一般序列化为json格式字符串。内建的消息附件主要有以下几种：
- LocationAttachment： 位置消息附件对象类型。
- FileAttachment: 文件消息附件对象类型
- ImageAttachment：图片消息附件对象类型。
- AudioAttachment：音频消息附件对象类型。
- VideoAttachment：视频消息附件对象类型。
- NotificationAttachment：通知类消息附件基类，目前主要用于群类各种操作事件的通知。该类型事件仅由服务器生成和发送，客户端不会生成该类型的消息。

### <span id="发送消息">发送消息</span>

发送原生支持的消息类型流程比较简单，先通过`MessageBuilder`提供的接口创建消息对象，然后调用`MsgService`的`sendMessage`接口发送出去即可。下面给出发送各种类型消息的示例代码：

```java
// 创建文本消息
IMMessage message = MessageBuilder.createTextMessage(
	sessionId, // 聊天对象的ID，如果是单聊，为用户账号，如果是群聊，为群组ID
	sessionType, // 聊天类型，单聊或群组
	content // 文本内容
	);

// 创建地理位置消息
IMMessage message = MessageBuilder.createLocationMessage(
	sessionId, // 聊天对象的ID，如果是单聊，为用户账号，如果是群聊，为群组ID
	sessionType, // 聊天类型，单聊或群组
	latitude, // 纬度
	longitude, // 经度
	address // 地址信息描述
	);

// 创建图片消息
IMMessage message = MessageBuilder.createImageMessage(
	sessionId, // 聊天对象的ID，如果是单聊，为用户账号，如果是群聊，为群组ID
	sessionType, // 聊天类型，单聊或群组
	file, // 图片文件对象
	displayName // 文件显示名字，如果第三方APP不关注，可以为null
	);

// 创建音频消息
IMMessage message = MessageBuilder.createAudioMessage(
    sessionId, // 聊天对象的ID，如果是单聊，为用户账号，如果是群聊，为群组ID
	sessionType, // 聊天类型，单聊或群组
    file, // 音频文件
    duration // 音频持续时间
    );

// 创建视频消息
IMMessage message = MessageBuilder.createVideoMessage(
	sessionId, // 聊天对象的ID，如果是单聊，为用户账号，如果是群聊，为群组ID
	sessionType, // 聊天类型，单聊或群组
	file, // 视频文件
	duration, // 视频持续时间
	width, // 视频宽度
	height, // 视频高度
	displayName // 视频显示名，可为空
	);

// 发送消息。如果需要关心发送结果，可设置回调函数。发送完成时，会收到回调。如果失败，会有具体的错误码。
NIMClient.getService(MsgService.class).sendMessage(message);
```

发送消息后，还需要持续关注消息的发送进度，以更新发送界面。发送消息的接口可以设置回调函数，但仅仅会通报发送结果，没有发送进度通知。开发者需要通过另一个观察者接口来完成。这样，开发者在任意界面都能接收到消息状态的改变。示例代码如下：

```java
NIMClient.getService(MsgServiceObserver.class).observeMsgStatus(
	new Observer<IMMessage> {
		public void onEvent(IMMessage message) {
			// 参数为有状态发生改变的消息对象，其msgStatus和attachStatus均为最新状态。
			// 发送消息和接收消息的状态监听均可以通过此接口完成。
		}
	}
}, true);
	
//如果发送的多媒体文件消息，还需要监听文件的上传进度。
NIMClient.getService(MsgServiceObserver.class).observerAttachProgress(
	new Observer(AttachProgress progress) {
		public void onEvent(AttachProgress progress) {
			// 参数为附件的传输进度，可根据progress中的uuid查找具体的消息对象，更新UI
			// 上传附件和下载附件的进度监听均可以通过此接口完成。
		}
	}, true);
```

### 接收消息

通过添加消息接收观察者，在有新消息到达时，第三方app就可以接收到通知。代码如下：

```java
Observer<List<IMMessage>> incomingMessageObserver =
	new Observer<List<IMMessage>>() {
	    @Override
	    public void onEvent(List<IMMessage> messages) {
		    // 处理新收到的消息，为了上传处理方便，SDK保证参数messages全部来自同一个聊天对象。
        }
    }
	NIMClient.getService(MsgServiceObserve.class)
		.observeReceiveMessage(incomingMessageObserver, true);
```

该代码的典型场景为消息对话界面，在界面`onCreate`里注册消息接收观察者，在`onDestroy`中注销观察者。在收到消息时，判断是否是当前聊天对象的消息，如果是，加入到列表中显示。
如果接收到消息是带文件附件的多媒体消息，SDK默认会在后台自动下载附件：如果是语音消息，直接下载文件，如果是图片或视频消息，下载缩略图文件。开发者可在`SDKOptions`中关闭自动下载，并在用户翻阅到对应消息，再通过以下代码手动下载。如果自动下载或手动下载失败，也可以通过这段代码重新下载：

```java
// 下载附件，参数1位消息对象，参数2为是下载缩略图还是下载原图。
// 因为下载的文件可能会很大，这个接口返回类型为AbortableFuture ，允许用户中途取消下载。
AbortableFuture future = NIMClient.getService(MsgService.class).downloadAttachment(message, true);
```

### 消息推送/提醒

SDK内置了新消息状态栏提醒功能，开发者可以在`SDKOptions`中开启或关闭，以及设置铃声/振动提醒，免打扰时段等选项。各选项配置可参见[在线API文档](http://dev.netease.im/doc/android/index.html) 中StatusBarNotificationConfig。同时，SDK也提供了更新消息提醒配置的接口：

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

除文本消息外，开发者可以通过`NimStrings`类修改这些默认提醒内容。
如果开发者需要为每条消息指定的具体的提醒内容，而不是显示默认内容，开发者可以调用`IMMessage`的`setContent`接口，对于文本消息，该接口会同时修改消息内容和提醒内容，对于其他格式消息，该接口仅仅修改提醒内容。
如果接收方是iOS客户端，消息推送的内容遵从相同的规则：如果设置了`setContent`字段，则使用设置的字符串作为推送内容，否则使用默认提醒内容。

如果用户正在与某一个人聊天，当这个人的消息到达时，是不应该有状态栏提醒的，如果用户停留在最近联系人列表界面，收到消息也不应该有提醒，因此，为了内置的新消息提醒功能正确工作，开发者需要在进入进出聊天界面以及最近联系人列表界面时，通知SDK。接口如下：

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

如果开发者希望自己定制消息提醒，SDK也能提供支持。由于接收消息是在SDK的service进程，消息到达的时候，第三方app进程可能没有运行，因此不能通过添加观察者的方式实现。为了实现此功能，SDK在新消息到达后，会发送一个广播，ACTION为App的包名加上`NimIntent#ACTION_RECEIVE_MSG`，例如demo的为："com.netease.nim.demo.ACTION.RECEIVE\_MSG"，开发者可在AndroidManifest.xml文件中注册此广播的接收者，在onReceive中实现自己的状态栏消息提醒。

### 管理消息记录

SDK提供了一个根据锚点查询消息历史记录的接口，根据提供的方向(direct)，查询比anchor更老（QUERY\_OLD）或更新（QUERY\_NEW）的最靠近anchor的limit条数据。调用者可使用asc参数指定结果排序规则，结果使用time作为排序字段。
当进行首次查询时，锚点可以用使用`MessageBuilder#createEmptyMessage`接口生成。查询结果不包含锚点。

```java
/**
 * @param anchor IMMessage 查询锚点
 * @param direction QueryDirectionEnum 查询方向
 * @param limit int 查询结果的条数限制
 * @param asc boolean 查询结果的排序规则，如果为true，结果按照时间升级排列，如果为false，按照时间降序排列
 * @return 调用跟踪，可设置回调函数，接收查询结果
 */
NIMClient.getService(MsgService.class).queryMessageListEx(anchor, direction, limit, asc);

```

SDK还提供了按照关键字搜索聊天记录的功能，可以对指定的聊天对象，输入关键字进行消息内容搜索。查询方向为从后往前。以锚点anchor作为起始点开始查询，返回最多limit条匹配key的记录。该接口目前仅搜索文本类型的消息，匹配规则为文本内容包含keyword，仅支持精确匹配，不支持模糊匹配和拼音匹配。
由于sdk并不存储用户数据，因此keyword不会匹配用户资料。如果调用者希望查询指定用户的说话记录，可提供fromAccounts参数。如果提供的fromAccounts参数不为空，那么anchor对应的会话（sessionId和sessionType）的消息记录中，凡是消息说话者在fromAccounts列表中的记录，也会被当做匹配结果，加入搜索结果中。

```java
/**
 * @param keyword String 文本消息的搜索关键字
 * @param fromAccounts List<String> 消息说话者账号列表，如果消息说话在该列表中，那么无需匹配keyword，对应的消息记录会直接加入搜索结果中。
 * @param anchor IMMessage 搜索的消息锚点
 * @param limit int 搜索结果的条数限制
 * @return 调用跟踪，可设置回调函数，接收查询结果
 */
 NIMClient.getService(MsgService.class).searchMessageHistory(keyword, fromAccounts, anchor, limit)
                .setCallback(new RequestCallbackWrapper<List<IMMessage>>() {
                });

```

删除消息记录：

```java
// 删除单条消息
NIMClient.getService(MsgService.class).deleteChattingHistory(message);
// 删除与某个聊天对象的全部消息记录
NIMClient.getService(MsgService.class).clearChattingHistory(account, sessionType);
```

### 最近联系人列表

最近联系人列表记录了与用户最近有过会话的联系人信息，包括联系人账号，联系人类型，最近一条消息的时间，消息状态，消息缩略，未读条数等信息。

```java
 NIMClient.getService(MsgService.class).queryRecentContacts()
	 .setCallback(new RequestCallbackWrapper<List<RecentContact>>() {
	   @Override
       public void onSuccess(List<RecentContact> contacts) {
       // 参数即为用户的最近联系人列表
       }
    });
```

在收发消息的同时，SDK会更新对应聊天对象的最近联系人资料。当有消息收发时，SDK会发出最近联系人更新通知：

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

如果用户在开始聊天时，开发者都调用了`setChattingAccount`接口，SDK会自动管理消息的未读条数。当收到新消息时，自动增加未读数，在`setChattingAccount`时，自动将未读数清零。如果第三方app需要不进入聊天窗口，就能主动将未读数清零，可以通过调用如下接口来实现，会触发`MsgServiceObserve#observeRecentContact(Observer, boolean)` 通知，通知中的RecentContact对象的未读数为0.

```java
NIMClient.getService(MsgService.class).clearUnreadCount(account, sessionType);
```

如果需要在最近联系人列表界面显示当前消息状态，还需要增加消息状态监听，操作见[发送消息](#发送消息) 一节。

### 自定义消息

除开内建消息类型外，SDK还支持收发自定义消息类型。SDK不负责定义和解析自定义消息的具体内容，解释工作由开发者完成。SDK会将自定义消息存入消息数据库，会和内建消息一并展现在消息记录中。
为了使用更加方便，自定义消息采用附件的方式展示给开发者。体现在`IMMessage`类中，自定义消息的内容会被解析为`MsgAttachment`对象，由于SDK并不知道自定义消息的格式，第三方APP需要注册一个自定义消息解析器。当第三方APP调用查询查询消息历史的接口，或者是SDK收到消息通知第三方APP时，就能将自定义消息内容转换为一个`MsgAttachment`对象，然后就可以同操作图片消息等带附件消息一样，操作自定义消息了。
为了增加自定义消息的灵活性，SDK还提供了对其生命周期和推送参数的一些配置选项，开发者可以自主设定一条自定义消息是否要保存云端消息记录，是否要漫游，发送时是否要多端同步等。详见 `CustomMessageConfig` api文档。

创建和发送自定义消息的代码如下：

```java
IMMessage message = MessageBuilder.createCustomMessage(
	sessionId, // 会话对象
	sessionType, // 会话类型
	attachment // 自定义消息附件
	)
NIMClient.getService(MsgService.class).sendMessage(message);
```

在demo中，我们提供了一个用自定义消息实现的“剪刀石头布”的游戏，下面以此为例，详细解析其实现步骤。

首先，我们先定义一个自定义消息附件的基类，负责解析你的自定义消息的公用字段，比如类型等等。还可以定义一些公共接口，用于一些便利性的调用。

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

	// 实现MsgAttachment的接口，封装公用字段，然后调用子类的封装函数。
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

最后，将该附件解析器注册到SDK中。为了保证生成历史消息时能够正确解析自定义附件，注册一般应放在Application的onCreate中完成。

```java
NIMClient.getService(MsgService.class).registerCustomAttachmentParser(new CustomAttachParser());
```

## 系统消息

系统消息是网易云信系统内建的消息/通知，其对应的数据结构为`SystemMessage`。由网易云信服务器推送给用户的通知类消息，用于网易云信系统类的事件通知。现在主要包括群变动的相关通知，例如入群申请，入群邀请等。类似于QQ中的系统消息。系统消息由SDK负责接收和存储，并提供较简单的未读数管理。

### 接收系统消息

开发者可通过`SystemMessageObserver`监听系统消息的到达事件：

```java
NIMClient.getService(SystemMessageObserver.class)
	.observeReceiveSystemMsg(new Observer<SystemMessage>() {
            @Override
            public void onEvent(SystemMessage message) {
            }
        }, register);
```

### 管理系统通知

查询系统消息列表：

```java
NIMClient.getService(SystemMessageService.class).querySystemMessages(offset, limit)
	.setCallback(new RequestCallbackWrapper<List<SystemMessage>>() {
    });
```

重置系统消息未读数：

```java
// 进入过系统消息列表后，可调用此函数将未读数值为0
NIMClient.getService(SystemMessageService.class)
	.resetSystemMessageUnreadCount();
```

## 自定义通知

系统消息属于网易云信的体系内，如果第三方APP需要自己的系统通知，可使用自定义通知，其数据结构为`CustomNotification`。

自定义通知提供的灵活性包括：
- 消息格式由第三方app自己定义，只要内容是`String`就可以了。
- 第三方app的客户端和服务器均可以发送自定义通知。
- 接收对象可以是个人，也可以是群组。
- 可设置通知的到达级别：保证必达，或是通知接收者只有当前在线才能收到。
- 如果需要向iOS用户推送，可自定义iOS的推送内容。

注意：自定义通知和自定义消息的不同之处在于，自定义消息归属于网易云信的消息体系内，适用于会话中，由SDK存储在消息数据库中，与网易云信的其他内建消息类型一同展现给用户。而自定义通知主要用于第三方的一些事件状态通知，网易云信不存储，也不解释这些通知，网易云信仅仅负责替第三方传递和通知这些事件，起到透传的作用。

### 发送自定义通知

通过SDK提供的接口，第三方app可以在客户端向其他用户或者群组发送自定义通知。SDK能发送的自定义通知主要分为两种。

一种是只有接收方当前在线才会收到，如果发送方发送时，指定的接收者不在线，这条通知将会被丢弃。在demo中，我们以此实现了"正在输入"这种状态的通知。

```java
// 构造自定义通知，指定接收者
CustomNotification notification = new CustomNotification();
notification.setSessionId(receiverId);
notification.setSessionType(sessionType);

// 构建通知的具体内容。为了可扩展性，这里采用json格式，以"id"作为类型区分。这里以类型“1”作为“正在输入”的状态通知。
JSONObject json = new JSONObject();
json.put("id", "1");
notification.setContent(json.toString());

// 发送自定义通知
NIMClient.getService(MsgService.class).sendCustomNotification(notification);
```

另外一种保证接收方一定会收到，如果接收方当前在线，会立即收到，如果当前不在线，则在下次登录后立即收到。如果接收方上次登录是iOS版本，还会收到APNS推送通知。
下面做了一个简单的示例：

```java
// 构造自定义通知，指定接收者
CustomNotification notification = new CustomNotification();
notification.setSessionId(receiverId);
notification.setSessionType(sessionType);

// 构建通知的具体内容。为了可扩展性，这里采用json格式，以"id"作为类型区分。
JSONObject json = new JSONObject();
json.put("id", "2");
JSONObject data = new JSONObject();
data.put("body", "the_content_for_display");
data.put("url", "url_of_the_game_or_anything_else");
json.put("data", data);
notification.setContent(json.toString());

// 设置该消息需要保证送达
notification.setSendToOnlineUserOnly(false);

// 设置apns的推送文本
notification.setApnsText("the_content_for_apns");

// 发送自定义通知
NIMClient.getService(MsgService.class).sendCustomNotification(notification);
```

### 接收自定义通知

上层有两种方式接收自定义通知，一是通过添加通知接收观察者的，二是通过广播的方式接收。SDK从版本1.4.0开始，推荐使用第一种方式接收。从这个版本起，收到消息后就会激活UI主进程，并通知到已注册的观察者。只要在主进程的入口添加自定义通知的观察者，就能收到该通知。
添加自定义通知的接收观察者代码如下：

```java
// 如果有自定义通知是作用于全局的，不依赖某个特定的Activity，那么这段代码应该在Application的onCreate中就调用
NIMClient.getService(MsgServiceObserve.class).observeCustomNotification(new Observer<CustomNotification>() {
    @Override
    public void onEvent(CustomNotification message) {
        // 在这里处理自定义通知。
    }
}, register);

```

由于SDK不存储自定义通知，因此收到自定义通知后，SDK会立即将这个通知转给第三方app。由于收到通知时，第三方app的主进程不一定存活，因此这里采用广播的方式通知。
为了接收自定义通知，首先需要在AndroidManifest.xml文件中声明一个接收器：

```xml
<!-- 声明自定义通知的广播接收器，第三方APP集成时，action中的com.netease.nim.demo请替换为自己的包名 -->
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

            // 从intent中取出自定义通知， intent中只包含了一个CustomNotification对象
            CustomNotification notification = (CustomNotification) intent.getSerializableExtra(NimIntent.EXTRA_BROADCAST_MSG);

            // 第三方app在此处理自定义通知：存储，处理，展示给用户等
            Log.i("demo", "receive custom notification: " + notification.getContent() + " from :" + notification.getSessionId() + "/" + notification.getSessionType());
        }
    }
```

## 群组服务

网易云信SDK提供了普通群(`TeamTypeEnum#Normal`)，以及高级群(`TeamTypeEnum#Advanced`)两种形式的群聊功能。高级群拥有更多的权限操作，两种群聊形式在共有操作上保持了接口一致。在群组中，当前会话的ID就是群组的ID。
- 普通群
普通群没有权限操作，适用于快速创建多人会话的场景。每个普通群只有一个管理员。管理员可以对群进行增减员操作，普通成员只能对群进行增员操作。在添加新成员的时候，并不需要经过对方同意。
- 高级群
高级群在权限上有更多的限制，权限分为群主、管理员、以及群成员。在添加成员的时候需要对方接受邀请。
- 群操作权限对比

| 权限   | 普通群             | 高级群     |
| :--| :---------------| :---------------|
| 邀请成员 | 群主、普通成员 | 群主、管理员、普通成员 |
| 踢出成员 | 群主 | 群主、管理员(管理员之间无法互相踢) |
| 解散群 | 群主 | 群主 |
| 退群 | 群主、普通成员 |  群主、管理员、普通成员 |
| 处理入群申请 | / | 群主、管理员 |
| 更改他人群昵称 | / | 群主、管理员 |
| 更改群名称 | 群主、普通成员 | 群主、管理员 |
| 更改群公告 | / | 群主、管理员 |
| 更改群介绍 | / | 群主、管理员 |
| 更新验证方式 | / | 群主、管理员 |
| 添加(删除)管理员 | / | 群主 |
| 移交群主 | / | 群主 |

### 群聊消息

群聊消息收发和管理和单人聊天完全相同，仅在`SessionTypeEnum`上做了区分，详见[消息服务](#消息服务)节。

### 关闭群聊消息提醒

群聊消息提醒可以单独打开或关闭，关闭提醒之后，用户仍然会收到这个群的消息。如果开发者使用的是云信内建的消息提醒，用户收到新消息后不会再用通知栏提醒，如果用户使用的iOS客户端，则他将收不到该群聊消息的apns推送。如果开发者自行实现状态栏提醒，可通过`Team`的`mute`接口获取提醒配置，并决定是不是要显示通知。
开发者可通过调用一下接口打开或关闭群聊消息提醒：

```java
NIMClient.getService(TeamService.class).muteTeam(teamId, mute);
```

### 获取群列表

SDK提供了两个获取自己加入的所有群的列表的接口，一个是获取所有群（包括高级群和普通群）的接口，另一个是根据类型获取列表的接口。开发者可根据实际产品需求选择使用。
获取所有群：

```java
NIMClient.getService(TeamService.class).queryTeamList()
	.setCallback(new RequestCallbackWrapper<List<Team>>() {
    });
```

你也可以直接使用这个函数的同步版本：
```java
List<Team> teams = NIMClient.getService(TeamService.class).queryTeamListBlock();
```

按照类型获取自己加入的群列表：

```java
NIMClient.getService(TeamService.class).queryTeamListByType(type)
	.setCallback(new RequestCallback<List<Team>>() {
    });
```

### 创建群组

网易云信群组分为两类：普通群和高级群，两种群组的消息功能都是相同的，区别在于管理功能。普通群所有人都可以拉人入群，除群主外，其他人都不能踢人。固定群则拥有完善的成员权限体系及管理功能，类似于QQ群。创建群的接口相同，传入不同的类型参数即可。

```java
// 群组类型
TeamTypeEnum type = TeamTypeEnum.Advanced;
// 创建时可以预设群组的一些相关属性，如果是普通群，仅群名有效。
// fields中，key为数据字段，value对对应的值，该值类型必须和field中定义的fieldType一致
HashMap<TeamFieldEnum, Serializable> fields = new HashMap<TeamFieldEnum, Serializable>();
fields.put(TeamFieldEnum.Name, teamName);
fields.put(TeamFieldEnum.Introduce, teamIntroduce);
fields.put(TeamFieldEnum.VerifyType, verifyType);
NIMClient.getService(TeamService.class).createTeam(fields, type, "", accounts)
     setCallback(new RequestCallback<Team> {
     });
```

### 修改群组资料

普通群所有人均可以修改群名，高级群仅拥有者和管理员可修改群名及其他群资料：

```java
// 每次仅修改群的一个属性
// 可修改的属性包括：群名，介绍，公告，验证类型等。
NIMClient.getService(TeamService.class).
	updateTeam(teamId, TeamFieldEnum, value)
	.setCallback(new RequestCallback<Void>() {
                });
```

修改后，群内所有成员会收到一条消息类型为`notification`的`IMMessage`，带有一个消息附件，类型为`UpdateTeamAttachment`。

### 监听群组资料改动

由于获取群组资料需要跨进程异步调用，开发者最好能在第三方app中做好群组资料缓存，查询群组资料时都从本地缓存中访问。在群组资料有变化时，SDK会告诉注册的观察者，此时，第三方app可更新缓存，并刷新界面。

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
    // 由于界面上可能仍需要显示群组名等信息，因此参数中会返回Team对象。
    // 该对象的isMyTeam接口返回为false。
    }
};
// 注册/注销观察者
NIMClient.getService(TeamServiceObserver.class).observeTeamRemove(teamRemoveObserver, register);
```

### 解散群组

普通群和高级群的群主均可以解散群：

```java
NIMClient.getService(TeamService.class).dismissTeam(teamId)
	.setCallback(new RequestCallback<Void>() {
            
        });
```

群被解散后，群内所有成员都会收到一条消息类型为`notification`的`IMMessage`，带有一个消息附件，类型为`DismissAttachment`,原群主为该消息的fromAccount。

### 获取群成员列表

由于群成员列表数据比较大，且除开进入群成员列表界面外，其他地方均不需要成员列表的数据，因此SDK不会在登录时同步成员列表数据，而是按照按需获取的原则，当上层主动调用获取指定群的成员列表时，才判断是否需要同步。获取群成员列表的示例代码如下：

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

### 拉人入群

普通群所有人都可以拉人入群，高级群仅管理员和拥有者可以邀请人入群，接口均为：

```java
NIMClient.getService(TeamService.class).addMembers(teamId, accounts)
	.setCallback(new RequestCallback<Void>() {
	});
```

普通群可直接将用户拉人群聊。
高级群不能直接拉入，被邀请的人会收到一条系统消息(`SystemMessage`)，类型为`SystemMessageType#TeamInvite`。用户接受该邀请后，才真正入群。
用户入群（普通群被拉人，高级群接受邀请）后，在收到之后的第一条消息时，群内所有成员（包括入群者）会收到一条入群消息，附件类型为`MemberChangeAttachment`。

### 踢人出群

普通群仅拥有者可以踢人，高级群拥有者和管理员可以踢人，且管理员不能踢拥有者和其他管理员。

```java
NIMClient.getService(TeamService.class).removeMember(teamId, account)
	.setCallback(new RequestCallback<Void>() {
	});
```

踢人后，群内所有成员(包括被踢者)会收到一条消息类型为`notification`的`IMMessage`，附件类型为`MemberChangeAttachment`。

### 主动退群

除拥有者外，其他用户均可以主动退群：

```java
NIMClient.getService(TeamService.class).quitTeam(teamId)
	.setCallback(new RequestCallback<Void>() {
	});
```

退群后，群内所有成员(包括退出者)会收到一条消息类型为`notification`的`IMMessage`，附件类型为`MemberChangeAttachment`。

### 接受/拒绝入群邀请

收到入群邀请后，用户可在系统消息中看到该邀请，并选择接受或拒绝：

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

接受邀请后，用户真正入群。如果拒绝邀请，邀请该用户的管理员会收到一条系统消息，类型为`SystemMessageType#DeclineTeamInvite`。

### 查询高级群资料

除了管理员邀请，用户也可以主动申请加入高级群。用户可以通过群号查询高级群信息：

```java
NIMClient.getService(TeamService.class).searchTeam(teamId)
	.setCallback(new RequestCallback<Team>() {
           
        });
```

### 申请加入高级群

```java
NIMClient.getService(TeamService.class)
	.applyJoinTeam(teamId, reason)
	.setCallback(new RequestCallback<Team>() {
            });
```

### 同意/拒绝入群申请

用户发出申请后，所有管理员都会收到一条系统消息，类型为`SystemMessageType#TeamApply`。管理员可选择同意或拒绝：

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
如果同意入群申请，群内所有成员(包括申请者)都会收到一条消息类型为`notification`的`IMMessage`，附件类型为`MemberChangeAttachment`。
如果拒绝申请，申请者会收到一条系统消息，类型为`SystemMessageType#RejectTeamApply`。

### 转让群组

高级群拥有者可以将群的拥有者权限转给群内的其他成员，转移后，被转让者变为新的拥有者，原拥有者变为普通成员。原拥有者还可以选择在转让的同时，直接退出该群。

```java
/**
 * 拥有者将群的拥有者权限转给另外一个人，转移后，另外一个人成为拥有者。
 *     原拥有者变成普通成员。若参数quit为true，原拥有者直接退出该群。
 * @param teamId 群ID
 * @param account 新任拥有者的用户账号
 * @param quit 转移时是否要同时退出该群
 * @return InvocationFuture 可以设置回调函数，如果成功，视参数quit值：
 *     quit为false：参数仅包含原拥有着和当前拥有者的(即操作者和account)，权限已被更新。
 *     quit为true: 参数为空。
 */
 NIMClient.getService(TeamService.class)
	 .transferTeam(teamId, account, quit)
	 .setCallback(new RequestCallback<List<TeamMember>>() {
                   });    
```

### 增加删除管理员

高级群中，拥有者可以增加和删除管理员。

```java
/**
 * 拥有者添加管理员
 * @param teamId 群ID
 * @param accounts 待提升为管理员的用户账号列表
 * @return InvocationFuture 可以设置回调函数,如果成功，参数为新增的群管理员列表
 */
NIMClient.getService(TeamService.class)
	.addManagers(teamId, accounts)
	.setCallback(new RequestCallback<List<TeamMember>>() {
}
/**
 * 拥有者撤销管理员权限 <br>
 * @param teamId 群ID
 * @param managers 待撤销的管理员的账号列表
 * @return InvocationFuture 可以设置回调函数，如果成功，参数为被撤销的群成员列表(权限已被降为Normal)。
 */
 NIMClient.getService(TeamService.class)
	.removeManagers(teamId, managers)
	.setCallback(new RequestCallback<List<TeamMember>>() {
}
```

操作完成后，群内所有成员都会收到一条消息类型为`notification`的`IMMessage`，附件类型为`MemberChangeAttachment`。

## 高清语音

### 录制

网易云信SDK提供了一套录制高清语音的接口，用于采集，编码，存储高清语音数据，被提供过程回调，供开发者进行只有的界面展现。
Recorder使用示例代码如下：

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

### 回放

网易云信的语音消息格式有aac和amr两种格式可选，由于2.x系统的原生MediaPlayer不支持aac格式，因此SDK也提供了一个AudioPlayer来播放网易云信的语音消息。同时，将MediaPlayer的接口进行了一些封装，使得在会话场景下播放语音更加方便。
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

    // 播放进度报告，每隔500ms会回调一次，告诉当前进度。 参数为当前进度，单位为毫秒，可用于更新UI
    public void onPlaying(long curPosition) {}
};

// 构造播放器对象
AudioPlayer player = new AudioPlayer(context, filePath, listener);

// 开始播放。需要传入一个Stream Type参数，表示是用听筒播放还是扬声器。取值可参见android.media.AudioManager#STREAM_***
player.start(streamType);

// 如果中途切换播放设备，重新调用start，传入指定的streamType即可。player会自动停止播放，然后再以新的streamType重新开始播放。
// 如果需要从中断的地方继续播放，需要外面自己记住已经播放过的位置，然后在onPrepared回调中调用seekTo
player.seekTo(pausedPosition);

// 主动停止播放
player.stop();
```
## 好友关系

网易云信提供了好友关系的托管，好友资料（用户资料）由第三方APP自行管理。

### 好友关系管理

#### 添加好友

目前添加好友有两种验证类型（见`VerifyType`）：直接添加为好友和发起好友验证请求。添加好友时需要构造`AddFriendData`，需要填入包括对方账号，好友验证类型及附言（可选）。代码示例如下：

```java
final VerifyType verifyType = VerifyType.VERIFY_REQUEST; // 发起好友验证请求
String msg = "好友请求附言";
NIMClient.getService(FriendService.class).addFriend(new AddFriendData(account, verifyType, msg)).setCallback(new RequestCallback<Void>() { ... });
```
添加好友请求发出后，对方会收到一条`SystemMessage`,可以通过SystemMessageObserver的observeReceiveSystemMsg函数来监听系统消息，通过getAttachObject
函数可以获取添加好友的通知`AddFriendNotify`，通知的事件类型见`AddFriendNotify.Event`

```java
if (message.getType() == SystemMessageType.AddFriend) {
    AddFriendNotify attachData = (AddFriendNotify) message.getAttachObject();
    if (attachData != null) {
    	// 针对不同的事件做处理
        if (attachData.getEvent() == AddFriendNotify.Event.RECV_ADD_FRIEND_DIRECT) {
            // 已添加你为好友
        } else if (attachData.getEvent() == AddFriendNotify.Event.RECV_AGREE_ADD_FRIEND) {
            // 对方通过了你的好友请求
        } else if (attachData.getEvent() == AddFriendNotify.Event.RECV_REJECT_ADD_FRIEND) {
    	    // 对方拒绝了你的好友请求
        } else if (attachData.getEvent() == AddFriendNotify.Event.RECV_ADD_FRIEND_VERIFY_REQUEST) {
            // 对方请求添加好友，一般场景会让用户选择同意或拒绝对方的好友请求。
            // 通过message.getContent()获取好友验证请求的附言
        }
    }
}
```
#### 通过/拒绝对方好友请求

收到好友的验证请求的系统消息后，可以通过或者拒绝，对方会收到一条系统通知，通知的事件类型为AddFriendNotify.Event.RECV_AGREE_ADD_FRIEND或者AddFriendNotify.Event.RECV_REJECT_ADD_FRIEND

```java
NIMClient.getService(FriendService.class).ackAddFriendRequest(account, true); // 通过对方的好友请求
```
#### 删除好友

删除好友后，将自动解除双方的好友关系，双方的好友列表中均不存在对方。

```java
NIMClient.getService(FriendService.class).deleteFriend(account).setCallback(new RequestCallback<Void>() { ... }
```

#### 监听好友关系的变化

第三方APP应在APP启动后监听好友关系的变化，当主动添加好友成功、被添加为好友、主动删除好友成功、被对方解好友关系、好友关系更新时都会收到通知：

```java
NIMClient.getService(FriendServiceObserve.class).observeFriendChangedNotify(friendChangedNotifyObserver, true);
private Observer<FriendChangedNotify> friendChangedNotifyObserver = new Observer<FriendChangedNotify>() {
    @Override
    public void onEvent(FriendChangedNotify friendChangedNotify) {
        final String account = friendChangedNotify.getAccount();
        if (friendChangedNotify.getChangeType() == FriendChangedNotify.ChangeType.ADD) {
            // 新增好友
        } else if (friendChangedNotify.getChangeType() == FriendChangedNotify.ChangeType.DELETE) {
            // 删除好友或者被解除好友关系
        }
    }
};
```

#### 获取好友列表

```java
List<Friend> friends = NIMClient.getService(FriendService.class).getFriends();
```
该方法是同步方法，返回的`Friend`集合，`Friend`中包含了账号、好友关系、好友来源、备注、扩展字段等信息。第三方APP获取了好友账号集合后可以向自己的应用服务器请求好友的用户资料来构建自己的通讯录。

#### 判断用户是否为我的好友

```java
boolean isMyFriend = NIMClient.getService(FriendService.class).isMyFriend(account);
```

### 黑名单管理

将用户加入黑名单后，将不在收到对方发来的任何消息或者请求。

#### 加入黑名单

```java
NIMClient.getService(FriendService.class).addToBlackList(account).setCallback(new RequestCallback<Void>() { ... }

```
#### 移出黑名单

```java
NIMClient.getService(FriendService.class).removeFromBlackList(user.getAccount()).setCallback(new RequestCallback<Void>() { ... }
```

#### 监听黑名单变化

```java
NIMClient.getService(FriendServiceObserve.class).observeBlackListChangedNotify(blackListChangedNotifyObserver, true);
private Observer<BlackListChangedNotify> blackListChangedNotifyObserver = new Observer<BlackListChangedNotify>() {
    @Override
    public void onEvent(BlackListChangedNotify blackListChangedNotify) {
        String account = blackListChangedNotify.getAccount();
        if (blackListChangedNotify.getChangeType() == BlackListChangedNotify.ChangeType.ADD) {
            // 拉黑
        } else if (blackListChangedNotify.getChangeType() == BlackListChangedNotify.ChangeType.REMOVE) {
            // 移出黑名单
        }
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

### 消息提醒

网易云信支持对用户设置或关闭消息提醒，关闭后，收到该用户发来的消息时，不再进行通知栏消息。

#### 设置/关闭消息提醒

```java
NIMClient.getService(FriendService.class).setMessageNotify(account, checkState).setCallback(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void param) {
        if (checkState) {
            Toast.makeText(UserProfileActivity.this, "开启消息提醒", Toast.LENGTH_SHORT).show();
        } else {
     		Toast.makeText(UserProfileActivity.this, "关闭消息提醒", Toast.LENGTH_SHORT).show();
        }
}
```
#### 判断用户是否需要消息提醒

```java
boolean notice = NIMClient.getService(FriendService.class).isNeedMessageNotify(account);
```
#### 获取所有静音账号（不进行消息提醒的用户）

```java
List<String> accounts = NIMClient.getService(FriendService.class).getMuteList();
```

## 语音视频通话

网易云信提供基于网络的点对点的语音、视频聊天功能。支持通话中音视频设备的控制，并支持音视频切换。


主叫方可以发起语音或者视频通话，通话类型见`AVChatTypeEnum`，如果发起的是视频通话，需要传入`VideoChatParam`，其中包含视频采集用的SurfaceView（一般只需要在界面布局里放置一个1×1的SurfaceView）及视频旋转角度，如果发起的是语音通话，该参数填null。

```java
AVChatManager.getInstance().call(account, callType, videoParam, new AVChatCallback<AVChatData>() {
}
```

#### 监听来电（被叫方）

一般是在APP启动时注册来电监听，例如在Application的onCreate里添加，这里可以进行铃声相关的配置，见`AVChatRingerConfig` 类，当监听到来电时，会返回来电信息`AVChatData`，其中包含呼叫方式（音频或者视频）、来电账号。

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

如果自己的账号有其他端在线（PC、Web），来电会被其他端做了回应，那么移动端会收到一条通知。因此，移动端在收到来电后需要监听PC端对主叫方的响应。

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
                aVChatUIManager.closeSessions(-1);
            }
        }
    };
```

#### 监听主叫方挂断（被叫方）

详见[监听对方挂断](#监听对方挂断)节

#### 同意接听（被叫方）

当监听到来电后启动通话界面，被叫方可以选择接听或者拒绝。当选择接听时，如果是视频通话，那么同样需要传入`VideoChatParam`，SDK会自动开启音视频设备，建立通话连接。

```java
AVChatManager.getInstance().accept(videoParam, new AVChatCallback<Boolean>() {
}
```

#### 拒绝接听（被叫方）

```java
AVChatManager.getInstance().hangUp(new AVChatCallback<Void>() {
}
```

#### 监听被叫方回应（主叫方）

主叫方在发起呼叫成功后需要监听被叫方的回应，监听接口`observeCalleeAckNotification`，回调返回`AVChatCalleeAckEvent`，其中包含被叫方的回应结果：
- 对方拒绝接听;
- 对方已经有来电了，此时会返回忙;
- 对方同意接听，此时SDK会自动开启音视频设备，建立通话连接，然后双方就可以进行音视频通话了。

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
AVChatManager.getInstance().requestSwitchToVideo(videoParam, new AVChatCallback<Void>() {
}

// 请求视频切换到音频
AVChatManager.getInstance().requestSwitchToAudio(new AVChatCallback<Void>() {
}
```

#### <span id="音视频切换请求的回应">音视频切换请求的回应</span>

一方发送音视频切换请求之后，对方会收到通知，见一下小节。当收到对方音频切换到视频的请求时，用户可以选择同意或者拒绝。对方将收到应答结果的通知，见下一小节。

```java
// 同意音频切换到视频
AVChatManager.getInstance().ackSwitchToVideo(true, videoParam, new AVChatCallback<Void>() {
}

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

主叫方在拨打网络通话时，超过45秒被叫方还未接听来电，则自动挂断。被叫方超过45秒未接听来听，也会自动挂断，在通话过程中网络超时30秒自动挂断。

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

发起结束通话的一方，调用`hangUp`接口。另外一方，需要注册`observeHangUpNotification`监听挂断通知，收到通知后，做相应处理，代码示例见[监听对方挂断（主叫方、被叫方）](#监听对方挂断) 。

```java
AVChatManager.getInstance().hangUp(new AVChatCallback<Void>() {}
```

### 获取当前通话信息及状态

实现`AVChatStateObserver`监听通话过程中状态变化。被叫方同意来电请求后，SDK自动进行音视频服务器连接，并返回相应信息供上层应用使用。

```java
public class AVChatActivity implements AVChatStateObserver {
	AVChatManager.getInstance().observeAVChatState(this, register);
} 
```

#### 当前音视频服务器连接回调

首先返回服务器连接是否成功的回调`onConnectedServer`，并通过返回的result code做相应的处理。

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

#### 加入当前音视频频道用户账号回调

音视频服务器连接成功后，会回调`onUserJoin`，可以获取当前通话的用户账号。

```java
@Override
public void onUserJoin(String account) {}
```

#### 当前用户离开频道回调

通话过程中，若有用户离开，则会回调`onUserLeave`。

```java
// @param event   －1,用户超时离开  0,正常退出
@Override
public void onUserLeave(String account, int event) {}
```

#### 版本协议不兼容回调

若音视频通话双方软件版本不兼容，则会回调`onProtocolIncompatible`。

```java
// @param status 0 自己版本过低  1 对方版本过低
@Override
public void onProtocolIncompatible(int status) {}
```

#### 服务器断开回调

通话过程中，服务器断开，会回调`onDisconnectServer`。

```java
@Override
public void onDisconnectServer() {}
```

#### 当前通话网络状况回调

通话过程中网络状态发生变化，会回调`onNetworkStatusChange`。

```java
// @param value 0~3 ,the less the better; 0 : best; 3 : worst
@Override
public void onNetworkStatusChange(int value) {}
```

#### 音视频连接成功建立回调

音视频连接建立，会回调`onCallEstablished`。音频切换到正在通话的界面，并开始计时等处理。视频则通过用户账户取出SurfaceView并添加到相应的layout上显示图像。

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

音视频设备打开错误，会回调`onOpenDeviceError`。

```java
/**
* 1音频录制错误
* 2音频播放错误
* 4视频采集错误
* 8视频渲染错误
*/
 @Override
public void onOpenDeviceError(int code) {}

```

#### 本地及远程摄像头图像获取

发起视频请求时，需要传递采集点SurfaceView给SDK。SDK会自动采集图像。界面需要显示图像时，通过用户账号获取采集好的图像SurfaceView。

```java
AVChatManager.getInstance().getSurfaceRender(account);
```

### 设备控制

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
AVChatManager.getInstance().setSpeaker(!AVChatManager.getInstance().speakerEnabled()); // 设置扬声器是否开启
```

#### 切换摄像头

```java
AVChatManager.getInstance().toggleCamera(); // 切换摄像头（主要用于前置和后置摄像头切换）
```

#### 关闭/开启摄像头

具体见[发送通话控制信息](#发送通话控制信息)

#### 切换通话模式

具体见[请求音视频切换](#请求音视频切换) 和 [音视频切换请求的回应](#音视频切换请求的回应)

## API文档
* [在线文档](http://dev.netease.im/doc/android/index.html "target=_blank")