## 1 获取通讯管理器
ImSDK一切操作都是由通讯管理器`TIMManager`开始，SDK操作第一步需要获取`TIMManager`单例：
原型：

```
public static TIMManager getInstance()
```

获取通讯管理器实例

**返回：**
TIMManager实例

**示例：**

```
TIMManager.getInstance();
```
## 2 初始化SDK配置
在初始化SDK之前，需要进行简单的SDK配置，包括SdkAppid、日志控制、Crash上报等。对应的配置类为`TIMSdkConfig`，具体api可以参考SDK下载包中的`javadoc`文档。

### 2.1 日志事件

SDK内部会进行打印日志，如果调用方有自己统一的日志收集方式，可以通过`TIMSdkConfig`中的`setLogListener`接口设置日志事件回调，把日志通过回调返给调用方，但ImSDK内部仍然会打印，如果需要禁掉，可以通过设置控制台不打印日志，或者设置日志级别。

**原型：**

```
/**
 * 设置当前日志回调监听器， 必须在sdk初始化之前设置
 * @param logListener 日志回调监听器
 */
public TIMSdkConfig setLogListener(TIMLogListener logListener) 
```

**示例：**

```
//设置日志回调，sdk输出的日志将通过此接口回传一份副本
//[NOTE] 请注意level定义在TIMManager中，如TIMManager.ERROR等， 并不同于Android系统定义
TIMManager.getInstance().setLogListener(new TIMLogListener() {
    @Override
    public void log(int level, String tag, String msg) {
        //可以通过此回调将sdk的log输出到自己的日志系统中
    }
});
```
### 2.2 设置日志级别

在权限允许的情况下，ImSDK的日志默认会写到日志文件中。通过`TIMSdkConfig`中的`setLogLevel`接口修改ImSDK内部写日志级别可以控制ImSDK的文件日志输出。

> 注意：
> 1. 设置写日志等级， **必须在sdk初始化之前调用**，在sdk初始化之后设置无效。
> 2. 可以通过设置日志级别为`TIMLogLevel.OFF`来关闭ImSDK的文件日志输出，提升性能，建议在开发期间打开日志，方便排查问题。

**原型：**

```
/**
 * 设置写日志等级，必须在sdk初始化之前调用，在sdk初始化之后设置无效
 * @param logLevel 日志等级
 */
public TIMSdkConfig setLogLevel(@NonNull TIMLogLevel logLevel) 
```


### 2.3 控制台不打印日志

默认ImSDK日志会打印到控制台，如果调试期间干扰太多，可选择通过`TIMSdkConfig`中的`enableLogPrint`关闭控制台日志（此时文件日志仍然会打印，可设置日志级别禁用）。

> 注意：
> 日志设置， **必须在sdk初始化之前调用**，在sdk初始化之后设置无效。

**原型：**
```
/**
 * 设置是否把日志输出到控制台， 必须在sdk初始化之前设置
 * @param logPrintEnabled true - 日志将会输出到控制台
 */
public TIMSdkConfig enableLogPrint(boolean logPrintEnabled) 
```

### 2.4 修改日志路径

为了方便统一管理日志，也可以修改默认的日志存储路径。通过`TIMSdkConfig`中的`setLogPath`接口可以设置日志文件存储路径。

> 注意：
> 1. 设置日志路径，**必须在sdk初始化之前调用**，在sdk初始化之后设置无效。
> 2. ImSDK默认日志存储路径为：*SD卡下，/tencent/imsdklogs/(your app package name)/*

**原型：**
```
/**
 * 设置日志路径，必须在sdk初始化之前调用，在sdk初始化之后设置无效
 * @param logPath 日志路径
 */
