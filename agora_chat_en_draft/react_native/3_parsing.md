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
| `e`  | 用户自定义扩展字段。 |

其中 `e` 为完全用户自定义扩展，数据来源为消息扩展的 `em_push_ext.custom`，数据结构如下：

```json
{
    "em_push_ext": {
        "custom": {
            "key1": "value1",
            "key2": "value2"
        }
    }
}
```

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