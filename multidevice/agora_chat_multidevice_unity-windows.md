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
<table width="807" height="327" border="1">
  <tbody>
    <tr>
      <td width="109" height="49">单端/多端登录</td>
      <td width="384">互踢策略</td>
      <td width="292">自动登录安全检查</td>
    </tr>
    <tr>
      <td height="52">单端登录</td>
      <td>新登录的设备会将当前在线设备踢下线。</td>
      <td rowspan="2">设备支持自动登录时，若设备下线后自动重连时需要判断是否踢掉当前在线的最早登录设备，请联系环信商务。 </td>
    </tr>
    <tr>
      <td height="156">多端登录</td>
      <td><p>若一端的登录设备数量达到了上限，最新登录的设备会将该端最早登录的设备踢下线。</p>
      <p>即时通讯 IM 仅支持同端互踢，不支持各端之间互踢。</p></td>
    </tr>
  </tbody>
</table>
</body>
</html>

## 技术原理

即时通讯 IM Unity SDK 初始化时会生成登录 ID 用于在多设备登录和消息推送时识别设备，并将该 ID 发送到服务器。服务器会自动将新消息发送到用户登录的设备，自动监听到其他设备上进行的操作，实现多个设备之间的同步。即时通讯 IM Unity SDK 提供以下多设备场景功能：

- 获取当前用户的其他已登录设备的登录 ID 列表；
- 获取指定账号的在线登录设备列表；
- 强制指定账号从单个设备下线；
- 强制指定账号从所有设备下线；
- 获取其他设备上的操作。

## 前提条件

开始前，确保将 SDK 初始化，连接到服务器。详见[快速开始](./agora_chat_get_started_unity)。

## 实现方法

### 获取当前用户的其他已登录设备的登录 ID 列表

你可以调用 `GetSelfIdsOnOtherPlatform` 方法获取其他登录设备的登录 ID 列表。选择目标登录 ID 作为消息接收方发出消息，则这些设备上的同一登录账号可以收到消息，实现不同设备之间的消息同步。

```csharp
SDKClient.Instance.ContactManager.GetSelfIdsOnOtherPlatform(new ValueCallBack<List<string>>(
  onSuccess: (list) =>
  {
    // 选择一个登录 ID 作为消息发送方。
    string toChatUserId = list[0];
    string content = "content";
    // 创建一条文本消息，content 为消息文字内容，toChatUserId 传入登录 ID 作为消息发送方。
    Message msg = Message.CreateTextSendMessage(toChatUserId, content);
    // 发送消息。
    SDKClient.Instance.ChatManager.SendMessage(ref msg, new CallBack(
	    onSuccess: () => {

	    },
	    onProgress: (progress) => {

	    },
	    onError: (code, desc) => {

	    }
    ));
  },
  onError: (code, desc) =>
  {

  }
));
```

### 获取指定账号的在线登录设备列表

你可以调用 `GetLoggedInDevicesFromServerWithToken` 方法通过传入用户 ID 和用户 token 从服务器获取指定账号的在线登录设备的列表。

```csharp
SDKClient.Instance.GetLoggedInDevicesFromServerWithToken(userId, token,
	callback: new ValueCallBack<List<DeviceInfo>>(

    onSuccess: (list) =>
    {

    },

    onError: (code, desc) =>
    {

    }
  )
);
```

### 强制指定账号从单个设备下线

你可以调用 `KickDeviceWithToken` 方法通过传入用户 ID 和用户 token 将指定账号从单个登录设备踢下线。调用这两种方法前，你需要首先通过 `SDKClient#GetLoggedInDevicesFromServer` 和 `DeviceInfo#Resource` 方法获取设备 ID。

:::tip
不登录也可以使用该接口。
:::

```csharp
// userId：用户 ID，token：用户 token。
SDKClient.Instance.GetLoggedInDevicesFromServerWithToken(userId, token,
      callback: new ValueCallBack<List<DeviceInfo>>(

          onSuccess: (list) =>
          {

          },

          onError: (code, desc) =>
          {

          }
       )
);

SDKClient.Instance.KickDeviceWithToken(userId, token, resource,
	callback: new CallBack(

    onSuccess: () =>
    {

    },

    onError: (code, desc) =>
    {

    }
  )
);
```

