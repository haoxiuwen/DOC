# 设置推送翻译 

推送通知与翻译功能协同工作。如果用户启用[自动翻译](./agora_chat_translation_android)功能并发送消息，SDK 会同时发送原始消息和翻译后的消息。

作为接收方，你可以设置你在离线时希望接收的推送通知的首选语言。如果翻译后消息的语言匹配你的设置，则翻译后消息显示在推送通知栏；否则，将显示原始消息。

以下示例代码显示如何设置和获取推送通知的首选语言：

```java
// 设置离线推送的首选语言。
ChatClient.getInstance().pushManager().setPreferredNotificationLanguage("en", new CallBack(){});

// 获取设置的离线推送的首选语言。
ChatClient.getInstance().pushManager().getPreferredNotificationLanguage(new ValueCallBack<String>(){});
```