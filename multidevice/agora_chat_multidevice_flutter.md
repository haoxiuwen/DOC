# 在多个设备上登录

即时通讯 IM 支持同一账号在多个设备上登录，每端默认最多支持 4 个设备同时在线。该功能默认不开通，如需使用该功能或增加支持的设备数量，可以联系 [support@agora.io](mailto:support@agora.io)。

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

即时通讯 IM Flutter SDK 初始化时会生成登录 ID 用于在多设备登录和消息推送时识别设备，并将该 ID 发送到服务器。服务器会自动将新消息发送到用户登录的设备，自动监听到其他设备上进行的操作，实现多个设备之间的同步。即时通讯 IM Flutter SDK 提供以下多设备场景功能：

- 获取当前用户的其他已登录设备的登录 ID 列表；
- 获取指定账号的在线登录设备列表；  
- 强制指定账号从单个设备下线；
- 强制指定账号从所有设备下线；
- 获取其他设备的好友或者群组操作。

## 前提条件

开始前，需确保完成 SDK 初始化，并连接到服务器。详见[快速开始](./agora_chat_get_started_flutter)。

## 实现方法

### 获取当前用户的其他登录设备的登录 ID 列表

你可以调用 `getSelfIdsOnOtherPlatform` 方法获取其他登录设备的登录 ID 列表，然后选择目标登录 ID 作为消息接收方向指定设备发送消息。

```dart
List<String> ids =
    await ChatClient.getInstance.contactManager.getSelfIdsOnOtherPlatform();

// 选择一个登录 ID 作为消息发送方。
String toChatUserId = ids.first;

// 创建一条文本消息，content 为消息文字内容，toChatUserId 传入登录 ID 作为消息发送方。
final msg = ChatMessage.createTxtSendMessage(
    targetId: toChatUserId, content: 'content');
// 发送消息。
ChatClient.getInstance.chatManager.sendMessage(msg);
```

### 获取指定账号的在线登录设备列表  

你可以调用 `fetchLoggedInDevices` 方法通过传入用户 ID 和用户 token 从服务器获取指定账号的在线登录设备的列表。

```dart
List<ChatDeviceInfo> deviceInfos =
    await ChatClient.getInstance.fetchLoggedInDevices(
  userId: 'userId',
  pwdOrToken: 'token',
  isPwd: false,
);
```

### 强制指定账号从单个设备下线

你可以调用 `kickDevice` 方法通过传入用户 ID 和用户 token 将指定账号从单个登录设备踢下线。调用这两种方法前，你需要首先通过 `ChatClient#fetchLoggedInDevices` 和 `DeviceInfo#getResource` 方法获取设备 ID。

:::notice
不登录也可以使用该接口。
:::

```dart
// userId：用户 ID，pwdOrToken：用户 token。
List<ChatDeviceInfo> deviceInfos =
    await ChatClient.getInstance.fetchLoggedInDevices(
  userId: 'userId',
  pwdOrToken: 'token',
  isPwd: false,
);
// userId：用户 ID，pwdOrToken：用户 token。
await ChatClient.getInstance.kickDevice(
  userId: 'userId',
  pwdOrToken: 'token',
  resource: deviceInfos[selectedIndex].resource!,
  isPwd: false,
);
```

### 强制指定账号从所有设备下线

你可以调用 `kickAllDevices` 方法通过传入用户 ID 和用户 token 将指定账号从所有登录设备踢下线。

:::notice
不登录也可以使用该接口。
:::

```dart
// userId：用户 ID，pwdOrToken：用户 token。
await ChatClient.getInstance.kickAllDevices(
  userId: 'userId',
  pwdOrToken: 'pwdOrToken',
  isPwd: false,
);
```

### 获取其他设备上的操作

例如，账号 A 同时在设备 A 和 B 上登录，账号 A 在设备 A 上进行操作，设备 B 会收到这些操作对应的通知。

你需要先实现 `ChatMultiDeviceEventHandler` 类监听其他设备上的操作，然后调用 `addMultiDeviceEventHandler` 方法添加多设备监听。

