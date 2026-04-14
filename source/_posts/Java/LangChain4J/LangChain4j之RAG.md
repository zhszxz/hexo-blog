---
title: LangChain4j之RAG
tags:
  - embedding
  - tika
  - DocumentSplitter
  - PromptTemplate
categories:
  - Java
  - LangChain4J
date: 2026-01-07 11:21:24
---

# RAG (检索增强生成)

LLM 的知识仅限于它已经训练过的数据。 如果你想让 LLM 了解特定领域的知识或专有数据，你可以：

- 使用 RAG，我们将在本节中介绍
- 用你的数据微调 LLM
- [结合 RAG 和微调](https://gorilla.cs.berkeley.edu/blogs/9_raft.html)

<!--more-->



## 什么是 RAG？

简单来说，RAG 是一种在发送给 LLM 之前，从你的数据中找到并注入相关信息片段到提示中的方法。 这样 LLM 将获得（希望是）相关信息，并能够使用这些信息回复， 这应该会降低产生幻觉的概率。

相关信息片段可以使用各种[信息检索](https://en.wikipedia.org/wiki/Information_retrieval)方法找到。 最流行的方法有：

- 全文（关键词）搜索。这种方法使用 TF-IDF 和 BM25 等技术， 通过匹配查询（例如，用户提问的内容）中的关键词与文档数据库进行搜索。 它根据每个文档中这些关键词的频率和相关性对结果进行排名。
- 向量搜索，也称为"语义搜索"。 文本文档使用嵌入模型转换为数字向量。 然后根据查询向量和文档向量之间的余弦相似度 或其他相似度/距离度量找到并排序文档， 从而捕捉更深层次的语义含义。
- 混合搜索。结合多种搜索方法（例如，全文 + 向量）通常可以提高搜索的有效性。

目前，本页主要关注向量搜索。 全文和混合搜索目前仅由 Azure AI Search 集成支持， 详情请参阅 `AzureAiSearchContentRetriever`。 我们计划在不久的将来扩展 RAG 工具箱，包括全文和混合搜索。

**RAG  的执行流程如下：**

```
用户问题
   ↓
Embedding（向量化）
   ↓
向量库相似度检索
   ↓
把「检索结果 + 用户问题」一起喂给 LLM
   ↓
回答
```



## RAG 阶段

RAG 过程分为两个不同的阶段：索引和检索。 LangChain4j 为这两个阶段提供了工具。



### 索引

在索引阶段，文档会被预处理，以便在检索阶段进行高效搜索。

这个过程可能因使用的信息检索方法而异。 对于向量搜索，这通常涉及清理文档、用额外数据和元数据丰富文档、 将文档分割成更小的片段（也称为分块）、嵌入这些片段，最后将它们存储在嵌入存储（也称为向量数据库）中。

索引阶段通常是离线进行的，这意味着最终用户不需要等待其完成。 例如，可以通过定时任务在周末每周重新索引一次公司内部文档来实现。 负责索引的代码也可以是一个单独的应用程序，只处理索引任务。

然而，在某些情况下，最终用户可能希望上传自己的自定义文档，使 LLM 能够访问这些文档。 在这种情况下，索引应该在线进行，并成为主应用程序的一部分。

以下是索引阶段的简化图表：

[![img](https://docs.langchain4j.info/assets/images/rag-ingestion-9b548e907df1c3c8948643795a981b95.png)](https://docs.langchain4j.info/tutorials/rag)



### 检索

检索阶段通常在线进行，当用户提交一个应该使用索引文档回答的问题时。

这个过程可能因使用的信息检索方法而异。 对于向量搜索，这通常涉及嵌入用户的查询（问题） 并在嵌入存储中执行相似度搜索。 然后将相关片段（原始文档的片段）注入到提示中并发送给 LLM。

以下是检索阶段的简化图表： [![img](https://docs.langchain4j.info/assets/images/rag-retrieval-f525d2937abc08fed5cec36a7f08a4c3.png)](https://docs.langchain4j.info/tutorials/rag)



## LangChain4j 中的 RAG 风格

LangChain4j 提供了三种 RAG 风格：

- [Easy RAG](https://docs.langchain4j.info/tutorials/rag/#easy-rag)：开始使用 RAG 的最简单方式
- [Naive RAG](https://docs.langchain4j.info/tutorials/rag/#naive-rag)：使用向量搜索的基本 RAG 实现
- [Advanced RAG](https://docs.langchain4j.info/tutorials/rag/#advanced-rag)：一个模块化的 RAG 框架，允许额外的步骤，如 查询转换、从多个来源检索和重新排序



## Easy RAG

LangChain4j 有一个"Easy RAG"功能，使开始使用 RAG 变得尽可能简单。 你不必了解嵌入、选择向量存储、找到合适的嵌入模型、 弄清楚如何解析和分割文档等。 只需指向你的文档，LangChain4j 将完成其魔法。

备注

当然，这种"Easy RAG"的质量会低于定制的 RAG 设置。 然而，这是开始学习 RAG 和/或制作概念验证的最简单方法。 之后，你将能够平稳地从 Easy RAG 过渡到更高级的 RAG， 调整和定制更多方面。

1. 导入 `langchain4j-easy-rag` 依赖：

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-easy-rag</artifactId>
    <version>1.0.0-beta3</version>
</dependency>
```



2. 让我们加载你的文档：

```java
List<Document> documents = FileSystemDocumentLoader.loadDocuments("/home/langchain4j/documentation");
```

这将加载指定目录中的所有文件。

> 底层使用 Apache Tika 库，它支持各种文档类型， 用于检测文档类型并解析它们。 由于我们没有明确指定使用哪个 `DocumentParser`， `FileSystemDocumentLoader` 将通过 SPI 加载 `ApacheTikaDocumentParser`， 由 `langchain4j-easy-rag` 依赖提供。



3. 现在，我们需要预处理文档并将其存储在专门的嵌入存储（也称为向量数据库）中。 这对于在用户提问时快速找到相关信息片段是必要的。 我们可以使用我们支持的 15+ [嵌入存储](https://docs.langchain4j.info/integrations/embedding-stores)中的任何一个， 但为了简单起见，我们将使用内存中的存储：

```java
InMemoryEmbeddingStore<TextSegment> embeddingStore = new InMemoryEmbeddingStore<>();
EmbeddingStoreIngestor.ingest(documents, embeddingStore);
```

> 1. `EmbeddingStoreIngestor` 通过 SPI 从 `langchain4j-easy-rag` 依赖加载 `DocumentSplitter`。 每个 `Document` 被分割成更小的片段（`TextSegment`），每个片段不超过 300 个令牌， 并有 30 个令牌的重叠。
> 2. `EmbeddingStoreIngestor` 通过 SPI 从 `langchain4j-easy-rag` 依赖加载 `EmbeddingModel`。 每个 `TextSegment` 使用 `EmbeddingModel` 转换为 `Embedding`。



### 案例：实现Easy RAG

```java
    //聊天模型
    @Bean
    public ChatLanguageModel chatLanguageModel() {
        OpenAiChatModel model = OpenAiChatModel.builder()
                .baseUrl(chatModelBaseUrl)
                .apiKey(chatModelApiKey)
                .modelName(chatModelName)
                .build();
        return model;
    }

    //向量化模型
    @Bean
    public EmbeddingModel embeddingModel() {
        OpenAiEmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
                .baseUrl(embeddingModelBaseUrl)
                .apiKey(embeddingModelApiKey)
                .modelName(embeddingModelName)
                .build();
        return embeddingModel;
    }

    //向量存储
    @Bean
    public EmbeddingStore<TextSegment> embeddingStore() {
        return new InMemoryEmbeddingStore<>();
    }

    /**
     * 文档入库
     *
     * @return
     */
    @GetMapping("/document/embedding")
    public String documentEmbedding() {
        //将文件加载为 Document
        List<Document> documents = FileSystemDocumentLoader.loadDocuments(documentsLocation);
        for (Document document : documents) {
            TextSegment textSegment = document.toTextSegment();//每个文档转为一个 TextSegment
            Embedding embedding = embeddingModel.embed(textSegment).content();//TextSegment 向量化
            embeddingStore.add(embedding, textSegment);//向量和原始文本段一起入库
        }
        return "success";
    }

    @GetMapping("/chat/rag/{question}")
    public String chat(@PathVariable("question") String question) {
        Embedding embedding = embeddingModel.embed(question).content();//问题向量化
        //检索相关文档
        List<EmbeddingMatch<TextSegment>> matches = embeddingStore.search(
                EmbeddingSearchRequest.builder()
                        .queryEmbedding(embedding)//问题向量
                        .minScore(0.5)//最小相似度得分
                        .maxResults(5)//最大召回文档数
                        .build()
        ).matches();

        //将命中的文档和问题一起发给 LLM
        String context = matches.stream().map(each -> each.embedded().text()).collect(Collectors.joining("\n"));
        String prompt = """
                请根据以下资料回答问题：
                ----------------
                %s
                ----------------
                问题：%s
                """.formatted(context, question);

        return chatLanguageModel.chat(prompt);
    }
```



1. 准备文档

```
咏鸡

鸡，鸡，鸡，
曲项向晨啼。
红冠映金羽，
白爪踏黄泥。
```

2. 文档解析入库
3. 用户提问

![使用RAG](01.png)



![不使用RAG](02.png)



### Document

`Document` 类表示整个文档，如单个 PDF 文件或网页。 目前，`Document` 只能表示文本信息， 但未来的更新将使其支持图像和表格。

> - `Document.text()` 返回 `Document` 的文本
> - `Document.metadata()` 返回 `Document` 的 `Metadata`（参见下面的"Metadata"部分）
> - `Document.toTextSegment()` 将 `Document` 转换为 `TextSegment`（参见下面的"TextSegment"部分）
> - `Document.from(String, Metadata)` 从文本和 `Metadata` 创建 `Document`
> - `Document.from(String)` 从文本创建带有空 `Metadata` 的 `Document`



### Metadata

每个 `Document` 包含 `Metadata`。 它存储关于 `Document` 的元信息，如其名称、来源、最后更新日期、所有者， 或任何其他相关细节。

`Metadata` 存储为键值映射，其中键为 `String` 类型， 值可以是以下类型之一：`String`、`Integer`、`Long`、`Float`、`Double`。

`Metadata` 有几个用途：

- 在将 `Document` 的内容包含在提示中时， 元数据条目也可以包括在内，为 LLM 提供额外的信息考虑。 例如，提供 `Document` 名称和来源可以帮助提高 LLM 对内容的理解。
- 在搜索相关内容以包含在提示中时， 可以通过 `Metadata` 条目进行过滤。 例如，你可以将语义搜索范围缩小到仅属于特定所有者的 `Document`。
- 当 `Document` 的来源更新时（例如，文档的特定页面）， 可以通过其元数据条目（例如，"id"、"source"等）轻松定位相应的 `Document`， 并在 `EmbeddingStore` 中更新它以保持同步。

> - `Metadata.from(Map)` 从 `Map` 创建 `Metadata`
> - `Metadata.put(String key, String value)` / `put(String, int)` / 等，向 `Metadata` 添加条目
> - `Metadata.putAll(Map)` 向 `Metadata` 添加多个条目
> - `Metadata.getString(String key)` / `getInteger(String key)` / 等，返回 `Metadata` 条目的值，将其转换为所需类型
> - `Metadata.containsKey(String key)` 检查 `Metadata` 是否包含指定键的条目
> - `Metadata.remove(String key)` 通过键从 `Metadata` 中删除条目

> - `TextSegment.text()` 返回 `TextSegment` 的文本
> - `TextSegment.metadata()` 返回 `TextSegment` 的 `Metadata`
> - `TextSegment.from(String, Metadata)` 从文本和 `Metadata` 创建 `TextSegment`
> - `TextSegment.from(String)` 从文本创建带有空 `Metadata` 的 `TextSegment`



### Document Splitter

LangChain4j 有一个 `DocumentSplitter` 接口，带有几个开箱即用的实现：

- `DocumentByParagraphSplitter`
- `DocumentByLineSplitter`
- `DocumentBySentenceSplitter`
- `DocumentByWordSplitter`
- `DocumentByCharacterSplitter`
- `DocumentByRegexSplitter`
- 递归：`DocumentSplitters.recursive(...)`

它们的工作方式如下：

1. 你实例化一个 `DocumentSplitter`，指定所需的 `TextSegment` 大小， 并可选择指定字符或令牌的重叠。
2. 你调用 `DocumentSplitter` 的 `split(Document)` 或 `splitAll(List<Document>)` 方法。
3. `DocumentSplitter` 将给定的 `Document` 分割成更小的单元， 这些单元的性质因分割器而异。例如，`DocumentByParagraphSplitter` 将 文档分成段落（由两个或更多连续的换行符定义）， 而 `DocumentBySentenceSplitter` 使用 OpenNLP 库的句子检测器将 文档分成句子，等等。
4. 然后，`DocumentSplitter` 将这些更小的单元（段落、句子、单词等）组合成 `TextSegment`， 尝试在不超过步骤 1 中设置的限制的情况下，在单个 `TextSegment` 中包含尽可能多的单元。 如果某些单元仍然太大而无法放入 `TextSegment`，它会调用子分割器。 这是另一个 `DocumentSplitter`，能够将不适合的单元分割成更细粒度的单元。 所有 `Metadata` 条目都从 `Document` 复制到每个 `TextSegment`。 每个文本片段都添加了一个唯一的元数据条目"index"。 第一个 `TextSegment` 将包含 `index=0`，第二个 `index=1`，依此类推。



### Text Segment Transformer

`TextSegmentTransformer` 类似于 `DocumentTransformer`（上面描述的），但它转换 `TextSegment`。

与 `DocumentTransformer` 一样，没有一种通用的解决方案， 所以我们建议实现你自己的 `TextSegmentTransformer`，根据你独特的数据定制。

一种对改善检索效果很好的技术是在每个 `TextSegment` 中包含 `Document` 标题或简短摘要。



### Embedding

`Embedding` 类封装了一个数值向量，表示已嵌入内容（通常是文本，如 `TextSegment`）的"语义含义"。

> - `Embedding.dimension()` 返回嵌入向量的维度（其长度）
> - `CosineSimilarity.between(Embedding, Embedding)` 计算两个 `Embedding` 之间的余弦相似度
> - `Embedding.normalize()` 规范化嵌入向量（就地）



### Embedding Model

`EmbeddingModel` 接口表示一种特殊类型的模型，将文本转换为 `Embedding`。

当前支持的嵌入模型可以在[这里](https://docs.langchain4j.info/category/embedding-models)找到。

> - `EmbeddingModel.embed(String)` 嵌入给定的文本
> - `EmbeddingModel.embed(TextSegment)` 嵌入给定的 `TextSegment`
> - `EmbeddingModel.embedAll(List<TextSegment>)` 嵌入所有给定的 `TextSegment`
> - `EmbeddingModel.dimension()` 返回此模型产生的 `Embedding` 的维度



### 案例： 生产级RAG落地

```java
    /**
     * 聊天模型
     */
    @Bean
    public ChatLanguageModel chatLanguageModel() {
        OpenAiChatModel model = OpenAiChatModel.builder()
                .baseUrl(chatModelProperties.getBaseurl())
                .apiKey(chatModelProperties.getApikey())
                .modelName(chatModelProperties.getModelname())
                .build();
        return model;
    }

    /**
     * 向量化模型
     */
    @Bean
    public EmbeddingModel embeddingModel() {
        OpenAiEmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
                .baseUrl(embeddingModelProperties.getBaseurl())
                .apiKey(embeddingModelProperties.getApikey())
                .modelName(embeddingModelProperties.getModelname())
                .build();
        return embeddingModel;
    }

    /**
     * 向量化存储配置
     *
     * @return
     */
    @Bean
    public EmbeddingStore<TextSegment> embeddingStore(EmbeddingModel embeddingModel) {

        int dimension = embeddingModel
                .embed("dimension-check")
                .content()
                .vector()
                .length;

        return MilvusEmbeddingStore.builder()
                .host(milvusProperties.getHost())
                .port(milvusProperties.getPort())
                .collectionName(milvusProperties.getCollectionname())
                .dimension(dimension)
                .retrieveEmbeddingsOnSearch(true)
                .build();
    }

    /**
     * 文档切分器
     */
    @Bean
    public DocumentSplitter documentSplitter() {
        //分片长度和字符重叠
        return new DocumentByParagraphSplitter(500, 50);
    }



    private final String PROMPT_TEMPLATE_WHEN_HITTING = """
            请根据以下资料回答问题：
                ----------------
               {{context}}
                ----------------
                问题：{{question}}
            """;
    private final String PROMPT_TEMPLATE_WHEN_NOT_HITTING = """
            请回答以下问题：
              {{question}}
            """;
    
    /**
     * 上传文档并向量化存储
     *
     * @param file
     * @return
     * @throws Exception
     */
    @PostMapping("/document/upload")
    public String upload(@RequestParam("file") MultipartFile file) throws Exception {
        //1.解析文本内容
        Tika tika = new Tika();
        String content = tika.parseToString(file.getInputStream());

        //2.文档分片
        Document document = Document.from(content);
        List<TextSegment> textSegments = documentSplitter.split(document);

        //3.分片向量化存储
        for (TextSegment textSegment : textSegments) {
            Embedding embedding = embeddingModel.embed(textSegment).content();
            embeddingStore.add(embedding, textSegment);
        }
        return "upload success, segments=" + textSegments.size();
    }

    @GetMapping("/chat/{question}")
    public String chat(@PathVariable("question") String question) {
        Embedding embedding = embeddingModel.embed(question).content();//问题向量化
        //检索相关文档
        List<EmbeddingMatch<TextSegment>> matches = embeddingStore.search(
                EmbeddingSearchRequest.builder()
                        .queryEmbedding(embedding)//问题向量
                        .minScore(0.6)//最小相似度得分
                        .maxResults(5)//最大召回文档数
                        .build()
        ).matches();

        //将命中的文档和问题一起发给 LLM
        String prompt = "";
        if (matches.size() > 0) {
            String context = matches.stream().map(each -> each.embedded().text()).collect(Collectors.joining("\n"));
            prompt = PromptTemplate.from(PROMPT_TEMPLATE_WHEN_HITTING)
                    .apply(Map.of(
                            "context", context,
                            "question", question
                    )).text();
        } else {
            prompt = PromptTemplate.from(PROMPT_TEMPLATE_WHEN_NOT_HITTING)
                    .apply(Map.of("question", question)).text();
        }
        return chatLanguageModel.chat(prompt);
    }
```

