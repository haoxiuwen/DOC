# 设置推送扩展功能

你可以利用扩展字段实现自定义推送设置，通知栏折叠、强制推送和发送静默消息。

## 设置自定义推送字段

创建推送消息时，可以在消息中添加自定义字段，满足个性化业务需求。

```java
// 本示例以文本消息为例，图片和文件等消息类型的设置方法相同。
ChatMessage message = ChatMessage.createSendMessage(ChatMessage.Type.TXT);
// 设置自定义推送字段。
JSONObject extObject = new JSONObject();
try {
    extObject.put("test1", "test 01"); 
} catch (JSONException e) {
    e.printStackTrace();
}
// 将推送扩展设置到消息中。
message.setAttribute("em_apns_ext", extObject);
```

自定义字段的数据结构如下：

```json
{
    "em_apns_ext": {
        "test1": "test 01"
    }
}
```

| 参数             | 描述               |
| :--------------- | :----------------- |
| `em_apns_ext`    | 内置的推送扩展字段。 |
| `test1`          | 用户添加的自定义 key，可添加多个。  |

## 设置某些群成员收到推送通知

在离线推送免打扰模式下，若你在群组中发送消息时只希望某些群成员收到离线推送通知，可通过设置消息扩展字段实现。

```java
// 本示例以文本消息为例，图片和文件等消息类型的设置方法相同。
ChatMessage message = ChatMessage.createSendMessage(ChatMessage.Type.TXT);
// 设置自定义推送字段。
JSONObject extObject = new JSONObject();
try {
    extObject.put("em_at_list", value); 
} catch (JSONException e) {
    e.printStackTrace();
}
// 将推送扩展设置到消息中。
message.setAttribute("em_apns_ext", extObject);
```

该扩展字段的数据结构如下：

```json
{
    "em_apns_ext": {
        "em_at_list": "All"
    }
}
```

| 参数             | 描述               |
| :--------------- | :----------------- |
| `em_apns_ext`    | 内置的推送扩展字段。 |
| `em_at_list`          | 用户添加的自定义 key。`value` 为字段的值，为数组类型，表示接收推送通知的群成员的用户 ID，设置为 `All` 表示向所有群成员推送通知。  |

## 设置通知栏折叠

你可以将通知栏中的多条消息折叠起来，示例代码如下：

```java
// 本示例以文本消息为例，图片和文件等消息类型的设置方法相同。
ChatMessage message = ChatMessage.createSendMessage(ChatMessage.Type.TXT);
// 设置自定义推送字段。
JSONObject extObject = new JSONObject();
try {
    extObject.put("em_push_collapse_key", "collapseKey"); 
} catch (JSONException e) {
    e.printStackTrace();
}
// 将推送扩展设置到消息中。
message.setAttribute("em_apns_ext", extObject);
```

通知栏折叠字段的数据结构如下：

```json
{
    "em_apns_ext": {
        "em_push_collapse_key": "collapseKey"
    }
}
```

| 参数             | 描述               |
| :--------------- | :----------------- |
| `em_apns_ext`    | 内置的推送扩展字段。 |
| `em_push_collapse_key`   | 指定一组可折叠的消息（例如，含有 collapse_key: “Updates Available”），以便恢复传送时只发送最后一条消息，从而避免设备恢复在线状态或变为活跃状态时重复发送过多相同的消息。   |

## 强制推送

设置强制推送后，用户发送消息时会忽略接收方的免打扰设置，不论是否处于免打扰时间段都会正常向接收方推送消息。

```java
// 本示例以文本消息为例，图片和文件等消息类型的设置方法相同。
ChatMessage message = ChatMessage.createSendMessage(ChatMessage.Type.TXT);
// 设置是否为强制推送，该字段为内置扩展字段，取值如下：`true`：强制推送；（默认）`false`：非强制推送。
message.setAttribute("em_force_notification", true);
```

## 发送静默消息

发送静默消息指发送方在发送消息时设置不推送消息，即用户离线时，即时通讯 IM 服务不会通过 FCM 推送服务向该用户的设备推送消息通知。因此，用户不会收到消息推送通知。当用户再次上线时，会收到离线期间的所有消息。

发送静默消息和免打扰模式下均为不推送消息，区别在于发送静默消息为发送方在发送消息时设置，而免打扰模式为接收方设置在指定时间段内不接收推送通知。

```java
// 本示例以文本消息为例，图片和文件等消息类型的设置方法相同。
ChatMessage message = ChatMessage.createSendMessage(ChatMessage.Type.TXT);
// 设置是否发送静默消息。该字段为内置扩展字段，取值如下：`true`：发送静默消息；（默认）`false`：推送该消息。
message.setAttribute("em_ignore_notification", true);
```