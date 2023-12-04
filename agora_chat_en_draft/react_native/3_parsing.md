# 解析 FCM 推送字段

收到推送通知后，你需要解析数据。

重写 `FirebaseMessagingService.onMessageReceived` 方法可以在 `RemoteMessage` 对象中获取自定义扩展字段，示例代码如下：

```typescript
messaging().onMessage(async (remoteMessage) => {
  console.log("A new FCM message arrived!", JSON.stringify(remoteMessage));
  // "t":"receiver",
  // "f":"fromUsername",
  // "m":"msg_id",
  // "g":"group_id",
  // "e":{}
});
```

| 参数 | 描述     |
| ---- | --------------- |
| `f`  | 推送通知的发送方的用户 ID。    |
| `t`  | 推送通知的接收方的用户 ID。   |
| `m`  | 消息 ID。消息唯一标识符。   |
| `g`  | 群组 ID，仅当消息为群组消息时，该字段存在。   |
| `e`  | 用户自定义扩展字段，数据来源为 `em_apns_ext` 或 `em_apns_ext.extern`。<br/> - 当 `extern` 不存在时，`e` 内容为 `em_apns_ext` 下推送服务未使用字段，即除 `em_push_title`，`em_push_content`，`em_push_name`，`em_push_channel_id` 和 `em_huawei_push_badge_class` 之外的其他字段。<br/> - 当 `extern` 存在时，使用 `extern` 下字段。 |

`RemoteMessage` 对象中的扩展信息的数据结构如下：

```json
{
  "t": "receiver",
  "f": "fromUsername",
  "m": "msg_id",
  "g": "group_id",
  "e": {}
}
```