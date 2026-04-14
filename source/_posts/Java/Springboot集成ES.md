---
title: Springboot集成ES
date: 2025-10-11 10:58:29
category: ElasticSearch
---

# Springboot集成ES



## ES的Java客户端

## 1. 早期阶段：Transport Client（基于 TCP）

<!--more-->

- **出现版本**：ES 早期版本（1.x ~ 6.x）
- 工作方式
  - 基于 **内部 TCP 协议** 与 ES 节点通信
  - 客户端必须与 ES 集群版本**完全一致**（主版本、次版本、修订版本）
- 优点
  - 性能较高（长连接，无 HTTP 解析开销）
- 缺点
  - 版本耦合度高，升级风险大
  - API 不统一，学习成本高
  - 安全性弱（没有原生 HTTPS 支持）
- 状态
  - ES 7.x 标记为 **废弃**
  - ES 8.x 完全 **移除**

------

## 2. 中期阶段：High Level REST Client（HLRC）

- **出现版本**：ES 5.0 引入，7.x 成为主力
- 工作方式
  - 基于 **HTTP/REST API** 通信
  - 通过封装好的 Java API 构造请求、解析响应
- 优点
  - 版本兼容性更宽松（主版本一致即可）
  - 支持 HTTPS 和各种认证方式
  - 功能覆盖大部分 ES 操作
- 缺点
  - API 设计较老，非类型安全（依赖字符串和 Map）
  - 构建复杂查询时容易出错
  - 对 ES 8.x 新特性支持有限
- 状态
  - ES 7.x ~ 8.x 可用
  - 官方明确不再新增功能，仅做 bug 修复
  - 未来会逐步废弃

------

## 3. 现阶段：Elasticsearch Java Client（新客户端）

- 出现版本
  - ES 7.16 开始预览
  - ES 8.x 正式成为官方唯一推荐的 Java 客户端
- 工作方式
  - 基于 **HTTP/REST API**（和 HLRC 一样）
  - 内部使用 **Java API Client Generator** 生成类型安全的请求 / 响应类
- 优点
  - **类型安全**：请求参数和响应结构在编译期检查
  - **DSL 风格**：链式调用构建查询，代码可读性高
  - **模块化**：按需引入依赖，减少体积
  - **完全支持 ES 8.x 新特性**（如向量搜索、数据分层等）
  - **官方长期维护**
- 缺点
  - 与 HLRC API 不兼容，迁移需要改代码
  - 上手有一定学习成本（尤其是复杂查询）
- 适用版本
  - 推荐用于 **ES 7.17+**
  - 最佳匹配 **ES 8.x**



*** 接下来以 High Level REST Client 作为演示（没找到更新的教程）***

![High Level REST Client](1.png)

## 准备工作

1. 引入依赖

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.6.1</version>
</dependency>
```

2. 配置类

```java
@Configuration
public class ESconfig {
    @Bean
    public RestHighLevelClient restHighLevelClient() {
        return new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("127.0.0.1", 9200, "http")));
    }
}
```

3. 依赖注入

```java
@Autowired private RestHighLevelClient client;
```



## API操作详解

```java
    /**
     * 创建索引
     * @throws IOException
     */
    @Test
    void testCreateIndex() throws IOException {
        CreateIndexRequest request = new CreateIndexRequest("test_index");
        CreateIndexResponse response = client.indices().create(request, RequestOptions.DEFAULT);
        System.out.println(response);
    }
```



```java
    /**
     * 判断索引是否存在
     * @throws IOException
     */
    @Test
    void testExistIndex() throws IOException {
        GetIndexRequest request = new GetIndexRequest("test_index");
        boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
        System.out.println(exists);
    }
```



```java
    /**
     * 删除索引
     * @throws IOException
     */
    @Test
    void testDeleteIndex() throws IOException {
        DeleteIndexRequest request = new DeleteIndexRequest("test_index");
        AcknowledgedResponse response = client.indices().delete(request, RequestOptions.DEFAULT);
        System.out.println(response.isAcknowledged());
    }
```



```java
    /**
     * 添加文档
     * @throws IOException
     */
    @Test
    void testAddDocument() throws IOException {
        User user = new User("拜登", 10);
        IndexRequest request = new IndexRequest("test_index").id("1").source(objectMapper.writeValueAsString(user), XContentType.JSON);
        IndexResponse response = client.index(request, RequestOptions.DEFAULT);
        System.out.println(response);
    }
```



```java
/**
 * 判断文档是否存在
 * @throws IOException
 */
@Test
void testDocExist() throws IOException {
    GetRequest request = new GetRequest("test_index", "1");
    boolean exists = client.exists(request, RequestOptions.DEFAULT);
    System.out.println(exists);
}
```



```java
    /**
     * 查询文档
     * @throws IOException
     */
    @Test
    void testGetDocument() throws IOException {
        GetRequest request = new GetRequest("test_index", "1");
        GetResponse response = client.get(request, RequestOptions.DEFAULT);
        System.out.println(response);
        System.out.println(response.getSourceAsString());
    }
```



```java
    /**
     * 更新文档
     * @throws IOException
     */
    @Test
    void testUpdateDocument() throws IOException {
        UpdateRequest request = new UpdateRequest("test_index", "1");
        request.timeout("1s");
        User user = new User("老毕登", 100);
        request.doc(objectMapper.writeValueAsString(user), XContentType.JSON);
        UpdateResponse response = client.update(request, RequestOptions.DEFAULT);
        System.out.println(response);
    }
```



```java
    /**
     * 删除文档
     * @throws IOException
     */
    @Test
    void testDeleteDocument() throws IOException {
        DeleteRequest request = new DeleteRequest("test_index", "1");
        DeleteResponse response = client.delete(request, RequestOptions.DEFAULT);
        System.out.println(response);
    }
```



```java
    /**
     * 批量操作
     * @throws IOException
     */
    @Test
    void testBulk() throws IOException {
        BulkRequest request = new BulkRequest("index1");
        request.add(new IndexRequest("index1").id("5").source(objectMapper.writeValueAsString(new Human("杜鲁门", 88, 1, "美国总统", Arrays.asList("冷战", "马歇尔计划", "原子弹"))), XContentType.JSON))
                .add(new IndexRequest("index1").id("6").source(objectMapper.writeValueAsString(new Human("特朗普", 70, 1, "共产党特工川建国", Arrays.asList("贸易战", "懂王", "红领巾"))), XContentType.JSON));
        BulkResponse response = client.bulk(request, RequestOptions.DEFAULT);
        System.out.println(response.status());
    }
```



```java
/**
 * 搜索文档
 * @throws IOException
 */
@Test
void testSearch() throws IOException {
    SearchRequest request = new SearchRequest("index1");
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    searchSourceBuilder.timeout(new TimeValue(10, TimeUnit.SECONDS));
    searchSourceBuilder.query(QueryBuilders.matchQuery("name", "杜鲁门"));
    request.source(searchSourceBuilder);
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    SearchHits hits = response.getHits();
    System.out.println(objectMapper.writeValueAsString(hits));
    for (SearchHit hit : hits.getHits()) {
        System.out.println(hit.getSourceAsString());
    }
}
```
