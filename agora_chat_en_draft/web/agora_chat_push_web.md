# 离线推送概述

即时通讯 IM 支持集成离线推送服务，为开发者提供低延时、高送达、高并发、不侵犯用户个人数据的离线消息推送服务。

客户端断开连接或应用进程被关闭等原因导致用户离线时，即时通讯 IM 会通过 FCM 消息推送服务向该离线用户的设备推送消息通知。当用户再次上线时，服务器会将离线期间的消息发送给用户。若应用在后台运行，则用户仍为在线状态，即时通讯 IM 不会向用户推送消息通知。多设备登录时，可在声网控制台的 **Push Certificate** 页面配置推送在所有设备离线或任一设备离线时发送推送消息，该配置对所有推送通道生效。

<div class="alert note">1. 应用在后台运行或手机锁屏等情况，若客户端未断开与声网服务器的连接，则即时通讯 IM 不会收到离线推送通知。<br/>2. 多端登录时若有设备被踢下线，即使接入了 IM 离线推送，也收不到离线推送消息。</div>

即时通讯 IM Web SDK 本身不支持离线推送，但支持移动平台的离线推送配置。

## 技术原理

即时通讯 IM SDK 可实现以下功能：

- 设置 app 的推送通知； 
- 获取 app 的推送通知；
- 设置会话的推送通知； 
- 获取会话的推送通知； 
- 清除会话的推送通知； 
- 设置推送翻译；
- 设置强制推送；
- 发送静默消息。

## 前提条件

使用离线推送前，确保满足以下条件：

