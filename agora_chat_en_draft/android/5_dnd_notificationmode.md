# 设置推送通知方式和免打扰模式

为优化用户在处理大量推送通知时的体验，即时通讯 IM 在 app 和会话层面提供了推送通知方式和免打扰模式的细粒度选项。 

**推送通知方式**

推送通知方式参数的说明如下表所示：

| 参数     | 描述   | App  | 单聊和群聊会话  |
| :------- | :----- | :----- | :----- |
| `ALL` | 接收所有离线消息的推送通知。 | ✓ | ✓ |
| `MENTION_ONLY` | 仅接收提及消息的推送通知。<br/>该参数推荐在群聊中使用。若提及一个或多个用户，需在创建消息时对 `ext` 字段传 "em_at_list":["user1", "user2" ...]；若提及所有人，对该字段传 "em_at_list":"all"。 | ✓ | ✓ |
| `NONE`   | 不接收离线消息的推送通知。 | ✓     | ✓ |

会话级别的推送通知方式设置优先于 app 级别的设置，未设置推送通知方式的会话默认采用 app 的设置。

例如，假设 app 的推送方式设置为 `MENTION_ONLY`，而指定会话的推送方式设置为 `ALL`。你会收到来自该会话的所有推送通知，而对于其他会话来说，你只会收到提及你的消息的推送通知。

**免打扰模式**

在你完成 SDK 初始化和成功登录 app 后，可以对 app 以及各类型的会话开启离线推送功能以及通过设置免打扰模式关闭推送。

你可以在 app 级别指定免打扰时间段和免打扰时长，即时通讯 IM 在这两个时间段内不发送离线推送通知。若既设置了免打扰时间段，又设置了免打扰时长，免打扰模式的生效时间为这两个时间段的累加。

免打扰时间参数的说明如下表所示：

| 参数     |  描述   |   App  | 单聊和群聊会话  |
| :--------| :----- | :------------------- | :----- |
| `SILENT_MODE_INTERVAL` | 免打扰时间段，精确到分钟，格式为 HH:MM-HH:MM，例如 08:30-10:00。该时间为 24 小时制，免打扰时间段的开始时间和结束时间中的小时数和分钟数的取值范围分别为 [00,23] 和 [00,59]。免打扰时间段的设置说明如下：<ul><li>开始时间和结束时间的设置立即生效，免打扰模式每天定时触发。例如，开始时间为 `08:00`，结束时间为 `10:00`，免打扰模式在每天的 8:00-10:00 内生效。若你在 11:00 设置开始时间为 `08:00`，结束时间为 `12:00`，则免打扰模式在当天的 11:00-12:00 生效，以后每天均在 8:00-12:00 生效。</li><li>若开始时间和结束时间相同，免打扰模式则全天生效。</li><li>若结束时间早于开始时间，则免打扰模式在每天的开始时间到次日的结束时间内生效。例如，开始时间为 `10:00`，结束时间为 `08:00`，则免打扰模式的在当天的 10:00 到次日的 8:00 生效。</li><li>目前仅支持在每天的一个指定时间段内开启免打扰模式，不支持多个免打扰时间段，新的设置会覆盖之前的设置。</li><li>若不设置该参数，开始时间和结束时间分别传 `00`。</li></ul> | ✓ | ✗ |
| `SILENT_MODE_DURATION`| 免打扰时长，单位为毫秒。免打扰时长的取值范围为 [0,604800000]，`0` 表示该参数无效，`604800000` 表示免打扰模式持续 7 天。<br/> 与免打扰时间段的设置长久有效不同，该参数为一次有效。    | ✓     | ✓ |