public TIMSdkConfig setLogPath(@NonNull String logPath)
```


### 2.2 Crash上报

ImSDK内部集成了[Bugly系统](http://bugly.qq.com)，当应用crash后，会自动上报到平台，用户可以根据Bugly文档指示上传符号表，显示crash详细信息，如果用户有自己的上报组件，可通过`TIMSdkConfig`中的`enableCrashReport`接口来进行禁用上报。

> 注意：
> 禁用Bugly的crash上报， **必须在sdk初始化之前调用**， 在sdk初始化之后设置无效。

**原型：**
```
/**
 * 设置是否开启bugly的crash上报功能， 必须在sdk初始化之前设置
 * @param crashReportEnabled true - 开启bugly的crash上报功能（需要集成ImSDK内部提供的bugly库）
 */
public TIMSdkConfig enableCrashReport(boolean crashReportEnabled)
```

## 3 初始化SDK

在使用SDK进一步操作之前，需要初始化SDK。

> 注意：
> **在存在多进程的情况下，请只在一个进程进行SDK初始化。**

**原型：**

```
/**
 * 初始化ImSDK
 * @param context  application context
 * @param config SDK全局配置
 * @return true - 初始化成功， false - 初始化失败
 */
public boolean init(@NonNull Context context, @NonNull TIMSdkConfig config)
```

**示例：**

```
//初始化SDK基本配置
TIMSdkConfig config = new TIMSdkConfig(sdkAppId)
		.enableCrashReport(false);
		.enableLogPrint(true)
		.setLogLevel(TIMLogLevel.DEBUG)
		.setLogPath(Environment.getExternalStorageDirectory().getPath() + "/justfortest/")

//初始化SDK
TIMManager.getInstance().init(getApplicationContext(), config);
```
## 4 用户配置

在初始化SDK后，登录SDK之前，需要设置用户配置。
ImSDK的用户配置分四部分，分别如下：
+ 基本用户配置 —— 通过`TIMUserConfig`进行配置。
+ 消息扩展用户配置 —— 通过`TIMUserConfigMsgExt`进行配置。
+ 群组管理扩展用户配置 —— 通过`TIMUserConfigGroupExt`进行配置。
+ 资料关系链管理扩展用户配置 —— 通过`TIMUserConfigSnsExt`进行配置。

配置完成后，**在登录前**，通过通讯管理器`TIMManager`的接口`setUserConfig`将用户配置与当前通讯管理器进行绑定。

**原型：**
```
/**
 * 设置当前用户的用户配置，登录前设置
 * @param userConfig 用户配置
 */
public void setUserConfig(TIMUserConfig userConfig) 
```

**示例：**
```
//基本用户配置
TIMUserConfig userConfig = new TIMUserConfig()
		//设置群组资料拉取字段
		.setGroupSettings(initGroupSettings())
		//设置资料关系链拉取字段
		.setFriendshipSettings(initFriendshipSettings())
		//设置用户状态变更事件监听器
		.setUserStatusListener(new TIMUserStatusListener() {
			@Override
			public void onForceOffline() {
				//被其他终端踢下线
				Log.i(tag, "onForceOffline");
			}

			@Override
			public void onUserSigExpired() {
				//用户签名过期了，需要刷新userSig重新登录SDK
				Log.i(tag, "onUserSigExpired");
			}
		})
		//设置连接状态事件监听器
		.setConnectionListener(new TIMConnListener() {
			@Override
			public void onConnected() {
				Log.i(tag, "onConnected");
			}

			@Override
			public void onDisconnected(int code, String desc) {
				Log.i(tag, "onDisconnected");
			}

			@Override
			public void onWifiNeedAuth(String name) {
				Log.i(tag, "onWifiNeedAuth");
			}
		})
		//设置群组事件监听器
		.setGroupEventListener(new TIMGroupEventListener() {
			@Override
			public void onGroupTipsEvent(TIMGroupTipsElem elem) {
				Log.i(tag, "onGroupTipsEvent, type: " + elem.getTipsType());
			}
		})
		//设置会话刷新监听器
		.setRefreshListener(new TIMRefreshListener() {
			@Override
			public void onRefresh() {
				Log.i(tag, "onRefresh");
			}

			@Override
			public void onRefreshConversation(List<TIMConversation> conversations) {
				Log.i(tag, "onRefreshConversation, conversation size: " + conversations.size());
			}
		});