- 已经完成即时通讯 IM SDK。详见[快速开始](https://docs.agora.io/en/agora-chat/get-started/get-started-sdk?platform=web)。
- 了解即时通讯 IM 套餐包中的 API 调用频率限制，详见[使用限制](./agora_chat_limitation)。

## 实现方法

为优化用户在处理大量推送通知时的体验，即时通讯 IM 在 app 和会话层面提供了推送通知方式和免打扰模式的细粒度选项。 

**推送通知方式**

| 参数     | 描述   | App  | 单聊和群聊会话  |
| :------- | :----- | :----- | :----- |
| `ALL` | 接收所有离线消息的推送通知。 | ✓ | ✓ |
| `AT` | 仅接收提及消息的推送通知。<br/>该参数推荐在群聊中使用。若提及一个或多个用户，需在创建消息时对 `ext` 字段传 "em_at_list":["user1", "user2" ...]；若提及所有人，对该字段传 "em_at_list":"all"。 | ✓ | ✓ |
| `NONE`   | 不接收离线消息的推送通知。 | ✓     | ✓ |

会话级别的推送通知方式设置优先于 app 级别的设置，未设置推送通知方式的会话默认采用 app 的设置。

例如，假设 app 的推送方式设置为 `AT`，而指定会话的推送方式设置为 `ALL`。你会收到来自该会话的所有推送通知，而对于其他会话来说，你只会收到提及你的消息的推送通知。

**免打扰模式**

在你完成 SDK 初始化和成功登录 app 后，可以对 app 以及各类型的会话开启离线推送功能以及通过设置免打扰模式关闭推送。

你可以在 app 级别指定免打扰时间段和免打扰时长，即时通讯 IM 在这两个时间段内不发送离线推送通知。若既设置了免打扰时间段，又设置了免打扰时长，免打扰模式的生效时间为这两个时间段的累加。

免打扰时间参数的说明如下表所示：

| 参数     |  描述   |   App  | 单聊和群聊会话  |
| :--------| :----- | :------------------- | :----- |
| `startTime & endTime` | 免打扰时间段，精确到分钟，格式为 HH:MM-HH:MM，例如 08:30-10:00。该时间为 24 小时制，免打扰时间段的开始时间和结束时间中的小时数和分钟数的取值范围分别为 [00,23] 和 [00,59]。免打扰时间段的设置说明如下：<ul><li>开始时间和结束时间的设置立即生效，免打扰模式每天定时触发。例如，开始时间为 `08:00`，结束时间为 `10:00`，免打扰模式在每天的 8:00-10:00 内生效。若你在 11:00 设置开始时间为 `08:00`，结束时间为 `12:00`，则免打扰模式在当天的 11:00-12:00 生效，以后每天均在 8:00-12:00 生效。</li><li>若开始时间和结束时间相同，免打扰模式则全天生效。</li><li>若结束时间早于开始时间，则免打扰模式在每天的开始时间到次日的结束时间内生效。例如，开始时间为 `10:00`，结束时间为 `08:00`，则免打扰模式的在当天的 10:00 到次日的 8:00 生效。</li><li>目前仅支持在每天的一个指定时间段内开启免打扰模式，不支持多个免打扰时间段，新的设置会覆盖之前的设置。</li><li>若不设置该参数，开始时间和结束时间分别传 `00`。</li></ul> | ✓ | ✗ |
| `duration` | 免打扰时长，单位为毫秒。免打扰时长的取值范围为 [0,604800000]，`0` 表示该参数无效，`604800000` 表示免打扰模式持续 7 天。<br/> 与免打扰时间段的设置长久有效不同，该参数为一次有效。    | ✓     | ✓ |

若在免打扰时段或时长生效期间需要对指定用户推送消息，需设置[强制推送](./7_push_extension#forced)。

**推送通知方式与免打扰时间设置之间的关系**

对于 app 和 app 中的所有会话，免打扰模式的设置优先于推送通知方式的设置。例如，假设在 app 级别指定了免打扰时间段，并将指定会话的推送通知方式设置为 `ALL`。免打扰模式与推送通知方式的设置无关，即在指定的免打扰时间段内，你不会收到任何推送通知。

或者，假设为会话指定了免打扰时长，而 app 没有任何免打扰设置，并且其推送通知方式设置为 `ALL`。在指定的免打扰时长内，你不会收到来自该会话的任何推送通知，而所有其他会话的推送保持不变。

### 设置 app 的推送通知

你可以调用 `setSilentModeForAll` 方法设置 app 级别的推送通知，并通过指定 `paramType` 字段设置推送通知方式和免打扰模式，如下代码示例所示：

```javascript
/**
  options // 推送通知配置选项。
	options: {
    paramType: 0, // 推送通知方式。
    remindType: 'ALL' // 可设置为 `ALL`、`AT` 或 `NONE`。
  }
  options: {
    paramType: 1, // 免打扰时长。
    duration: 7200000 // 免打扰时长，单位为毫秒。
  }
  options: {
    paramType: 2, // 免打扰时间段。
    startTime: {
    	hours: 8, // 免打扰时间段的开始时间中的小时数。
    	minutes: 0 // 免打扰时间段的开始时间中的分钟数。
    }，
    endTime: {
    	hours: 12, // 免打扰时间段的结束时间中的小时数。
    	minutes: 0 // 免打扰时间段的结束时间中的分钟数。
    }
  }
*/
const params = {
  options: {
    paramType: 0,
    remindType: "ALL",
  },
};
connection.setSilentModeForAll(params);
```

### 获取 app 的推送通知设置

你可以调用 `getSilentModeForAll` 获取 app 的推送通知设置，示例代码如下：

```javascript
connection.getSilentModeForAll();
```

### 设置单个会话的推送通知

你可以调用 `setSilentModeForConversation` 设置指定会话的推送通知设置，示例代码如下：

```javascript
/**
	const params = {
    conversationId: 'conversationId', // 会话 ID：单聊为对方用户 ID，群聊为群组 ID。
    type: 'singleChat', // 会话类型：singleChat（单聊）、groupChat（群聊）。
    options: {
      paramType: 0, // 推送通知方式。
      remindType: 'ALL' // 可设置为 `ALL`、`AT` 或 `NONE`。
    }
  }
	
	const params = {
    conversationId: 'conversationId',
    type: 'groupChat',
    options: {
      paramType: 1, // 免打扰时长。
      duration: 7200000 // 免打扰时长，单位为毫秒。
    }
  }
  
  const params = {
    conversationId: 'conversationId',
    type: 'chatRoom',
    options: {
      paramType: 2, // 免打扰时间段。
      startTime: {
        hours: 8, // 免打扰时间段的开始时间中的小时数。
        minutes: 0 // 免打扰时间段的开始时间中的分钟数。
      }，
      endTime: {
        hours: 12, // 免打扰时间段的结束时间中的小时数。
        minutes: 0 // 免打扰时间段的结束时间中的分钟数。
      }
    }
  }
*/
const params = {
  conversationId: "conversationId",
  type: "groupChat",
  options: {
    paramType: 0,
    remindType: "ALL",
  },
};
connection.setSilentModeForConversation(params);
```

### 获取单个会话的推送通知设置

调用 `getSilentModeForConversation` 获取单个会话的推送通知设置，示例代码如下：

```javascript
const params = {
  conversationId: "conversationId", // 会话 ID：单聊为对方用户 ID，群聊为群组 ID。
  type: "singleChat", // 会话类型：singleChat 为单聊，groupChat 为群聊。
};
connection.getSilentModeForConversation(params);
```

### 获取多个会话的推送通知设置

1. 每次最多可获取 20 个会话的推送通知设置。
2. 如果会话使用了 app 设置或其推送通知设置已过期，则返回的字典不包含此会话。

你可以调用 `getSilentModeForConversations` 获取多个会话的推送通知设置，示例代码如下：

```javascript
const params = {
  conversationList: [
    {
      id: "conversationId", // 会话 ID：单聊为对方用户 ID，群聊为群组 ID。
      type: "singleChat", // 会话类型：singleChat 为单聊，groupChat 为群聊。
    },
    {
      id: "conversationId",
      type: "groupChat",
    },
  ],
};
connection.getSilentModeForConversations(params);
```

### 清除单个会话的推送通知方式的设置

你可以调用 `clearRemindTypeForConversation` 方法清除指定会话的推送通知方式的设置。清除后，默认情况下，此会话会使用 app 的设置。示例代码如下：

```javascript
const params = {
  conversationId: "conversationId", // 会话 ID：单聊为对方用户 ID，群聊为群组 ID。
  type: "groupChat", // 会话类型：singleChat 为单聊，groupChat 为群聊。
};
connection.clearRemindTypeForConversation(params);
```

### 设置推送翻译

推送通知与翻译功能配合使用。如果用户启用[自动翻译](./agora_chat_translation_web)功能并发送消息，SDK 会同时发送原始消息和翻译后的消息。

作为接收方，你可以设置你在离线时希望接收的推送通知的首选语言。如果翻译后消息的语言匹配你的首选语言，则翻译后的消息显示在推送通知中；否则，将显示原始消息。

你可以调用 `setPushPerformLanguage` 方法设置推送通知的首选语言，示例代码如下：

```javascript
// 设置推送通知的首选语言。
const params = {
  language: "EU",
};
connection.setPushPerformLanguage(params);

// 获取推送通知的首选语言。
connection.getPushPerformLanguage();
```

### 设置推送模板

默认推送模板主要用于服务器提供的默认配置不满足你的需求时，可使你设置全局范围的推送标题和推送内容。例如，服务器提供的默认设置为中文和英文的推送标题和内容，你若需要使用韩语或日语的推送标题和内容，则可以设置对应语言的推送模板。

#### 使用默认推送模板

要使用默认模板，你需要在声网控制台或[调用 RESTful 接口](./agora_chat_restful_push#set-up-push-templates)创建默认推送模板，模板名称为 `default`。设置完毕，消息推送时自动使用默认模板，创建消息时无需传入模板名称。

按照以下步骤在声网控制台创建默认推送模板：

1. 登录[声网控制台](https://console.agora.io/)，在左侧导航栏中单击 **Project Management**。
2. 在 **Project Management** 页面上，在启用了即时通讯 IM 的项目的 **Action** 一栏中单击 **Config**。
3. 在 **Edit Project** 页面的 **Features** 区域，单击 **Chat** 对应的 **Enable/Config**。
4. 在项目配置页面，选择 **Features** > **Push Template**，单击 **Add Push Template**，在弹出的对话框中配置字段，如下图所示。

![image](\agora_doc_source\en\markdown\agora-chat\images\push\push_add_template.png)

5. 将 **Template Name** 设置为 **default**，然后设置 **Title** 和 **Content** 参数，点击 **OK**。

| 参数      | 类型 | 描述 | 是否必须 |
| :--------- | :----- | :----- | :----- |
| **Template Name**   | String  | 推送模板名称，默认模板为 **default**。       | Yes |
| **Title**     | Array | 推送标题。可通过以下方式设置：<ul><li>输入固定的推送标题。</li><li>使用内置变量，输入 **{$fromNickname}, {$msg}**。</li><li>通过 value 数组设置自定义变量，字段格式为: **{0} {1} {2} ... {n}**。</li></ul><br/>若使用默认模板，前两种设置方式在创建消息时无需传入该参数，服务器自动获取，第三种设置方式则需要通过扩展字段传入。  | Yes |
| **Content**   | Array | 推送内容。可通过以下方式设置：<ul><li>输入固定的推送内容。</li><li>使用变量，输入 **{$fromNickname}, {$msg}**。</li><li>通过 value 数组设置自定义变量，字段格式为: **{0} {1} {2} ... {n}**。</li></ul><br/>若使用默认模板，前两种设置方式在创建消息时无需传入参数，服务器自动获取，第三种设置方式则需要通过扩展字段传入。| Yes |

#### 使用自定义推送模板

使用自定义推送模板的步骤如下：

1. 若使用自定义推送模板，你需要在声网控制台或[调用 RESTful 接口](./agora_chat_restful_push#set-up-push-templates)创建自定义推送模板。**Add Push Template** 对话框中参数的描述，详见[使用默认推送模板](#使用默认推送模板)。使用自定义模板时，**Title** 和 **Content** 参数无论通过哪种方式设置，创建消息时均需通过扩展字段传入。

2. 创建消息时需通过使用扩展字段传入模板名称、推送标题和推送内容，通知栏中的推送标题和内容分别使用模板中的格式。

```javascript
// 下面以文本消息为例，其他类型的消息设置方法相同。

const sendTextMsg = () => {
  let option: AgoraChat.CreateTextMsgParameters = {
    chatType: chatType,
    type: "txt",
    to: targetUserId,
    msg: msgContent,
    ext: {
      em_push_template: {
        name: "templateName", // 设置推送模板名称。
        title_args: ["titleValue1"], // 设置模板中推送标题的 value 数组。如果模板中指定的推送标题为占位数据，则在这里可自定义标题；若指定的标题为固定值，则使用该模板时标题为固定值。
        content_args: ["contentValue1"], // 设置模板中的推送内容的 value 数组。如果模板中指定的模板内容为占位数据，则在这里可自定义内容；若指定的内容为固定值，则使用该模板时内容为固定值。
      },
    },
  };
  let msg = AC.message.create(option);
  connection.send(msg);
};
```

推送模板的 JSON 结构如下：

```json
"em_push_template":{
        "name":"test6",
        "title_args":[
            "test1"
        ],
        "content_args":[
            `${fromNickname}`,
            `${msg}`
        ]
}
```

### 强制推送

设置强制推送后，用户发送消息时会忽略接收方的免打扰设置，不论是否处于免打扰时间段都会正常向接收方推送消息。

```javascript
// 下面以文本消息为例，其他类型的消息设置方法相同。
const sendTextMsg = () => {
  let option: AgoraChat.CreateTextMsgParameters = {
    chatType: chatType,
    type: "txt",
    to: targetUserId,
    msg: msgContent,
    ext: {
      // 设置是否为强制推送，该字段为内置内置字段，取值如下：`YES`：强制推送；（默认）`NO`：非强制推送。
      em_force_notification: "YES"
    },
  };
  let msg = AC.message.create(option);
  connection.send(msg);
};
```

### 发送静默消息

发送静默消息指发送方在发送消息时设置不推送消息，即用户离线时，即时通讯 IM 服务不会通过推送服务向该用户的设备推送消息通知。因此，用户不会收到消息推送通知。当用户再次上线时，会收到离线期间的所有消息。

发送静默消息和免打扰模式下均为不推送消息，区别在于发送静默消息为发送方在发送消息时设置，而免打扰模式为接收方设置在指定时间段内不接收推送通知。

```javascript
// 下面以文本消息为例，其他类型的消息设置方法相同。
const sendTextMsg = () => {
  let option: AgoraChat.CreateTextMsgParameters = {
    chatType: chatType,
    type: "txt",
    to: targetUserId,
    msg: msgContent,
    ext: {
      // 设置是否发送静默消息。该字段为内置内置字段，取值如下：`YES`：发送静默消息；（默认）`NO`：推送该消息。 
      em_ignore_notification: "NO"
    },
  };
  let msg = AC.message.create(option);
  connection.send(msg);
};
```
