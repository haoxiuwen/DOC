# 离线推送

即时通讯 IM 支持集成 FCM 和 APNs 消息推送服务，为 iOS 开发者提供低延时、高送达、高并发、不侵犯用户个人数据的离线消息推送服务。

当客户端应用进程被关闭等原因导致用户离线，即时通讯 IM 会通过 FCM 或 APNs 消息推送服务向该离线用户的设备推送消息通知。当用户再次上线时，服务器会将离线期间的消息发送给用户。仅单聊和群聊会话支持离线推送，聊天室不支持。

本文介绍如何在客户端应用中实现 APNs 推送服务。

## 技术原理

下图展示了消息推送的基本工作流程：

![](push_apns_workflow.apns)

消息推送流程如下：

1. 用户 B（消息接收者）检查设备支持哪种推送渠道，即 app 配置了哪种第三方推送服务且满足该推送的使用条件。
2. 用户 B 根据配置的第三方推送 SDK 从第三方推送服务器获取推送 token。
3. 第三方推送服务器向用户 B 返回推送 token。
4. 用户 B 向声网即时通讯服务器上传推送证书名称和推送 token。
5. 用户 A 向 用户 B 发送消息。
6. 声网即时通讯服务器检查用户 B 是否在线。若在线，声网即时通讯服务器直接将消息发送给用户 B。
7. 若用户 B 离线，声网即时通讯服务器判断该用户的设备使用的推送服务类型。
8. 声网即时通讯服务器将将消息发送给第三方推送服务器。
9. 第三方推送服务器将消息发送给用户 B。

<div class="alert info">device token 是第三方推送厂商提供的推送 token，该 token 用于标识每台设备上每个应用。各推送厂商通过该 token 明确要发送的消息是发送给哪个设备的，然后将消息转发给设备，设备再通知应用程序。对于 APNs 推送，该 device token 指初次启动你的应用时，APNs SDK 为客户端应用实例生成的推送 token，你可以调用 registerForRemoteNotifications 方法获得 token。另外，如果退出即时通讯 IM 登录时不解绑 device token（调用 `logout` 方法时对 `aIsUnbindDeviceToken` 参数传 `NO` 表示不解绑 device token，传 `YES` 表示解绑 token），用户在推送证书有效期和 token 有效期内仍会接收到离线推送通知。</div>

## 前提条件

- 已开启即时通讯 IM，详见[开启和配置即时通讯服务](./enable_agora_chat)；
- 了解即时通讯 IM 套餐包中的 API 调用频率限制，详见[使用限制](./agora_chat_limitation)。

## 在即时通讯 IM 中集成 APNs

按以下步骤在即时通讯 IM 中集成 APNs：

