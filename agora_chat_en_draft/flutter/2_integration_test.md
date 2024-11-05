# 在即时通讯 IM 中集成 FCM 并测试推送

本页介绍如何在即时通讯 IM 中集成 FCM 并测试推送是否成功集成。

## 集成 FCM 推送

Note: Currently, the official Chat website shows the latest configuration on the Firebase Console and Agora Console. 
Therefore, you can ignore this part enclosed by "///////////////" during translation. 
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

按以下步骤在即时通讯 IM 中集成 FCM：

1. 在 [Firebase 控制台](https://console.firebase.google.com/)创建 Android 项目。
2. 在[声网控制台](https://console.agora.io/)上传 FCM 推送证书。
3. 在客户端集成 FCM。

<div class="alert info">对于使用 Android 系统的设备，若同时开启了 FCM 和其他厂商的推送服务，优先使用 FCM 推送服务。</div>

### 在 Firebase 控制台创建项目

1. 登录 [Firebase 控制台](https://console.firebase.google.com/)，在 Firebase 控制台 添加项目，点击**Add project**, 并根据需求创建项目;

2. 创建成功后，Add App, 选择 flutter 平台;

3. 根据 firebase 提示，安装 `Firebase CLI`并登录 Firebase ,创建 flutter 项目;

4. 安装 FlutterFire CLI, `dart pub global activate flutterfire_cli`;

5. 在需要添加FCM的 flutter 项目根目录执行 `flutterfire configure --project=your project name`;

6. 查询 Sender ID。在 Project settings 页面，选择 Cloud Messaging 页签，查看 Server ID。在声网控制台上传 FCM 证书时，需要将证书名称设置为 FCM 的发送者 ID;

    [img](push_fcm_senderid.png)

7. 在 **Project settings** 页面，选择 **Service accounts** 页签，点击 **Generate new private key** 按钮生成密钥（JSON 文件）。请保存该文件，使用 V1 证书时需在声网控制台上传该文件;

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
| **Push Key** | String | 是       | FCM 的服务器密钥（Server Key）。 你需在 Firebase 控制台的 **Project settings** > **Cloud Messaging** 页面，在 **Cloud Messaging API (Legacy)** 区域中获取服务器密钥。|
| **Certificate Name** | String | 是       | 配置为 FCM 的发送者 ID。你可以在 FCM 控制台的 **Project settings** > **Cloud Messaging** 页面查看 **Sender ID** 参数的值。<br/> 证书名称是声网即时通讯服务器用于判断目标设备使用哪种推送通道的唯一条件，因此必须确保你[在即时通讯 IM 中集成 FCM 时设置的 Sender ID](#initialization)与这里设置的一致。 |
| **Sound** | String | 否       | 接收方收到推送通知时的铃声标记。|
| **Push Priority** |  | 否       | 消息传递优先级，详见 [FCM 官网](https://firebase.google.cn/docs/cloud-messaging/concept-options#setting-the-priority-of-a-message)。 |
| **Push Msg Type** |  | 否       | 通过 FCM 向客户端发送的消息的类型，详见 [FCM 消息简介](https://firebase.google.cn/docs/cloud-messaging/concept-options#notifications_and_data_messages) 。<ul><li>**Data**：数据消息，由客户端应用处理。</li><li>**Notification**：通知消息，由 FCM SDK 自动处理。</li><li>**Both**：可以通过 FCM 客户端发送通知消息和数据消息。</li></ul>|

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

### 在客户端配置 FCM

1. 安装 `firebase_core`：在项目根目录执行 `flutter pub add firebase_core`。
2. 安装 `firebase_messaging`：在项目根目录执行 `flutter pub add firebase_messaging`。

**针对 Android 平台**

在你 flutter 项目的 `Android/build.gradle` 文件中，查看 `google-services` 版本，确保版本为 4.3.14 或以上。

```gradle

dependencies {
    // START: FlutterFire Configuration
    classpath 'com.google.gms:google-services:4.3.15'
    // END: FlutterFire Configuration
}

```

**针对 iOS 平台**

您必须先在 Xcode 项目中启用推送通知和后台模式，您的应用才能开始接收消息。

1. 打开 Xcode 项目工作区 (ios/Runner.xcworkspace)。
2. [启用推送通知](https://help.apple.com/xcode/mac/current/#/devdfd3d04a1)。
3. 启用后台提取和远程通知[后台执行模式](https://developer.apple.com/documentation/xcode/configuring-background-execution-modes)。


#### SDK 集成

1. 初始化即时通讯 IM SDK 并开启 FCM。<a name="initialization"></a>

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  // 替换为你的 Appkey。
  var options = ChatOptions(appKey: "Your Appkey", autoLogin: false);
  // 替换为你的 FCM 的 Sender ID。
  options.enableFCM("Your FCM sender id");
  // 初始化即时通讯 IM SDK。
  await ChatClient.getInstance.init(options);
  // 初始化 FCM SDK。
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(const MyApp());
}
```

2. 将 FCM 的 device token 传递给服务器。

在应用初始化时，FCM SDK 会为用户的设备上的客户端应用生成唯一的注册 token。由于 FCM 使用该 token 确定要将推送消息发送给哪个设备，因此，声网服务器需要获得客户端应用的注册 token 才能将通知请求发送给 FCM，然后 FCM 验证注册 token，将通知消息发送给设备。


```dart
// 请求推送权限
await FirebaseMessaging.instance.requestPermission();

FirebaseMessaging.instance.onTokenRefresh.listen((event) async {
    debugPrint('onTokenRefresh: $event');
    await ChatClient.getInstance.pushManager.updateFCMPushToken(event);
});

// 获取token 并上传到即使通讯。
final fcmToken = await FirebaseMessaging.instance.getToken();
if (fcmToken != null) {
  debugPrint('fcmToken: $fcmToken');
  await ChatClient.getInstance.pushManager.updateFCMPushToken(fcmToken);
} else {
  debugPrint('fcmToken is null');
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