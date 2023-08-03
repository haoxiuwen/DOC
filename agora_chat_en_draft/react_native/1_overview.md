# 离线推送概述

即时通讯 IM 支持集成 FCM 消息推送服务，为 React-Native 开发者提供低延时、高送达、高并发、不侵犯用户个人数据的离线消息推送服务。

客户端断开连接或应用进程被关闭等原因导致用户离线时，即时通讯 IM 会通过 FCM 消息推送服务向该离线用户的设备推送消息通知。当用户再次上线时，服务器会将离线期间的消息发送给用户。若应用在后台运行，则用户仍为在线状态，即时通讯 IM 不会向用户推送消息通知。多设备登录时，可在声网控制台的 **Push Certificate** 页面配置推送在所有设备离线或任一设备离线时发送推送消息，该配置对所有推送通道生效。

<div class="alert note">1. 应用在后台运行或手机锁屏等情况，若客户端未断开与声网服务器的连接，则即时通讯 IM 不会收到离线推送通知。<br/>2. 多端登录时若有设备被踢下线，即使接入了 IM 离线推送，也收不到离线推送消息。</div>

除了满足用户离线条件外，要使用 FCM 离线推送，用户还需声网控制台配置 FCM 推送证书信息，例如 **Private Key** 和 **Certificate Name**，并向声网即时通讯服务器上传 device token。

要体验离线推送的 demo，请点击[这里](https://github.com/AgoraIO/Agora-Chat-API-Examples/tree/main/Chat-RN)。

## 推送流程

![](push_fcm_flow.png)

消息推送流程如下：

1. 用户 B 初始化 FCM 推送 SDK，检查是否使用了 FCM 推送。
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

使用 FCM 推送前，确保满足以下条件：

- 已开启即时通讯 IM ，详见[开启和配置即时通讯服务](./enable_agora_chat)。
- 了解即时通讯 IM 套餐包中的 API 调用频率限制，详见[使用限制](./agora_chat_limitation)。