//消息扩展用户配置
userConfig = new TIMUserConfigMsgExt(userConfig)
		//禁用消息存储
		.enableStorage(false)
		//开启消息已读回执
		.enableReadReceipt(true);

//资料关系链扩展用户配置
userConfig = new TIMUserConfigSnsExt(userConfig)
		//开启资料关系链本地存储
		.enableFriendshipStorage(true)
		//设置关系链变更事件监听器
		.setFriendshipProxyListener(new TIMFriendshipProxyListener() {
			@Override
			public void OnAddFriends(List<TIMUserProfile> users) {
				Log.i(tag, "OnAddFriends");
			}

			@Override
			public void OnDelFriends(List<String> identifiers) {
				Log.i(tag, "OnDelFriends");
			}

			@Override
			public void OnFriendProfileUpdate(List<TIMUserProfile> profiles) {
				Log.i(tag, "OnFriendProfileUpdate");
			}

			@Override
			public void OnAddFriendReqs(List<TIMSNSChangeInfo> reqs) {
				Log.i(tag, "OnAddFriendReqs");
			}

			@Override
			public void OnAddFriendGroups(List<TIMFriendGroup> friendgroups) {
				Log.i(tag, "OnAddFriendGroups");
			}

			@Override
			public void OnDelFriendGroups(List<String> names) {
				Log.i(tag, "OnDelFriendGroups");
			}

			@Override
			public void OnFriendGroupUpdate(List<TIMFriendGroup> friendgroups) {
				Log.i(tag, "OnFriendGroupUpdate");
			}
		});

//群组管理扩展用户配置
userConfig = new TIMUserConfigGroupExt(userConfig)
		//开启群组资料本地存储
		.enableGroupStorage(true)
		//设置群组资料变更事件监听器
		.setGroupAssistantListener(new TIMGroupAssistantListener() {
			@Override
			public void onMemberJoin(String groupId, List<TIMGroupMemberInfo> memberInfos) {
				Log.i(tag, "onMemberJoin");
			}

			@Override
			public void onMemberQuit(String groupId, List<String> members) {
				Log.i(tag, "onMemberQuit");
			}

			@Override
			public void onMemberUpdate(String groupId, List<TIMGroupMemberInfo> memberInfos) {
				Log.i(tag, "onMemberUpdate");
			}

			@Override
			public void onGroupAdd(TIMGroupCacheInfo groupCacheInfo) {
				Log.i(tag, "onGroupAdd");
			}

			@Override
			public void onGroupDelete(String groupId) {
				Log.i(tag, "onGroupDelete");
			}

			@Override
			public void onGroupUpdate(TIMGroupCacheInfo groupCacheInfo) {
				Log.i(tag, "onGroupUpdate");
			}
		});

//将用户配置与通讯管理器进行绑定
TIMManager.getInstance().setUserConfig(userConfig);
```

### 4.1 网络事件通知

可选设置，如果要用户感知是否已经连接服务器，需要通过`TIMUserConfig`来设置此回调，用于通知调用者跟通讯后台链接的连接和断开事件，另外，如果断开网络，等网络恢复后会自动重连，自动拉取消息通知用户，用户无需关心网络状态，仅作通知之用。注意：这里的网络事件不表示用户本地网络状态，仅指明SDK是否与IM云Server连接状态。只要用户处于登录状态，**ImSDK内部会进行断网重连，用户无需关心**。

**原型：**

```
/**
 * 设置连接监听器
 * @param listener 连接监听器
 */
