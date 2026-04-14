---
title: LangChain4j入门
tags:
  - ChatLanguageModel
  - 多模态
  - 流式输出
categories:
  - Java
  - LangChain4J
date: 2026-01-05 09:08:25
---



## 1.什么是LangChain4J

LangChain4j 的目标是简化将 LLM 集成到 Java 应用程序中的过程。

具体方式如下：

1. **统一 API：** LLM 提供商（如 OpenAI 或 Google Vertex AI）和嵌入（向量）存储（如 Pinecone 或 Milvus） 使用专有 API。LangChain4j 提供统一的 API，避免了学习和实现每个特定 API 的需求。 要尝试不同的 LLM 或嵌入存储，您可以在它们之间轻松切换，无需重写代码。 LangChain4j 目前支持 [15+ 个流行的 LLM 提供商](https://docs.langchain4j.info/integrations/language-models/) 和 [20+ 个嵌入存储](https://docs.langchain4j.info/integrations/embedding-stores/)。
2. **全面的工具箱：** 自 2023 年初以来，社区一直在构建众多 LLM 驱动的应用程序， 识别常见的抽象、模式和技术。LangChain4j 将这些提炼成一个即用型包。 我们的工具箱包含从低级提示模板、聊天记忆管理和函数调用 到高级模式如代理和 RAG 的工具。 对于每个抽象，我们提供一个接口以及基于常见技术的多个即用型实现。 无论您是在构建聊天机器人还是开发包含从数据摄取到检索完整管道的 RAG， LangChain4j 都提供多种选择。
3. **丰富的示例：** 这些[示例](https://github.com/langchain4j/langchain4j-examples)展示了如何开始创建各种 LLM 驱动的应用程序， 提供灵感并使您能够快速开始构建。

LangChain4j 始于 2023 年初 ChatGPT 热潮期间。 我们注意到与众多 Python 和 JavaScript LLM 库和框架相比，缺少 Java 对应物， 我们必须解决这个问题！ 虽然我们的名字中有"LangChain"，但该项目是 LangChain、Haystack、 LlamaIndex 和更广泛社区的想法和概念的融合，并加入了我们自己的创新。

<!--more-->



LangChain4j 在两个抽象层次上运行：

- 低层次。在这个层次上，您拥有最大的自由度和访问所有低级组件的权限，如 [ChatLanguageModel](https://docs.langchain4j.info/tutorials/chat-and-language-models)、`UserMessage`、`AiMessage`、`EmbeddingStore`、`Embedding` 等。 这些是您的 LLM 驱动应用程序的"原语"。 您可以完全控制如何组合它们，但需要编写更多的粘合代码。
- 高层次。在这个层次上，您使用高级 API（如 [AI 服务](https://docs.langchain4j.info/tutorials/ai-services)）与 LLM 交互， 它隐藏了所有复杂性和样板代码。 您仍然可以灵活地调整和微调行为，但是以声明式方式完成。



## 2.为什么选择LangChain4J

![img](01.png)



## 3.快速入门

LangChain4j 提供[与多种 LLM 提供商的集成](https://docs.langchain4j.info/integrations/language-models/)。 每种集成都有自己的 Maven 依赖项。 最简单的开始方式是使用 OpenAI 集成：

- 对于 Maven 在 `pom.xml` 中：

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai</artifactId>
    <version>1.0.0-beta3</version>
</dependency>
```



如果您希望使用高级 [AI 服务](https://docs.langchain4j.info/tutorials/ai-services) API，您还需要添加以下依赖项：

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j</artifactId>
    <version>1.0.0-beta3</version>
</dependency>
```



- 配置你的 **chatLanguageModel**

```properties
openai.baseurl=http://langchain4j.dev/demo/openai/v1
openai.apikey=
openai.modelname=gpt-4o-mini
```



```java
    @Bean
    public ChatLanguageModel chatLanguageModel() {
        OpenAiChatModel model = OpenAiChatModel.builder()
                .baseUrl(baseUrl)
                .apiKey(apiKey)
                .modelName(modelName)
                .build();
        return model;
    }
```



- 现在，开始聊天

```java
    @Autowired
    public ChatLanguageModel chatLanguageModel;

    @GetMapping("/chat/{question}")
    public String chat(@PathVariable(value = "question") String question) {
        return chatLanguageModel.chat(question);
    }
```



## 4.聊天和语言模型：低阶API

LLM 目前有两种 API 类型：

- `LanguageModel`。它们的 API 非常简单 - 接受 `String` 作为输入并返回 `String` 作为输出。 这种 API 现在正在被聊天 API（第二种 API 类型）所取代。
- `ChatLanguageModel`。这些接受多个 `ChatMessage` 作为输入并返回单个 `AiMessage` 作为输出。 `ChatMessage` 通常包含文本，但某些 LLM 也支持其他模态（例如，图像、音频等）。 

LangChain4j 不会再扩展对 `LanguageModel` 的支持， 因此在所有新功能中，我们将使用 `ChatLanguageModel` API。

`ChatLanguageModel` 是 LangChain4j 中与 LLM 交互的低级 API，提供最大的能力和灵活性。

除了 `ChatLanguageModel` 和 `LanguageModel` 外，LangChain4j 还支持以下类型的模型：

- `EmbeddingModel` - 这种模型可以将文本转换为 `Embedding`。

- `ImageModel` - 这种模型可以生成和编辑 `Image`。

- `ModerationModel` - 这种模型可以检查文本是否包含有害内容。

- `ScoringModel` - 这种模型可以对查询的多个文本片段进行评分（或排名）， 本质上确定每个文本片段与查询的相关性。这对 [RAG](https://docs.langchain4j.info/tutorials/rag) 很有用。 这些将在后面介绍。

  

现在，让我们仔细看看 `ChatLanguageModel` API。

```java
public interface ChatLanguageModel {

    String chat(String userMessage);
    
    ...
}
```

简单的 `chat` 方法，它接受 `String` 作为输入并返回 `String` 作为输出。



以下是其他聊天 API 方法：

```java
    ...
    
    ChatResponse chat(ChatMessage... messages);

    ChatResponse chat(List<ChatMessage> messages);
        
    ...
```

这些版本的 `chat` 方法接受一个或多个 `ChatMessage` 作为输入。 `ChatMessage` 是表示聊天消息的基本接口。



如果您希望自定义请求（例如，指定温度、工具或 JSON 模式等）， 您可以使用 `chat(ChatRequest)` 方法：

```java
    ...
    
    ChatResponse chat(ChatRequest chatRequest);
        
    ...
```



```java
ChatRequest request = ChatRequest.builder()
    .messages(UserMessage.from())
    .parameters(ChatRequestParameters.builder()
        .temperature(0.5)
        .toolSpecifications(toolSpecifications)
        .build())
    .build();
```



### `ChatMessage` 的类型

目前有四种类型的聊天消息，每种对应消息的一个"来源"：

- `UserMessage`：这是来自用户的消息。根据 LLM 支持的模态，`UserMessage` 可以只包含文本（`String`）， 或[其他模态](https://docs.langchain4j.info/tutorials/chat-and-language-models#multimodality)。
- `AiMessage`：这是由 AI 生成的消息，通常是对 `UserMessage` 的回应。`AiMessage` 可以包含文本响应（`String`）或执行工具的请求（`ToolExecutionRequest`）。
- `ToolExecutionResultMessage`：这是 `ToolExecutionRequest` 的结果。
- `SystemMessage`：这是来自系统的消息。通常，您会在这里写入关于 LLM 角色是什么、它应该如何行为、以什么风格回答等指令。 LLM 被训练为比其他类型的消息更加关注 `SystemMessage`， 所以要小心，最好不要让最终用户自由定义或在 `SystemMessage` 中注入一些输入。 通常，它位于对话的开始。
- `CustomMessage`：这是一个可以包含任意属性的自定义消息。这种消息类型只能由 支持它的 `ChatLanguageModel` 实现使用（目前只有 Ollama）。



现在我们了解了所有类型的 `ChatMessage`，让我们看看如何在对话中组合它们。

在最简单的情况下，我们可以向 `chat` 方法提供单个 `UserMessage` 实例。返回 `ChatResponse`。 除了 `AiMessage` 外，`ChatResponse` 还包含 `ChatResponseMetadata`。 `ChatResponseMetadata` 包含 `TokenUsage`，其中包含有关输入包含多少令牌的统计信息， 输出（在 `AiMessage` 中）生成了多少令牌，以及总计（输入 + 输出）。  然后，`ChatResponseMetadata` 还包含 `FinishReason`， 这是一个枚举，包含生成停止的各种原因。 通常，如果 LLM 自己决定停止生成，它将是 `FinishReason.STOP`。



### 案例：统计 token 耗费

```java
    @GetMapping("/chat2/{question}")
    public String chat2(@PathVariable(value = "question") String question) {
        UserMessage message = UserMessage.from(question);
        ChatResponse response = chatLanguageModel.chat(message);
        //获取token消耗
        ChatResponseMetadata metadata = response.metadata();
        TokenUsage tokenUsage = metadata.tokenUsage();
        //或者直接
        //TokenUsage tokenUsage = response.tokenUsage();
        Integer inputTokenCount = tokenUsage.inputTokenCount();//输入token
        Integer outputTokenCount = tokenUsage.outputTokenCount();//输出token
        Integer totalTokenCount = tokenUsage.totalTokenCount();//总token
        log.info("调用模型完成，输入token={}，输出token={}，总token={}", inputTokenCount, outputTokenCount, totalTokenCount);
        //返回模型响应
        AiMessage aiMessage = response.aiMessage();
        return aiMessage.text();
    }
```



```
2026-01-05T11:37:19.670+08:00  INFO 21180 --- [demo1] [nio-8080-exec-1] c.e.demo1.controller.ChatController      : 调用模型完成，输入token=10，输出token=670，总token=680
```



### 多模态

`UserMessage` 不仅可以包含文本，还可以包含其他类型的内容。 `UserMessage` 包含 `List<Content> contents`。 `Content` 是一个接口，有以下实现：

- `TextContent`
- `ImageContent`
- `AudioContent`
- `VideoContent`
- `PdfFileContent`

您可以在[这里](https://docs.langchain4j.info/integrations/language-models)的比较表中查看哪些 LLM 提供商支持哪些模态。



#### 文本内容

`TextContent` 是最简单的 `Content` 形式，表示纯文本并包装单个 `String`。 `UserMessage.from(TextContent.from("Hello!"))` 等同于 `UserMessage.from("Hello!")`。

可以在 `UserMessage` 中提供一个或多个 `TextContent`：

```java
UserMessage userMessage = UserMessage.from(
    TextContent.from("Hello!"),
    TextContent.from("How are you?")
);
```



#### 图像内容

根据 LLM 提供商的不同，`ImageContent` 可以从**远程**图像的 URL 创建， 或从 Base64 编码的二进制数据创建：

```java
    @GetMapping("/chat3")
    public String chat3() throws IOException {
        File file = new File("C:/Users/user/Pictures/Saved Pictures/image.jpg");
        FileInputStream fis = new FileInputStream(file);
        byte[] bytes = fis.readAllBytes();
        fis.close();
        String base64 = Base64.getEncoder().encodeToString(bytes);
        UserMessage message = UserMessage.from(
                TextContent.from("以下两张图片内容是什么？"),
                ImageContent.from("https://picsum.photos/200/300"),
                ImageContent.from(base64, "image/jpg")
        );
        ChatResponse response = chatLanguageModel.chat(message);
        return response.aiMessage().text();
    }
```



```tex
这两张图片的内容分别如下： --- **第一张图片：** 这是一张具有宁静、略带怀旧氛围的室内照片。画面主体是一个带有深色木质窗格的老式窗户，窗外是茂密的绿色树林。阳光透过树叶和窗格洒入室内，在窗框和窗台上投下斑驳的光影。整体色调偏暗，突出了窗外自然光的明亮与生机，营造出一种静谧、沉思或略带孤寂的感觉。 这张图常被用作背景、壁纸或表达“窗外风景”、“时光流逝”、“自然与建筑”等意境。 --- **第二张图片：** 这是一位动漫风格的女性角色特写。她有着深蓝色短发、明亮的蓝色眼睛，表情略显淡漠或平静。她身穿红色和服（或类似传统服饰），衣领处有白色装饰，耳侧佩戴着白色的花朵状发饰。她手中似乎拿着筷子，面前有一个红色碗，可能正在用餐。 该角色出自日本动画《**Re:从零开始的异世界生活**》（Re:Zero − Starting Life in Another World），是其中的重要角色——**雷姆（Rem）** 的Q版（chibi）形象。在原作中，雷姆是罗兹瓦尔宅邸的女仆，性格温柔忠诚，深受观众喜爱。这个Q版形象常用于轻松搞笑或可爱风格的周边、头像或表情包。 --- ✅ 总结： - 图一：窗外绿树成荫的复古窗景，充满诗意与静谧感。 - 图二：动漫角色雷姆的Q版形象，来自《Re:从零开始的异世界生活》，正准备用餐。 如需进一步了解某张图的出处、用途或相关作品，可继续提问！
```



还可以指定 `DetailLevel` 枚举（带有 `LOW`/`HIGH`/`AUTO` 选项）来控制模型如何处理图像。 更多详情请参见[这里](https://platform.openai.com/docs/guides/vision#low-or-high-fidelity-image-understanding)。



#### 其他模态

`AudioContent` 类似于 `ImageContent`，但表示音频内容。

`VideoContent` 类似于 `ImageContent`，但表示视频内容。

`PdfFileContent` 类似于 `ImageContent`，但表示 PDF 文件的二进制内容。



## 5.流式输出

LLM 一次生成一个标记（token），因此许多 LLM 提供商提供了一种方式，可以逐个标记地流式传输响应，而不是等待整个文本生成完毕。 这显著改善了用户体验，因为用户不需要等待未知的时间，几乎可以立即开始阅读响应。

对于 `ChatLanguageModel` 接口，有相应的 `StreamingChatLanguageModel` 。 这些接口有类似的 API，但可以流式传输响应。 它们接受 `StreamingChatResponseHandler` 接口的实现作为参数。

```java
public interface StreamingChatResponseHandler {

    void onPartialResponse(String partialResponse);//LLM每生成一个token后调用

    void onCompleteResponse(ChatResponse completeResponse);//LLM token生成完成后调用

    void onError(Throwable error);//出现异常时调用
}
```



### 案例：StreamingChatLanguageModel 加 SseEmitter 实现流式输出

```java
    @Bean("streamingChatLanguageModel")
    public StreamingChatLanguageModel StreamingChatLanguageModel() {
        OpenAiStreamingChatModel model = OpenAiStreamingChatModel.builder()
                .baseUrl(baseUrl)
                .apiKey(apiKey)
                .modelName(modelName)
                .build();
        return model;
    }

    @GetMapping("/chat4/stream")
    public SseEmitter chatStream(@RequestParam String question) {
        SseEmitter sseEmitter = new SseEmitter(0L);//不超时
        streamingChatLanguageModel.chat(question, new StreamingChatResponseHandler() {
            @Override
            public void onPartialResponse(String s) {
                try {
                    sseEmitter.send(s);
                } catch (Exception e) {
                    sseEmitter.completeWithError(e);
                }
            }

            @Override
            public void onCompleteResponse(ChatResponse chatResponse) {
                sseEmitter.complete();
            }

            @Override
            public void onError(Throwable throwable) {
                sseEmitter.completeWithError(throwable);
            }
        });
        return sseEmitter;
    }
```





![img](02.png)
