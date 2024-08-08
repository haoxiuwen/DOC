# 设置推送扩展

你可以利用扩展字段实现自定义推送设置，自定义铃声、通知栏折叠、强制推送、发送静默消息以及富文本推送功能。

## 自定义推送字段

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

自定义字段的数据结构如下：

```json
{
    "em_apns_ext": {
      "extern": {"test": 123}
    }
}
```

| 参数             | 描述                |
| :--------------- | :------------------ |
| `em_apns_ext`    | 内置的消息扩展字段。      |
| `extern`         | 用户添加的自定义 key，可添加多个。 |

## 设置某些群成员收到推送通知

在离线推送免打扰模式下，若你在群组中发送消息时只希望某些群成员收到离线推送通知，可通过设置消息扩展字段实现。

```Objective-C
AgoraChatTextMessageBody* body = [[AgoraChatTextMessageBody alloc] initWithText:@"hello"];
    AgoraChatMessage* msg = [[AgoraChatMessage alloc] initWithConversationID:@"groupId" body:body ext:nil];
    // 推送给群中所有成员时，设置为 `All`。
    msg.ext = @{@"em_apns_ext":@{@"em_at_list":@"All"}};
    //推送给指定群成员,设置为成员列表。
    msg.ext = @{@"em_apns_ext":@{@"em_at_list":@[@"userId1",@"userId2"]}};
    message.chatType = EMChatTypeGroupChat;
    [AgoraChatClient.sharedClient.chatManager sendMessage:msg progress:nil completion:nil];
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
| `em_at_list`          | 内置的扩展字段 key，对应的 value 为数组类型，表示接收推送通知的群成员的用户 ID，设置为 `All` 表示向所有群成员推送通知。  |

## 设置通知栏折叠 

你可以将通知栏中的多条消息折叠起来，示例代码如下：

```Objective-C
AgoraChatTextMessageBody *body = [[AgoraChatTextMessageBody alloc] initWithText:@"test"];
AgoraChatMessage *message = [[AgoraChatMessage alloc] initWithConversationID:conversationId body:body ext:nil];
message.ext = @{@"em_apns_ext":@{@"em_push_collapse_key":@"collapseKey"}};
message.chatType = AgoraChatTypeChat; 
[AgoraChatClient.sharedClient.chatManager sendMessage:message progress:nil completion:nil];
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
| `em_apns_ext`    | 内置的消息扩展字段。  |
| `em_push_collapse_key`   | 指定一组可折叠的消息（例如，含有 collapse_key: “Updates Available”），以便恢复传送时只发送最后一条消息，从而避免设备恢复在线状态或变为活跃状态时重复发送过多相同的消息。   |

## 自定义铃声

创建推送消息时，你可以自定义接收方收到消息时的提示音。你需要将音频文件加入到 app 中，并在推送中配置使用的音频文件名称。详见[苹果官方文档](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification?language=objc)。

```Objective-C
AgoraChatTextMessageBody *body = [[AgoraChatTextMessageBody alloc] initWithText:@"test"];
    AgoraChatMessage *message = [[AgoraChatMessage alloc] initWithConversationID:conversationId from:AgoraChatClient.sharedClient.currentUsername to:conversationId body:body ext:nil];
    message.ext = @{@"em_apns_ext":@{@"em_push_sound":@"custom.caf"}};
    message.chatType = AgoraChatTypeChat;
    [AgoraChatClient.sharedClient.chatManager sendMessage:message progress:nil completion:nil]; 
```

自定义铃声字段的数据结构如下：

```json
{
  "ext": {
    "em_apns_ext": {
      "em_push_sound":"custom.caf"
    }
  }
}
```

| 参数             | 描述                 |
| :--------------- | :------------------- |
| `em_apns_ext`    | 内置的消息扩展字段。       |
| `em_push_sound`  | 自定义提示铃声字段的 Key。该字段为内置字段，字段名不可修改。    |
| `custom.caf`     | 铃声的音频文件名称。 |

## 强制推送

设置强制推送后，用户发送消息时会忽略接收方的免打扰设置，不论是否处于免打扰时间段都会正常向接收方推送消息。

```Objective-C
AgoraChatTextMessageBody *body = [[AgoraChatTextMessageBody alloc] initWithText:@"test"];
AgoraChatMessage *message = [[AgoraChatMessage alloc] initWithConversationID:conversationId from:AgoraChatClient.sharedClient.currentUsername to:conversationId body:body ext:nil];
// 设置是否为强制推送，该扩展字段为内置字段，取值如下：`YES`：强制推送；（默认）`NO`：非强制推送。
message.ext = @{@"em_force_notification":@YES};
message.chatType = AgoraChatTypeChat;
[AgoraChatClient.sharedClient.chatManager sendMessage:message progress:nil completion:nil];
```

## 发送静默消息

发送静默消息指发送方在发送消息时设置不推送消息，即用户离线时，即时通讯 IM 服务不会通过第三方厂商的消息推送服务向该用户的设备推送消息通知。因此，用户不会收到消息推送通知。当用户再次上线时，会收到离线期间的所有消息。

发送静默消息和免打扰模式下均为不推送消息，区别在于发送静默消息为发送方在发送消息时设置，而免打扰模式为接收方设置在指定时间段内不接收推送通知。

```Objective-C
AgoraChatTextMessageBody *body = [[AgoraChatTextMessageBody alloc] initWithText:@"test"];
AgoraChatMessage *message = [[AgoraChatMessage alloc] initWithConversationID:conversationId from:AgoraChatClient.sharedClient.currentUsername to:conversationId body:body ext:nil];
// 设置是否发送静默消息。该字段为内置扩展字段，取值如下：`YES`：发送静默消息；（默认）`NO`：推送该消息。
message.ext = @{@"em_ignore_notification":@YES};
message.chatType = AgoraChatTypeChat;
[AgoraChatClient.sharedClient.chatManager sendMessage:message progress:nil completion:nil];
```

## 实现富文本推送

如果你的目标平台是 iOS 10.0 或以上版本，你可以参考如下代码实现 [`UNNotificationServiceExtension`](https://developer.apple.com/documentation/usernotifications/unnotificationserviceextension?language=objc) 的富文本推送功能。

```Objective-C
AgoraChatTextMessageBody *body = [[AgoraChatTextMessageBody alloc] initWithText:@"test"];
AgoraChatMessage *message = [[AgoraChatMessage alloc] initWithConversationID:conversationId from:AgoraChatClient.sharedClient.currentUsername to:conversationId body:body ext:nil];
// em_apns_ext：消息扩展字段
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
| `em_apns_ext`             | 内置的消息扩展字段。 |
| `em_push_mutable_content` | 是否使用富文本推送通知（`em_apns_ext`）：<ul><li>`YES`：富文本推送通知；</li><li>（默认）`NO`：普通推送通知。</li></ul><br/>该字段为内置字段，字段名不可修改。  |

接收方收到富文本推送时，会进入回调 `didReceiveNotificationRequest:withContentHandler:`，示例代码如下：

```Objective-C
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    // 推送扩展字段
    NSDictionary *userInfo = request.content.userInfo;
    // 通知内容
    UNNotificationContent *content = [request.content mutableCopy];
    contentHandler(content);
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