若在免打扰时段或时长生效期间需要对指定用户推送消息，需设置[强制推送](./7_push_extension#强制推送)。

**推送通知方式与免打扰时间设置之间的关系**

对于 app 和 app 中的所有会话，免打扰模式的设置优先于推送通知方式的设置。例如，假设在 app 级别指定了免打扰时间段，并将指定会话的推送通知方式设置为 `ALL`。免打扰模式与推送通知方式的设置无关，即在指定的免打扰时间段内，你不会收到任何推送通知。

或者，假设为会话指定了免打扰时长，而 app 没有任何免打扰设置，并且其推送通知方式设置为 `ALL`。在指定的免打扰时长内，你不会收到来自该会话的任何推送通知，而所有其他会话的推送保持不变。

## 设置 app 的推送通知

你可以调用 `setSilentModeForAll` 方法设置 app 级别的推送通知，并通过指定 `SilentModeParam` 字段设置推送通知方式和免打扰模式，如下代码示例所示：

```java       
//设置推送通知方式为 `MENTION_ONLY`。
SilentModeParam param = new SilentModeParam(SilentModeParam.SilentModeParamType.REMIND_TYPE)
                                .setRemindType(PushManager.PushRemindType.MENTION_ONLY);
//设置离线推送免打扰时长为 15 分钟。
SilentModeParam param = new SilentModeParam(SilentModeParam.SilentModeParamType.SILENT_MODE_DURATION)
                                .setSilentModeDuration(15);
//设置离线推送的免打扰时间段为 8:30 至 15:00。
SilentModeParam param = new SilentModeParam(SilentModeParam.SilentModeParamType.SILENT_MODE_INTERVAL)
                                .setSilentModeInterval(new SilentModeTime(8, 30), new SilentModeTime(15, 0));
//设置 app 的离线推送通知。
ChatClient.getInstance().pushManager().setSilentModeForAll(param, new ValueCallBack<SilentModeResult>(){});
```

## 获取 app 的推送通知设置

你可以调用 `getSilentModeForAll` 方法获取 app 级别的推送通知设置，如以下代码示例所示：

```java
ChatClient.getInstance().pushManager().getSilentModeForAll(new ValueCallBack<SilentModeResult>(){
    @Override
    public void onSuccess(SilentModeResult result) {
        // 获取 app 的推送通知方式的设置。
        PushManager.PushRemindType remindType = result.getRemindType();
        // 获取 app 的离线推送免打扰过期的 Unix 时间戳。
        long timestamp = result.getExpireTimestamp();
        // 获取 app 的离线推送免打扰时间段的开始时间。
        SilentModeTime startTime = result.getSilentModeStartTime();
        startTime.getHour(); // 免打扰时间段的开始时间中的小时数。
        startTime.getMinute(); // 免打扰时间段的开始时间中的分钟数。
        // 获取 app 的离线推送免打扰时间段的结束时间。
        SilentModeTime endTime = result.getSilentModeEndTime();
        endTime.getHour(); // 免打扰时间段的结束时间中的小时数。
        endTime.getMinute(); // 免打扰时间段的结束时间中的分钟数。
    }
    @Override
    public void onError(int error, String errorMsg) {}
});
```

## 设置单个会话的推送通知

你可以调用 `setSilentModeForConversation` 方法设置指定会话的推送通知，并通过指定 `SilentModeParam` 字段设置推送通知方式和免打扰模式，如以下代码示例所示：

```java
//设置推送通知方式为 `MENTION_ONLY`。
SilentModeParam param = new SilentModeParam(SilentModeParam.SilentModeParamType.REMIND_TYPE)
                                .setRemindType(PushManager.PushRemindType.MENTION_ONLY);

//设置离线推送免打扰时长为 15 分钟。
SilentModeParam param = new SilentModeParam(SilentModeParam.SilentModeParamType.SILENT_MODE_DURATION)
                                .setSilentDuration(15);
//设置会话的离线推送免打扰模式。目前，暂不支持设置会话免打扰时间段。
ChatClient.getInstance().pushManager().setSilentModeForConversation(conversationId, conversationType, param, new ValueCallBack<SilentModeResult>(){});
```

## 获取单个会话的推送通知设置

你可以调用 `getSilentModeForConversation` 方法获取指定会话的推送通知设置，如以下代码示例所示：

```java
ChatClient.getInstance().pushManager().getSilentModeForConversation(conversationId, conversationType, new ValueCallBack<SilentModeResult>(){
    @Override
    public void onSuccess(SilentModeResult result) {
        // 获取指定会话是否设置了推送通知方式。
        boolean enable = result.isConversationRemindTypeEnabled();
        // 检查会话是否设置了推送通知方式。
        if(enable){
            //获取会话的推送通知方式。
            PushManager.PushRemindType remindType = result.getRemindType();
        }
        // 获取会话的离线推送免打扰过期 Unix 时间戳。
        long timestamp = result.getExpireTimestamp();
    }
    @Override
    public void onError(int error, String errorMsg) {}
});
```

## 获取多个会话的推送通知设置

1. 你可以在每次调用中最多获取 20 个会话的推送通知设置。

2. 如果会话使用了 app 设置或其推送通知设置已过期，则返回的字典不包含此会话。

你可以调用 `getSilentModeForConversations` 方法获取多个会话的推送通知设置，如以下示例代码所示：

```java
ChatClient.getInstance().pushManager().getSilentModeForConversations(conversationList, new ValueCallBack<Map<String, SilentModeResult>>(){
    @Override
    public void onSuccess(Map<String, SilentModeResult> value) {}
    @Override
    public void onError(int error, String errorMsg) {}
});
```

## 清除单个会话的推送通知方式的设置

你可以调用 `clearRemindTypeForConversation` 方法清除指定会话的推送通知方式的设置。清除后，默认情况下，该会话会使用 app 的设置。

以下代码示例显示了如何清除会话的推送通知方式：

```java
ChatClient.getInstance().pushManager().clearRemindTypeForConversation(conversationId, conversationType, new CallBack(){});
```