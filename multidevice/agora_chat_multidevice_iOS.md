# 在多个设备上登录

即时通讯 IM 支持同一账号在多个设备上登录，每端默认最多支持 4 个设备同时在线。该功能默认不开通，如需使用该功能或增加支持的设备数量，可以联系 [sales@agora.io](mailto:sales@agora.io)。

多设备登录情况下，所有已登录的设备同步以下信息和操作：

- 消息：包括在线消息、离线消息、推送通知（若开启了第三方推送服务，离线设备收到）以及对应的回执和已读状态等；
- 好友和群组相关操作；
- 子区相关操作；
- 会话相关操作。

单端和多端登录场景下的互踢策略和自动登录时安全检查如下：

<html>
<head>
<meta charset="utf-8">
<title>无标题文档</title>
</head>

<body>
<table width="915" height="327" border="1">
  <tbody>
    <tr>
      <td width="109" height="49">单端/多端登录</td>
      <td width="442">互踢策略</td>
      <td width="342">自动登录安全检查</td>
    </tr>
    <tr>
      <td height="52">单端登录</td>
      <td>新登录的设备会将当前在线设备踢下线。</td>
      <td rowspan="2">设备支持自动登录时，若设备下线后自动重连时需要判断是否踢掉当前在线的最早登录设备，请联系环信商务。 </td>
    </tr>
    <tr>
      <td height="156">多端登录</td>
      <td>若一端的登录设备数量达到了上限，最新登录的设备会将该端最早登录的设备踢下线。&lt;br/&gt;即时通讯 IM 仅支持同端互踢，不支持各端之间互踢。</td>
    </tr>
  </tbody>
</table>
</body>
</html>

## 技术原理  

iOS SDK 初始化时会生成登录 ID 用于在多设备登录和消息推送时识别设备，并将该 ID 发送到服务器。服务器会自动将新消息发送到用户登录的设备，自动监听到其他设备上进行的操作，实现多个设备之间的同步。即时通讯 IM iOS SDK 提供以下多设备场景功能：

- 获取当前用户的其他已登录设备的登录 ID 列表；
- 获取指定账号的在线登录设备列表；  
- 强制指定账号从单个设备下线；
- 强制指定账号从所有设备下线；
- 获取其他设备上的操作。

## 前提条件

开始前，需确保完成 SDK 初始化，并连接到服务器。详见[快速开始](./agora_chat_get_started_ios)。

## 实现方法

### 获取当前用户的其他登录设备的登录 ID 列表

你可以调用 `getSelfIdsOnOtherPlatform` 方法获取你的其他登录设备的登录 ID 列表。选择目标登录 ID 作为消息接收方发出消息，则这些设备上的同一登录账号可以收到消息，实现不同设备之间的消息同步。

```swift
AgoraChatClient.shared().contactManager?.getSelfIdsOnOtherPlatform(completion: { ids, err in
            if err == nil,
               let userId = ids?.first {
                // 选择一个登录 ID 作为消息接收方。创建一条文本消息，content 为消息文字内容，conversationId 传入登录 ID 作为消息接收方。
                let textMessage = AgoraChatMessage(conversationId: userId, body: .text(content: "hello"), ext: nil)
                AgoraChatClient.shared().chatManager?.send(textMessage, progress: nil, completion: { msg, e in
                    
                })
            }
        })
```

### 获取指定账号的在线登录设备列表  

你可以调用 `getLoggedInDevicesFromServerWithToken` 方法通过传入用户 ID 和用户 token 从服务器获取指定账号的在线登录设备的列表。

```swift
    AgoraChatClient.shared().getLoggedInDevicesFromServer(withUserId: "userId", token: "token") { deviceConfigs, err in
            if err == nil {
                
            }
        }
```

### 强制指定账号从单个设备下线

你可以调用 `kickDeviceWithUserId` 方法通过传入用户 ID 和用户 token 将指定账号从单个登录设备踢下线。调用该方法前，你需要首先通过 `AgoraChatClient#getLoggedInDevicesFromServer` 和 `AgoraChatDeviceInfo#resource` 方法获取设备 ID。

:::notice
不登录也可以使用该接口。
:::

```swift
AgoraChatClient.shared().getLoggedInDevicesFromServer(withUserId: "userId", token: "token") { deviceConfigs, err in
            if err == nil,
               let resource = deviceConfigs?.first?.resource {
                AgoraChatClient.shared().kickDevice(withUserId: "userId", token: "token", resource: resource) { e in
                    
                }
            }
        }
```

### 强制指定账号从所有设备下线

你可以调用 `kickAllDevicesWithUserId` 方法通过传入用户 ID 和用户 token 将指定账号从所有登录设备踢下线。

:::notice
不登录也可以使用该接口。
:::

```swift
    AgoraChatClient.shared().kickAllDevices(withUserId: "userId", token: "token") { err in
            
        }
```

### 获取其他设备上的操作

