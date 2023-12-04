# 解析收到的推送字段

当设备收到推送通知并点击时，iOS 系统会通过 `launchOptions` 将推送中的 JSON 传递给 app，这样就可以根据推送的内容定制 app 的一些行为，比如页面跳转等。当收到推送通知并点击推送时，app 获取推送内容的方法：

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
            "body":"您有一条新消息"
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
| `g`     | 群组 ID，仅当消息为群组消息时，该字段存在。 |
| `m`     | 消息 ID。               |