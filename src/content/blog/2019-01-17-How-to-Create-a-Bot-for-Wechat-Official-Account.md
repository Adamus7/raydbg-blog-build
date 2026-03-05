---
layout: '[post]'
title: 给微信公众号加个 Bot
pubDate: 2019-01-17 10:42:23
tags:
- Azure
- Microsoft Bot Framework
- WeChat
---
Bot，会话助手类工具，一直有着相当广泛的应用。最早的 Bot 可能类似电信的电话查询系统：“查询账单请按 1，修改密码请按 2……”。这种机械式的应用渐渐被更为智能的对话助理所代替，典型代表有 Siri 和 Cortana 这种个人助理类应用。对于个人和企业，如何开发一个具有一定业务能力的 Bot，一直是一个比较热门的话题。在这篇博客中，我将记录如何基于 Microsoft Bot Framework 为微信公众号添加一个智能对话机器人。
<!-- more -->
# 背景
在商业环境中，一个 Bot 不仅要有趣，还要有业务能力。一个优秀的 Bot 要有自然语言识别能力、上下文处理能力和在对话中实现业务的能力。例如，如果要开发一个 Bot 来处理一家咖啡店的订座服务，这个 Bot 需要像服务员一样能够理解客户的语言，了解客户的意图，获取语句中的关键信息（时间，数量，地点等），最后用对话方式在前台完成业务。
Microsoft Bot Framework 提供了一套 Bot 开发的框架和工具，包括 Azure Bot Service，Bot Builder SDK 和相对丰富的 Bot Channels。这套工具可以帮助开发者在短时间内开发出一个个人或企业级 Bot。这套 Framework 可以轻松集成自然语言处理服务，认知服务并打通不同的沟通渠道（channels）。Bot Channels 目前支持了不少流行的社交和通信应用，因此开发者只要开发一套 Bot，就可以部署到不同的社交通信应用中，包括 Facebook Pages、Microsoft Teams、Skype、Slack 等等。
![500](/images/2019-01-17-How-to-Create-a-Bot-for-Wechat-Official-Account/what-is-bot.png)
微信朋友圈和公众号作为一个非常特殊的社交网络，内容非常丰富，但生态也相对封闭。给公众号加个 Bot，让用户更自然地与公众号交流，说不定还挺好玩。目前 Bot Channels 并没有官方的支持微信和公众号 Channel。公众号本身提供了基于关键字的自动回复功能，但是这个选项没法实现更多的业务功能。另一方面，公众号同时也提供了开发接口，可以把用户输入的消息转发到设定的服务器上，这给集成 Bot 服务提供了可能。这里主要记录一下如何通过公众号开发接口来接入 Azure Bot Service。

# 公众号开发设置
首先需要申请注册公众号。然后接入微信公众平台开发，开发者需要按照如下步骤完成：`填写服务器配置`, `验证服务器地址的有效性`, `依据接口文档实现业务逻辑`。
## 填写服务器配置
![500](/images/2019-01-17-How-to-Create-a-Bot-for-Wechat-Official-Account/dev-config-1.png)
这里需要填写服务接口 URL，注意这里微信开发平台会且只会给这个 URL 发送 HTTP GET/POST 请求，开发者无法定义更多 API 接口。Token 可以随便填一个，在接下来的消息来源验证中会用到。EncodingAESKey 可以随机生成一个，用以后面的消息加密。加密方式可以按需设置。
## 服务器验证
点击提交时，微信平台会立刻发送一个 HTTP GET 请求到设定的服务 URL 上，服务器必须的原样返回请求中所带的 echostr 才能通过验证。因此，在提交前，要保证你的的服务 URL 可以正常工作。这里我在 Azure App Services 上部署了一个基于 ASP.NET Core 的 Web API 应用，针对微信的验证请求原样返回 echostr。
```csharp
[HttpGet("")]
public string ReturnEchostr([FromQuery]string signature, [FromQuery]string nonce, [FromQuery]string timestamp, [FromQuery]string echostr)
{
        if (WXHelper.IsMessageFromWX(signature, nonce, timestamp, WxToken))
    {
            return echostr;
    }
    else
    {
            return "Failed to authenticate the request";
    }
}
```
```csharp
//根据微信开发接入指南的规则来确认请求的来源
public static bool IsMessageFromWX(string signature, string nonce, string timestamp, string wxToken)
{
    string[] tempArr = { nonce, timestamp, wxToken };
    Array.Sort(tempArr);
    string tempStr = string.Join("", tempArr);
    var sha1 = SHA1.Create();
    byte[] hashBytes = sha1.ComputeHash(Encoding.ASCII.GetBytes(tempStr));
    string calculatedSignature = BitConverter.ToString(hashBytes).Replace("-", "").ToLower();
    return calculatedSignature == signature;
}

```
验证 URL 有效性成功后即接入生效，成为开发者。