### 强制指定账号从所有设备下线

你可以调用 `KickAllDevicesWithToken` 方法通过传入用户 ID 和用户 token 将指定账号从所有登录设备踢下线。

:::tip
不登录也可以使用该接口。
:::

```csharp
SDKClient.Instance.KickAllDevicesWithToken(username, token,
  callback: new CallBack(

    onSuccess: () =>
    {

    },

    onError: (code, desc) =>
    {

    }
   )
);
```

### 获取其他设备上的操作

账号 A 同时在设备 A 和设备 B 上登录，账号 A 在设备 A 上进行一些操作，设备 B 上会收到这些操作对应的通知。

你需要先实现 `IMultiDeviceDelegate` 监听其他设备上的操作，再设置多设备监听器。

```csharp
//继承并实现 IMultiDeviceDelegate。
public class MultiDeviceDelegate : IMultiDeviceDelegate {
	public void onContactMultiDevicesEvent(MultiDevicesOperation operation, string target, string ext) {
            ......
            switch (operation) {
            //当前用户在其他设备上删除好友。
            case CONTACT_REMOVE: 
            //当前用户在其他设备上接受好友请求。  
                break;
            case CONTACT_ACCEPT:
            //当前用户在其他设备上拒绝好友请求。
                break;    
            case CONTACT_DECLINE: 
            //当前用户在其他设备加某人进入黑名单。
                break;    
            case CONTACT_BAN: 
            //当前用户在其他设备将某个用户移出黑名单。
                break;   
            case CONTACT_ALLOW:
                break; 
        }
    }

    public void onGroupMultiDevicesEvent(MultiDevicesOperation operation, string target, List<string> usernames) {
            ......
            switch (operation) {
            //当前⽤户在其他设备创建了群组。
            case GROUP_CREATE:
                break;
            //当前⽤户在其他设备销毁了群组。
            case GROUP_DESTROY:
                break;
            //当前⽤户在其他设备加⼊了群组。
            case GROUP_JOIN:
                break;
            //当前⽤户在其他设备离开了群组。
            case GROUP_LEAVE:
                break;
            //当前⽤户在其他设备发起了群组申请。
            case GROUP_APPLY:
                break;
            //当前⽤户在其他设备同意了群组申请。
            case GROUP_APPLY_ACCEPT:
                break;
            //当前⽤户在其他设备拒绝了群组申请。
            case GROUP_APPLY_DECLINE:
                break;
            //当前⽤户在其他设备邀请了群成员。
            case GROUP_INVITE:
                break;
            //当前⽤户在其他设备同意了群组邀请。
            case GROUP_INVITE_ACCEPT:
                break;
            //当前⽤户在其他设备拒绝了群组邀请。
            case GROUP_INVITE_DECLINE:
                break;
            //当前⽤户在其他设备将某⼈踢出群。
            case GROUP_KICK:
                break;
            //当前⽤户在其他设备将成员加⼊群组⿊名单。
            case GROUP_BAN:
                break;
            //当前⽤户在其他设备将成员移除群组⿊名单。
            case GROUP_ALLOW:
                break;
            //当前⽤户在其他设备屏蔽群组。
            case GROUP_BLOCK:
                break;
            //当前⽤户在其他设备取消群组屏蔽。
            case GROUP_UNBLOCK:
                break;
            //当前⽤户在其他设备转移群主。
            case GROUP_ASSIGN_OWNER:
                break;
            //当前⽤户在其他设备添加管理员。
            case GROUP_ADD_ADMIN:
                break;
            //当前⽤户在其他设备移除管理员。
            case GROUP_REMOVE_ADMIN:
                break;
            //当前⽤户在其他设备禁⾔⽤户。
            case GROUP_ADD_MUTE:
                break;
            //当前⽤户在其他设备移除禁⾔。
            case GROUP_REMOVE_MUTE:
                break;
            default:
                break;
        }
        ......
    }
}

//注册监听器。
MultiDeviceDelegate adelegate = new MultiDeviceDelegate();
SDKClient.Instance.AddMultiDeviceDelegate(adelegate);

//移除监听器。
SDKClient.Instance.DeleteMultiDeviceDelegate(adelegate);
```