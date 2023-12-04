# 设置推送通知的显示内容

通知收到后，通知栏中显示的推送标题和内容可通过以下方式设置，配置的优先级为由低到高：

1. 设置推送通知的显示属性；
2. 使用默认推送模板；
3. 使用消息扩展字段；
4. 使用自定义推送模板。

## 设置推送通知的显示属性

你可以分别调用 `updatePushNickname` 和 `updatePushDisplayStyle` 方法设置推送通知中显示的昵称（`nickname`）和通知显示样式（`displayStyle`），确定通知栏中的推送标题和推送内容。

```typescript
ChatClient.getInstance()
  .pushManager.updatePushNickname(nickname)
  .then(() => {
    console.log("Succeeded in updating the nickname.");
  })
  .catch((reason) => {
    console.log("Failed to update the nickname.", reason);
  });
```

```typescript
ChatClient.getInstance()
  .pushManager.updatePushDisplayStyle(displayStyle)
  .then(() => {
    console.log("Succeeded in updating the display style.");
  })
  .catch((reason) => {
    console.log("Failed to update the display style.", reason);
  });
```

你可以调用 `fetchPushOptionFromServer` 方法获取推送通知中的显示属性，如以下代码示例所示：

```typescript
ChatClient.getInstance()
  .pushManager.fetchPushOptionFromServer()
  .then(() => {
    console.log("Succeeded in getting the push configurations.");
  })
  .catch((reason) => {
    console.log("Failed to get the push configuration.", reason);
  });
```

若要在通知栏中显示消息内容，需要设置通知显示样式 `displayStyle`。该参数有如下两种设置：

- （默认）`Simple`：不论 `nickname` 是否设置，对于推送任何类型的消息，通知栏采用默认显示设置，即推送标题为“您有一条新消息”，推送内容为“请点击查看”。
- `Summary`：显示消息内容。设置的昵称只在 `displayStyle` 为 `Summary` 时生效，在 `Simple` 时不生效。

下表以单聊文本消息为例介绍这显示属性的设置。

<div class="alert info">对于群聊，下表中的“消息发送方的推送昵称”和“消息发送方的 IM 用户 ID”显示为“群组 ID”。</div>
| 参数设置      | 推送显示 | 图片    |
| :--------- | :----- |:------------- |
| <ul><li>`displayStyle`：（默认）`Simple`</li><li>`nickname`：设置或不设置</li></ul>  | <ul><li>推送标题：“您有一条新消息”</li><li>推送内容：“请点击查看”</li></ul>   | push_displayattribute_1.png      |
| <ul><li>`displayStyle`：`Summary`</li><li>`nickname`：设置具体值</li></ul>| <ul><li>推送标题：“您有一条新消息”</li><li>推送内容：“消息发送方的推送昵称：消息内容”</li></ul>      | push_displayattribute_2.png     |
| <ul><li>`displayStyle`：`Summary`</li><li>`nickname`：不设置</li></ul>    | <ul><li>推送标题：“您有一条新消息”</li><li>推送内容：“消息发送方的 IM 用户 ID: 消息内容”</li></ul>       | push_displayattribute_3.png         |

## 使用默认推送模板

默认推送模板主要用于服务器提供的默认配置不满足你的需求时，可使你设置全局范围的推送标题和推送内容。例如，服务器提供的默认设置为中文和英文的推送标题和内容，你若需要使用韩语或日语的推送标题和内容，则可以设置对应语言的推送模板。

