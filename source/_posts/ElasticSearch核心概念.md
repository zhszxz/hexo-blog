---
title: ElasticSearch核心概念
date: 2025-09-22 15:04:39
tags:
- 倒排索引
- 分词
- 文档
category: ElasticSearch
---

# ES核心概念

> ElasticSearch是面向文档的，与关系型数据库对比如下

| 关系型数据库        | es              |
| ------------------- | --------------- |
| 数据库（databases） | 索引（indices） |
| 表（tables）        | types           |
| 行（rows）          | documents       |
| 字段（columns）     | fields          |

<!--more-->

es（集群）中包含多个索引，每个索引可以包含多个类型，每个类型下又包含多个文档，文档又包含多个字段。

**物理设计：**

es把每个**索引划分为多个分片**，每个分片可以在集群的不同服务器间迁移，一个es节点就是一个集群。

**逻辑设计：**

一个索引当中，包含多个文档，可以通过**索引->类型->文档ID**索引到到某个具体的文档



> 文档

文档是索引和搜索数据的最小单位，以JSON格式存储，具备天然的灵活性，无需预先严格定义，但是文档中每个字段的类型非常重要，es会保存字段与类型间的映射

>类型

类型是文档的逻辑容器，就像关系型数据库的表一样，类型对字段的定义称为映射，类型的概念逐渐被抛弃，高版本的es中一个索引只包含一个类型：_doc

> 索引

索引是类型的容器，或者说文档的集合，索引存储了映射类型的字段和其他设置，创建索引默认包含一个主分片和副本分片，分布在集群不同节点上

> 倒排索引

倒排索引是包括es在内的全文搜索引擎的核心数据结构，通过将文档中的 ***关键词*** 与 ***包含该关键词的文档*** 建立映射，实现快速全文检索。与正排索引（由文档找关键词，逐篇文档读内容）不同，倒排索引是由关键词直接定位包含它的文档

#### 1. 准备原始文档

假设我们有 3 篇简单的文档，内容如下：

| 文档 ID | 文档内容                      |
| ------- | ----------------------------- |
| 1       | Elasticsearch 是搜索引擎      |
| 2       | 搜索引擎 Elasticsearch 很强大 |
| 3       | 我用 Elasticsearch 查数据     |

#### 2. 分词

ES 会对每个文档的文本字段进行分词，拆成一个个独立的关键词。以上文档分词后得到关键词列表（忽略 “是”“很” 等无意义虚词）：

- 文档 1 分词：`Elasticsearch`、`搜索引擎`
- 文档 2 分词：`搜索引擎`、`Elasticsearch`、`强大`
- 文档 3 分词：`Elasticsearch`、`查`、`数据`

#### 3. 构建倒排索引表

将 “关键词” 作为键，“包含该关键词的文档 ID 列表” 作为值，生成映射表（实际还会记录关键词在文档中的位置、频次等，简化示例仅保留文档 ID）：

| 关键词（Term） | 包含该关键词的文档 ID 列表（Posting List） |
| -------------- | ------------------------------------------ |
| Elasticsearch  | [1, 2, 3]                                  |
| 搜索引擎       | [1, 2]                                     |
| 强大           | [2]                                        |
| 查             | [3]                                        |
| 数据           | [3]                                        |

#### 4. 基于倒排索引搜索

当用户搜索关键词 `Elasticsearch 搜索引擎` 时，ES 会：

1. 对搜索词分词，得到 `Elasticsearch` 和 `搜索引擎`；
2. 查倒排索引表，找到两个关键词对应的文档列表：`[1,2,3]` 和 `[1,2]`；
3. 取两个列表的交集（即同时包含两个关键词的文档）：`[1,2]`；
4. 按相关性（如关键词出现频次、位置等）排序后返回文档 1 和文档 2。



### IK分词器

> 什么是IK分词器？

Ik是es常用的中文分词插件，解决中文分词问题（中文不像英文有天然空格分割），将中文句子分为有意义的词语，提升中文搜索准确性

IK有两种分词模式：

- 「ik_max_word」：最大化分词（细粒度），如 “Elasticsearch 很强大” 会拆分为 “Elasticsearch、很、强大”

- 「ik_smart」：智能分词（粗粒度），同样内容可能拆分为 “Elasticsearch、很强大”

  

#### 1. 测试`ik_max_word`模式（细粒度）

```bash
GET /_analyze
{
  "analyzer": "ik_max_word",
  "text": " Elasticsearch是一款强大的搜索引擎"
}
```

**分词结果**：

```json
{
  "tokens" : [
    {
      "token" : "elasticsearch",
      "start_offset" : 1,
      "end_offset" : 14,
      "type" : "ENGLISH",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 14,
      "end_offset" : 15,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "一款",
      "start_offset" : 15,
      "end_offset" : 17,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "一",
      "start_offset" : 15,
      "end_offset" : 16,
      "type" : "TYPE_CNUM",
      "position" : 3
    },
    {
      "token" : "款",
      "start_offset" : 16,
      "end_offset" : 17,
      "type" : "COUNT",
      "position" : 4
    },
    {
      "token" : "强大",
      "start_offset" : 17,
      "end_offset" : 19,
      "type" : "CN_WORD",
      "position" : 5
    },
    {
      "token" : "的",
      "start_offset" : 19,
      "end_offset" : 20,
      "type" : "CN_CHAR",
      "position" : 6
    },
    {
      "token" : "搜索引擎",
      "start_offset" : 20,
      "end_offset" : 24,
      "type" : "CN_WORD",
      "position" : 7
    },
    {
      "token" : "搜索",
      "start_offset" : 20,
      "end_offset" : 22,
      "type" : "CN_WORD",
      "position" : 8
    },
    {
      "token" : "索引",
      "start_offset" : 21,
      "end_offset" : 23,
      "type" : "CN_WORD",
      "position" : 9
    },
    {
      "token" : "引擎",
      "start_offset" : 22,
      "end_offset" : 24,
      "type" : "CN_WORD",
      "position" : 10
    }
  ]
}

```

#### 2. 测试`ik_smart`模式（粗粒度）

```bash
GET /_analyze
{
  "analyzer": "ik_smart",
  "text": " Elasticsearch是一款强大的搜索引擎"
}
```

**分词结果（示例）**：

```json
{
  "tokens" : [
    {
      "token" : "elasticsearch",
      "start_offset" : 1,
      "end_offset" : 14,
      "type" : "ENGLISH",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 14,
      "end_offset" : 15,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "一款",
      "start_offset" : 15,
      "end_offset" : 17,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "强大",
      "start_offset" : 17,
      "end_offset" : 19,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "的",
      "start_offset" : 19,
      "end_offset" : 20,
      "type" : "CN_CHAR",
      "position" : 4
    },
    {
      "token" : "搜索引擎",
      "start_offset" : 20,
      "end_offset" : 24,
      "type" : "CN_WORD",
      "position" : 5
    }
  ]
}

```

> Ik分词器基于预置的词库进行中文分词，预置词库覆盖面有限，而且互联网经常有新的词语出现，为此IK支持通过自定义词库扩展词汇