```dart
//实现 `ChatMultiDeviceEventHandler` 监听其他设备上的操作。
private class ChatMultiDeviceListener implements MultiDeviceListener {
final eventHandler = ChatMultiDeviceEventHandler(
  onContactEvent: (event, userId, ext) {
    switch (event) {
      //当前用户在其他设备上删除好友。
      case ChatMultiDevicesEvent.CONTACT_REMOVE:
        break;
      //当前用户在其他设备上接受好友请求。
      case ChatMultiDevicesEvent.CONTACT_ACCEPT:
        break;
      //当前用户在其他设备上拒绝好友请求。
      case ChatMultiDevicesEvent.CONTACT_DECLINE:
        break;
      //当前用户在其他设备将某个用户添加至黑名单。
      case ChatMultiDevicesEvent.CONTACT_BAN:
        break;
      // 当前用户在其他设备将某个用户移出黑名单。
      case ChatMultiDevicesEvent.CONTACT_ALLOW:
        break;
      default:
    }
  },
  onGroupEvent: (event, groupId, userIds) {
    switch (event) {
      //当前⽤户在其他设备创建了群组。
      case ChatMultiDevicesEvent.GROUP_CREATE:
        break;
      //当前⽤户在其他设备销毁了群组。
      case ChatMultiDevicesEvent.GROUP_DESTROY:
        break;
      //当前⽤户在其他设备加⼊了群组。
      case ChatMultiDevicesEvent.GROUP_JOIN:
        break;
      //当前⽤户在其他设备离开了群组。
      case ChatMultiDevicesEvent.GROUP_LEAVE:
        break;
      //当前⽤户在其他设备发起了入群申请。
      case ChatMultiDevicesEvent.GROUP_APPLY:
        break;
      //当前⽤户在其他设备同意了入群申请。
      case ChatMultiDevicesEvent.GROUP_APPLY_ACCEPT:
        break;
      //当前⽤户在其他设备拒绝了入群申请。
      case ChatMultiDevicesEvent.GROUP_APPLY_DECLINE:
        break;
      //当前⽤户在其他设备邀请了群成员。
      case ChatMultiDevicesEvent.GROUP_INVITE:
        break;
      //当前⽤户在其他设备同意了入群邀请。
      case ChatMultiDevicesEvent.GROUP_INVITE_ACCEPT:
        break;
      //当前⽤户在其他设备拒绝了入群邀请。
      case ChatMultiDevicesEvent.GROUP_INVITE_DECLINE:
        break;
      //当前⽤户在其他设备将成员踢出群。
      case ChatMultiDevicesEvent.GROUP_KICK:
        break;
      //当前⽤户在其他设备将成员加⼊群组⿊名单。
      case ChatMultiDevicesEvent.GROUP_BAN:
        break;
      //当前⽤户在其他设备将成员移除群组⿊名单。
      case ChatMultiDevicesEvent.GROUP_ALLOW:
        break;
      //当前⽤户在其他设备屏蔽了群组。
      case ChatMultiDevicesEvent.GROUP_BLOCK:
        break;
      //当前⽤户在其他设备取消群组屏蔽。
      case ChatMultiDevicesEvent.GROUP_UNBLOCK:
        break;
      //当前⽤户在其他设备转移群所有权。
      case ChatMultiDevicesEvent.GROUP_ASSIGN_OWNER:
        break;
      //当前⽤户在其他设备添加管理员。
      case ChatMultiDevicesEvent.GROUP_ADD_ADMIN:
        break;
      //当前⽤户在其他设备移除管理员。
      case ChatMultiDevicesEvent.GROUP_REMOVE_ADMIN:
        break;
      //当前⽤户在其他设备禁⾔⽤户。
      case ChatMultiDevicesEvent.GROUP_ADD_MUTE:
        break;
      //当前⽤户在其他设备移除禁⾔。
      case ChatMultiDevicesEvent.GROUP_REMOVE_MUTE:
        break;
      //当前⽤户在其他设备设置了群成员自定义属性。
      case ChatMultiDevicesEvent.GROUP_MEMBER_ATTRIBUTES_CHANGED:
        break;
      default:
    }
  },
  onChatThreadEvent: (event, chatThreadId, userIds) {
    switch (event) {
      case ChatMultiDevicesEvent.CHAT_THREAD_CREATE:
        //当前用户在其他设备上创建子区。
        break;
      case ChatMultiDevicesEvent.CHAT_THREAD_DESTROY:
        //当前用户在其他设备上销毁子区。
        break;
      case ChatMultiDevicesEvent.CHAT_THREAD_JOIN:
        //当前用户在其他设备上加入子区。
        break;
      case ChatMultiDevicesEvent.CHAT_THREAD_LEAVE:
        //当前用户在其他设备上离开子区。
        break;
      case ChatMultiDevicesEvent.CHAT_THREAD_UPDATE:
        //当前用户在其他设备上更新子区。
        break;
      case ChatMultiDevicesEvent.CHAT_THREAD_KICK:
        //当前用户在其他设备上将某个成员踢出子区。
        break;
      default:
    }
  },
  onConversationEvent: (event, conversationId, type) {
    switch (event) {
      case ChatMultiDevicesEvent.CONVERSATION_PINNED:
        //当前用户在其他设备上将某个会话置顶
        break;
      case ChatMultiDevicesEvent.CONVERSATION_UNPINNED:
        //当前用户在其他设备上将某个会话取消置顶
        break;
      case ChatMultiDevicesEvent.CONVERSATION_DELETE:
        //当前用户在其他设备上删除某个会话
        break;
      default:
    }
  },
  onRemoteMessagesRemoved: (conversationId, deviceId) {
    // 当前用户在其他设备上单向某个会话的删除服务端的历史消息。
  },
);

//设置多设备监听。
ChatClient.getInstance.addMultiDeviceEventHandler(
  "UNIQUE_HANDLER_ID",
  eventHandler,
);
//移除多设备监听。
ChatClient.getInstance.removeMultiDeviceEventHandler('UNIQUE_HANDLER_ID');
}
```