# 服务开发
## 消息的转发
上面我们提到，Azure Bot Service 没有官方的微信 Channel，因此我们需要通过程序把用户在微信公众号中输入的消息转发给 Bot。这里需要用到 Bot 的一个特殊 Channel, **Directline Channel**。 Directline Channel 支持开发者通过 HTTP GET 或者 WebSocket stream 来和 Bot 进行交互。所以，我们需要把服务器上收到的用户消息，通过 Directline 发送给 Bot；再通过 Directline 获取 Bot 的回复，并把结果返回给用户。
这里需要注意的是：微信公众号不支持主动消息（特定动作交互除外，具体见 [这里](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140547)），必须是客户先说话，服务器紧接着只能回复一条消息，而通过 Microsfot Bot Framework 开发的 Bot 则没有这个要求。因此，在设计 Bot 的对话逻辑的时候，要注意兼容微信公众号的这种形式。服务器回复的消息内容为一个 XML 格式的字符串，文本消息的 XML 格式如下：
```
<xml> <ToUserName>< ![CDATA[toUser] ]></ToUserName> <FromUserName>< ![CDATA[fromUser] ]></FromUserName> <CreateTime>12345678</CreateTime> <MsgType>< ![CDATA[text] ]></MsgType> <Content>< ![CDATA[你好] ]></Content> </xml>"
```
Bot 端的功能开发我希望在另外一篇博客中详细的解释，这里用一个样例 Bot 作为接入演示。
## DirectLine 的配置
在 Azure Bot Service 上， 需要开启 DirectLine Channel 并获取对应的接入 Secret Key。
![500](/images/2019-01-17-How-to-Create-a-Bot-for-Wechat-Official-Account/bot-directline.png)

## 通过 DirectLine 接入 Bot
消息转发的功能同样在上面的。NET Core Web API 应用中实现。我们可以用`Microsoft.Bot.Connector.DirectLine`来帮我们简化开发工作，避免手动的构造 HTTP 请求。在 Directline 中，一个完整的简单对话流程一般如下：
1. Start Conversation（获取一个`conversation id`）
2. Send an activity to the bot（向 DirectLine 的 endpoint 发送一个类型为`message`的`Activity`）
3. Receive activities from the bot（通过`conversation id`从 DirectLine 的 endpoint 获取 Bot 的回复）
4. End a conversation（结束对话）

步骤 2 和 3 可以重复进行，不需要严格的一一对应。步骤 3 中，每一次的返回值除了有一个`activity Set`，还有一个`watermark`标识。再次向 Bot 请求回复时，可以带上之前获得的`watermark`，那么已经回复过的`activity`将不会出现在新的`activity set`中。作用类似 TCP 协议中的 ACK，通过 watermark 标识可以保证 Bot 回复的消息不会丢失。
一个需要注意是，针对不同微信用户发过来的消息，需要建立不同的对话。可以维护一个`微信用户<-->服务器`和`服务器<-->Bot`两种对话之间的映射关系。样例代码如下：
```csharp
// WX Platform Will Post the Message to the Endpoint 
[HttpPost("")]
public async Task<string> WxPost([FromQuery]string signature, [FromQuery]string nonce, [FromQuery]string timestamp)
{
    // WX Message Validataion
    if (WXHelper.IsMessageFromWX(signature, nonce, timestamp, WxToken))
    {
        using (var reader = new StreamReader(Request.Body))
        {
            var body = reader.ReadToEnd();
            if (String.IsNullOrEmpty(body))
            {
                return "Failed to get message";
            }
            // Parse WX Message
            WXMsg msg = WXHelper.ParseWXMsgFromBodyString(body);
            string wxUserId = msg.FromUserName.Trim();
            string responseXML = "";

            // In This Demo, We Only Care About Text Message
            if (msg.MsgType == WXMsgType.Text)
            {
                if (!conMap.activeConversations.ContainsKey(wxUserId))
                {
                    // Create a Directline Clint
                    var createdCon = await botClient.Conversations.StartConversationAsync();
                    conMap.activeConversations.Add(wxUserId, new ConversationInfo(createdCon, ""));
                }

                // Create a Bot Message Activity
                Activity userMessage = new Activity
                {
                    From = new ChannelAccount(wxUserId),
                    Text = msg.Content,
                    Type = ActivityTypes.Message
                };
                
                // Post the message to Bot
                var thisConverstaionID = conMap.activeConversations[wxUserId].Conversation.ConversationId;
                await botClient.Conversations.PostActivityAsync(thisConverstaionID, userMessage);

                // Get the Activity Set from Bot
                var activitySet = await botClient.Conversations.GetActivitiesAsync(thisConverstaionID, conMap.activeConversations[wxUserId].Waltermark);
                conMap.activeConversations[wxUserId].Waltermark = activitySet.Watermark;
                var activities = from x in activitySet.Activities
                                 where x.From.Id == botId
                                 select x;

                // Porcess the Activities
                var returnString = "";
                foreach (Activity activity in activities)
                {
                    returnString += activity.Text + "||";
                }
                responseXML = WXHelper.ConstructWXTextMessage(msg, returnString);
            }
            return responseXML;
        }
    }
    else
    {
        return "";
    }
}
```
在公众号上测试效果：
![300](/images/2019-01-17-How-to-Create-a-Bot-for-Wechat-Official-Account/official-account-test.png)
至此，用户已经可以和公众号的 Bot 进行对话，Bot 也能正确识别和记录对话的状态。在后面的博客中，我将记录 Bot 端更多功能的实现。

Reference:
* [Azure Bot Service Documentation](https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0)
* [如何将 Microsoft Bot Framework 链接至微信公共号](https://www.cnblogs.com/sonic1abc/p/5941442.html)
* [Direct Line Bot Sample](https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLine)
