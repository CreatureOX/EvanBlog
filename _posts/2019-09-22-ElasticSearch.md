---
title: "ElasticSearch"
share: false
categories:
  - 后端技术研究
tags:
  - 大数据
  - 搜索
---

## ElastciSearch 简介
ElastciSearch 是一个分布式可扩展的实时搜索和分析引擎，建立于全文搜索引擎 Apache Lucene(TM) 基础之上，具有以下功能：
1. 全文搜索

2. 分布式实时文件存储，将每一个字段都编入索引，使其可被搜索

3. 实时分析的分布式搜索引擎

4. 可扩展到上百台服务器，处理PB级别的结构化/非结构化数据

## 基本概念
ElastciSearch 是面向文档型数据库。一条数据相当于一个文档，用JSON格式作为文档序列化的格式。ElastciSearch 和 关系型数据库的术语可按如下对照表关联起来。

||||||    
-|-|-|-|-
关系型数据库 | 数据库 | 表 | 行 | 列 |
ElastciSearch | 索引(index) | 类型(type) | 文档(Documents) | 字段(Fields) |

## 使用 Docker 部署 ElasticSearch
1. 使用 docker命令 搜索相关镜像并启动容器

``` bash

# 搜索对应的 es镜像
docker search elasticsearch: 7.3.0

# 拉取对应的 es镜像
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.3.0

# 创建容器运行 
# 9200 是 供 http 访问端口, 9300 是 供 tcp 访问端口
# bdaab402b220 为镜像名
docker run --name es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" bdaab402b220

# 查看对应运行容器的status是否是up，即是否成功运行 elasticsearch
docker ps

```

2. 打开浏览器访问 http://localhost:9200/ 可看到如下结果

``` json

{
  "name" : "48872fe3c7bd",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "TWGpZLsRTbGT33km2mCEXA",
  "version" : {
    "number" : "7.3.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "de777fa",
    "build_date" : "2019-07-24T18:30:11.767338Z",
    "build_snapshot" : false,
    "lucene_version" : "8.1.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

```

## ElasticSearch 实战
### Restful API 使用方式
* 创建一条索引(customer)

``` bash
curl -X PUT "localhost:9200/customer?pretty"
```

``` json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "customer"
}
```

* 查看一条索引(customer)
``` bash
curl -X GET "localhost:9200/customer?pretty"
```

``` json
{
  "customer" : {
    "aliases" : { },
    "mappings" : { },
    "settings" : {
      "index" : {
        "creation_date" : "1569151701083",
        "number_of_shards" : "1",
        "number_of_replicas" : "1",
        "uuid" : "VpgjnrIgT6GzYqTjRMkamg",
        "version" : {
          "created" : "7030099"
        },
        "provided_name" : "customer"
      }
    }
  }
}
```

* 删除一条索引(customer)
``` bash
curl -X DELETE "localhost:9200/customer?pretty"
```

``` json
{
  "acknowledged" : true
}
```

* 查看一个索引下所有文档(customer)
``` bash
curl -XGET http://localhost:9200/customer/_doc/_search?pretty
```

``` json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "customer",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "John"
        }
      }
    ]
  }
}
```

* 插入一个文档(customer/1)
``` bash
# 指定文档ID
curl -H "Content-Type: application/json" -XPUT "localhost:9200/customer/_doc/1?pretty" -d "{"""name""":"""John"""}"
```

``` json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

``` bash
# 不指定文档ID
curl -H "Content-Type: application/json" -XPOST "localhost:9200/customer/_doc?pretty" -d "{"""name""":"""Mike"""}"
```

``` json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "7gqAWW0BScc1USLGH5Kk",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

* 查看一个文档(customer/1)
``` bash
curl -X GET "localhost:9200/customer/_doc/1?pretty"
```

``` json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "John"
  }
}
```

* 更新一个文档(customer/1)
1. 再次执行指定ID文档插入命令完成更新

2. 执行如下命令
``` bash
curl -H "Content-Type: application/json" -XPOST "localhost:9200/customer/_doc/1/_update?pretty" -d "{ """doc""":{"""name""":"""JohnUpdate"""}}"
```

``` json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

* 删除一个文档(customer/1)
``` bash
curl -X DELETE "localhost:9200/customer/_doc/1?pretty"
```

``` json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

### JAVA 接入使用方式
1. 创建一个 maven 项目在 pom.xml dependencies中引入相关依赖

```xml
        <!-- elasticsearch -->
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>7.3.0</version>
        </dependency>

        <!-- elasticsearch-rest-client -->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-client</artifactId>
            <version>7.3.0</version>
        </dependency>

        <!-- elasticsearch-rest-high-level-client -->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>7.3.0</version>
            <exclusions>
                <exclusion>
                    <groupId>org.elasticsearch.client</groupId>
                    <artifactId>elasticsearch-rest-client</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.elasticsearch</groupId>
                    <artifactId>elasticsearch</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        
```

2. 编写对应的JAVA配置

``` java
    public void makeConnection(){
        restHighLevelClient = new RestHighLevelClient(RestClient.builder(
                new HttpHost("localhost", 9200, "http")));
    }

    public void shutdown(){
        if (restHighLevelClient != null){
            try {
                restHighLevelClient.close();
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }

```

3. 编写对应的DEMO代码请求 customer 中 id = 1的信息

``` java
    public void highLevelESTest(){
        makeConnection();
        GetResponse getResponse = null;
        try {
            GetRequest getRequest = new GetRequest("customer", "1");
            getResponse = restHighLevelClient.get(getRequest, RequestOptions.DEFAULT);
        }catch (IOException e){
            e.printStackTrace();
        }
        Map<String, Object> source = getResponse.getSource();
        shutdown();
    }
    
```

4. 更进一步: 从es中查询出不同平台售出的商品的数目  
    Step 1. 决定从哪些索引中进行本次操作并获取到这些符合条件的索引  
    Step 2. 拼接搜索语句  
    Step 3. 拼接聚合语句 (es对于多条聚合结果会将其分别放在不同的 bucket 内)

``` java
    public void esDemo(){
        SearchRequest searchRequest = new SearchRequest();
        SearchResponse searchResponse = null;
        try {
            // es get indices
            String[] indexArrInEs = restHighLevelClient.indices().get(new GetIndexRequest(String.format("%s", "indice-test")),
                    RequestOptions.DEFAULT).getIndices();
            searchRequest.indices(indexArrInEs[0]);

            // es search
            SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
                    BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery()
                .must(QueryBuilders.termsQuery("platform", "platform"));
            sourceBuilder.query(boolQueryBuilder);

            // es aggregation
            TermsAggregationBuilder termsAgg = AggregationBuilders.terms("platform").field("platform");
            SumAggregationBuilder sumAgg = AggregationBuilders.sum("sold").field("sold");
            sourceBuilder.aggregation(termsAgg.subAggregation(sumAgg));

            searchRequest.source(sourceBuilder);
            searchResponse = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
        }catch (Exception e){
            log.info("Oops! Something wrong!");
        }
        Terms aggregation = searchResponse.getAggregations().get("platform");
        for (Terms.Bucket bucket: aggregation.getBuckets()){
            String term1 = bucket.getKeyAsString();
            Sum sum = bucket.getAggregations().get("sold");
        }
    }
```