public TIMUserConfig setConnectionListener(TIMConnListener listener) 
```

**示例：**

请参考[用户配置](#4-.E7.94.A8.E6.88.B7.E9.85.8D.E7.BD.AE9)中的示例。

### 4.2 用户状态变更

用户状态变更的时候，SDK会有相应的通知。通过`TIMUserConfig`设置用户状态变更通知监听器来对相应的通知进行监听。目前用户状态变更有两种通知，具体可参见 [用户被踢下线通知](#4.2.1-.E7.94.A8.E6.88.B7.E8.A2.AB.E8.B8.A2.E4.B8.8B.E7.BA.BF.E9.80.9A.E7.9F.A512) 和 [用户票据过期通知](#4.2.2-.E7.94.A8.E6.88.B7.E7.A5.A8.E6.8D.AE.E8.BF.87.E6.9C.9F.E9.80.9A.E7.9F.A513)。

**原型：**

```
/**
 * 设置用户状态通知回调
 * @param userStatusListener 用户状态通知回调
 */
public TIMUserConfig setUserStatusListener(TIMUserStatusListener userStatusListener)
```

用户状态变更通知监听器`TIMUserStatusListener`的定义如下：

```
/**
 * 用户状态变更通知监听器
 */
public interface TIMUserStatusListener {

    /**
     * 被踢下线时回调
     */
    public void onForceOffline();

    /**
     * 票据过期时回调
     */
    public void onUserSigExpired();
}
```


**示例：**

请参考[用户配置](#4-.E7.94.A8.E6.88.B7.E9.85.8D.E7.BD.AE9)中的示例。


### 4.2.1 用户被踢下线通知

用户如果在其他终端登录，会被踢下线，这时会收到用户被踢下线的通知。如果设置了用户状态变更通知监听器（参见 [用户状态变更](#4.2-.E7.94.A8.E6.88.B7.E7.8A.B6.E6.80.81.E5.8F.98.E6.9B.B411)），则可以在监听器的回调方法 `onForceOffline` 中进行相应的处理，出现这种情况常规的做法是提示用户进行操作（退出，或者再次把对方踢下线）。


>注意：
用户如果在离线状态下被踢，下次登录将会失败，可以给用户一个非常强的提醒（登录错误码ERR_IMSDK_KICKED_BY_OTHERS：6208），开发者也可以选择忽略这次错误，再次登录即可。

用户在线情况下的互踢情况如下图所示：

![](//avc.qcloud.com/wiki2.0/im/imgs/20151015021645_19906.png)

图示中，用户在设备1登录，保持在线状态下，该用户又在设备2登录，这时用户会在设备1上强制下线，收到onForceOffline回调。用户在设备1上收到回调后，提示用户，可继续调用login上线，强制设备2下线。这里是在线情况下互踢过程。
用户离线状态互踢如下图所示：

![](//avc.qcloud.com/wiki2.0/im/imgs/20151015021702_68733.png)

用户在设备1登录，没有进行logout情况下杀掉应用进程。该用户在设备2登录，此时由于应用进程已不在了，设备1无法感知此事件，为了显式提醒用户，避免无感知的互踢，用户重新在设备1登录时，会返回（ERR_IMSDK_KICKED_BY_OTHERS：6208）错误码，表明之前被踢，是否需要把对方踢下线。如果需要，则再次调用login强制上线，设备2的登录的实例将会收到onForceOffline回调。

### 4.2.2 用户票据过期通知

在用户登录（参见 [登录](/doc/product/269/9233#1-.E7.99.BB.E5.BD.951)）的时候，需要提供一个用户票据，而这个用户票据在生成的时候是有一个有效使用期限的。在正常使用过程中，如果超过了用户票据的使用期限时，SDK与服务器的交互会因为票据验证失败而操作失败，这个时候SDK会给出用户票据过期的通知。如果设置了用户状态变更通知监听器（参见 [用户状态变更](#4.2-.E7.94.A8.E6.88.B7.E7.8A.B6.E6.80.81.E5.8F.98.E6.9B.B411)），则可以在监听器的回调方法 `onUserSigExpired` 中进行相应的处理，出现这种情况，如果仍需要继续与服务器进行交互，则需要更换票据后重新登录。


### 4.3 禁用存储
默认情况ImSDK会进行消息的存储，如无需存储，可选择通过`TIMUserConfigMsgExt`关闭存储来提升处理性能。

> 注意：
> 禁用消息存储，**需要在登录之前调用**。

**原型：**
```
/**
 * 设置是否开启消息本地储存
 * @param storageEnabled true - 开启， false - 不开启
 */
