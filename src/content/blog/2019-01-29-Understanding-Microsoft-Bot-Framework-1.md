---
title: 理解 Microsoft Bot Framework：BotBuilder
pubDate: 2019-01-29 17:53:03
tags:
- Azure
- Microsoft Bot Framework
- WeChat
---
微软为对话机器人（Chatbot）的开发提供了一整套的解决方案，其中主要的部件包括 Azure Bot Service 和 BotBuilder。Azure Bot Service 用来在云端运行开发好的 Bot 应用；[BotBuilder](https://github.com/Microsoft/BotBuilder) 则是一个开源的 Bot 开发工具集，包括了 SDK，CLI，Emulator 和一个 webchat client。BotBuilder 作为 Microsoft Bot Framework 的一个亮点，给予开发者极大的自由度去设计和完成一个 Bot 应用的开发。在这里记录一下如何理解 Microsoft Bot Framework 的架构，解释相关的概念。
<!-- more -->
> BotBuilder SDK V4 作为最新版本，和前一代 V3 相比，几乎是全新的一套 SDK。从概念到细节都听取了开发者的大量意见，并做出了巨大的改变。因此，这里就只关注基于 V4 版本的开发。

# 初识 BotBuilder
对于开发者来说，一套趁手的开发套件需要包括成熟的 SDK，自动化工具，调试工具，测试和部署工具。BotBuilder 为 Chatbot 的开发提供如下工具：
1.  Bot Builder V4 SDK，支持 C#(.NETCore 2.2)，JavaScript, Java(preview)，Python(Preview) 四种开发语言。
2.  Bot Framework Emulator，本地化的 Chatbot 的调试工具。
3.  Bot Builder CLI tools，为不同的 Bot 组件提供命令行工具，方便实现自动化的开发部署工作。
4.  [Bot Framework webchat](https://github.com/microsoft/botframework-webchat)，可以定制化的 chatbot web 前端，可以把开发好的 Bot 直接集成到现有的网页中。
5.  [BotBuilder Samples](https://github.com/microsoft/botbuilder-samples)，实现不同功能的 Sample Code。

# Microsoft Bot SDK 中的一些概念
在上手 Bot SDK 之前，最好先简单了解一些基本概念和流程。
## Activity
在 BotBuilder SDK 的设计中，Activity 是会话系统中最基本的信息单元对象。任何用户和 Bot 之间的交互都需要通过一个 Activity Object 来承载，用户发送一条文字信息是一个 activity，用户加入对话是一个 activity，用户正在输入这个状态也是一个 activity。从 BotBuilder 的源码中可以看到目前定义的 Activity 有如下类型：
```csharp
//botbuilder-dotnet/libraries/Microsoft.Bot.Schema/ActivityTypes.cs
public static class ActivityTypes
{
    public const string Message = "message";
    public const string ContactRelationUpdate = "contactRelationUpdate";
    public const string ConversationUpdate = "conversationUpdate";
    public const string Typing = "typing";
    public const string EndOfConversation = "endOfConversation";
    public const string Event = "event";
    public const string Invoke = "invoke";
    public const string DeleteUserData = "deleteUserData";
    public const string MessageUpdate = "messageUpdate";
    public const string MessageDelete = "messageDelete";
    public const string InstallationUpdate = "installationUpdate";
    public const string MessageReaction = "messageReaction";
    public const string Suggestion = "suggestion";
    public const string Trace = "trace";
    public const string Handoff = "handoff";
}
```
## Turn
会话，必然是一来一回，一个 Turn 就是`从用户给 Bot 发送一个 Activity 开始，到 Bot 给用户返回一条 Activity 结束`的一个过程。把 Turn 这种行为抽象出来的目的是为了 Bot 可以获取 Turn Context。前面我们知道 Activity 只定义了信息的结构，那么这个消息是谁发的，发给谁的，怎么处理这个消息等等同样重要，我们需要一个 Turn Context 来承载。可以看一下 ITurnContext 这个接口的定义
```csharp
//botbuilder-dotnet/libraries/Microsoft.Bot.Builder/ITurnContext.cs
//注释已删除
public interface ITurnContext
{
    BotAdapter Adapter { get; }
    TurnContextStateCollection TurnState { get; }
    Activity Activity { get; }
    bool Responded { get; }
    Task<ResourceResponse> SendActivityAsync(string textReplyToSend, string speak = null,string inputHint = InputHints.AcceptingInput, CancellationToken cancellationToken =default(CancellationToken));
    Task<ResourceResponse> SendActivityAsync(IActivity activity, CancellationTokencancellationToken = default(CancellationToken));
    Task<ResourceResponse[]> SendActivitiesAsync(IActivity[] activities,CancellationTokencancellationToken = default(CancellationToken));
    Task<ResourceResponse> UpdateActivityAsync(IActivity activity,CancellationTokencancellationToken = default(CancellationToken));
    Task DeleteActivityAsync(string activityId, CancellationToken cancellationToken = defaul(CancellationToken));
    Task DeleteActivityAsync(ConversationReference conversationReference,CancellationTokencancellationToken = default(CancellationToken));
    ITurnContext OnSendActivities(SendActivitiesHandler handler);
    ITurnContext OnUpdateActivity(UpdateActivityHandler handler);
    ITurnContext OnDeleteActivity(DeleteActivityHandler handler);
}
```
## Adapter
Activity 和 Turn 是 Bot 会话层面的抽象设计，和实际数据的传输是隔离的。我们当前的 Bot 应用本质上是一个基于 HTTP 协议的 Web 应用。所以在设计上，Adapter 的作用是利用 HTTP 协议发送和接受 Activity，并在接收到 Activity 的时候创建相应的 TurnContext。另外，Adapter 还一个重要的工作是调用定义好的 Middleware（见下）。Adapter 通过 HTTP POST 的 Body 接收一个 JSON 格式的 Activity，实例化一个 Activity 对象，再创建对应的 TurnContext 对象，然后调用 Middlerware 作对应的处理，最后送给 Bot 做相应的逻辑处理。
![500](/images/2019-01-29-Understanding-Microsoft-Bot-Framework-1/bot-builder-activity-processing-stack.png)

## Middleware
Middleware 工作在 Adaptor 和 Bot 逻辑单元之间，跟其他的消息系统的 middleware 的作用类似，如果有些处理工作对每个 Activite 都要做，那么可以把这类工作抽象成一个 Middleware 放在消息处理的 pipeline 上。举例来说，如果你的 Bot 只能处理英文消息，那你肯定希望 Bot 的输入消息都是英文。这时如果收到一个中文消息，则可以调用一个 Translator 的 Middlerware，在 message 到达 Bot 之前先把它翻译成英文。反过来，也可以通过 Middleware 把发送的 message 翻译成用户需要的语言。Bot SDK 的 [文档](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0) 中另外一个关于 dialog state 的 middleware 的例子如下：
![500](/images/2019-01-29-Understanding-Microsoft-Bot-Framework-1/bot-builder-dialog-state-solution.png)

## Bot State
对于一个智能对话系统来说，“记忆力”是非常重要的。上面定义的 Turn 是一个无状态的结构。一场对话（Conversation）往往有多个 Turn 组成，怎么在对话的上下文中“记住”一些事情是非常重要的。比如说开场的时候 Bot 问过了用户的姓名，后面的对话中，Bot 应该记住这个信息并加以利用。在 BotBuilde SDK V4 中，Bot State 被定义成一个 Key-Value 对，并提供了相应的 Bot State 方案，主要包括三个层面的组件：Storage，State Management 和 State Property Accessors。他们的关系可以见下面的示意图：
![500](/images/2019-01-29-Understanding-Microsoft-Bot-Framework-1/bot-builder-state.png)
### Storage
目前 Bot SDK 直接支持三种 Storage 方案：
1.  Memory storage
2.  Azure Blob storage
3.  Azure Cosmos DB (NoSQL) storage

通过简单的配置，可以把 State 保存到以上三种不同的地方，而不用考虑数据结构或者数据库的设计。另外，当前是支持自定义其他 Storage 方案的，不在此详细记录。
### State Management
如何从上述的 Storage 层读取和写入数据，是 State Management 做的事情。开发者不需要关心具体的实现细节，SDK 会自动处理数据的保存。除此支持，SDK 粗略的划分了三种 State 的类型：`User state`, `Conversation state`, 
`Private conversation state`。例如，你可以在 ConversationState 里面记录`Bot 在上一个 Turn 里面问问题`，`当前 turn 的主题`或者`上一个 turn 的主题`等等。
### State Property Accessors
Accessor 为定义的 State 提供了`get`和`set`方法，方便开发者获取和设置数据。值得注意的是`set`方法只是把 State 数据存储到 Cache 中，要把 State 数据持久化到 Storage 中，需要调用对应 State Object 的`SaveChanges()`方法。如前面的例子所说，当 Bot 在一个 Turn 中获取了用户的姓名，就可以用 Accessor 把这个姓名数据放到 UserState 中。在处理后面的 Turn 时，如果需要再用 Accessor 把这个数据取出来。

## 小结
总的来说，Microsoft Bot SDK 给开发者定义了一套 Bot 的开发范式，从 Activity 到 Turn，再到 Bot State。开发者可以专注于 Bot 逻辑的开发，而不需要关心和设计对话系统本身。

# Bot Framework Emulator
我们开发好的 Bot 是一个后端应用，并没有一个现成的前端。只有部署到 Azure Bot Service 上之后，我们才可以通过不同的 Channel 去和 Bot 进行对话。那么在开发阶段，Bot Framework Emulator 就非常有用了。在本地调试 Bot 的时候，Emulator 充当了一个前端的作用，可以直接和本地或者部署在云端的 Bot 应用进行交互，极大的方便了本地调试。
其中对调试开发最有用的功能就是 Inspector，可以帮助 trace activity，查看 Activity 的 payload：
![500](/images/2019-01-29-Understanding-Microsoft-Bot-Framework-1/V4-Emulator-inspector.png)

# Bot Framework webchat
之前我们提到 Bot Service 有很多的 Channels，用户可以通过不同的 Channels 来和 Bot 进行交互。如果开发者希望用户通过 Web 端和 Bot 进行交互，那么直接集成 [Bot Framework Webchat](https://github.com/microsoft/botframework-webchat) 到网页上：
```html
<!DOCTYPE html>
<html>
  <body>
    <div id="webchat" role="main"></div>
    <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
    <script>
      window.WebChat.renderWebChat({
        directLine: window.WebChat.createDirectLine({ secret: 'YOUR_BOT_SECRET_FROM_AZURE_PORTAL' }),
        userID: 'YOUR_USER_ID'
      }, document.getElementById('webchat'));
    </script>
  </body>
</html>
```
Reference:
* [How bots work](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-basics?view=azure-bot-service-4.0&tabs=cs)
* [Managing state](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0)
* [Middleware](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0)