要使用默认模板，你需要在声网控制台或[调用 RESTful 接口](./agora_chat_restful_push#set-up-push-templates)创建默认推送模板，模板名称为 `default`。设置完毕，消息推送时自动使用默认模板，创建消息时无需传入模板名称。

按照以下步骤在声网控制台创建默认推送模板：

1. 登录[声网控制台](https://console.agora.io/)，在左侧导航栏中单击 **Project Management**。
2. 在 **Project Management** 页面上，在启用了即时通讯 IM 的项目的 **Action** 一栏中单击 **Config**。
3. 在 **Edit Project** 页面的 **Features** 区域，单击 **Chat** 对应的 **Enable/Config**。
4. 在项目配置页面，选择 **Features** > **Push Template**，单击 **Add Push Template**，在弹出的对话框中配置字段，如下图所示。

![image](\agora_doc_source\en\markdown\agora-chat\images\push\push_add_template.png)

5. 将 **Template Name** 设置为 **default**，然后设置 **Title** 和 **Content** 参数，点击 **OK**。

| 参数              | 类型   | 描述                                                                                                                                                                                                                                                                                                                            | 是否必须 |
| :---------------- | :----- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :------- |
| **Template Name** | String | 推送模板名称，默认模板为 **default**。                                                                                                                                                                                                                                                                                          | Yes      |
| **Title**         | Array  | 推送标题。可通过以下方式设置：<ul><li>输入固定的推送标题。</li><li>使用内置变量，输入 **{$fromNickname}, {$msg}**。</li><li>通过 value 数组设置自定义变量，字段格式为: **{0} {1} {2} ... {n}**。</li></ul><br/>若使用默认模板，前两种设置方式在创建消息时无需传入该参数，服务器自动获取，第三种设置方式则需要通过扩展字段传入。 | Yes      |
| **Content**       | Array  | 推送内容。可通过以下方式设置：<ul><li>输入固定的推送内容。</li><li>使用变量，输入 **{$fromNickname}, {$msg}**。</li><li>通过 value 数组设置自定义变量，字段格式为: **{0} {1} {2} ... {n}**。</li></ul><br/>若使用默认模板，前两种设置方式在创建消息时无需传入参数，服务器自动获取，第三种设置方式则需要通过扩展字段传入。       | Yes      |

## 使用消息扩展字段

创建推送消息时，你可以设置消息扩展字段自定义要显示的推送标题 `em_push_title` 和推送内容 `em_push_content`。

```typescript
msg.attributes = {
  // 消息扩展字段。该字段为内置内置字段，字段名不可修改。
  em_apns_ext: {
    em_push_title: "custom push title", // 自定义推送消息标题。该字段为内置内置字段，字段名不可修改。
    em_push_content: "custom push content", // 自定义推送消息内容。该字段为内置内置字段，字段名不可修改。
  },
};
```

自定义显示字段的数据结构如下：

```java
{
    "em_apns_ext": {
        "em_push_title": "custom push title",
        "em_push_content": "custom push content"
    }
}
```

## 使用自定义推送模板

使用自定义推送模板的步骤如下：

1. 若使用自定义推送模板，你需要在声网控制台或[调用 RESTful 接口](./agora_chat_restful_push#set-up-push-templates)创建自定义推送模板。**Add Push Template** 对话框中参数的描述，详见[使用默认推送模板](#使用默认推送模板)。使用自定义模板时，**Title** 和 **Content** 参数无论通过哪种方式设置，创建消息时均需通过扩展字段传入。

2. 创建消息时需通过使用扩展字段传入模板名称、推送标题和推送内容，通知栏中的推送标题和内容分别使用模板中的格式。

```typescript
// 下面以文本消息为例，其他类型的消息设置方法相同。
const msg = ChatMessage.createTextMessage(targetId, content, targetType);
msg.attributes = {
  em_push_template: {
    name: "foo",
    // 设置模板中推送标题的 value 数组。如果模板中指定的推送标题为占位数据，则在这里可自定义标题；若指定的标题为固定值，则使用该模板时标题为固定值。
    title_args: "title",
    // 设置模板中的推送内容的 value 数组。如果模板中指定的模板内容为占位数据，则在这里可自定义内容；若指定的内容为固定值，则使用该模板时内容为固定值。
    content_args: "test",
  },
};
ChatClient.getInstance()
  .chatManager.sendMessage(msg, {
    onError: (error) => {},
    onSuccess: (msg) => {},
  } as any)
  .then(() => {
    console.log("send message success");
  })
  .catch((reason) => {
    console.log("set message fail.", reason);
  });
```

推送模板的 JSON 结构如下：

```json
"em_push_template":{
        "name":"test6",
        "title_args":[
            "test1"
        ],
        "content_args":[
            "{$fromNickname}",
            "{$msg}"
        ]
}
```

消息接收方可以调用 `selectPushTemplate` 方法传入推送模板名称，选择要使用的模板。

<div class="alert info">若发送方在发送消息时使用了推送模板，则推送通知栏中的显示内容以发送方的推送模板为准。</div>

```typescript
ChatClient.getInstance()
  .pushManager.selectPushTemplate(templateName)
  .then(() => {
    console.log("Succeeded in updating the nickname.");
  })
  .catch((reason) => {
    console.log("Failed to update the nickname.", reason);
  });
```
