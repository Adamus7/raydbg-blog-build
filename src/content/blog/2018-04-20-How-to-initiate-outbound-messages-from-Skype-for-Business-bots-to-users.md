---
title: How to Initiate Outbound Messages from Bot to Skype for Business User
pubDate: 2018-04-20 11:24:07
tags: 
- Microsoft Bot Framework 
- Skype for Business
---
A common scenario in BOT world is you want to notify something or send some message to a backend agent proactively. The [official domo](https://docs.microsoft.com/en-us/azure/bot-service/dotnet/bot-builder-dotnet-proactive-messages) of Microsoft Bot demonstrate how to achieve that. But the conversation is initiated by the user, not start from the Bot.
Skype for Business team published a [document](https://msdn.microsoft.com/en-us/skype/skype-for-business-bot-framework/docs/overview) about SfB channel and Bot Framework. In this doc, they mentioned how to create outbound bots using C# Bot Builder SDK. However, the sample code is incomplete and doesn't tell us how to send a message to the conversation.
<!-- more -->
Prerequisite：
*   A Bot service with Skype for Business channel enabled.
*   The instance of your bot with a Skype for Business Online tenant is registered.
See more:
    https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-skypeforbusiness
    https://msdn.microsoft.com/en-us/skype/skype-for-business-bot-framework/docs/overview

After further testing, here is my sample code which can initial a conversation from bot to a specific SfB user and send message to the user in same conversation:
```csharp
//Initiate a Conversation
string trustServiceUri = "https://api.skypeforbusiness.com/platformservice/botframework";
MicrosoftAppCredentials.TrustServiceUrl(trustServiceUri);
ConnectorClient connector = new ConnectorClient(new Uri(trustServiceUri));
List<ChannelAccount> participants = new List<ChannelAccount>();
participants.Add(new ChannelAccount("sip:yourAgentaccount@contoso.com", "Agent"));
ConversationParameters parameters = new ConversationParameters(true, new ChannelAccount("sip:yourBotaccount@contoso.com", "Bot"), participants, "TestTopic");
ConversationResourceResponse response = connector.Conversations.CreateConversationAsync(parameters).Result;

//Initiate another connector with the ServiceURL from above response.
ConnectorClient connectorSend = new ConnectorClient(new Uri(response.ServiceUrl));
IMessageActivity msg = Activity.CreateMessageActivity();
msg.Recipient = new ChannelAccount("sip: yourAgentaccount@contoso.com ", "Agent");
msg.Text = "text message";
msg.Locale = "en-Us";
msg.From = new ChannelAccount("sip: yourBotaccount@contoso.com", "Bot");
msg.Type = ActivityTypes.Message;
msg.Timestamp = DateTime.UtcNow;
msg.ChannelId = "skypeforbusiness";
msg.ServiceUrl = response.ServiceUrl;
msg.Conversation = new ConversationAccount(isGroup: true, id: response.Id, name: null);
//Send the message from Bot to User
var result = connectorSend.Conversations.SendToConversationAsync((Activity)msg, response.Id).Result;
{% endcodeblock  %}