public TIMUserConfigMsgExt enableStorage(boolean storageEnabled) 
```

### 4.4 会话刷新监听

默认登录后会异步获取C2C离线消息、最近联系人以及同步资料数据（如果有开启ImSDK存储，可参见[关系链资料存储](/doc/product/269/9231#7.-.E5.85.B3.E7.B3.BB.E9.93.BE.E8.B5.84.E6.96.99.E5.AD.98.E5.82.A836)及[群资料存储](/doc/product/269/9236#8.-.E7.BE.A4.E8.B5.84.E6.96.99.E5.AD.98.E5.82.A837)），同步完成后会通过会话刷新监听器`TIMRefreshListener`中的`onRefresh`回调通知更新界面，用户得到这个消息时，可以刷新界面，比如会话列表的未读等。
> 注意：
> 1. 如果不需要最近联系人可通过接口禁用： [最近联系人漫游](/doc/product/269/9232#4.2-.E6.9C.80.E8.BF.91.E8.81.94.E7.B3.BB.E4.BA.BA.E6.BC.AB.E6.B8.B835) 。
> 2. 如果不需要离线消息，可以再发消息时使用：[发送在线消息](/doc/product/269/9232#1.10-.E5.9C.A8.E7.BA.BF.E6.B6.88.E6.81.AF11)。

在多终端情况下，未读消息计数由Server下发同步通知，SDK在本地更新未读计数后，通知用户更新会话。通知会通过`TIMRefreshListener`中的`onRefreshConversation`接口来进行回调，对于关注多终端同步的用户，可以在这个接口中进行相关的同步处理。

所以建议在登录之前，通过`TIMUserConfig`中的`setRefreshListener`接口来设置会话刷新监听。

**原型：**

```
/**
 * 设置数据刷新通知监听器
 * @param listener 数据刷新通知监听器
 */
public TIMUserConfig setRefreshListener(TIMRefreshListener listener)
```

## 5 新消息通知

在多数情况下，用户需要感知新消息的通知，这时只需注册新消息通知回调 `TIMMessageListener`，在用户登录的时候，会拉取C2C离线消息和最近联系人，为了不漏掉消息通知，建议在登录之前注册新消息通知。

> 注意：
> 只要是本地没有的消息，SDK都会通过注册的消息通知回调给上层应用。

**原型：**

```
/**
 * 添加一个消息监听器
 * @param listener 消息监听器
 *                 默认情况下所有消息监听器都将按添加顺序被回调一次
 *                 除非用户在OnNewMessages回调中返回true，此时将不再继续回调下一个消息监听器
 */
public void addMessageListener(TIMMessageListener listener) 
```

添加一个消息监听器，默认情况下所有消息监听器都将按添加顺序被回调一次。除非用户在`OnNewMessages`回调中返回true，此时将不再继续回调下一个消息监听器。


```
public boolean onNewMessages(java.util.List msgs)
```

收到新消息回调。


```
public void removeMessageListener(TIMMessageListener listener)
```

删除一个消息监听器，消息监听器被删除后，将不再被调用。

回调消息内容通过参数`TIMMessage`传递，通过`TIMMessage`可以获取消息和相关会话的详细信息，如消息文本，语音数据，图片等等，可参阅（[消息解析](/doc/product/269/消息收发（Android%20SDK）#2.1-.E6.B6.88.E6.81.AF.E8.A7.A3.E6.9E.90)）部分。

**示例：**


```
//设置消息监听器，收到新消息时，通过此监听器回调
TIMManager.getInstance().addMessageListener(new TIMMessageListener() {//消息监听器
 
    @Override
    public boolean onNewMessage(Listmsgs) {//收到新消息
		//消息的内容解析请参考消息收发文档中的消息解析说明
		return true; //返回true将终止回调链，不再调用下一个新消息监听器
    }
});
```









