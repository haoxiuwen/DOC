# Offline push

Chat provides an offline push notification service that features low latency, high delivery, high concurrency, and no violation of the users' personal data.

When the user gets offline because of the client disconnection, application process closure, or other reasons, Chat will push a message notification to the user's device through a push service. When the user gets online again, all messages sent during the offline period will be received. Note that the user can receive push notifications on the device only after the chat server is successfully connected at least once from the device.

The Web SDK does not support offline push, but supports the offline push configurations for mobile platforms.

## [Understand the tech](https://docs.agora.io/en/agora-chat/develop/offline-push?platform=web#understand-the-tech)

The Chat SDK allows you to implement the following features:

- Set the push notifications of an app
- Retrieve the push notification settings of an app
- Set the push notifications of a conversation
- Retrieve the push notification settings of one or more conversations
- Clear the push notification mode of a conversation
- Set and retrieve the preferred language of push notifications

## [Prerequisites](https://docs.agora.io/en/agora-chat/develop/offline-push?platform=web#prerequisites)

Before proceeding, ensure that you meet the following requirements:

- You have initialized the Chat SDK. For details, see [SDK quickstart](https://docs.agora.io/en/agora-chat/get-started/get-started-sdk?platform=web).
- You understand the call frequency limit of the Chat APIs supported by different pricing plans as described in [Limitations](https://docs.agora.io/en/agora-chat/reference/limitations).

## [Implementation](https://docs.agora.io/en/agora-chat/develop/offline-push?platform=web#implementation)

To optimize user experience when dealing with an influx of push notifications, Chat provides fine-grained options for the push notification mode and do-not-disturb (DND) mode at both the app and conversation levels, as shown in the following table:

**Push notification mode**

| Parameter       | Description   | Application Scope   |
| :--------- | :----- | :-------------------------------------- | 
| `ALL`  | Receives push notifications for all offline messages.  | App, one-to-one conversations, and group conversations.  | 
| `AT` | Receives push notifications only for mentioned messages.      | App, one-to-one conversations, and group conversations.                  | 
| `NONE` | Receives no push notifications for offline messages.     | App, one-to-one conversations, and group conversations. | 

The setting of the push notification mode at the conversation level takes precedence over that at the app level, and those conversations that do not have specific settings for the push notification mode inherit the app setting by default.
 
For example, assume that the push notification mode of the app is set to `AT`, while that of the specified conversation is set to `ALL`. You receive all the push notifications from this conversation, while you only receive the push notifications for mentioned messages from all the other conversations.

**Do-not-disturb mode**

You can specify both the DND duration and DND time frame at the app level. If you set both time periods, no push notifications will be received in the cumulative period.

Conversations only support the DND duration, but not the DND time frame.

DNS parameters are described as follows:

| Parameter       | Description   | Application Scope   |
| :--------- | :----- | :-------------------------------------- | 
| `startTime & endTime`  | Receive no push notifications in the specified time frame that is accurate to minutes. The time is based on the 24-hour clock, in the format of HH:MM-HH:MM, for example, 08:30-10:00. The value range for the hour and minute in a parameter value is respectively [00,23] and [00,59]. Note the following when you set this parameter: <ul><li> Once the start time and end time are set, they take effect immediately and the DND mode is triggered regularly every day. For example, the start time is `08:00` and the end time is `10:00`, the DND mode operates during 8:00-10:00  every day. If you set the start time to `08:00` at 11:00 and the end time is 12:00, the DND mode works during 11:00-12:00 on the current day, but 08:00-12:00 thereafter. </li><li> If the start time is the same as the end time, the DND mode operates the whole day. </li><li> If the end time is earlier than the start time, the DND mode remains activated from the start time on the current day to the end time the next day. For example, if the start time and end time are set to `10:00` and `08:00` respectively, the DND mode works from `10:00` on the current day until 8:00 the next day. </li><li> Currently, you can specify only one DND time frame and the new setting will overwrite the previous one.</li></ul> | App only | 
| `duration` | Receive no push notifications for the specified duration in millisecond. The value range is [0,604800000], where `0` indicates that this parameter does not take effect and `604800000` indicates that the DND mode lasts for seven days. Unlike the DND time frame specified with `startTime` and `endTime`, this parameter specifies that the DND mode only lasts for a specific interval, not on a regular basis.     | App, one-to-one conversations, and group conversations.     | 

**Relationship between the push notification mode and the DND mode**

For both the app and conversations in the app, the DND mode setting takes precedence over the setting of the push notification mode.

For example, assume that a DND time period is specified at the app level and the push notification mode of the specified conversation is set to `ALL`. The DND mode takes effect regardless of the setting of the push notification mode, that is, no push notifications will be received during the specified DND time period.

Alternatively, assume that a DND duration is specified for a conversation, while the app does not have any DND settings and its push notification mode is set to `ALL`. You do not receive any push notifications from this conversation during the specified DND duration, while the push of all the other conversations remains the same.

### [Set the push notifications of an app](https://docs.agora.io/en/agora-chat/develop/offline-push?platform=web#set-the-push-notifications-of-an-app)

You can call `setSilentModeForAll` to set the push notifications at the app level and set the push notification mode and DND mode by specifying the `paramType` field, as shown in the following code sample:

```javascript
/**
  // options: The push notification options.
	options: {
    paramType: 0, // The push notification mode.
    remindType: 'ALL' // Sets to `ALL`, `AT`, or `NONE`.
  }
  options: {
    paramType: 1, // The DND duration.
    duration: 7200000 // Sets the DND duration to `7200000` in milliseconds.
  }
  options: {
    paramType: 2, // The DND time frame.
    startTime: {
    	hours: 8, // Sets the start hour to `8`.
    	minutes: 0 // Sets the start minute to `0`.
    }，
    endTime: {
    	hours: 12, // Sets the end hour to `12`.
    	minutes: 0 // Sets the end minute to `0`.
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

### [Retrieve the push notification setting of an app](https://docs.agora.io/en/agora-chat/develop/offline-push?platform=web#retrieve-the-push-notification-setting-of-an-app)

You can call `getSilentModeForAll` to retrieve the push notification settings at the app level, as shown in the following code sample:

```javascript
connection.getSilentModeForAll();
```

### [Set the push notifications of a conversation](https://docs.agora.io/en/agora-chat/develop/offline-push?platform=web#set-the-push-notifications-of-a-conversation)

You can call `setSilentModeForConversation` to set the push notifications for the conversation specified by the `conversationId` and `type` fields, as shown in the following code sample:

```javascript
/**
	const params = {
    conversationId: 'conversationId', // The conversation ID. For one-to-one chats, sets to the ID of the peer user. For group chats, sets to the ID of the chat group or chat room.
    type: 'singleChat', // The chat type. Sets the chat type to `singleChat`, `groupChat`, or `chatRoom`.
    options: {
      paramType: 0, // The push notification mode.
      remindType: 'ALL' // Sets to `ALL`, `AT`, or `NONE`.
    }
  }
	
	const params = {
    conversationId: 'conversationId',
    type: 'groupChat',
    options: {
      paramType: 1, // The DND duration.
      duration: 7200000 // Sets the DND duration to `7200000` in milliseconds.
    }
  }
  
  const params = {
    conversationId: 'conversationId',
    type: 'chatRoom',
    options: {
      paramType: 2, // The DND time frame.
      startTime: {
        hours: 8, // Sets the start hour to `8`.
        minutes: 0 // Sets the start minute to `0`.
      }，
      endTime: {
        hours: 12, // Sets the start hour to `12`.
        minutes: 0 // Sets the start hour to `0`.
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

### [Retrieve the push notification setting of a conversation](https://docs.agora.io/en/agora-chat/develop/offline-push?platform=web#retrieve-the-push-notification-setting-of-a-conversation)

You can call `getSilentModeForConversation` to retrieve the push notification settings of the specified conversation, as shown in the following code sample:

```javascript
const params = {
  conversationId: "conversationId", // The conversation ID. For one-to-one chats, sets to the ID of the peer user. For group chats, sets to the ID of the chat group or chat room.
  type: "singleChat", // The chat type. Sets the chat type to `singleChat`, `groupChat`, or `chatRoom`.
};
connection.getSilentModeForConversation(params);
```

### [Retrieve the push notification settings of multiple conversations](https://docs.agora.io/en/agora-chat/develop/offline-push?platform=web#retrieve-the-push-notification-settings-of-multiple-conversations)

1. You can retrieve the push notification settings of up to 20 conversations at each call.
2. If a conversation inherits the app setting or its push notification setting has expired, the returned dictionary does not include this conversation.

You can call `getSilentModeForConversations` to retrieve the push notification settings of multiple conversations, as shown in the following code sample:

```javascript
const params = {
  conversationList: [
    {
      id: "conversationId", // The conversation ID. For one-to-one chats, sets to the ID of the peer user. For group chats, sets to the ID of the chat group or chat room.
      type: "singleChat", // The chat type. Sets the chat type to `singleChat`, `groupChat`, or `chatRoom`.
    },
    {
      id: "conversationId",
      type: "groupChat",
    },
  ],
};
connection.getSilentModeForConversations(params);
```

### [Clear the push notification mode of a conversation](https://docs.agora.io/en/agora-chat/develop/offline-push?platform=web#clear-the-push-notification-mode-of-a-conversation)

You can call `clearRemindTypeForConversation` to clear the push notification mode of the specified conversation. Once the specific setting of a conversation is cleared, this conversation inherits the app setting by default.

The following code sample shows how to clear the push notification mode of a conversation:

```javascript
const params = {
  conversationId: "conversationId", // The conversation ID. For one-to-one chats, sets to the ID of the peer user. For group chats, sets to the ID of the chat group or chat room.
  type: "groupChat", // The chat type. Sets the chat type to `singleChat`, `groupChat`, or `chatRoom`.
};
connection.clearRemindTypeForConversation(params);
```

### [Set up push translations](https://docs.agora.io/en/agora-chat/develop/offline-push?platform=web#set-up-push-translations)

Push notifications work in tandem with the translation feature. If a user enables [automatic translation](https://docs.agora.io/en/agora-chat/client-api/messages/translate-messages?platform=web#automatic-translation) and sends a message, the SDK sends both the original message and the translated message.

As a recipient, you can set the preferred language of push notifications. If the translated message is in the language matching your setting, the translated message is displayed as the push notification; otherwise, the original message is displayed.

The following code sample shows how to set and retrieve your preferred language of push notifications:

```javascript
// Sets the preferred language of push notifications.
const params = {
  language: "EU",
};
connection.setPushPerformLanguage(params);

// Retrieves the preferred language of push notifications.
connection.getPushPerformLanguage();
```

## [What's next](https://docs.agora.io/en/agora-chat/develop/offline-push?platform=web#whats-next)

This section includes more versatile push notification features that you can use to implement additional functions if needed.

If the ready-made templates do not meet your requirements, Chat also enables you to customize your push notifications.