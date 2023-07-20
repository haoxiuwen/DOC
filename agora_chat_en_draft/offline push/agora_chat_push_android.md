# 离线推送

即时通讯 IM 支持集成 FCM 消息推送服务，为 Android 开发者提供低延时、高送达、高并发、不侵犯用户个人数据的离线消息推送服务。

当客户端断开连接或应用进程被关闭等原因导致用户离线，即时通讯 IM 会通过 FCM 消息推送服务向该离线用户的设备推送消息通知。当用户再次上线时，服务器会将离线期间的消息发送给用户。仅单聊和群聊会话支持离线推送，聊天室不支持。

本文介绍如何在客户端应用中实现 FCM 推送服务。 

## 技术原理

![1655713515869](https://web-cdn.agora.io/docs-files/1655713515869)

消息推送流程如下：

1. 用户 B 初始化 FCM 推送 SDK，检查是否支持 FCM 推送。
2. 用户 B 根据配置的 FCM 推送 SDK 从 FCM 推送服务器获取 device token。
3. FCM 推送服务器向用户 B 返回 device token。
4. 用户 B 向声网即时通讯服务器上传推送证书名称和 device token。
5. 用户 A 向 用户 B 发送消息。
6. 声网即时通讯服务器检查用户 B 是否在线。若在线，声网即时通讯服务器直接将消息发送给用户 B。
7. 若用户 B 离线，声网即时通讯服务器判断该用户是否使用了 FCM 推送。
8. 声网即时通讯服务器将消息发送给 FCM 推送服务器。
9. FCM 推送服务器将消息发送给用户 B。

<div class="alert info">device token 是 FCM 推送提供的推送 token，即初次启动你的应用时，FCM SDK 为客户端应用实例生成的注册令牌 (registration token)。该 token 用于标识每台设备上的每个应用，FCM 通过该 token 明确消息是发送给哪个设备的，然后将消息转发给设备，设备再通知应用程序。你可以调用 FirebaseMessaging.getInstance().getToken() 方法获得 token。另外，如果退出即时通讯 IM 登录时不解绑 device token（调用 `logout` 方法时对 `unbindToken` 参数传 `false` 时不解绑 device token，传 `true` 表示解绑 token），用户在推送证书有效期和 token 有效期内仍会接收到离线推送通知。</div>

## 前提条件

- 已开启即时通讯 IM ，详见[开启和配置即时通讯服务](./enable_agora_chat)。
- 了解即时通讯 IM 套餐包中的 API 调用频率限制，详见[使用限制](./agora_chat_limitation)。

## 在即时通讯 IM 中集成 FCM 

按以下步骤在即时通讯 IM 中集成 FCM：

1. 在 [Firebase 控制台](https://console.firebase.google.com/)创建 Android 项目。
2. 在[声网控制台](https://console.agora.io/)配置 FCM 推送。
3. 在客户端集成 FCM。

<div class="alert info">对于使用 Android 系统的设备，若同时开启了 FCM 和其他厂商的推送服务，优先使用 FCM 推送服务。</div>

### 在 Firebase 控制台创建 Android 项目

1. 登录 [Firebase 控制台](https://console.firebase.google.com/)，单击 **Add project**。
2. 在 **Create a project** 页面输入项目名称，点击 **Continue**。
3. (可选) 选择 **Enable Google Analytics for this project**。该选项默认启用，若不需要，可将其关闭。
4. 项目创建完毕，界面提示 **Your new project is ready**。点击 **Continue** 打开项目页面，点击 **Android** 图标注册 Android 项目。
5. 在 **Add Firebase to your Android app** 页面中，进行如下操作：
    i. 在 **Register app** 步骤中，输入 Android 包名、应用昵称（可选）和调试签名证书 SHA-1（可选），然后单击 **Register app**。
    ii. 在 **Download and then add config file** 步骤中，点击 **Download google-services.json**，将该文件放入你的 Android 应用模块的根目录，然后单击 **Next**。
    iii. 在 **Add Firebase SDK** 步骤中，修改 **build.gradle** 文件以便使用 Firebase，然后单击 **Next**。
    iv. 在 **Next steps** 步骤中，单击 **Continue to console** 返回项目页面。
6. 在项目页面中，单击你创建的 Android 项目，然后单击左侧导航栏右上角 **Project Overview** 旁边的项目设置图标。
7. 在 **Project settings** 页面，选择 **Cloud Messaging** 页签，查看 **Server ID**。在声网控制台上传 FCM 证书时，需要将证书名称设置为 FCM 的发送者 ID。
8. 在 **Project settings** 页面，选择 **Service accounts** 页签，点击 **Generate new private key** 按钮生成密钥（JSON 文件）。请保存该文件，使用 V1 证书时需在声网控制台上传该文件。

[img](fcm_private-key)

### 在声网控制台配置 FCM 推送

登录即时通讯 IM SDK 成功后，可在声网控制台配置多设备登录场景下的推送策略以及上传 FCM 推送证书。
1. 登录[声网控制台](https://console.agora.io/)，在左侧导航栏中单击 **Project Management**。
2. 在 **Project Management** 页面上，在启用了即时通讯 IM 的项目的 **Action** 一栏中单击 **Config**。
3. 在 **Edit Project** 页面的 **Features** 区域，单击 **Chat** 对应的 **Enable/Config**。
4. 在项目配置页面，选择 **Features** > **Push Certificate**。

![image](push_multidevice_policy.png)

**配置多设备登录时的推送策略**

在 **Push Certificate** 页面，配置多设备登录场景下的推送策略：
- 所有设备离线时，才发送推送消息。
- 任一设备离线时，都发送推送消息。

**上传 FCM 证书**

在 **Push Certificate** 页面，单击 **Add Push Certificate**。 在弹出的对话框中，选择 **Google** 页签，配置字段，单击**保存**。

[img](FCM_add_push_certificate.png)

| 参数       | 类型   | 是否必需 | 描述        |
| :--------- | :----- | :------- | :----------------------- |
| **Certificate Type**     |  | 否  | 选择使用旧版证书还是 V1 证书。<br/> - **V1**： 你需要配置 **Private Key**。推荐使用 V1 证书。<br/> - **Old Version**：你需要配置 **Push Key**。老版本证书即将被弃用。|
| **Private Key**     | file | 是       | 点击 **Upload** 上传推送证书文件（.json）。你需要在 Firebase 控制台的 **Project settings** > **Service accounts** 页面点击 **Generate new private key** 生成的推送证书文件（.json）。 |
| **Push Key** | String | 是       | FCM 的服务器密钥（Server Key）。 你需在 Firebase 控制台的 **Project settings** > **Cloud Messaging** 页面，在 **Cloud Messaging API (Legacy)** 区域中获取服务器密钥。|
| **Certificate Name** | String | 是       | 配置为 FCM 的发送者 ID。你可以在 FCM 控制台的 **Project settings** > **Cloud Messaging** 页面查看 **Sender ID** 参数的值。<br/> - 证书名称是声网即时通讯服务器用于判断目标设备使用哪种推送通道的唯一条件，因此必须确保你[在即时通讯 IM 中集成 FCM 时设置的 Sender ID](#initialization)与这里设置的一致。|
| **Sound** | String | 否       | 接收方收到推送通知时的铃声提醒。  |  是否生效？待验证
| **Push Priority** |  | 否       | 消息传递优先级，详见 [FCM 官网](https://firebase.google.cn/docs/cloud-messaging/concept-options#setting-the-priority-of-a-message)。 |
| **Push Msg Type** |  | 否       | 通过 FCM 向客户端发送的消息的类型，详见 [FCM 消息简介](https://firebase.google.cn/docs/cloud-messaging/concept-options#notifications_and_data_messages) 。<br/> - **Data**：数据消息，由客户端应用处理。<br/> - **Notification**：通知消息，由 FCM SDK 自动处理。<br/> - **Both**：可以通过 FCM 客户端发送通知消息和数据消息。|

### 在客户端集成 FCM

1. 在你的 app 项目的 build.gradle 文件中，配置 FCM 库的依赖：

```gradle
plugins {
    id 'com.android.application'
    // Add the Google services Gradle plugin
    id 'com.google.gms.google-services'
}

dependencies {
    // ...
    // 导入 Firebase BoM。
    implementation platform('com.google.firebase:firebase-bom:28.4.1')
    // 声明 FCM 的依赖。
    // 使用 BoM 时，不要在 Firebase 库依赖中指定版本。
    implementation 'com.google.firebase:firebase-messaging'
}
```

在你的根级（项目级）Gradle 文件 (<project>/build.gradle)：

```gradle
plugins {
  // ...

  // Add the dependency for the Google services Gradle plugin
  id 'com.google.gms.google-services' version '4.3.15' apply false

}
```

对于 Gradle 5.0 及以上版本，BoM 自动开启，而对于 Gradle 5.0 之前的版本，你需要开启 BoM 特性。详见[Firebase Android BoM](https://firebase.google.cn/docs/android/learn-more#bom)和[Firebase Android SDK Release Notes](https://firebase.google.cn/support/release-notes/android)。

2. 将你的项目与 gradle 文件同步，扩展 `FirebaseMessagingService`，并将其在项目的 `AndroidManifest.xml` 文件中注册。

```java
public class FCMMSGService extends FirebaseMessagingService {
    private static final String TAG = "FCMMSGService";

    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        super.onMessageReceived(remoteMessage);
        if (remoteMessage.getData().size() > 0) {
            String message = remoteMessage.getData().get("alert");
            Log.d(TAG, "onMessageReceived: " + message);
        }
    }

    @Override
    public void onNewToken(@NonNull String token) {
        super.onNewToken(token);
        Log.d(TAG, "onNewToken: " + token);
        ChatClient.getInstance().sendFCMTokenToServer(token);
    }
}
```

```xml
<service
    android:name=".java.MyFirebaseMessagingService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT" />
    </intent-filter>
</service>
```

3. 在即时通讯 IM SDK 中初始化并开启 FCM。<a name="initialization"></a>

```java
ChatOptions options = new ChatOptions();
...
PushConfig.Builder builder = new PushConfig.Builder(this);
// 替换为你的 FCM 的 Sender ID。
builder.enableFCM("Your FCM sender id");
// 在 ChatOptions 类中配置推送设置。
options.setPushConfig(builder.build());
// 初始化即时通讯 IM SDK。
ChatClient.getInstance().init(this, options);
// 设置推送监听。
PushHelper.getInstance().setPushListener(new PushListener() {
    @Override
    public void onError(PushType pushType, long errorCode) {
        EMLog.e("PushClient", "Push client occur a error: " + pushType + " - " + errorCode);
    }
    @Override
    public boolean isSupportPush(PushType pushType, PushConfig pushConfig) {
        // Sets whether FCM is enabled.
        if(pushType == PushType.FCM) {
            return GoogleApiAvailabilityLight.getInstance().isGooglePlayServicesAvailable(MainActivity.this)
                        == ConnectionResult.SUCCESS;
        }
        return super.isSupportPush(pushType, pushConfig);
    }
});
```

4. 将 FCM 的 device token 传递给即时通讯 IM SDK。

```java
// 检查是否启用了 FCM。
if(GoogleApiAvailabilityLight.getInstance().isGooglePlayServicesAvailable(MainActivity.this) == ConnectionResult.SUCCESS) {
    return;
}
FirebaseMessaging.getInstance().getToken().addOnCompleteListener(new OnCompleteListener<String>() {
    @Override
    public void onComplete(@NonNull Task<String> task) {
        if (!task.isSuccessful()) {
            EMLog.d("PushClient", "Fetching FCM registration token failed:"+task.getException());
            return;
        }
        // 获取 FCM 的注册 token，即 device token。
        String token = task.getResult();
        EMLog.d("FCM", token);  
        ChatClient.getInstance().sendFCMTokenToServer(token);
    }
});
```

5. 监控 device token 生成。

重写 `FirebaseMessagingService` 中的 `onNewToken` 回调。device token 生成后，该回调会及时将新 token 更新到即时通讯 IM SDK。

```java
@Override
public void onNewToken(@NonNull String token) {
    Log.i("MessagingService", "onNewToken: " + token);
    // 若要向该应用发送消息或在服务器端管理应用订阅，需向你的 App Server 发送 FCM 注册 token。
    if(ChatClient.getInstance().isSdkInited()) {
        ChatClient.getInstance().sendFCMTokenToServer(token);
    }
}
```

## 测试 FCM 推送

在即时通讯 IM 中集成并启用 FCM 后，可测试推送是否成功集成。

### 前提条件

准备一台在海外正式发售的 Android 设备用于接收推送通知，确保该设备满足以下条件：
- 使用国外 IP 地址与即时通讯 IM 建立连接。
- 支持 Google GMS 服务（Google Mobile Services）。
- 可以正常访问 Google 网络服务，否则该设备无法从 FCM 服务接收推送通知。

为了确保测试效果可靠，请避免使用模拟器进行测试。

### 测试步骤

1. 在设备上登录应用，并确认 device token 绑定成功。
2. 开启应用通知栏权限。
3. 杀掉应用进程。
4. 在声网控制台发送测试消息。
   在左侧导航栏中选择 **Operation Management** > **User**。在用户管理页面中，在对应用户 ID 的 **Action** 栏中选择 **Send Admin Message**。
   在弹出的对话框中选择消息类型，输入消息内容，然后点击 **Send**。
5. 查看设备是否收到推送通知。

### 故障排除

1. 检查在即时通讯 IM 中是否正确集成或启用了 FCM 推送。
   在左侧导航栏中选择 **Operation Management > User**。在用户管理页面中，在对应用户 ID 的 **Action** 栏中选择 **Push Certificate**。
   在弹出框中查看是否正确显示了证书名称和 device token。
2. 检查是否在声网控制台上传了正确的 FCM 证书。
3. 检查是否在聊天室中推送消息。聊天室不支持离线消息推送。
4. 检查设备是否为国行手机的 ROM。一些品牌的国产手机不支持 GMS 服务，需替换为海外发售的设备。
5. 检查发送消息时是否设置了只发在线(deliverOnlineOnly = true)。只发在线的消息不推送。

## 设置推送通知 

为优化用户在处理大量推送通知时的体验，即时通讯 IM 在 app 和会话层面提供了推送通知方式和免打扰模式的细粒度选项。 

**推送通知方式**

| 推送通知方式参数     | 描述   | 应用范围   |
| :------- | :----- | :----- |
| `ALL` | 接收所有离线消息的推送通知。 | App 和单聊/群聊会话 |
| `MENTION_ONLY` | 仅接收提及消息的推送通知。<br/>该参数推荐在群聊中使用。若提及一个或多个用户，需在创建消息时对 `ext` 字段传 "em_at_list":["user1", "user2" ...]；若提及所有人，对该字段传 "em_at_list":"all"。 | App 和单聊/群聊会话 |
| `NONE`   | 不接收离线消息的推送通知。 | App 和单聊/群聊会话      |

会话级别的推送通知方式设置优先于 app 级别的设置，未设置推送通知方式的会话默认采用 app 的设置。

例如，假设 app 的推送方式设置为 `MENTION_ONLY`，而指定会话的推送方式设置为 `ALL`。你会收到来自该会话的所有推送通知，而对于其他会话来说，你只会收到提及你的消息的推送通知。

**免打扰模式**

你可以在 app 级别指定免打扰时间段和免打扰时长，即时通讯 IM 在这两个时间段内不发送离线推送通知。若既设置了免打扰时间段，又设置了免打扰时长，免打扰模式的生效时间为这两个时间段的累加。

免打扰时间参数的说明如下表所示：

| 免打扰时间参数     |  描述   |   应用范围 |
| :--------| :----- | :----------------------------------------------------------- |
| `SILENT_MODE_INTERVAL` | 免打扰时间段，精确到分钟，格式为 HH:MM-HH:MM，例如 08:30-10:00。该时间为 24 小时制，免打扰时间段的开始时间和结束时间中的小时数和分钟数的取值范围分别为 [00,23] 和 [00,59]。免打扰时间段的设置说明如下：<ul><li>开始时间和结束时间的设置立即生效，免打扰模式每天定时触发。例如，开始时间为 `08:00`，结束时间为 `10:00`，免打扰模式在每天的 8:00-10:00 内生效。若你在 11:00 设置开始时间为 `08:00`，结束时间为 `12:00`，则免打扰模式在当天的 11:00-12:00 生效，以后每天均在 8:00-12:00 生效。</li><li>若开始时间和结束时间相同，免打扰模式则全天生效。</li><li>若结束时间早于开始时间，则免打扰模式在每天的开始时间到次日的结束时间内生效。例如，开始时间为 `10:00`，结束时间为 `08:00`，则免打扰模式的在当天的 10:00 到次日的 8:00 生效。</li><li>目前仅支持在每天的一个指定时间段内开启免打扰模式，不支持多个免打扰时间段，新的设置会覆盖之前的设置。</li><li>若不设置该参数，开始时间和结束时间分别传 `00`。</li></ul> | 仅用于 app 级别，对单聊或群聊会话不生效。 |
| `SILENT_MODE_DURATION`|  免打扰时长，单位为毫秒。免打扰时长的取值范围为 [0,604800000]，`0` 表示该参数无效，`604800000` 表示免打扰模式持续 7 天。<br/> 与免打扰时间段的设置长久有效不同，该参数为一次有效。    | App 或单聊/群聊会话。  |

若在免打扰时段或时长生效期间需要对指定用户推送消息，需设置[强制推送](#forced)。

**推送通知方式与免打扰时间设置之间的关系**

对于 app 和 app 中的所有会话，免打扰模式的设置优先于推送通知方式的设置。例如，假设在 app 级别指定了免打扰时间段，并将指定会话的推送通知方式设置为 `ALL`。免打扰模式与推送通知方式的设置无关，即在指定的免打扰时间段内，你不会收到任何推送通知。

或者，假设为会话指定了免打扰时长，而 app 没有任何免打扰设置，并且其推送通知方式设置为 `ALL`。在指定的免打扰时长内，你不会收到来自该会话的任何推送通知，而所有其他会话的推送保持不变。

### 设置 app 的推送通知

你可以调用 `setSilentModeForAll` 设置 app 级别的推送通知，并通过指定 `SilentModeParam` 字段设置推送通知方式和免打扰模式，如下代码示例所示：

```java       
//设置推送通知方式为 `MENTION_ONLY`。
SilentModeParam param = new SilentModeParam(SilentModeParam.SilentModeParamType.REMIND_TYPE)
                                .setRemindType(PushManager.PushRemindType.MENTION_ONLY);
//设置离线推送免打扰时长为 15 分钟。
SilentModeParam param = new SilentModeParam(SilentModeParam.SilentModeParamType.SILENT_MODE_DURATION)
                                .setSilentModeDuration(15);
//设置离线推送的免打扰时间段为 8:30 至 15:00。
SilentModeParam param = new SilentModeParam(SilentModeParam.SilentModeParamType.SILENT_MODE_INTERVAL)
                                .setSilentModeInterval(new SilentModeTime(8, 30), new SilentModeTime(15, 0));
//设置 app 的离线推送通知。
ChatClient.getInstance().pushManager().setSilentModeForAll(param, new ValueCallBack<SilentModeResult>(){});
```

### 获取 app 的推送通知设置

你可以调用 `getSilentModeForAll` 获取 app 级别的推送通知设置，如以下代码示例所示：

```java
ChatClient.getInstance().pushManager().getSilentModeForAll(new ValueCallBack<SilentModeResult>(){
    @Override
    public void onSuccess(SilentModeResult result) {
        // 获取 app 的推送通知方式的设置。
        PushManager.PushRemindType remindType = result.getRemindType();
        // 获取 app 的离线推送免打扰过期的 Unix 时间戳。
        long timestamp = result.getExpireTimestamp();
        // 获取 app 的离线推送免打扰时间段的开始时间。
        SilentModeTime startTime = result.getSilentModeStartTime();
        startTime.getHour(); // 免打扰时间段的开始时间中的小时数。
        startTime.getMinute(); // 免打扰时间段的开始时间中的分钟数。
        // 获取 app 的离线推送免打扰时间段的结束时间。
        SilentModeTime endTime = result.getSilentModeEndTime();
        endTime.getHour(); // 免打扰时间段的结束时间中的小时数。
        endTime.getMinute(); // 免打扰时间段的结束时间中的分钟数。
    }
    @Override
    public void onError(int error, String errorMsg) {}
});
```

### 设置单个会话的推送通知

你可以调用 `setSilentModeForConversation` 方法设置指定会话的推送通知，并通过指定 `SilentModeParam` 字段设置推送通知方式和免打扰模式，如以下代码示例所示：

```java
//设置推送通知方式为 `MENTION_ONLY`。
SilentModeParam param = new SilentModeParam(SilentModeParam.SilentModeParamType.REMIND_TYPE)
                                .setRemindType(PushManager.PushRemindType.MENTION_ONLY);

//设置离线推送免打扰时长为 15 分钟。
SilentModeParam param = new SilentModeParam(SilentModeParam.SilentModeParamType.SILENT_MODE_DURATION)
                                .setSilentDuration(15);
//设置会话的离线推送免打扰模式。目前，暂不支持设置会话免打扰时间段。
ChatClient.getInstance().pushManager().setSilentModeForConversation(conversationId, conversationType, param, new ValueCallBack<SilentModeResult>(){});
```

### 获取单个会话的推送通知设置

你可以调用 `getSilentModeForConversation` 方法获取指定会话的推送通知设置，如以下代码示例所示：

```java
ChatClient.getInstance().pushManager().getSilentModeForConversation(conversationId, conversationType, new ValueCallBack<SilentModeResult>(){
    @Override
    public void onSuccess(SilentModeResult result) {
        // 获取指定会话是否设置了推送通知方式。
        boolean enable = result.isConversationRemindTypeEnabled();
        // 检查会话是否设置了推送通知方式。
        if(enable){
            //获取会话的推送通知方式。
            PushManager.PushRemindType remindType = result.getRemindType();
        }
        // 获取会话的离线推送免打扰过期 Unix 时间戳。
        long timestamp = result.getExpireTimestamp();
    }
    @Override
    public void onError(int error, String errorMsg) {}
});
```

### 获取多个会话的推送通知设置

1. 你可以在每次调用中最多获取 20 个会话的推送通知设置。

2. 如果会话继承了 app 设置或其推送通知设置已过期，则返回的字典不包含此会话。

你可以调用 `getSilentModeForConversations` 方法获取多个会话的推送通知设置，如以下示例代码所示：

```java
ChatClient.getInstance().pushManager().getSilentModeForConversations(conversationList, new ValueCallBack<Map<String, SilentModeResult>>(){
    @Override
    public void onSuccess(Map<String, SilentModeResult> value) {}
    @Override
    public void onError(int error, String errorMsg) {}
});
```

### 清除单个会话的推送通知方式的设置

你可以调用 `clearRemindTypeForConversation` 方法清除指定会话的推送通知方式的设置。清除后，默认情况下，该会话会使用 app 的设置。

以下代码示例显示了如何清除会话的推送通知方式：

```java
ChatClient.getInstance().pushManager().clearRemindTypeForConversation(conversationId, conversationType, new CallBack(){});
```

## 设置显示属性

### 设置推送通知的显示属性<a name="display"></a>

你可以调用 `updatePushNickname` 方法设置推送通知中显示的昵称，如以下代码示例所示：

```java
// 需要异步处理。
ChatClient.getInstance().pushManager().updatePushNickname("pushNickname");
```

你也可以调用 `updatePushDisplayStyle` 方法设置推送通知的显示样式，如下代码示例所示：

```java
// `DisplayStyle` 设置为简单样式 `SimpleBanner`，只显示 "你有一条新消息"。若要显示消息内容，需设置为 `MessageSummary`。
PushManager.DisplayStyle displayStyle = PushManager.DisplayStyle.SimpleBanner;
// 需要异步处理。
ChatClient.getInstance().pushManager().updatePushDisplayStyle(displayStyle);
```

添加四个截图？单聊和群聊界面分别添加
a. 设置了推送昵称，`DisplayStyle` 设置为简单样式 `SimpleBanner`。
b. 设置了推送昵称，`DisplayStyle` 设置为显示消息内容 `MessageSummary`。

### 获取推送通知的显示属性

你可以调用 `getPushConfigsFromServer` 方法获取推送通知中的显示属性，如以下代码示例所示：

```java
PushConfigs pushConfigs = ChatClient.getInstance().pushManager().getPushConfigsFromServer();
// 获取推送显示昵称。
String nickname = pushConfigs.getDisplayNickname();
// 获取推送通知的显示样式。
PushManager.DisplayStyle style = pushConfigs.getDisplayStyle();
```

## 设置离线推送的首选语言 

推送通知与翻译功能协同工作。如果用户启用[自动翻译](./agora_chat_translation_android)功能并发送消息，SDK 会同时发送原始消息和翻译后的消息。

作为接收方，你可以设置你在离线时希望接收的推送通知的首选语言。如果翻译后消息的语言匹配你的设置，则翻译后消息显示在推送通知栏；否则，将显示原始消息。

以下示例代码显示如何设置和获取推送通知的首选语言：

```java
// 设置离线推送的首选语言。
ChatClient.getInstance().pushManager().setPreferredNotificationLanguage("en", new CallBack(){});

// 获取设置的离线推送的首选语言。
ChatClient.getInstance().pushManager().getPreferredNotificationLanguage(new ValueCallBack<String>(){});
```

## 设置推送模板

即时通讯 IM 支持自定义推送通知模板。使用前，你需要调用 RESTful 接口或参考以下步骤在声网控制台创建推送模板：

1. 登录[声网控制台](https://console.agora.io/)，在左侧导航栏中单击 **Project Management**。
2. 在 **Project Management** 页面上，在启用了即时通讯 IM 的项目的 **Action** 一栏中单击 **Config**。
3. 在 **Edit Project** 页面的 **Features** 区域，单击 **Chat** 对应的 **Enable/Config**。
4. 在项目配置页面，选择 **Features** > **Push Template**，单击 **Add Push Template**，在弹出的对话框中配置字段，如下图所示。

![image](\agora_doc_source\en\markdown\agora-chat\images\push\push_add_template.png)

5. 创建推送模板后，用户可以在发送消息时选择使用此模板，代码示例如下所示。

```java
// 下面以文本消息为例，其他类型的消息设置方法相同。
ChatMessage message = ChatMessage.createSendMessage(ChatMessage.Type.TXT);
TextMessageBody txtBody = new TextMessageBody("message content");
// 设置推送通知接收方。单聊为接收方的用户 ID，群聊为群组 ID。
message.setTo("receiver");  
// 将在声网控制台上或调用 RESTful 接口创建的推送模板设置为默认推送模板。
JSONObject pushObject = new JSONObject();
JSONArray titleArgs = new JSONArray();
JSONArray contentArgs = new JSONArray();
try {
        // 设置推送模板名称。
        pushObject.put("name", "test6");
        // 设置模板中推送标题的 value 数组。如果模板中指定的推送标题为占位数据，则在这里可自定义标题；若指定的标题为固定值，则使用该模板时标题为固定值。
        titleArgs.put("test1");
        //...
        pushObject.put("title_args", titleArgs);
        // 设置模板中的推送内容的 value 数组。如果模板中指定的模板内容为占位数据，则在这里可自定义内容；若指定的内容为固定值，则使用该模板时内容为固定值。
        contentArgs.put("$fromNickname");
        contentArgs.put("$msg");
        //...
        pushObject.put("content_args", contentArgs);
} catch (JSONException e) {
        e.printStackTrace();
}
// 将推送模板添加到消息中。
message.setAttribute("em_push_template", pushObject);
// 设置消息状态回调。
message.setMessageStatusCallback(new CallBack() {...});
// 发送消息。
ChatClient.getInstance().chatManager().sendMessage(message);
```

推送模板的 JSON 结构如下：

```json
"em_push_template":{
        "name":"test6",
        "title_args":[
            "test1"
        ],
        "content_args":[
            "{$fromNickname}",
            "{$msg}"
        ]
}
```

## 解析 FCM 推送字段 

收到推送通知后，你需要解析数据。

重写 `FirebaseMessagingService.onMessageReceived` 方法可以在 `RemoteMessage` 对象中获取自定义扩展字段，示例代码如下：

```java
public class FCMMSGService extends FirebaseMessagingService {
    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        super.onMessageReceived(remoteMessage);
        if (remoteMessage.getData().size() > 0) {
            String f = remoteMessage.getData().get("f");
            String t = remoteMessage.getData().get("t");
            String m = remoteMessage.getData().get("m");
            String g = remoteMessage.getData().get("g");
            Object e = remoteMessage.getData().get("e");
        }
    }
}
```

| 参数    | 描述           |
| ------- | -------------- |
| `f` | 推送通知的发送方的用户 ID。     |
| `t` | 推送通知的接收方的用户 ID。     |
| `m` | 消息 ID。消息唯一标识符。     |
| `g` | 群组 ID，仅当消息为群组消息时，该字段存在。     |
| `e` | 用户自定义扩展字段，数据来源为 `em_apns_ext` 或 `em_apns_ext.extern`。<br/> - 当 `extern` 不存在时，`e` 内容为 `em_apns_ext` 下推送服务未使用字段，即除 `em_push_title`，`em_push_content`，`em_push_name`，`em_push_channel_id` 和 `em_huawei_push_badge_class` 之外的其他字段。<br/> - 当 `extern` 存在时，使用 `extern` 下字段。 |

`RemoteMessage` 对象中的扩展信息的数据结构如下：

```java
{
    "t":"receiver",
    "f":"fromUsername",
    "m":"msg_id",
    "g":"group_id",
    "e":{}
}
```

### 自定义推送字段

创建推送消息时，可以在消息中添加自定义字段，满足个性化业务需求。

```java
// 本示例以文本消息为例，图片和文件等消息类型的设置方法相同。
ChatMessage message = ChatMessage.createSendMessage(ChatMessage.Type.TXT);
TextMessageBody txtBody = new TextMessageBody("message content");
// 设置消息接收方：单聊为对端用户的用户 ID；群聊为群组 ID。
message.setTo("receiver");
// 设置自定义推送字段。
JSONObject extObject = new JSONObject();
try {
    extObject.put("test1", "test 01"); 
} catch (JSONException e) {
    e.printStackTrace();
}
// 将推送扩展设置到消息中。
message.setAttribute("em_apns_ext", extObject);
// 设置消息体。
message.addBody(txtBody);
// 设置消息回调。
message.setMessageStatusCallback(new CallBack() {...});
// 发送消息。
ChatClient.getInstance().chatManager().sendMessage(message);
```

包含自定义字段的消息的数据结构如下：

```json
{
    "em_apns_ext": {
        "test1": "test 01",
        "em_push_collapse_key": "collapseKey"
    }
}
```
| 参数             | 描述               |
| :--------------- | :----------------- |
| `em_apns_ext`    | 消息扩展字段。该字段名固定，不可修改。     |
| `test1`          | 用户添加的自定义 key。  |
| `em_push_collapse_key`   | 指定一组可折叠的消息（例如，含有 collapse_key: “Updates Available”），以便当恢复传送时只发送最后一条消息。这是为了避免当设备恢复在线状态或变为活跃状态时重复发送过多相同的消息。   |

### 自定义推送显示

创建推送消息时，你可以设置消息扩展字段自定义要显示的推送内容。

对于推送通知的显示属性，即推送昵称和推送通知显示样式，除了调用具体方法设置，你还可以通过自定义字段设置。若你同时采用了这两种方法，设置的自定义字段优先级较高。

```java
// 本示例以文本消息为例，图片和文件等消息类型的设置方法相同。
ChatMessage message = ChatMessage.createSendMessage(ChatMessage.Type.TXT);
TextMessageBody txtBody = new TextMessageBody("message content");
// 设置消息接收方：单聊为对端用户的用户 ID；群聊为群组 ID。
message.setTo("receiver"); 
// 设置自定义推送显示。
JSONObject extObject = new JSONObject();
try {
    extObject.put("em_push_title", "custom push title");
    extObject.put("em_push_content", "custom push content");
} catch (JSONException e) {
    e.printStackTrace();
}
// 将推送扩展设置到消息中。
message.setAttribute("em_apns_ext", extObject);
// 设置消息体。
message.addBody(txtBody);
// 设置消息回调。
message.setMessageStatusCallback(new CallBack() {...});
// 发送消息。
ChatClient.getInstance().chatManager().sendMessage(message);
```

包含自定义显示字段的消息的结构如下：

```java
{
    "em_apns_ext": {
        "em_push_title": "custom push title",
        "em_push_content": "custom push content"
    }
}
```

| 参数              | 描述               |
| :---------------- | :----------------- |
| `em_apns_ext`     | 消息扩展字段。该字段名固定，不可修改。     |
| `em_push_title`   | 自定义推送消息标题。该字段名固定，不可修改。     |
| `em_push_content` | 自定义推送消息内容。该字段名固定，不可修改。     |

### 强制推送<a name="forced"></a>

设置强制推送后，用户发送消息时会忽略接收方的免打扰设置，不论是否处于免打扰时间段都会正常向接收方推送消息。

```java
// 本示例以文本消息为例，图片和文件等消息类型的设置方法相同。
ChatMessage message = ChatMessage.createSendMessage(ChatMessage.Type.TXT);
TextMessageBody txtBody = new TextMessageBody("test");
// 设置消息接收方：单聊为对端用户的用户 ID；群聊为群组 ID。
message.setTo("receiver");
// 设置自定义扩展字段。
message.setAttribute("em_force_notification", true);
// 设置消息回调。
message.setMessageStatusCallback(new CallBack() {...});
// 发送消息。
ChatClient.getInstance().chatManager().sendMessage(message);
```

| 参数                    | 描述                                                        |
| :---------------------- | :---------------------------------------------------------- |
| `txtBody`               | 推送消息内容。                                              |
| `receiver`        | 消息接收方：<ul><li>单聊为对端用户的用户 ID；</li><li>群聊为群组 ID。</li></ul>                                        |
| `em_force_notification` | 是否为强制推送：<ul><li>`true`：强制推送；</li><li>（默认）`false`：非强制推送。</li></ul><br/>该字段名固定，不可修改。 |

### 发送静默消息

发送静默消息指发送方在发送消息时设置不推送消息，即用户离线时，即时通讯 IM 服务不会通过 FCM 推送服务向该用户的设备推送消息通知。因此，用户不会收到消息推送通知。当用户再次上线时，会收到离线期间的所有消息。

发送静默消息和免打扰模式下均为不推送消息，区别在于发送静默消息为发送方在发送消息时设置，而免打扰模式为接收方设置在指定时间段内不接收推送通知。

```java
// 本示例以文本消息为例，图片和文件等消息类型的设置方法相同。
ChatMessage message = ChatMessage.createSendMessage(ChatMessage.Type.TXT);
TextMessageBody txtBody = new TextMessageBody("test");
// 设置消息接收方：单聊为对端用户的用户 ID；群聊为群组 ID。
message.setTo("receiver");
// 设置自定义扩展字段。
message.setAttribute("em_ignore_notification", true);
// 设置消息回调。
message.setMessageStatusCallback(new CallBack() {...});
// 发送消息。
ChatClient.getInstance().chatManager().sendMessage(message);
```

| 参数                    | 描述                                                        |
| :---------------------- | :---------------------------------------------------------- |
| `txtBody`               | 消息内容。                                              |
| `receiver`        | 消息接收方：<ul><li>单聊为对端用户的用户 ID；</li><li>群聊为群组 ID。</li></ul>       |
| `em_ignore_notification` | 是否发送静默消息：<ul><li>`true`：发送静默消息；</li><li>（默认）`false`：推送该消息。</li></ul><br/>该字段名固定，不可修改。 |