1. 在[苹果的开发者平台](https://developer.apple.com/)创建推送证书。
2. 在[声网控制台](https://console.agora.io/)配置 APNs 推送。
3. 在客户端集成 APNs。

<a name="certificate"></a>

### 创建推送证书

APNs 支持 p8 和 p12 证书。声网服务端需要具备你的 APNs 证书才能与 APNs 通信，向客户端发送推送通知。

以下步骤介绍如何在[苹果的开发者平台](https://developer.apple.com/)创建 p12 证书： 

1. 申请证书签名请求 Certificate Signing Request (CSR) 文件。<a name="step1-1"></a>
   i. 在设备上打开 **Keychain Access** 应用，选择 **Keychain Access** > **Certificate Assistant** > **Request a Certificate from a Certificate Authority**。
   ii. 在 **Certificate Assistant** 对话框中填写 **User Email Address**（电子邮件地址）和 **Common Name**（常用名称），对 **Request is** 选择 **Saved to disk**，点击 **Continue**，添加存储路径保存文件。
   ![](https://web-cdn.agora.io/docs-files/1642564150801)
   iii. 该存储路径下生成了 CSR 文件 `CertificateSigningRequest.certSigningRequest`。

2. 创建 App ID。<a name="step1-2"></a>
    i. 登录 [iOS Developer Center](https://developer.apple.com/cn/)，选择 **Account** > **Certificates, Identifiers & Profiles** > **Identifiers**。
    ii. 在 **Identifiers** 页签，点击 **Identifiers** 右侧的 **+**。
    iii. 在 **Register a new identifier** 页面中，选择 `App ID`，点击 `Continue`。
    iv. 对 **Select a type** 选择 **App**，点击 **Continue**。
    v. 在 **Register an App ID** 页面中，配置如下字段：
       - **Description**: App ID 的描述信息。
       - **Bundle ID**: 可以设置为 `com.YourCompany.YourProjectName`。
       - **Capabilities**: 选择 **Push Notification**。
    
    vi. 确定信息无误，点击 `Register`。
    
3. 分别创建开发环境和生产环境的消息推送证书。<a name="step1-3"></a>
   i. 在 **Identifiers** 页签中，选择[步骤 2](#step1-2)中创建的 **App ID**。
   ii. 在 **Edit your App ID Configuration** 页面，找到 **Push Notifications**，点击 **Configure**。
   iii. 在 **Apple Push Notification service SSL Certificates** 对话框中，点击 **Create Certificate** 创建适用于开发环境或生产环境的推送证书。
   iv. 在 **Create a New Certificate** 页面，**Platform** 选择 **iOS**，上传[步骤 1](#step1-1) 中创建的 CSR 文件，点击  **Continue**。
   v. 在 **Download Your Certificate** 页面，点击 **Download** 生成 [APNs](https://help.apple.com/xcode/mac/current/?spm=a2c4g.11186623.0.0.14864088B1zf4p#/dev80c6204ec) 证书。

4. 生成推送证书。<a name="step1-4"></a>
   i. 双击导入[步骤 3](#step1-3)中 Keychain 中创建的推送证书。
   ii. 打开 **Keychain Access**，选择  **login** > **Certificates**，找到已经导入的证书，右键选择该证书导出为 `.p12`  文件，设置证书密钥。

5. 生成 Provisioning Profile 文件。
   i. 登录 [iOS Developer Center](https://developer.apple.com/cn/)，选择 **Account** > **Certificates, Identifiers & Profiles** > **Profiles**。
   ii. 在 **Provisioning** 页签，点击 **Profiles** 右侧的 **+** 图标。
   iii. 在 **Register a New Provisioning Profile** 页面，**Development** 选择 **iOS App Development**，**Distribution** 选择 **Ad Hoc**，点击 **Continue**。
    对应于 App Store 上的正式版本，**Distribution** 选择 **App Store** 。
   iv. 在 **Generate a Provisioning Profile** 页面，配置如下字段：
      - **App ID**：填写[步骤 2](#step1-2)创建的 App ID。
      - **Select Certificates**：选择[步骤 4](#step1-4)中生成的 `.p12` 文件。
      - **Select Devices**：选择待开发的设备。
      - **Provisioning Profile Name**：填写 Provisioning Profile 文件名称。
   v. 确认信息，点击 `Download` 生成 Provisioning Profile 文件。

### 在声网控制台配置 APNs 推送

登录即时通讯 IM SDK 成功后，可在声网控制台配置多设备登录场景下的推送策略以及上传 APNs 推送证书。
1. 登录[声网控制台](https://console.agora.io/)，在左侧导航栏中单击 **Project Management**。
2. 在 **Project Management** 页面上，在启用了即时通讯 IM 的项目的 **Action** 一栏中单击 **Config**。
3. 在 **Edit Project** 页面的 **Features** 区域，单击 **Chat** 对应的 **Enable/Config**。
4. 在项目配置页面，选择 **Features** > **Push Certificate**。

![image](push_multidevice_policy.png)

**配置多设备登录时的推送策略**

在 **Push Certificate** 页面，配置多设备登录场景下的推送策略：
- 所有设备离线时，才发送推送消息。
- 任一设备离线时，都发送推送消息。

**上传 APNs 推送证书**

在 **Push Certificate** 页面，单击 **Add Push Certificate**。 在弹出的对话框中，选择 **Apple** 页签，配置字段。

<div class="alert info">若使用 FCM 推送，需选择 **Google** 页签，配置 FCM 推送参数。</div>

[image](push_apns_add_certificate.png)

| 参数       | 类型   | 是否必需 | 描述        |
| :--------- | :----- | :------- | :----------------------- |
| **Certificate Type**      |  | 是 | 消息推送证书类型，目前支持 **p8** 和 **p12**。        |
| **Certificate Name**      | String  | 是 | 消息推送证书名称。填写在[创建推送证书](#certificate)的[第三步](##step1-3)中创建的消息推送证书名称。必须确保你[在即时通讯 IM 中集成 APNs 时设置的证书名称]() 一致 |
| **Push Key**      | String  | 是 | 消息推送证书密钥。填写在[创建推送证书](#certificate)的[第四步](#step1-4)中导出消息推送证书文件时设置的证书密钥。该参数仅在使用 p12 证书时需要配置。  |
| **Upload Certificate**      | file  | 是 | 点击 `Upload` 上传[创建推送证书](#certificate)的[第四步](#step1-4)中获取的消息推送证书文件。  |
| **Key ID**      | String  | 是 | 输入推送证书的 Key ID。该参数仅在使用 p8 证书时需要配置。  |
| **Team ID**      | String  | 是 | 输入推送证书的 Team ID。该参数仅在使用 p8 证书时需要配置。  |
| **Integration Environment**      | | 是 | 集成环境：<br/> - **Development**：开发环境；<br/> - **Production**：生产环境。 |
| **Bundle ID**      | String  | 是 | 绑定 ID。[创建推送证书](#certificate)的[第二步](#step1-2)中创建 App ID 时设置的 Bundle ID。 |
| **sound**      | String  | 否 | 接收方收到推送通知时的铃声提醒。 |

### 在客户端集成 APNs 推送

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
    
    // 填写上传证书时设置的名称
    options.apnsCertName = @"PushCertName";
    
    [AgoraChatClient.sharedClient initializeSDKWithOptions:options];
    
    return YES;
}
```

3. 获取 Device Token 并传递给 SDK。

Device Token 注册后，iOS 系统会通过以下方式将 Device Token 回调给你，你需要将 Device Token 传给 SDK。

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

   在左侧导航栏中选择 **Operation Management** > **User**。在用户管理页面中，在对应用户 ID 的 **Action** 栏中选择 **Send Admin Message**。

    在弹出的对话框中选择消息类型，输入消息内容，然后点击 **Send**。

4. 查看设备是否收到推送通知。

### 故障排除

1. 检查在即时通讯 IM 中是否正确集成或启用了 APNs 推送。

   在左侧导航栏中选择 **Operation Management** > **User**。在用户管理页面中，在对应用户 ID 的 **Action** 栏中选择 **Push Certificate**。

   在弹出框中查看是否正确显示了证书名称和 device token。

2. 检查是否在声网控制台上传了正确的 APNs 证书且设置了正确的证书环境。

3. 检查是否在聊天室中推送消息。聊天室不支持离线消息推送。

4. 检查发送消息时是否设置了只发在线(AgoraChatMessage#deliverOnlineOnly = YES)。只发在线的消息不推送。

## 设置推送通知 

为优化用户在处理大量推送通知时的体验，即时通讯 IM 在 app 和会话层面提供了推送通知方式和免打扰模式的细粒度选项。

**推送通知方式**

| 推送通知方式参数     | 描述   | 应用范围   |
| :------- | :----- | :----- |
| `All` | 接收所有离线消息的推送通知。 | App 和单聊/群聊会话 |
| `MentionOnly` | 仅接收提及消息的推送通知。<br/>该参数推荐在群聊中使用。若提及一个或多个用户，需在创建消息时对 `ext` 字段传 "em_at_list":["user1", "user2" ...]；若提及所有人，对该字段传 "em_at_list":"all"。 | App 和单聊/群聊会话 |
| `NONE`   | 不接收离线消息的推送通知。 | App 和单聊/群聊会话      |

会话级别的推送通知方式设置优先于 app 级别的设置，未设置推送通知方式的会话默认采用 app 的设置。

例如，假设 app 的推送方式设置为 `MentionOnly`，而指定会话的推送方式设置为 `All`。你会收到来自该会话的所有推送通知，而对于其他会话来说，你只会收到提及你的消息的推送通知。

**免打扰模式**

在你完成 SDK 初始化和成功登录 app 后，可以对 app 以及各类型的会话开启离线推送功能以及通过设置免打扰模式关闭推送。

你可以在 app 级别指定免打扰时间段和免打扰时长，即时通讯 IM 在这两个时间段内不发送离线推送通知。若既设置了免打扰时间段，又设置了免打扰时长，免打扰模式的生效时间为这两个时间段的累加。

免打扰时间参数的说明如下表所示：

| 免打扰时间参数     |  描述   |   应用范围 |
| :--------| :----- | :----------------------------------------------------------- |
| `silentModeStartTime & silentModeEndTime`  | 免打扰时间段，精确到分钟，格式为 HH:MM-HH:MM，例如 08:30-10:00。该时间为 24 小时制，免打扰时间段的开始时间和结束时间中的小时数和分钟数的取值范围分别为 [00,23] 和 [00,59]。免打扰时间段的设置说明如下：<ul><li>开始时间和结束时间的设置立即生效，免打扰模式每天定时触发。例如，开始时间为 `08:00`，结束时间为 `10:00`，免打扰模式在每天的 8:00-10:00 内生效。若你在 11:00 设置开始时间为 `08:00`，结束时间为 `12:00`，则免打扰模式在当天的 11:00-12:00 生效，以后每天均在 8:00-12:00 生效。</li><li>若开始时间和结束时间相同，免打扰模式则全天生效。</li><li>若结束时间早于开始时间，则免打扰模式在每天的开始时间到次日的结束时间内生效。例如，开始时间为 `10:00`，结束时间为 `08:00`，则免打扰模式的在当天的 10:00 到次日的 8:00 生效。</li><li>目前仅支持在每天的一个指定时间段内开启免打扰模式，不支持多个免打扰时间段，新的设置会覆盖之前的设置。</li><li>若不设置该参数，开始时间和结束时间分别传 `00`。</li></ul>  | 仅用于 app 级别，对单聊或群聊会话不生效。 |
| `silentModeDuration` |  免打扰时长，单位为毫秒。免打扰时长的取值范围为 [0,604800000]，`0` 表示该参数无效，`604800000` 表示免打扰模式持续 7 天。<br/> 与免打扰时间段的设置长久有效不同，该参数为一次有效。    | App 或单聊/群聊会话。  |

**推送通知方式与免打扰时间设置之间的关系**

对于 app 和 app 中的所有会话，免打扰模式的设置优先于推送通知方式的设置。例如，假设在 app 级别指定了免打扰时间段，并将指定会话的推送通知方式设置为 `All`。免打扰模式与推送通知方式的设置无关，即在指定的免打扰时间段内，你不会收到任何推送通知。

或者，假设为会话指定了免打扰时长，而 app 没有任何免打扰设置，并且其推送通知方式设置为 `All`。在指定的免打扰时长内，你不会收到来自该会话的任何推送通知，而所有其他会话的推送保持不变。

### 设置 app 的推送通知

你可以调用 `setSilentModeForAll` 设置 app 级别的推送通知，并通过指定 `AgoraChatSilentModeParam` 字段设置推送通知方式和免打扰模式，如下代码示例所示：

```Objective-C
// 设置推送通知方式为 `MentionOnly`。
AgoraChatSilentModeParam *param = [[AgoraChatSilentModeParam alloc]initWithParamType:AgoraChatSilentModeParamTypeRemindType];
    param.remindType = AgoraChatPushRemindTypeMentionOnly;
// 设置 app 的离线推送通知。
[[AgoraChatClient sharedClient].pushManager setSilentModeForAll:param completion:^(AgoraChatSilentModeResult *aResult, AgoraChatError *aError) {
            if (aError) {
                NSLog(@"setSilentModeForAll error---%@",aError.errorDescription);
            }
        }];
// 设置离线推送免打扰时长为 15 分钟。
AgoraChatSilentModeParam *param = [[AgoraChatSilentModeParam alloc]initWithParamType:AgoraChatSilentModeParamTypeDuration];
    param.silentModeDuration = 15;
//设置离线推送的免打扰时间段为 8:30 到 15:00。
AgoraChatSilentModeParam *param = [[AgoraChatSilentModeParam alloc]initWithParamType:AgoraChatSilentModeParamTypeInterval];
    param.silentModeStartTime = [[AgoraChatSilentModeTime alloc]initWithHours:8 minutes:30];
    param.silentModeEndTime = [[AgoraChatSilentModeTime alloc]initWithHours:15 minutes:0];
```

### 获取 app 的推送通知设置

你可以调用 `getSilentModeForAllWithCompletion` 获取 app 级别的推送通知设置，如以下代码示例所示：

```Objective-C
[[AgoraChatClient sharedClient].pushManager getSilentModeForAllWithCompletion:^(AgoraChatSilentModeResult *aResult, AgoraChatError *aError) {
            if (!aError) {
                // 获取 app 的推送通知方式的设置。
                AgoraChatPushRemindType remindType = aResult.remindType;
                // 获取 app 的离线推送免打扰过期的 Unix 时间戳。
                NSTimeInterval ex = aResult.expireTimestamp;
                // 获取 app 的离线推送免打扰时间段的开始时间。
                AgoraChatSilentModeTime *startTime = aResult.silentModeStartTime;
                // 获取 app 的离线推送免打扰时间段的结束时间。
                AgoraChatSilentModeTime *endTime = aResult.silentModeEndTime;
            }else{
                NSLog(@"getSilentModeForAll error---%@",aError.errorDescription);
            }
        }];
```

### 设置单个会话的推送通知

你可以调用 `setSilentModeForConversation` 方法设置指定会话的推送通知，并通过指定 `SilentModeParam` 字段设置推送通知方式和免打扰模式，如以下代码示例所示：

```Objective-C
// 设置推送通知方式为 `MentionOnly`。
AgoraChatSilentModeParam *param = [[AgoraChatSilentModeParam alloc]initWithParamType:AgoraChatSilentModeParamTypeRemindType];
    param.remindType = AgoraChatPushRemindTypeMentionOnly;

// 设置离线推送免打扰时长为 15 分钟。
AgoraChatSilentModeParam *param = [[AgoraChatSilentModeParam alloc]initWithParamType:AgoraChatSilentModeParamTypeDuration];
    param.silentModeDuration = 15;
// 设置会话的离线推送免打扰模式。目前，暂不支持设置会话免打扰时间段。
AgoraChatConversationType conversationType = AgoraChatConversationTypeGroupChat;
[[AgoraChatClient sharedClient].pushManager setSilentModeForConversation:@"conversationId" conversationType:conversationType params:param completion:^(AgoraChatSilentModeResult *aResult, AgoraChatError *aError) {
        if (aError) {
                NSLog(@"setSilentModeForConversation error---%@",aError.errorDescription);
         }
     }];
```

### 获取单个会话的推送通知设置

你可以调用 `getSilentModeForConversation` 获取指定会话的推送通知设置，如以下代码示例所示：

```Objective-C
[[AgoraChatClient sharedClient].pushManager getSilentModeForConversation:@"conversationId" conversationType:AgoraChatConversationTypeChat completion:^(AgoraChatSilentModeResult * _Nullable aResult, AgoraChatError * _Nullable aError) {
    }];
```

### 获取多个会话的推送通知设置

1. 你可以在每次调用中最多获取 20 个会话的推送通知设置。

2. 如果会话继承了 app 设置或其推送通知设置已过期，则返回的字典不包含此会话。

你可以调用 `getSilentModeForConversations` 获取多个会话的推送通知设置，如以下代码示例所示：

```Objective-C
AgoraChatConversation* conv1 = [AgoraChatClient.sharedClient.chatManager getConversationWithConvId:@"conversationId1"];
NSArray *conversations = @[conv1];
    [[AgoraChatClient sharedClient].pushManager getSilentModeForConversations:conversations completion:^(NSDictionary<NSString*,AgoraChatSilentModeResult*>*aResult, AgoraChatError *aError) {
            if (aError) {
                NSLog(@"getSilentModeForConversations error---%@",aError.errorDescription);
            }
        }];
```

### 清除单个会话的推送通知方式的设置

你可以调用 `clearRemindTypeForConversation` 方法清除指定会话的推送通知方式的设置。清除后，默认情况下，此会话会使用 app 的设置。

以下代码示例显示了如何清除会话的推送通知方式的设置：

```Objective-C
    [[AgoraChatClient sharedClient].pushManager clearRemindTypeForConversation:@"" conversationType:conversationType completion:^(AgoraChatSilentModeResult *aResult, AgoraChatError *aError) {
            if (aError) {
                NSLog(@"clearRemindTypeForConversation error---%@",aError.errorDescription);
            }
    }];
```

## 设置显示属性

### 设置推送通知的显示属性

你可以调用 `updatePushDisplayName` 设置推送通知中显示的昵称，如以下代码示例所示：

```Objective-C
[AgoraChatClient.sharedClient.pushManager updatePushDisplayName:@"displayName" completion:^(NSString * aDisplayName, AgoraChatError * aError) {
    if (aError) 
    {
        NSLog(@"update push display name error: %@", aError.errorDescription);
    }
}];
```

你也可以调用 `updatePushDisplayStyle` 设置推送通知的显示样式，如下代码示例所示：

```Objective-C
// 设置为简单样式 `AgoraChatPushDisplayStyleSimpleBanner`，只显示 "你有一条新消息"。若要显示消息内容，需设置为 `AgoraPushDisplayStyleMessageSummary`。
[AgoraChatClient.sharedClient.pushManager updatePushDisplayStyle:AgoraChatPushDisplayStyleSimpleBanner completion:^(AgoraChatError * aError)
{
    if(aError)
    {
        NSLog(@"update display style error --- %@", aError.errorDescription);
    }
}];
```

### 获取推送通知的显示属性

你可以调用 `getPushNotificationOptionsFromServerWithCompletion` 方法获取推送通知中的显示属性，如以下代码示例所示：

```Objective-C
[AgoraChatClient.sharedClient.pushManager getPushNotificationOptionsFromServerWithCompletion:^(AgoraChatPushOptions *aOptions, AgoraChatError *aError) {
        if (!aError) {
            // 获取推送通知中的显示昵称。
            NSString *displayName = aOptions.displayName;
            // 获取推送通知的显示样式。
            AgoraChatPushDisplayStyle displayStyle = aOptions.displayStyle;
        }
    }];
```

## 设置离线推送的首选语言

推送通知与翻译功能协同工作。如果用户启用[自动翻译功能](./agora_chat_translation_ios)并发送消息，SDK 会同时发送原始消息和翻译后的消息。

作为接收方，你可以设置你在离线时希望接收的推送通知的首选语言。如果翻译消息的语言匹配你的设置，则翻译消息显示在推送通知中；否则，将显示原始消息。

以下代码示例显示了如何设置和获取推送通知的首选语言：

```Objective-C
// 设置离线推送的首选语言。
[[AgoraChatClient sharedClient].pushManager setPreferredNotificationLanguage:@"EU" completion:^(AgoraChatError *aError) {
    if (aError) {
        NSLog(@"setPushPerformLanguageCompletion error---%@",aError.errorDescription);
    }
}];
// 获取设置的离线推送的首选语言。
[[AgoraChatClient sharedClient].pushManager getPreferredNotificationLanguageCompletion:^(NSString *aLanguageCode, AgoraChatError *aError) {
    if (!aError) {
        NSLog(@"getPushPerformLanguage---%@",aLanguageCode);
    }
}];
```

## 设置推送模板

即时通讯 IM 支持自定义推送通知模板。使用前，你需要调用 RESTful 接口或参考以下步骤在声网控制台创建推送模板：

1. 登录[声网控制台](https://console.agora.io/)，在左侧导航栏中单击 **Project Management**。
2. 在 **Project Management** 页面上，在启用了即时通讯 IM 的项目的 **Action** 一栏中单击 **Config**。
3. 在 **Edit Project** 页面的 **Features** 区域，单击 **Chat** 对应的 **Enable/Config**。
4. 在项目配置页面，选择 **Features** > **Push Template**，单击 **Add Push Template**，在弹出的对话框中配置字段，如下图所示。

![image](\agora_doc_source\en\markdown\agora-chat\images\push\push_add_template.png)

5. 创建推送模板后，用户可以在发送消息时选择使用此模板，代码示例如下所示。

```objective-C
// 下面以文本消息为例，其他类型的消息设置方法相同。
AgoraChatTextMessageBody *body = [[AgoraChatTextMessageBody alloc]initWithText:@"test"];
AgoraChatMessage *message = [[AgoraChatMessage alloc]initWithConversationID:@"conversationId" from:@"currentUsername" to:@"conversationId" body:body ext:nil];
       // 将在声网控制台上创建的推送模板设置为默认推送模板。
       NSDictionary *pushObject = @{
           @"name":@"templateName",// 设置推送模板名称。
           @"title_args":@[@"titleValue1"],// 设置推送标题变量。如果模板中指定的推送标题为占位数据，则在这里可自定义标题；若指定的标题为固定值，则使用该模板时标题为固定值。
           @"content_args":@[@"contentValue1"]// 设置推送内容变量。如果模板中指定的推送标题为占位数据，则在这里可自定义标题；若指定的标题为固定值，则使用该模板时标题为固定值。
       };
       message.ext = @{
           @"em_push_template":pushObject,
       };
       message.chatType = AgoraChatTypeChat;
[[AgoraChatClient sharedClient].chatManager sendMessage:message progress:nil completion:nil];
```
### 解析收到的推送字段

当设备收到推送并点击时，iOS 会通过 launchOptions 将推送中的 JSON 传递给 app，这样就可以根据推送的内容定制 app 的一些行为，比如页面跳转等。 当收到推送通知并点击推送时，app 获取推送内容的方法：

```objective-C
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
      NSDictionary *userInfo = launchOptions[UIApplicationLaunchOptionsRemoteNotificationKey];
  }
```

`userInfo` 的数据结构如下：

```json
{
    "aps":{
        "alert":{
            "body":"你有一条新消息"
        },   
        "badge":1,               
        "sound":"default"   
    },
    "f":"6001",                  
    "t":"6006", 
    "g":"1421300621769",    
    "m":"373360335316321408"
}
```

| 参数    | 描述                    |
| :------ | :---------------------- |
| `body`  | 显示内容。              |
| `badge` | 角标数。                |
| `sound` | 提示铃声。              |
| `f`     | 消息发送方 ID。         |
| `t`     | 消息接收方 ID。         |
| `g`     | 群组 ID，单聊无该字段。 |
| `m`     | 消息 ID。               |


## 更多功能

如果现有的模板不能满足你的要求，你还可以自定义推送通知。

### 自定义推送字段

创建推送消息时，可以在消息中添加自定义字段，满足个性化业务需求。

```Objective-C
AgoraChatTextMessageBody *body = [[AgoraChatTextMessageBody alloc] initWithText:@"test"];
    NSString* currentUsername = AgoraChatClient.sharedClient.currentUsername;
    NSString* conversationId = @"remoteId";
    AgoraChatMessage *message = [[AgoraChatMessage alloc] initWithConversationID:conversationId from:currentUsername to:conversationId body:body ext:nil];
    message.ext = @{@"em_apns_ext":@{@"extern":@{@"test":123}}};
    message.chatType = AgoraChatTypeChat;
    [AgoraChatClient.sharedClient.chatManager sendMessage:message progress:nil completion:nil];
```

| 参数             | 描述                |
| :--------------- | :------------------ |
| `body`           | 消息内容。      |
| `conversationId` | 消息所属的会话 ID。 |
| `from`           | 消息发送方的用户 ID。  |
| `to`             | 消息接收方的用户 ID。  |
| `em_apns_ext`    | 消息扩展字段。      |
| `extern`         | 消息扩展字段的 Key，该名称固定，不可修改。  |

包含自定义字段的消息的数据结构如下：

```json
{
  "payload": {
    "ext": {
      "em_apns_ext": {
        "extern":{}
      }
    }
  }
}
```

### 自定义推送显示

创建推送消息时，你可以设置消息扩展字段自定义要显示的推送内容。

对于推送通知的显示属性，即推送通知的显示属性和显示样式，除了调用具体方法，你还可以通过自定义字段设置。若你同时采用了这两种方法，设置的自定义字段优先级较高。

```Objective-C
AgoraChatTextMessageBody *body = [[AgoraChatTextMessageBody alloc] initWithText:@"test"];
    AgoraChatMessage *message = [[AgoraChatMessage alloc] initWithConversationID:conversationId from:AgoraChatClient.sharedClient.currentUsername to:conversationId body:body ext:nil];
    message.ext = @{@"em_apns_ext":@{@"em_push_content":@"custom push content"}};
    message.chatType = AgoraChatTypeChat;
    [AgoraChatClient.sharedClient.chatManager sendMessage:message progress:nil completion:nil];
```

| 参数              | 描述                |
| :---------------- | :------------------ |
| `body`            | 消息内容。     |
| `conversationId`  | 消息所属的会话 ID。 |
| `from`            | 消息发送方的用户 ID。  |
| `to`              | 消息接收方的用户 ID。  |
| `em_apns_ext`     | 消息扩展字段。      |
| `em_push_content` | 消息扩展字段的 Key，该名称固定，不可修改。  |

包含自定义显示字段的消息的结构如下：

```json
{
  "payload": {
    "ext": {
      "em_apns_ext": {
        "em_push_content":"自定义推送显示内容"
      }
    }
  }
}
```

| 参数              | 描述               |
| :---------------- | :----------------- |
| `em_push_title` | 自定义推送消息标题。该字段名固定，不可修改。     |
| `em_push_content` | 自定义推送消息内容。该字段名固定，不可修改。     |

### 自定义铃声

创建推送消息时，你可以自定义接收方收到消息时的提示音。你需要将音频文件加入到 app 中，并在推送中配置使用的音频文件名称。详见[苹果官方文档](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification?language=objc)。

```Objective-C
AgoraChatTextMessageBody *body = [[AgoraChatTextMessageBody alloc] initWithText:@"test"];
    AgoraChatMessage *message = [[AgoraChatMessage alloc] initWithConversationID:conversationId from:AgoraChatClient.sharedClient.currentUsername to:conversationId body:body ext:nil];
    message.ext = @{@"em_apns_ext":@{@"em_push_sound":@"custom.caf"}};
    message.chatType = AgoraChatTypeChat;
    [AgoraChatClient.sharedClient.chatManager sendMessage:message progress:nil comple****tion:nil];
```

| 参数             | 描述                 |
| :--------------- | :------------------- |
| `body`           | 消息内容。       |
| `conversationId` | 消息所属的会话 ID。  |
| `from`           | 消息发送方的用户 ID。   |
| `to`             | 消息接收方的用户 ID。   |
| `em_apns_ext`    | 消息扩展字段。       |
| `em_push_sound`  | 自定义提示铃声字段的 Key，该名称固定，不可修改。     |
| `custom.caf`     | 铃声的音频文件名称。 |

包含自定义铃声的消息的结构如下：

```json
{
  "payload": {
    "ext": {
      "em_apns_ext": {
        "em_push_sound":"custom.caf"
      }
    }
  }
}
```

### 强制推送

设置强制推送后，用户发送消息时会忽略接收方的免打扰设置，不论是否处于免打扰时间段都会正常向接收方推送消息。

```Objective-C
AgoraChatTextMessageBody *body = [[AgoraChatTextMessageBody alloc] initWithText:@"test"];
AgoraChatMessage *message = [[AgoraChatMessage alloc] initWithConversationID:conversationId from:AgoraChatClient.sharedClient.currentUsername to:conversationId body:body ext:nil];
message.ext = @{@"em_force_notification":@YES};
message.chatType = AgoraChatTypeChat;
[AgoraChatClient.sharedClient.chatManager sendMessage:message progress:nil completion:nil];
```

| 参数                    | 描述                                        |
| :---------------------- | :------------------------------------------ |
| `body`                  | 推送消息内容。                              |
| `conversationId`        | 消息所属的会话 ID。                         |
| `from`                  | 消息发送方的用户 ID。                          |
| `to`                    | 消息接收方的用户 ID。                          |
| `em_force_notification` | 是否为强制推送：<ul><li>`YES`：强制推送</li><li> （默认）`NO`：非强制推送。</li></ul><br/>该字段名固定，不可修改。|

### 发送静默消息

发送静默消息指发送方在发送消息时设置不推送消息，即用户离线时，即时通讯 IM 服务不会通过第三方厂商的消息推送服务向该用户的设备推送消息通知。因此，用户不会收到消息推送通知。当用户再次上线时，会收到离线期间的所有消息。

发送静默消息和免打扰模式下均为不推送消息，区别在于发送静默消息为发送方在发送消息时设置，而免打扰模式为接收方设置在指定时间段内不接收推送通知。

```Objective-C
AgoraChatTextMessageBody *body = [[AgoraChatTextMessageBody alloc] initWithText:@"test"];
AgoraChatMessage *message = [[AgoraChatMessage alloc] initWithConversationID:conversationId from:AgoraChatClient.sharedClient.currentUsername to:conversationId body:body ext:nil];
message.ext = @{@"em_ignore_notification":@YES};
message.chatType = AgoraChatTypeChat;
[AgoraChatClient.sharedClient.chatManager sendMessage:message progress:nil completion:nil];
```

| 参数                    | 描述                                        |
| :---------------------- | :------------------------------------------ |
| `body`                  | 推送消息内容。                              |
| `conversationId`        | 消息所属的会话 ID。                         |
| `from`                  | 消息发送方的用户 ID。                          |
| `to`                    | 消息接收方的用户 ID。                          |
| `em_ignore_notification` | 是否发送静默消息：<ul><li>`YES`：发送静默消息；</li><li> （默认）`NO`：推送该消息。</li></ul><br/>该字段名固定，不可修改。|

### 通知服务的扩展

如果你的目标平台是 iOS 10.0 或以上版本，你可以参考如下代码实现 [`UNNotificationServiceExtension`](https://developer.apple.com/documentation/usernotifications/unnotificationserviceextension?language=objc) 的富文本推送功能。

```Objective-C
AgoraChatTextMessageBody *body = [[AgoraChatTextMessageBody alloc] initWithText:@"test"];
AgoraChatMessage *message = [[AgoraChatMessage alloc] initWithConversationID:conversationId from:AgoraChatClient.sharedClient.currentUsername to:conversationId body:body ext:nil];
message.ext = @{@"em_apns_ext":@{@"em_push_mutable_content":@YES}};
message.chatType = AgoraChatTypeChat;
[AgoraChatClient.sharedClient.chatManager sendMessage:message progress:nil completion:nil];
```

| 参数                      | 描述                           |
| :------------------------ | :----------------------------- |
| `body`                    | 推送消息内容。                 |
| `conversationId`          | 消息所属的会话 ID。            |
| `from`                    | 消息发送方的用户 ID。             |
| `to`                      | 消息接收方的用户 ID。             |
| `em_apns_ext`             | 消息扩展字段，该字段名固定，不可修改。该字段用于配置富文本推送通知，包含自定义字段。 |
| `em_push_mutable_content` | 是否使用富文本推送通知（`em_apns_ext`）：<ul><li>`YES`：富文本推送通知；</li><li> （默认）`NO`：普通推送通知。<br/>该字段名固定，不可修改。  |

声网服务器会解析该推送消息，如下所示。也就是说，接收方收到如下结构的推送消息。

```json
{
    "aps":{
        "alert":{
            "body":"test"
        },  
        "badge":1,  
        "sound":"default",
        "mutable-content":1  
    },
    "f":"6001",  
    "t":"6006",  
    "m":"373360335316321408"  
}
```

| 参数              | 描述                                               |
| :---------------- | :------------------------------------------------- |
| `body`            | 推送消息内容。                                     |
| `badge`           | 角标数字。                                           |
| `sound`           | 接收方收到消息后的提示铃声。                                         |
| `mutable-content` | 设置为 `1` 表示激活 `UNNotificationServiceExtension`。 |
| `f`               | 消息发送方的用户 ID。                                 |
| `t`               | 消息接收方的用户 ID。                                 |
| `m`               | 消息 ID。                                          |