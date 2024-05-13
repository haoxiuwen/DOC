# 在即时通讯 IM 中集成 FCM 并测试推送

本页介绍如何在即时通讯 IM 中集成 FCM 并测试推送是否成功集成。

## 集成 FCM 推送

按以下步骤在即时通讯 IM 中集成 FCM：

1. 在 [Firebase 控制台](https://console.firebase.google.com/)创建 Android 项目。
2. 在[声网控制台](https://console.agora.io/)上传 FCM 推送证书。
3. 在客户端集成 FCM。

<div class="alert info">对于使用 Android 系统的设备，若同时开启了 FCM 和其他厂商的推送服务，优先使用 FCM 推送服务。</div>

### 在 Firebase 控制台创建 Android 项目

1. 登录 [Firebase 控制台](https://console.firebase.google.com/)，[添加项目](https://firebase.google.com/docs/android/setup/#create-firebase-project)。

2. 在项目中[注册应用](https://firebase.google.com/docs/android/setup/#register-app)。

   在 **Add Firebase to your Android app** 页面中的 **Download and then add config file** 步骤中，点击 **Download google-services.json**，将该文件放入你的 Android 应用模块的根目录。

   [img](push_fcm_download_googleservice.png)

3. 查询 Sender ID。在 **Project settings** 页面，选择 **Cloud Messaging** 页签，查看 **Server ID**。在声网控制台上传 FCM 证书时，需要将证书名称设置为 FCM 的发送者 ID。

   [img](push_fcm_senderid.png)

4. 在 **Project settings** 页面，选择 **Service accounts** 页签，点击 **Generate new private key** 按钮生成密钥（JSON 文件）。请保存该文件，使用 V1 证书时需在声网控制台上传该文件。

  [img](push_fcm_private_key.png)

### 在声网控制台上传 FCM 推送证书

登录即时通讯 IM SDK 成功后，可在声网控制台上传 FCM 推送证书。

1. 登录[声网控制台](https://console.agora.io/)，在左侧导航栏中单击 **Project Management**。
2. 在 **Project Management** 页面上，在启用了即时通讯 IM 的项目的 **Action** 一栏中单击 **Config**。
3. 在 **Edit Project** 页面的 **Features** 区域，单击 **Chat** 对应的 **Enable/Config**。
4. 在项目配置页面，选择 **Features** > **Push Certificate**。
5. 在 **Push Certificate** 页面，单击 **Add Push Certificate**。在弹出的对话框中，选择 **Google** 页签，配置字段，单击**保存**。

[img](push_fcm_add_certificate.png)

| 参数       | 类型   | 是否必需 | 描述        |
| :--------- | :----- | :------- | :----------------------- |
| **Certificate Type**     |  | 否  | 选择使用旧版证书还是 V1 证书。<ul><li>**V1**： 你需要配置 **Private Key**。推荐使用 V1 证书。</li><li>**Old Version**：你需要配置 **Push Key**。老版本证书即将被弃用。</li></ul>|
| **Private Key**     | file | 是       | 点击 **Upload** 上传推送证书文件（.json）。你需要在 Firebase 控制台的 **Project settings** > **Service accounts** 页面点击 **Generate new private key** 生成的推送证书文件（.json）。 |
| **Push Key** | String | 是       | FCM 的服务器密钥（Server Key）。 你需在 Firebase 控制台的 **Project settings** > **Cloud Messaging** 页面，在 **Cloud Messaging API (Legacy)** 区域中获取服务器密钥。**该参数仅对旧版本证书有效。**|
| **Certificate Name** | String | 是       | 配置为 FCM 的发送者 ID。你可以在 FCM 控制台的 **Project settings** > **Cloud Messaging** 页面查看 **Sender ID** 参数的值。<br/> 证书名称是声网即时通讯服务器用于判断目标设备使用哪种推送通道的唯一条件，因此必须确保你[在即时通讯 IM 中集成 FCM 时设置的 Sender ID](#initialization)与这里设置的一致。 |
| **Sound** | String | 否       | 接收方收到推送通知时的铃声标记。|
| **Push Priority** |  | 否       | 消息传递优先级，详见 [FCM 官网](https://firebase.google.cn/docs/cloud-messaging/concept-options#setting-the-priority-of-a-message)。 |
| **Push Msg Type** |  | 否       | 通过 FCM 向客户端发送的消息的类型，详见 [FCM 消息简介](https://firebase.google.cn/docs/cloud-messaging/concept-options#notifications_and_data_messages) 。<ul><li>**Data**：数据消息，由客户端应用处理。</li><li>**Notification**：通知消息，由 FCM SDK 自动处理。</li><li>**Both**：可以通过 FCM 客户端发送通知消息和数据消息。</li></ul>|

#### **旧版证书无缝切换至 V1 证书**

若你仍使用旧版证书，即 **Certificate Type** 选择 **Old Version**，你需要将 **Certificate Name** 设置为 FCM 的发送者 ID，**Push Key** 设置为 FCM 的服务器密钥。你需在 [Firebase 控制台](https://console.firebase.google.com)的 ***Project settings > Cloud Messaging** 页面中，在 **Cloud Messaging API (Legacy)** 区域中获取发送者 ID 和服务器密钥，如下图所示。配置完毕，设置 **Sound**、**Push Priority** 和 **Push Msg Type** 参数。

![image](@static/images/android/push/fcm_old_version.png)

**旧版 HTTP 或 XMPP API 于 2024 年 6 月 20 日停用，请尽快迁移到最新的 FCM API（HTTP v1）版本证书。详见 [FCM 控制台](https://console.firebase.google.com)。请确保 V1 证书可用，因为执行转换证书后，旧证书会被删除，若此时新证书不可用，会导致推送失败。**

你可以参考以下步骤从旧版证书无缝切换到 V1 新证书：

1. 在 **Push Certificate** 页面的旧版证书的 **Action** 栏中点击 **Edit**。

![image](push_fcm_oldcertificate_edit.png)

2. 在**Edit Push Certificate** 窗口的 **Google** 页签，将**Certificate Type**切换为 **V1**。

![image](push_fcm_oldcertificate_switch.png)

3. 点击 **Upload** 上传本地保存的 V1 证书文件（.json）。

![fcmapp](push_fcm_newcertificate_upload.png)

4. 点击 **OK** 完成切换。


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
    android:name=".FCMMSGService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT" />
    </intent-filter>
</service>
```

3. 初始化即时通讯 IM SDK 并开启 FCM。<a name="initialization"></a>

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
        // 设置是否开启 FCM。
        if(pushType == PushType.FCM) {
            return GoogleApiAvailabilityLight.getInstance().isGooglePlayServicesAvailable(MainActivity.this)
                        == ConnectionResult.SUCCESS;
        }
        return super.isSupportPush(pushType, pushConfig);
    }
});
```

4. 将 FCM 的 device token 传递给服务器。

在应用初始化时，FCM SDK 会为用户的设备上的客户端应用生成唯一的注册 token。由于 FCM 使用该 token 确定要将推送消息发送给哪个设备，因此，声网服务器需要获得客户端应用的注册 token 才能将通知请求发送给 FCM，然后 FCM 验证注册 token，将通知消息发送给 Android 设备。建议该段代码放在成功登录即时通讯 IM 后的主页面。

```java
// 检查是否启用了 FCM。
if(GoogleApiAvailabilityLight.getInstance().isGooglePlayServicesAvailable(MainActivity.this) != ConnectionResult.SUCCESS) {
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

FCM SDK 成功生成注册 token（device token）后，会传递给 `onNewToken` 回调。

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
   可以查看日志或调用[获取用户详情的 RESTful 接口](https://docs.agora.io/en/agora-chat/restful-api/user-system-registration#querying-a-user)确认 device token 是否绑定成功。
2. 开启应用通知栏权限。
3. 杀掉应用进程。
4. 在声网控制台发送测试消息。
   在左侧导航栏中选择 **Operation Management** > **User**。在 Users 页面中，在对应用户 ID 的 **Action** 栏中选择 **Send Admin Message**。在弹出的对话框中选择消息类型，输入消息内容，然后点击 **Send**。

   <div class="alert note">在 **Push Certificate** 页面中证书列表中，在每个证书的 **Action** 一栏中，点击 **More**，会出现 **Test**，这里是直接调用第三方接口推送，而 **Users** 页面中的消息发送测试是先调用即时通讯 IM 的发消息的接口，满足条件后（即用户离线、推送证书有效且绑定了 device token）再调第三方的接口进行推送。</div>

5. 查看设备是否收到推送通知。

### 故障排除

1. 检查在即时通讯 IM 中是否正确集成或启用了 FCM 推送。
   在左侧导航栏中选择 **Operation Management > User**。在用户管理页面中，在对应用户 ID 的 **Action** 栏中选择 **Push Certificate**。在弹出框中查看是否正确显示了证书名称和 device token。
2. 检查是否在声网控制台上传了正确的 FCM 证书。
3. 检查是否在聊天室中推送消息。聊天室不支持离线消息推送。
4. 检查设备是否为国行手机的 ROM。一些品牌的国产手机不支持 GMS 服务，需替换为海外发售的设备。
5. 检查发送消息时是否设置了只发在线(deliverOnlineOnly = true)。只发在线的消息不推送。