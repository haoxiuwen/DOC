# 设置离线推送翻译

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