# 在即时通讯 IM 中集成 FCM 并测试推送

本页介绍如何在即时通讯 IM 中集成 FCM 并测试推送是否成功集成。

## 集成 FCM 推送

Note: Currently, the official Chat website shows the latest configuration on the Firebase Console and Agora Console. 
Therefore, you can ignore this part enclosed by "///////////////" during translation. 
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

按以下步骤在即时通讯 IM 中集成 FCM：

1. 在 [Firebase 控制台](https://console.firebase.google.com/)创建项目和应用。
2. 在[声网控制台](https://console.agora.io/)上传 FCM 推送证书。
3. 在客户端集成 FCM 组件。

### 在 Firebase 控制台创建项目

**对于 Android 平台:**

1. 登录 [Firebase 控制台](https://console.firebase.google.com/)，[添加项目](https://firebase.google.com/docs/android/setup#create-firebase-project)。

2. 在项目中[注册应用](https://firebase.google.com/docs/web/setup/#create-firebase-project)。

a. 在 **Add Firebase to your Android app** 页面中的 **Download and then add config file** 步骤中，点击 **Download google-services.json**，将该文件放入你的 Android 应用模块的根目录。

[img] push_fcm_download_googleservice.png

b. 首先，将 `google-services` 插件作为信赖，添加到 `/android/build.gradle` 文件中：

```gradle
buildscript {
  dependencies {
    classpath 'com.google.gms:google-services:4.3.15'
  }
}
```

c. 最后，将下面的代码添加到你的 `/android/app/build.gradle` 文件中以执行该插件：

```gradle
apply plugin: 'com.android.application'
apply plugin: 'com.google.gms.google-services' // <- Add this line
```

3. 查询 **Sender ID**。在 **Project settings** 页面，选择 **Cloud Messaging** 页签，查看 **Server ID**。在声网控制台上传 FCM 证书时，需要将证书名称设置为 FCM 的发送者 ID。

[img] push_fcm_senderid.png

4. 在 **Project settings** 页面，选择 **Service accounts** 页签，点击 **Generate new private key** 按钮生成密钥（JSON 文件）。请保存该文件，使用 V1 证书时需在声网控制台上传该文件。

[img](fcm_private-key)

**对于 iOS 平台：**

1. 登录 [Firebase 控制台](https://console.firebase.google.com/)，[添加项目](https://firebase.google.com/docs/ios/setup#create-firebase-project)。

2. 在项目中[注册应用](https://firebase.google.com/docs/ios/setup#register-app)。

   a. 在 **Add Firebase to your Apple app** 页面中的 **Download config file** 步骤中，点击 **Download GoogleService-Info.plist**，将该文件放入你的 Xcode 项目的根目录，然后再将其放入所有目标。

   [img](push_googleservice_Info_plist.png)

   b. 在文件顶部，在 `#import "AppDelegate.h"` 后面导入 Firebase SDK。

   ```objective-C
   #import <Firebase.h>
   ```

   c. 在当前的 `didFinishLaunchingWithOptions` 方法的顶部，添加以下代码：

   ```objective-C
   - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
   [FIRApp configure];}
   ```

3. 在 **Project settings** 页面的 **Cloud Messaging** 选项卡下，选择该应用，添加 [APNs 身份验证密钥](https://developer.apple.com/help/account/manage-keys/create-a-private-key)或者[证书](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/establishing_a_certificate-based_connection_to_apns?language=objc)。

4. 查询 **Sender ID** 和生成密钥（JSON 文件），与 Android 应用相同。

### 在声网控制台上传 FCM 推送证书

登录即时通讯 IM SDK 成功后，可在声网控制台上传 FCM 推送证书。

1. 登录[声网控制台](https://console.agora.io/)，在左侧导航栏中单击 **Project Management**。
2. 在 **Project Management** 页面上，在启用了即时通讯 IM 的项目的 **Action** 一栏中单击 **Config**。
3. 在 **Edit Project** 页面的 **Features** 区域，单击 **Chat** 对应的 **Enable/Config**。
4. 在项目配置页面，选择 **Features** > **Push Certificate**。

![image](push_multidevice_policy.png)

5. 在 **Push Certificate** 页面，单击 **Add Push Certificate**。 在弹出的对话框中，选择 **Google** 页签，配置字段，单击**保存**。

[img](push_fcm_add_certificate.png)

| 参数        | 类型   | 是否必需 | 描述        |
| :------------------- | :----- | :------- | :--------------- |
| **Certificate Type** |        | 否       | 选择使用旧版证书还是 V1 证书。<ul><li>**V1**：你需要配置 **Private Key**。推荐使用 V1 证书。</li><li>**Old Version**：你需要配置 **Push Key**。老版本证书即将被弃用。</li></ul>   |
| **Private Key**      | file   | 是       | 点击 **Upload** 上传推送证书文件（.json）。你需要在 Firebase 控制台的 **Project settings** > **Service accounts** 页面点击 **Generate new private key** 生成的推送证书文件（.json）。                |
| **Push Key**         | String | 是       | FCM 的服务器密钥（Server Key）。你需在 Firebase 控制台的 **Project settings** > **Cloud Messaging** 页面，在 **Cloud Messaging API (Legacy)** 区域中获取服务器密钥。    |
| **Certificate Name** | String | 是       | 配置为 FCM 的发送者 ID。你可以在 FCM 控制台的 **Project settings** > **Cloud Messaging** 页面查看 **Sender ID** 参数的值。<br/>证书名称是声网即时通讯服务器用于判断目标设备使用哪种推送通道的唯一条件。   |
| **Sound**            | String | 否       | 接收方收到推送通知时的铃声标记。       |
| **Push Priority**    |        | 否       | 消息传递优先级，详见 [FCM 官网](https://firebase.google.cn/docs/cloud-messaging/concept-options#setting-the-priority-of-a-message)。      |
| **Push Msg Type**    |        | 否       | 通过 FCM 向客户端发送的消息的类型，详见 [FCM 消息简介](https://firebase.google.cn/docs/cloud-messaging/concept-options#notifications_and_data_messages) 。<ul><li>**Data**：数据消息，由客户端应用处理。</li><li>**Notification**：通知消息，由 FCM SDK 自动处理。</li><li>**Both**：可以通过 FCM 客户端发送通知消息和数据消息。</li></ul> |

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

### 在客户端集成 FCM

对于 React Native 平台，在客户端集成 FCM 分为三个步骤：

1. 添加依赖。
2. 添加原生平台配置。
3. FCM 推送代码实现。

#### 添加依赖

```sh
yarn add @react-native-firebase/app @react-native-firebase/messaging react-native-agora-chat
```

#### 添加原生平台配置

**Android 平台**

1. 下载 `google-services.json` 文件，放入你的项目中的以下路径中：`/android/app/google-services.json`。

2. 在你的 `/android/build.gradle` 文件中，添加 google-services 插件作为依赖：

```groovy
buildscript {
  dependencies {
    classpath 'com.google.gms:google-services:4.3.15'
  }
}
```

3. 将 google-services 插件添加到 `/android/app/build.gradle` 文件中，执行插件：

```groovy
apply plugin: 'com.google.gms.google-services' // <- Add this line
```

**对于 iOS 平台**

1. 修改 `Podfile` 配置文件：

```ruby
# 设置 Reaction Native app 中集成的 Firebase SDK 的版本。
$FirebaseSDKVersion = '10.12.0'
# 设置 Firebase 使用静态框架。
$RNFirebaseAsStaticFramework = true
```

```ruby
target 'rn_firebase_push_demo' do
  pod 'GoogleUtilities', :modular_headers => true
  pod 'FirebaseCore', :modular_headers => true
end
```

2. 运行 `pod install` 命令在 `ios` 目录中安装依赖。

3. 在 Xcode 中打开 `/ios/{projectName}.xcworkspace` 文件。

4. 右击项目名称，选择 **Add files**，从本地选择下载的 `GoogleService-Info.plist` 文件，确保选择 `Copy items if needed`。

5. 为此，打开 `/ios/{projectName}/AppDelegate.mm` 文件（在 React Native 较早版本中打开 `AppDelegate.m`）。

6. 在文件顶部，在 `#import "AppDelegate.h"` 后面导入 Firebase SDK。

```objective-c
#import <Firebase.h>
```

7. 在 `didFinishLaunchingWithOptions` 方法中，在顶部添加以下代码：

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [FIRApp configure];
}
```

#### FCM 推送代码实现

1. 设置消息通知接收。

```typescript
React.useEffect(() => {
  const sub1 = messaging().onMessage(async (remoteMessage) => {
    // 进行离线消息通知的处理，例如打印日志和消息展示等。
    console.log("A new FCM message arrived!", JSON.stringify(remoteMessage));
  });
  return sub1;
}, []);
```

2. 初始化即时通讯 IM SDK。

```typescript
// 获取 device token。
fcmToken.current = await messaging().getToken();
// 推送设置。
const pushConfig = new ChatPushConfig({
  deviceId: senderId,
  deviceToken: fcmToken.current,
});
// 在 ChatOptions 类中配置推送设置。
let o = new ChatOptions({
  autoLogin: false,
  appKey: appKey,
  pushConfig: pushConfig,
});
// 执行设置，初始化即时通讯 IM SDK。
ChatClient.getInstance()
  .init(o)
  .then(() => {
    // 初始化成功。
  })
  .catch((error) => {
    // 初始化失败，返回错误信息。
  });
```

3. 登录服务器。

```typescript
// 使用用户 ID 和 token 登录服务器。
ChatClient.getInstance()
  .loginWithAgoraToken(username, token)
  .then(() => {
    // 登录成功。
  })
  .catch((reason) => {
    // 登录失败，返回错误信息。
  });
```

4. 发送 device token 到服务器。

```typescript
// 发送 device token 到服务器。
ChatClient.getInstance()
  .updatePushConfig(
    new ChatPushConfig({ deviceId: senderId, deviceToken: fcmToken.current })
  )
  .then(() => {
    // 发送成功。
  })
  .catch((reason) => {
    // 发送失败，返回错误信息。
  });
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
