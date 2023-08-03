# 在即时通讯 IM 中集成 APNs 并测试推送

本页介绍如何在即时通讯 IM 中集成 APNs 并测试推送是否成功集成。

## 集成 APNs 推送

按以下步骤在即时通讯 IM 中集成 APNs：

1. 在[苹果的开发者平台](https://developer.apple.com/)创建推送证书。
2. 在[声网控制台](https://console.agora.io/)上传 APNs 证书。
3. 在客户端集成 APNs。

<a name="certificate"></a>

### 在苹果开发者平台创建推送证书

APNs 支持 p8 和 p12 证书。声网服务端需要具备你的 APNs 证书才能与 APNs 通信，向客户端发送推送通知。

以下步骤介绍如何在[苹果的开发者平台](https://developer.apple.com/)创建 p12 证书： 

#### 步骤 1 申请证书签名请求 Certificate Signing Request (CSR) 文件<a name="step1-1"></a>

1. 在设备上打开 **Keychain Access** 应用，选择 **Keychain Access** > **Certificate Assistant** > **Request a Certificate from a Certificate Authority**。
2. 在 **Certificate Assistant** 对话框中填写 **User Email Address**（电子邮件地址）和 **Common Name**（常用名称），对 **Request is** 选择 **Saved to disk**，点击 **Continue**，添加存储路径保存文件。
   ![](https://web-cdn.agora.io/docs-files/1642564150801)
3. 该存储路径下生成了 CSR 文件 `CertificateSigningRequest.certSigningRequest`。

#### 步骤 2 创建 App ID<a name="step1-2"></a>

1. 登录 [iOS Developer Center](https://developer.apple.com/cn/)，选择 **Account** > **Certificates, Identifiers & Profiles** > **Identifiers**。
2.  在 **Identifiers** 页签，点击 **Identifiers** 右侧的 **+**。
3. 在 **Register a new identifier** 页面中，选择 `App IDs`，点击 `Continue`。
4. 对 **Select a type** 选择 **App**，点击 **Continue**。
5. 在 **Register an App ID** 页面中，配置如下字段：
 - **Description**: App ID 的描述信息。
 - **Bundle ID**: 可以设置为 `com.YourCompany.YourProjectName`。
 - **Capabilities**: 选择 **Push Notification**。   
6. 确定信息无误，点击 `Register`。
    
#### 步骤 3 分别创建开发环境和生产环境的消息推送证书<a name="step1-3"></a>

1. 在 **Identifiers** 页签中，选择[步骤 2](#step1-2)中创建的 **App ID**。
2. 在 **Edit your App ID Configuration** 页面，找到 **Push Notifications**，点击 **Configure**。
3. 在 **Apple Push Notification service SSL Certificates** 对话框中，点击 **Create Certificate** 创建适用于开发环境或生产环境的推送证书。
4. 在 **Create a New Certificate** 页面，**Platform** 选择 **iOS**，上传[步骤 1](#step1-1) 中创建的 CSR 文件，点击  **Continue**。
5. 在 **Download Your Certificate** 页面，点击 **Download** 生成 [APNs](https://help.apple.com/xcode/mac/current/?spm=a2c4g.11186623.0.0.14864088B1zf4p#/dev80c6204ec) 证书。

#### 步骤 4 生成推送证书<a name="step1-4"></a>

 1. 双击导入[步骤 3](#step1-3)中 Keychain 中创建的推送证书。
 2. 打开 **Keychain Access**，选择  **login** > **Certificates**，找到已经导入的证书，右键选择该证书导出为 `.p12`  文件，设置证书密钥。

#### 步骤 5 生成 Provisioning Profile 文件

1. 登录 [iOS Developer Center](https://developer.apple.com/cn/)，选择 **Account** > **Certificates, Identifiers & Profiles** > **Profiles**。
2. 在 **Provisioning** 页签，点击 **Profiles** 右侧的 **+** 图标。
3. 在 **Register a New Provisioning Profile** 页面，**Development** 选择 **iOS App Development**，**Distribution** 选择 **Ad Hoc**，点击 **Continue**。
    对应于 App Store 上的正式版本，**Distribution** 选择 **App Store** 。
4. 在 **Generate a Provisioning Profile** 页面，配置如下字段：
 - **App ID**：填写[步骤 2](#step1-2)创建的 App ID。
 - **Select Certificates**：选择[步骤 4](#step1-4)中生成的 `.p12` 文件。
 - **Select Devices**：选择待开发的设备。
 - **Provisioning Profile Name**：填写 Provisioning Profile 文件名称。
5. 确认信息，点击 **Download** 生成 Provisioning Profile 文件。

### 步骤 6 在声网控制台上传 APNs 证书

登录即时通讯 IM SDK 成功后，可在声网控制台配置多设备登录场景下的推送策略以及上传 APNs 推送证书。

1. 登录[声网控制台](https://console.agora.io/)，在左侧导航栏中单击 **Project Management**。
2. 在 **Project Management** 页面上，在启用了即时通讯 IM 的项目的 **Action** 一栏中单击 **Config**。
3. 在 **Edit Project** 页面的 **Features** 区域，单击 **Chat** 对应的 **Enable/Config**。
4. 在项目配置页面，选择 **Features** > **Push Certificate**。
5. 在 **Push Certificate** 页面，单击 **Add Push Certificate**。在弹出的对话框中，选择 **Apple** 页签，配置字段。

<div class="alert info">若使用 FCM 推送，需选择 **Google** 页签，配置 FCM 推送参数。</div>

[image](push_apns_add_certificate.png)

| 参数       | 类型   | 是否必需 | 描述        |
| :--------- | :----- | :------- | :----------------------- |
| **Certificate Type**      |  | 是 | 消息推送证书类型，目前支持 **p8** 和 **p12**。        |
| **Certificate Name**      | String  | 是 | 消息推送证书名称。填写在[创建推送证书](#certificate)的[第三步](##step1-3)中创建的消息推送证书名称。 |
| **Push Key**      | String  | 是 | 消息推送证书密钥。填写在[创建推送证书](#certificate)的[第四步](#step1-4)中导出消息推送证书文件时设置的证书密钥。该参数仅在使用 p12 证书时需要配置。  |
| **Upload Certificate**      | File  | 是 | 点击 `Upload` 上传[创建推送证书](#certificate)的[第四步](#step1-4)中获取的消息推送证书文件。  |
| **Key ID**      | String  | 是 | 输入推送证书的 Key ID。该参数仅在使用 p8 证书时需要配置。  |
| **Team ID**      | String  | 是 | 输入推送证书的 Team ID。该参数仅在使用 p8 证书时需要配置。  |
| **Integration Environment**      | | 是 | 集成环境：<br/> - **Development**：开发环境；<br/> - **Production**：生产环境。 |
| **Bundle ID**      | String  | 是 | 绑定 ID。[创建推送证书](#certificate)的[第二步](#step1-2)中创建 App ID 时设置的 Bundle ID。 |
| **sound**      | String  | 否 | 接收方收到推送通知时的铃声提醒。 |

### 在客户端集成 APNs

1. 打开 Xcode，选择 **Targets** > **Capability** > **Push Notifications** 开启消息推送权限。

2. 将证书名称传递给 SDK。

```Objective-C
#import <AgoraChat/AgoraChat.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // 申请通知权限
    UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
        [center requestAuthorizationWithOptions:(UNAuthorizationOptionBadge | UNAuthorizationOptionSound | UNAuthorizationOptionAlert) completionHandler:^(BOOL granted, NSError * _Nullable error) {
            if (granted) {
                NSLog(@"request authorization succeed");
            }
        }];
    
    // 注册推送
    [application registerForRemoteNotifications];
    
    // 初始化选项，并设置 App Key
    AgoraChatOptions *options = [AgoraChatOptions optionsWithAppkey:@"XXXX#XXXX"];
    
    // 填写上传证书时设置的名称。确保这里设置的证书名称与在声网控制台上填写的证书名称一致。
    options.apnsCertName = @"PushCertName";
    
    [AgoraChatClient.sharedClient initializeSDKWithOptions:options];
    
    return YES;
}
```

3. 获取 device token 并传递给即时通讯 IM SDK。

Device token 注册后，iOS 系统会通过以下方式将 device token 回调给你，你需要将 device token 传给 SDK。

```Objective-C
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    [AgoraChatClient.sharedClient registerForRemoteNotificationsWithDeviceToken:deviceToken completion:^(AgoraChatError *aError) {
        if (aError) {
            NSLog(@"bind deviceToken error: %@", aError.errorDescription);
        }
    }];
}
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error {
    NSLog(@"Register Remote Notifications Failed");
}
```

## 测试 APNs 推送

在即时通讯 IM 中集成并启用 APNs 推送后，可测试推送是否成功集成。

### 前提条件

准备一台使用 iOS 系统的未越狱的设备。

为了确保测试效果可靠，请避免使用模拟器进行测试。

### 测试步骤

1. 在设备上登录应用，并确认 device token 绑定成功。
2. 杀掉应用进程。
3. 在声网控制台发送测试消息。
   在左侧导航栏中选择 **Operation Management** > **User**。在用户管理页面中，在对应用户 ID 的 **Action** 栏中选择 **Send Admin Message**。在弹出的对话框中选择消息类型，输入消息内容，然后点击 **Send**。
4. 查看设备是否收到推送通知。

### 故障排除

1. 检查在即时通讯 IM 中是否正确集成或启用了 APNs 推送。
   在左侧导航栏中选择 **Operation Management** > **User**。在用户管理页面中，在对应用户 ID 的 **Action** 栏中选择 **Push Certificate**。在弹出框中查看是否正确显示了证书名称和 device token。
2. 检查是否在声网控制台上传了正确的 APNs 证书且设置了正确的证书环境。
3. 检查是否在聊天室中推送消息。聊天室不支持离线消息推送。
4. 检查发送消息时是否设置了只发在线(`AgoraChatMessage#deliverOnlineOnly = YES`)。只发在线的消息不推送。