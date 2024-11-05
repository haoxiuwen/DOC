# 解析 FCM 推送字段 

收到推送通知后，你需要解析数据。

通过 `FirebaseMessaging.instance.getInitialMessage` 方法可以获取 app 打开时的自定义扩展字段，示例代码如下：

```dart
FirebaseMessaging.instance.getInitialMessage().then((message) {
      Map<String, dynamic>? data = message?.data;
      if (data != null) {
        String f = data['f'] ?? '';
        String t = data['t'] ?? '';
        String m = data['m'] ?? '';
        String g = data['g'] ?? '';
        Object e = data['e'] ?? '';
      }
});
```

| 参数    | 描述           |
| ------- | -------------- |
| `f` | 推送通知的发送方的用户 ID。     |
| `t` | 推送通知的接收方的用户 ID。     |
| `m` | 消息 ID。消息唯一标识符。     |
| `g` | 群组 ID，仅当消息为群组消息时，该字段存在。     |
| `e` | 用户自定义扩展字段。 |

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

`RemoteMessage.data` 对象中的扩展信息的数据结构如下：

```dart
{
    "t":"receiver",
    "f":"fromUsername",
    "m":"msg_id",
    "g":"group_id",
    "e":{}
}
```