例如，账号 A 同时在设备 A 和 B 上登录，账号 A 在设备 A 上进行操作，设备 B 会收到这些操作对应的通知。

你需要先实现 `AgoraChatMultiDevicesDelegate` 类监听其他设备上的操作，然后调用 `addMultiDevicesDelegate` 方法添加多设备监听。

```swift
extension AppDelegate: AgoraChatMultiDevicesDelegate {
    func multiDevicesContactEventDidReceive(_ aEvent: AgoraChatMultiDevicesEvent, username aUsername: String, ext aExt: String?) {
        switch aEvent {
            //当前用户在其他设备上删除好友。
        case .contactRemove:
            break
            //当前用户在其他设备上接受好友请求。
        case .contactAccept:
            break
            //当前用户在其他设备上拒绝好友请求。 
        case .contactDecline:
            break
            //当前用户在其他设备将某个用户添加至黑名单。
        case .contactBan:
            break
            //当前用户在其他设备将某个用户移出黑名单。
        case .contactAllow:
            break
        default:
            break
        }
    }
    
    func multiDevicesGroupEventDidReceive(_ aEvent: AgoraChatMultiDevicesEvent, groupId aGroupId: String, ext aExt: Any?) {
        switch aEvent {
            //当前⽤户在其他设备创建了群组
        case .groupCreate:
            break
        //当前⽤户在其他设备销毁了群组
        case .groupDestroy:
            break
        //当前⽤户在其他设备加⼊了群组
        case .groupJoin:
            break
        //当前⽤户在其他设备离开了群组
        case .groupLeave:
            break
        //当前⽤户在其他设备发起了入群申请
        case .groupApply:
            break
        //当前⽤户在其他设备同意了入群申请
        case .groupApplyAccept:
            break
        //当前⽤户在其他设备拒绝了入群申请
        case .groupApplyDecline:
            break
        //当前⽤户在其他设备邀请了群成员
        case .groupInvite:
            break
        //当前⽤户在其他设备同意了入群邀请
        case .groupInviteAccept:
            break
        //当前⽤户在其他设备拒绝了入群邀请
        case .groupInviteDecline:
            break
        //当前⽤户在其他设备将成员踢出群
        case .groupKick:
            break
        //当前⽤户在其他设备将成员加⼊群组⿊名单
        case .groupBan:
            break
        //当前⽤户在其他设备将成员移除群组⿊名单
        case .groupAllow:
            break
        //当前⽤户在其他设备屏蔽了群组
        case .groupBlock:
            break
        //当前⽤户在其他设备取消群组屏蔽
        case .groupUnBlock:
            break
        //当前⽤户在其他设备转移群所有权
        case .groupAssignOwner:
            break
        //当前⽤户在其他设备添加管理员
        case .groupAddAdmin:
            break
        //当前⽤户在其他设备移除管理员
        case .groupRemoveAdmin:
            break
        //当前⽤户在其他设备禁⾔⽤户
        case .groupAddMute:
            break
        //当前⽤户在其他设备移除禁⾔
        case .groupRemoveMute:
            break
        //当前⽤户在其他设备设置了群成员自定义属性
        case .groupMemberAttributesChanged:
            break
        default:
            break
        }
    }
    
    func multiDevicesChatThreadEventDidReceive(_ aEvent: AgoraChatMultiDevicesEvent, threadId aThreadId: String, ext aExt: Any?) {
        switch aEvent {
        case  .chatThreadCreate:
            //当前用户在其他设备上创建子区。
            break
        case  .chatThreadDestroy:
            //当前用户在其他设备上销毁子区。
            break
        case  .chatThreadJoin:
            //当前用户在其他设备上加入子区。
            break
        case  .chatThreadLeave:
            //当前用户在其他设备上离开子区。
            break
        case  .chatThreadUpdate:
            //当前用户在其他设备上更新子区。
            break
        case  .chatThreadKick:
            //当前用户在其他设备上将某个成员踢出子区。
            break
        default:
            break
        }
    }
    
    func multiDevicesConversationEvent(_ event: AgoraChatMultiDevicesEvent, conversationId: String, conversationType: AgoraChatConversationType) {
        switch event {
        case .conversationPinned:
            //当前用户在其他设备上将某个会话置顶。
            break
        case .conversationUnpinned:
            //当前用户在其他设备上将某个会话取消置顶。
            break
        case .conversationDelete:
            //当前用户在其他设备上删除某个会话。
            break
            //当前用户在其他设备上更新了会话标记，包括添加和移除会话标记。
        case .conversationUpdateMark:
        default:
            break
        }
    }
    
    func multiDevicesMessageBeRemoved(_ conversationId: String, deviceId: String) {
        // 当前用户在其他设备上单向某个会话的删除服务端的历史消息。
    }
}

//设置多设备监听。
AgoraChatClient.shared().addMultiDevices(delegate: self, queue: nil)
```