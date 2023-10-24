# 							Postman

(Postman)

### 1.创建索引

​		PUT	 http://127.0.0.1:9200/shopping 	创建一个'shopping' 的index

​		

#### 1.1查询所有索引

GET	 http://127.0.0.1:9200/_cat/indices?v



#### 1.2 获取索引详细信息

GET	http://127.0.0.1:9200/shopping



#### 1.3 删除索引

DELETE	 http://127.0.0.1:9200/shopping



### 2. 创建文档数据

DOCUMENT---> INDEX



POST	http://127.0.0.1:9200/shopping/_doc

会返回一个随机 _id, 该操作不是幂等性,PUT请求是幂等性，所以不能使用PUT

```JSON
{
    "title":"小米手机",
    "category":"小米",
    "image":"https://cdn.cnbj0.fds.api.mi-img.com/b2c-shopapi-pms/pms_1681726094.73687921.png",
    "price":3999.00
}
```







#### 2.1 自定义 _id 

幂等性

POST	 http://127.0.0.1:9200/shopping/_doc/1001

```json
{
    "title":"小米手机",
    "category":"小米",
    "image":"https://cdn.cnbj0.fds.api.mi-img.com/b2c-shopapi-pms/pms_1681726094.73687921.png",
    "price":3999.00
}
```



幂等性

PUT 	http://127.0.0.1:9200/shopping/_doc/1002



PUT  http://127.0.0.1:9200/shopping/_create/1003

```json
{
    "title":"小米手机",
    "category":"小米",
    "image":"https://cdn.cnbj0.fds.api.mi-img.com/b2c-shopapi-pms/pms_1681726094.73687921.png",
    "price":3999.00
}
```

​			

#### 2.2 查询

GET 	http://127.0.0.1:9200/shopping/_doc/1001



全部查询

​	GET 	http://127.0.0.1:9200/shopping/_search

#### 2.3 修改

全量修改

PUT 	http://127.0.0.1:9200/shopping/_doc/1001

```json
{
    "title":"小米手机",
    "category":"小米",
    "image":"https://cdn.cnbj0.fds.api.mi-img.com/b2c-shopapi-pms/pms_1681726094.73687921.png",
    "price":4999.00
}
```



局部修改

非幂等性

POST 	http://127.0.0.1:9200/shopping/_update/1001

```json
{
    "doc": {
        "title": "华为手机",
        "category":"华为"
    }
}
```



#### 2.4删除

DELETE	http://127.0.0.1:9200/shopping/_doc/1001



### 3.查询



#### 3.1条件查询



GET	http://127.0.0.1:9200/shopping/_search?q=category:小米	查询category为小米





在请求体中加入查询条件

GET	http://127.0.0.1:9200/shopping/_search

```json
{
    "query":{
        "match":{
            "category":"小米"
        }
    }
}
```



tips: query is in the request body as the following examples.

##### 3.1.1 全量查询

GET	http://127.0.0.1:9200/shopping/_search

```json
{
    "query":{
        "match_all":{
            
        }
    }
}
```



##### 3.1.2 分页查询

GET	http://127.0.0.1:9200/shopping/_search

```
{
    "query":{
        "match_all":{
            
        }
    },
    "from":0,
    "size":2
}
```

查询第二页  	页数= (页码-1)*size

```json
{
    "query":{
        "match_all":{
            
        }
    },
    "from":2,
    "size":2
}
```



显示 指定field

```json
{
    "query":{
        "match_all":{
            
        }
    },
    "from":2,
    "size":2,
    "_source": ["title"]
}
```



##### 3.1.3 查询排序

GET	http://127.0.0.1:9200/shopping/_search

```json
{
    "query":{
        "match_all":{
            
        }
    },
    "from":0,
    "size":2,
    "_source": ["title"],
    "sort":{
        "price":{
            "order":"desc"
        }
    }
}
```



#### 3.2 多条件查询



同时成立

GET	http://127.0.0.1:9200/shopping/_search

```json
{
    "query" : {
        "bool": {
            "must": [
                {
                    "match":{
                        "category":"小米"
                    }
                },
                {
                    "match":{
                        "price":2999
                    }
                }
            ]
        }
    }
}
```



非同时成立

GET 	http://127.0.0.1:9200/shopping/_search

```json
{
    "query" : {
        "bool": {
            "should": [
                {
                    "match":{
                        "category":"小米"
                    }
                },
                {
                    "match":{
                        "category":"华为"
                    }
                }
            ]
        }
    }
}
```



范围查询

GET	http://127.0.0.1:9200/shopping/_search

```json
{
    "query" : {
        "bool": {
            "should": [
                {
                    "match":{
                        "category":"小米"
                    }
                },
                {
                    "match":{
                        "category":"华为"
                    }
                }
            ],
            "filter":{
                "range":{
                    "price":{
                        "gt":2999
                    }
                }
            }
        }
    }
}
```



#### 3.3匹配查询

全文检索匹配

GET	http://127.0.0.1:9200/shopping/_search

```json
{
    "query":{
        "match":{
            "category":"小华"
        }
    }
}
```

会匹配出 category 存在 '小'  '米'  '华'  '为' 	(倒排索引)



完全匹配

 GET	http://127.0.0.1:9200/shopping/_search

```json
{
    "query":{
        "match_phrase":{
            "category":"小华"
        }
    }
}
```



高亮查询

 GET	http://127.0.0.1:9200/shopping/_search

```json
{
    "query":{
        "match_phrase":{
            "category":"华为"
        }
    },
    "highlight": {
        "fields":{
            "category": {}
        }
    }
}
```



#### 3.4聚合查询



GET	http://127.0.0.1:9200/shopping/_search

```json
{
    "aggs":{    // 聚合操作
        "price_group" :{    // 统计结果的名称
            "terms":{   // 分组
                "field":"price"     // 分组字段
            }
        }
    }
}
```

会拿到原始数据



GET	http://127.0.0.1:9200/shopping/_search

```
{
    "aggs":{    // 聚合操作
        "price_group" :{    // 统计结果的名称
            "terms":{   // 分组
                "field":"price"     // 分组字段
            }
        }
    },
    "size": 0
}
```

只会拿到统计结果



```json
{
    "aggs":{    // 聚合操作
        "price_avg" :{    // 统计结果的名称
            "avg":{   // 平均值
                "field":"price"     // 分组字段
            }
        }
    },
    "size": 0
}
```



### 4.映射关系

MAPPING



#### 1.create an index

PUT	http://127.0.0.1:9200/user



#### 2.create mapping

PUT	http://127.0.0.1:9200/user/_mapping

request body

```json
{
    "properties" : {
        "name" : {
            "type" : "text",    // 可分词
            "index" : true
        },
        "sex": {
            "type" : "keyword",    // 不能分词，必须完整匹配
            "index" : true
        },
        "tel": {
            "type" : "keyword",
            "index" : false
        }
    }
}
```





#### 3.show mapping

GET	http://127.0.0.1:9200/user/_mapping



#### 4.test

####  

_create

PUT	http://127.0.0.1:9200/user/_create/1001

```json
{
    "name":"小米",
    "sex":"男的",
    "tel":"1234"
}
```



_search

GET 	http://127.0.0.1:9200/user/_search

```json
{
    "query":{
        "match":{
            "name":"小"
        }
    }
}
```



```json
{
    "query":{
        "match":{
            "sex":"男"
        }
    }
}
```

keyword 不能被分词，不能查到



```json
{
    "query":{
        "match":{
            "sex":"男的"
        }
    }
}
```





```json
{
    "query":{
        "match":{
            "tel":"1234"
        }
    }
}
```

failed to create query: Cannot search on field [tel] since it is not indexed.





### 5.集群部署



#### 5.1 windows



##### 5.1.1 node-1001

配置 elasticsearch.yml 文件

```yaml
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
cluster.name: my-application
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
node.name: node-1001
node.master: true
node.data: true
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
#path.data: /path/to/data
#
# Path to log files:
#
#path.logs: /path/to/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
#network.host: 192.168.0.1
network.host: localhost
#
# Set a custom port for HTTP:
#
#http.port: 9200
http.port: 1001
transport.tcp.port: 9301
#cluster.initial_master_nodes: ["master","node"]
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.seed_hosts: ["host1", "host2"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
#cluster.initial_master_nodes: ["node-1", "node-2"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
#gateway.recover_after_nodes: 3
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true
http.cors.enabled: true
http.cors.allow-origin: "*"
```





postman 中测试



GET	http://127.0.0.1:1001/_cluster/health



response

```json
{
    "cluster_name": "my-application",
    "status": "green",
    "timed_out": false,
    "number_of_nodes": 1,
    "number_of_data_nodes": 1,
    "active_primary_shards": 0,
    "active_shards": 0,
    "relocating_shards": 0,
    "initializing_shards": 0,
    "unassigned_shards": 0,
    "delayed_unassigned_shards": 0,
    "number_of_pending_tasks": 0,
    "number_of_in_flight_fetch": 0,
    "task_max_waiting_in_queue_millis": 0,
    "active_shards_percent_as_number": 100.0
}
```



##### 5.1.2 node-1002

配置 elasticsearch.yml 文件

```yaml
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
cluster.name: my-application
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
node.name: node-1002
node.master: true
node.data: true
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
#path.data: /path/to/data
#
# Path to log files:
#
#path.logs: /path/to/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
#network.host: 192.168.0.1
network.host: localhost
#
# Set a custom port for HTTP:
#
#http.port: 9200
http.port: 1002
transport.tcp.port: 9302
discovery.seed_hosts: ["localhost:9301"]
discovery.zen.fd.ping_timeout: 1m
discovery.zen.fd.ping_retries: 5
#cluster.initial_master_nodes: ["master","node"]
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.seed_hosts: ["host1", "host2"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
#cluster.initial_master_nodes: ["node-1", "node-2"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
#gateway.recover_after_nodes: 3
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true
http.cors.enabled: true
http.cors.allow-origin: "*"
```



GET	http://127.0.0.1:1001/_cluster/health

response

```json
{
    "cluster_name": "my-application",
    "status": "green",
    "timed_out": false,
    "number_of_nodes": 2,
    "number_of_data_nodes": 2,
    "active_primary_shards": 0,
    "active_shards": 0,
    "relocating_shards": 0,
    "initializing_shards": 0,
    "unassigned_shards": 0,
    "delayed_unassigned_shards": 0,
    "number_of_pending_tasks": 0,
    "number_of_in_flight_fetch": 0,
    "task_max_waiting_in_queue_millis": 0,
    "active_shards_percent_as_number": 100.0
}
```



##### 5.1.3 node-1003



```yaml
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
cluster.name: my-application
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
node.name: node-1003
node.master: true
node.data: true
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
#path.data: /path/to/data
#
# Path to log files:
#
#path.logs: /path/to/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
#network.host: 192.168.0.1
network.host: localhost
#
# Set a custom port for HTTP:
#
#http.port: 9200
http.port: 1003
transport.tcp.port: 9303
discovery.seed_hosts: ["localhost:9301","localhost:9302"]
discovery.zen.fd.ping_timeout: 1m
discovery.zen.fd.ping_retries: 5
#cluster.initial_master_nodes: ["master","node"]
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.seed_hosts: ["host1", "host2"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
#cluster.initial_master_nodes: ["node-1", "node-2"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
#gateway.recover_after_nodes: 3
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true
http.cors.enabled: true
http.cors.allow-origin: "*"
```



GET	http://127.0.0.1:1001/_cluster/health

```json
{
    "cluster_name": "my-application",
    "status": "green",
    "timed_out": false,
    "number_of_nodes": 3,
    "number_of_data_nodes": 3,
    "active_primary_shards": 0,
    "active_shards": 0,
    "relocating_shards": 0,
    "initializing_shards": 0,
    "unassigned_shards": 0,
    "delayed_unassigned_shards": 0,
    "number_of_pending_tasks": 0,
    "number_of_in_flight_fetch": 0,
    "task_max_waiting_in_queue_millis": 0,
    "active_shards_percent_as_number": 100.0
}
```





























##### ERRORS

1.master not discoverd yet

[ELK集群部署报错（master not discovered yet, this node has not previously joined a bootstrapped ）_熊博主的博客-CSDN博客](https://blog.csdn.net/weixin_36522099/article/details/112732301)

2.org.elasticsearch.discovery.MasterNotDiscoveredException: null

[elasticsearch报错org.elasticsearch.discovery.MasterNotDiscoveredException: null_奔跑吧邓邓子的博客-CSDN博客](https://blog.csdn.net/u012069313/article/details/123377280)



## CORE



### 1.Index

index --->  table

### 2.Type

7.x 默认不再支持自定义索引类型,默认类型为: _doc

### 3.Document

document ---> record

文档以json格式来表示

### 4.Field

field ---> field

json中属性对应着field

### 5.Mapping

Mapping ---> Schema

### 6.Shards

shards --->  分表

### 7.Replicas

* 在分片/节点 失败的情况下，提供了高可用性。因为这个原因，注意到复制分片从不与 原/主要 分片置于同一节点是非常重要的。
* 扩展搜索量/吞吐量，因为搜索可以在所有的副本上并行运行。

### 8.Allocation

master节点完成





## 分布式集群



### 1.单节点集群

在包含一个空节点的集群内创建名为users的索引，演示：分配3个主分片和一份副本(每个主分片拥有一个副本分片)



PUT	http://127.0.0.1:1001/users

(request_body)

```json
{
    "settings":{
        "number_of_shards":3,
        "number_of_replicas":1
    }
}
```



查询

GET	http://127.0.0.1:1001/users

(response)

```
{
    "users": {
        "aliases": {},
        "mappings": {},
        "settings": {
            "index": {
                "creation_date": "1688522032425",
                "number_of_shards": "3",
                "number_of_replicas": "1",
                "uuid": "87nNwoOfQ8ODwI9IgJ0RkQ",
                "version": {
                    "created": "7080099"
                },
                "provided_name": "users"
            }
        }
    }
}
```





#### 1.2 水平扩容

无法修改index的shards，因为已经定义好了。但可以修改replicas，提高吞吐量。

PUT	http://127.0.0.1:1001/users/_settings

```json
{
    "number_of_replicas":2
}
```



#### 1.3 应对故障

如果node-1001 down

**cluster health: yellow (6 of 9)**



再重启node-1001

修改elasticsearch.yaml

```yaml
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
#network.host: 192.168.0.1
network.host: localhost
#
# Set a custom port for HTTP:
#
#http.port: 9200
http.port: 1001
transport.tcp.port: 9301
discovery.seed_hosts: ["localhost:9302","localhost:9302"]
#
# For more information, consult the network module documentation.
```



#### 1.4 路由计算

主分片

hash(id) % 主分片数 ---> [0,1,2]



查数据

**分片控制：用户可以访问任何一个节点来获取数据，因为存放规则是一致的，那么查询规则也是一致的。**

**这个节点称之为协调节点。**

分片控制为轮询操作。



#### 1.5 数据读写



写

1.客户端请求集群节点(任意) ---协调节点

2.协调节点将请求转发到指定的节点

3.主分片需要将数据保存

4.主分片需要将数据发送给副本

5.副本保存后，进行反馈

6.主分片进行反馈

7.客户端获取反馈



读

1.客户端发送查询请求到协调节点

2.协调节点计算数据所在的分片以及全部的副本位置

3.为了能够负载均衡，可以轮询所有节点

4.将请求转发给具体的节点

5.节点返回查询结构，将结果反馈给客户端





#### 1.6 更新流程

请求到协调节点，根据路由请求跳转到具体的数据节点，把数据查出来后要不断的更新 (别的进程抢占)，

写入成功同步副本，最后给反馈客户端。



多文档操作流程

批量操作只是单点操作并行处理。





#### 1.7分片原理

分片是因为一个索引的数据量过多后会影响效率，所以把一个数据量大的索引分解为各个部分。所有的分片组合到一块就是一个完成的索引数据。每个分片是ElasticSearch最小的工作单元。写入和读取都是基于分片来完成的。





##### 1.7.1 倒排索引





keyword 不能分词

text 分词取决于分词器



ik_max_word 细拆分

ik_smart 粗拆



词条: 	索引中最小存储和查询单元

词典:	字典，词条的集合,B+,HashMap

倒排表:	关键词出现在什么位置，出现的频率是什么，倒排表中每条记录称为倒排项



倒排索引搜索过程：搜索的时候查询单词的词典，判断单词是否在词典中，如果在词典中，进而查找倒排列表中的指针，通过倒排列表获取单词所对应的文档id的列表，通过文档id去对应数据。



##### 1.7.2 文档搜索

...



##### 1.7.x 文档分析

 分析器



测试分析器



GET http://127.0.0.1:9200/_analyze

(request_body)

```json
{
    "analyzer":"standard",
    "text":"Text to analyze"
}
```

(response)

```json
{
    "tokens": [
        {
            "token": "text",
            "start_offset": 0,
            "end_offset": 4,
            "type": "<ALPHANUM>",
            "position": 0
        },
        {
            "token": "to",
            "start_offset": 5,
            "end_offset": 7,
            "type": "<ALPHANUM>",
            "position": 1
        },
        {
            "token": "analyze",
            "start_offset": 8,
            "end_offset": 15,
            "type": "<ALPHANUM>",
            "position": 2
        }
    ]
}
```



指定分词器

###### IK分词器

GET http://127.0.0.1:9200/_analyze

(request_body)

```json
{
    "text":"测试单词"
}
```

(response)

```json
{
    "tokens": [
        {
            "token": "测",
            "start_offset": 0,
            "end_offset": 1,
            "type": "<IDEOGRAPHIC>",
            "position": 0
        },
        {
            "token": "试",
            "start_offset": 1,
            "end_offset": 2,
            "type": "<IDEOGRAPHIC>",
            "position": 1
        },
        {
            "token": "单",
            "start_offset": 2,
            "end_offset": 3,
            "type": "<IDEOGRAPHIC>",
            "position": 2
        },
        {
            "token": "词",
            "start_offset": 3,
            "end_offset": 4,
            "type": "<IDEOGRAPHIC>",
            "position": 3
        }
    ]
}
```



GET http://127.0.0.1:9200/_analyze

(request_body)

```json
{
    "text":"测试单词",
    "analyzer":"ik_max_word"
}
```

(response)

```json
{
    "tokens": [
        {
            "token": "测试",
            "start_offset": 0,
            "end_offset": 2,
            "type": "CN_WORD",
            "position": 0
        },
        {
            "token": "单词",
            "start_offset": 2,
            "end_offset": 4,
            "type": "CN_WORD",
            "position": 1
        }
    ]
}
```



ik_max_word:	会将文本做最细粒度的拆分

ik_smart:	会将文本做最粗粒度的拆分





```json
{
    "text":"中国人",
    "analyzer":"ik_max_word"
}
```

```json
{
    "tokens": [
        {
            "token": "中国人",
            "start_offset": 0,
            "end_offset": 3,
            "type": "CN_WORD",
            "position": 0
        },
        {
            "token": "中国",
            "start_offset": 0,
            "end_offset": 2,
            "type": "CN_WORD",
            "position": 1
        },
        {
            "token": "国人",
            "start_offset": 1,
            "end_offset": 3,
            "type": "CN_WORD",
            "position": 2
        }
    ]
}
```





```json
{
    "text":"中国人",
    "analyzer":"ik_smart"
}
```

```json
{
    "tokens": [
        {
            "token": "中国人",
            "start_offset": 0,
            "end_offset": 3,
            "type": "CN_WORD",
            "position": 0
        }
    ]
}
```





**custom**

plugins ---> ik--->config

创建 custom.dic文件,写入所需的自定义词。同时打开 IKAnalyzer.cfg.xml,将新建的custom.dic配置其中

(IKAnalyzer.cfg.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">custom.dic</entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords"></entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<!-- <entry key="remote_ext_dict">words_location</entry> -->
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>

```





GET	http://127.0.0.1:9200/_analyze

(request_body)

```
{
    "text":"弗雷尔卓德",
    "analyzer":"ik_max_word"
}
```

(response)

```json
{
    "tokens": [
        {
            "token": "弗雷尔卓德",
            "start_offset": 0,
            "end_offset": 5,
            "type": "CN_WORD",
            "position": 0
        }
    ]
}
```





##### 1.7.x1 文档控制 

内部版本控制

乐观锁

PUT	http://127.0.0.1:9200/shopping/_create/1001

...

(resopnse)

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1001",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```



修改 

POST	http://127.0.0.1:9200/shopping/_update/1001

...

(response)

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1001",
    "_version": 2,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 1,
    "_primary_term": 1
}
```



再更新

POST	http://127.0.0.1:9200/shopping/_update/1001?version=1	(早期)

...

(response)

```json
{
    "error": {
        "root_cause": [
            {
                "type": "action_request_validation_exception",
                "reason": "Validation Failed: 1: internal versioning can not be used for optimistic concurrency control. Please use `if_seq_no` and `if_primary_term` instead;"
            }
        ],
        "type": "action_request_validation_exception",
        "reason": "Validation Failed: 1: internal versioning can not be used for optimistic concurrency control. Please use `if_seq_no` and `if_primary_term` instead;"
    },
    "status": 400
}
```





POST	http://127.0.0.1:9200/shopping/_update/1001?if_seq_no=0&if_primary_term=0

...

(response)

```json
{
    "error": {
        "root_cause": [
            {
                "type": "action_request_validation_exception",
                "reason": "Validation Failed: 1: ifSeqNo is set, but primary term is [0];"
            }
        ],
        "type": "action_request_validation_exception",
        "reason": "Validation Failed: 1: ifSeqNo is set, but primary term is [0];"
    },
    "status": 400
}
```





POST	http://127.0.0.1:9200/shopping/_update/1001?if_seq_no=1&if_primary_term=1

(request_body)

```json
{
    "doc": {
        "title": "华为手机",
        "category":"华为"
    }
}
```

(response)

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1001",
    "_version": 2,
    "result": "noop",
    "_shards": {
        "total": 0,
        "successful": 0,
        "failed": 0
    },
    "_seq_no": 1,
    "_primary_term": 1
}
```





POST	http://127.0.0.1:9200/shopping/_update/1001?if_seq_no=1&if_primary_term=1

(resquest_body)

```json
{
    "doc": {
        "title": "华为手机",
        "category":"华为123"
    }
}
```

(response)

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1001",
    "_version": 3,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 2,
    "_primary_term": 1
}
```





外部版本控制

GET	http://127.0.0.1:9200/shopping/_doc/1001

(response)

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1001",
    "_version": 3,
    "_seq_no": 2,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "title": "华为手机",
        "category": "华为123",
        "image": "https://cdn.cnbj0.fds.api.mi-img.com/b2c-shopapi-pms/pms_1681726094.73687921.png",
        "price": 3999.0
    }
}
```



POST	http://127.0.0.1:9200/shopping/_doc/1001?version=1&version_type=external

(request_body)

```json
{
    "title":"测试手机"
}
```

(response)

```json
{
    "error": {
        "root_cause": [
            {
                "type": "version_conflict_engine_exception",
                "reason": "[1001]: version conflict, current version [3] is higher or equal to the one provided [1]",
                "index_uuid": "2xb8E_JeSKS-UjoiVpceFA",
                "shard": "0",
                "index": "shopping"
            }
        ],
        "type": "version_conflict_engine_exception",
        "reason": "[1001]: version conflict, current version [3] is higher or equal to the one provided [1]",
        "index_uuid": "2xb8E_JeSKS-UjoiVpceFA",
        "shard": "0",
        "index": "shopping"
    },
    "status": 409
}
```



POST	http://127.0.0.1:9200/shopping/_doc/1001?version=4&version_type=external

...

(response)

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1001",
    "_version": 4,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 3,
    "_primary_term": 1
}
```



查询结果

GET	http://127.0.0.1:9200/shopping/_doc/1001

(response)

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1001",
    "_version": 4,
    "_seq_no": 3,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "title": "测试手机"
    }
}
```





## Kibana

### START

1.下载

```
https://www.elastic.co/downloads/kibana
```

2.修改 config -> kibana.yml 文件

```yml
# 默认端口
server.port: 5601
# ES服务器的地址
elasticsearch.hosts: ["http://localhost:9200"]
# 索引名
kibana.index: ".kibana"
# 支持中文
i18n.locale: "zh-CN"
```

3.win 执行 bin->kibana.bat



### ERROR

 FATAL  [index_closed_exception] closed, with { index_uuid="YlwaIU5QTkWDqKp9kp5FzQ" & index=".kibana_1" }



postman

GET	http://127.0.0.1:9200/_cat/indices?v

查看status是否为close



POST	http://127.0.0.1:9200/.kibana_1/_open

health status index                          uuid                   pri rep docs.count docs.deleted store.size pri.store.size

green  open   .kibana_1                      YlwaIU5QTkWDqKp9kp5FzQ   1   0        200            4     79.7kb         79.7kb











# RESTART

(Kibana)

## INDEX

### 1.创建索引库

```json
# 创建索引库
PUT /xxx
{
  "mappings": {
    "properties": {
      "info": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "email": {
        "type": "keyword",
        "index": false
      },
      "name": {
        "type": "object",
        "properties": {
          "firstName": {
            "type": "keyword"
          },
          "lastName": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

### 2.查看、删除索引库

查看

```json
GET /xxx
```



删除

```json
DELETE /xxx
```

### 3.修改索引库

索引库和mapping一旦创建无法修改。



但是可以添加新字段

```json
# 修改索引库，添加新字段
PUT /xxx/_mapping
{
  "properties":{
    "age":{
      "type":"integer"
    }
  }
}
```







## DOCUMENT



### 1.添加文档

```json
# 插入一个文档
POST /xxx/_doc/1
{
  "info": "zzzzzz",
  "email": "zy@itcast.cn",
  "name": {
    "firstName": "云",
    "lastName": "赵"
  }
}
```



### 2.查询、删除文档

```json
# 查询文档
GET /xxx/_doc/1
```

```json
# 删除
DELETE /xxx/_doc/1
```



### 3.修改文档



全量修改

id存在就是修改，不存在是新增

```json
# 全量修改文档
PUT /xxx/_doc/1
{
  "info": "zzz",
  "email": "zhaoYun@itcast.cn",
  "name": {
    "firstName": "云",
    "lastName": "赵"
  }
}
```



局部修改



```json
# 局部修改
POST /xxx/_update/1
{
  "doc": {
    "email":"zYunn@itcast.cn"
  }
}
```





## RestClient



python

```http
https://www.elastic.co/guide/en/elasticsearch/client/python-api/current/index.html
```





hotel index (mapping)

```json
PUT /hotel
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "name": {
        "type": "text",
        "analyzer": "ik_max_word",
        "copy_to": "all"
      },
      "address": {
        "type": "keyword",
        "index": false
      },
      "price": {
        "type": "integer"
      },
      "score": {
        "type": "integer"
      },
      "brand": {
        "type": "keyword",
        "copy_to": "all"
      },
      "city": {
        "type": "keyword"
      },
      "starName": {
        "type": "keyword"
      },
      "business": {
        "type": "keyword",
        "copy_to": "all"
      },
      "location": {
        "type": "geo_point"
      },
      "pic": {
        "type": "keyword",
        "index": false
      },
      "all": {
        "type": "text",
        "analyzer": "ik_max_word"
      }
    }
  }
}
```





table (django -> models.py)

```python
class HotelData(models.Model):
    """酒店数据表"""
    id = models.BigIntegerField(verbose_name="酒店id", max_length=20, primary_key=True)
    name = models.CharField(verbose_name="酒店名称", max_length=255)
    address = models.CharField(verbose_name="酒店地址", max_length=255)
    price = models.IntegerField(verbose_name="酒店价格", max_length=10)
    score = models.IntegerField(verbose_name="酒店评分", max_length=2)
    brand = models.CharField(verbose_name="酒店品牌", max_length=32)
    city = models.CharField(verbose_name="所在城市", max_length=32)
    star_name = models.CharField(verbose_name="酒店星级", max_length=16)
    business = models.CharField(verbose_name="商圈", max_length=255)
    latitude = models.CharField(verbose_name="维度", max_length=32)
    longitude = models.CharField(verbose_name="精度", max_length=32)
    pic = models.CharField(verbose_name="酒店图片", max_length=255)
```









## Query DSL



```html
https://www.elastic.co/guide/en/elasticsearch/reference/8.8/getting-started.html
```



Match all：	eg.	match_all



Full text queries:	eg.	match_query, multi_match_query



Term-level queries:		eg.	ids, range, term



Geo queries:		eg. 	geo_distance,  geo_bounding_box



compound queries:		eg.	bool, function_score





#### Getting Start

Should have data migrated to es doc from database to es index.

(I'm still learning...)



```json
# create a doc in index named hotel
POST /hotel/_doc/1
{
  "address": "南京西路88号",
  "brand": "丽笙",
  "business": "人民广场",
  "city": "上海",
  "id": "1341",
  "location": "31.23462,121.473227",
  "name": "上海新世界丽笙大酒店",
  "pic": "xxx",
  "price": 1341,
  "score": 46,
  "starName": "五星级"
}
```





(match all)

```json
# 查询所有
GET /xxx/_search
{
  "query": {
    "FIELD": "TEXT"
  }
}
```



##### 1. Full text queries



###### match

Returns documents that match a provided text, number, date or boolean value. The provided text is analyzed before matching.



(template)

```json
GET /hotel/_search
{
  "query": {
    "match": {
      "FIELD": "TEXT"
    }
  }
}
```

(eg.)

```json
GET /hotel/_search
{
  "query": {
    "match": {
      "all": "人民广场"
    }
  }
}
```

###### multi-match



(template)

```json
GET /hotel/_search
{
  "query": {
    "multi_match": {
      "query": "TEXT",
      "fields": ["FIELD1","FIELD2"]
    }
  }
}
```

(eg.)

```json
GET /hotel/_search
{
  "query": {
    "multi_match": {
      "query": "人民广场",
      "fields": ["brand","name","business"]
    }
  }
}
```



搜索字段越多，效率越低，可以将多个字段copy到一个字段进行搜索。



##### 2. Term-level queries

一般查找keyword、数值、日期、boolean等类型字段。所以不会对搜索条件分词。



###### term



(template)

```json
GET /hotel/_search
{
  "query": {
    "term": {
      "FIELD": {
        "value": "VALUE"
      }
    }
  }
}
```



(eg.)

```json
GET /hotel/_search
{
  "query": {
    "term": {
      "city": {
        "value": "上海"
      }
    }
  }
}
```





###### range



(template)

```json
GET /hotel/_search
{
  "query": {
    "range": {
      "FIELD": {
        "gte": 10,
        "lte": 20
      }
    }
  }
}
```



(eg.)

```json
GET /hotel/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 100,
        "lt": 1341
      }
    }
  }
}
```



##### 3. Geo queries



###### geo_bounding_box

(template)

```json
GET /xxx/_search
{
  "query": {
    "geo_bounding_box": {
      "FIELD": {
        "top_left": {
          "lat": "XXX",
          "lon": "xxx"
        },
        "bottom_right": {
          "lat": "xxx",
          "lon": "xxx"
        }
      }
    }
  }
}
```



###### geo_distance

```json
GET /hotel/_search
{
  "query": {
    "geo_distance": {
      "distance": "15km",
      "location": "31.21,121.5"
    }
  }
}
```





##### 4. compound queries



相关性算分

当利用match查询是，文档结果会根据搜索词条的关联度打分(_score),返回结果是按照分值降序排列

TF(词条评率) = 词条出现次数 / 文档中词条总数



TF-IDF

IDF(逆文档频率)=log(文档总数/包含词条的文档总数)

_score = sigma(n,i) 	TF * IDF



BM25

...



###### function score



![](C:\Users\user\Desktop\ElasticSearch_Doc\assets\func_score.jpg)



过滤条件、算分函数、加权方式

```json
GET /hotel/_search
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "all": "上海"
        }
      },
      "functions": [
        {
          "filter": {
            "term": {
              "brand": "丽笙"
            }
          },
          "weight": 10
        }
      ],
      "boost_mode": "sum"
    }
  }
}
```



###### boolean

A query that matches documents matching boolean combinations of other queries. The bool query maps to Lucene `BooleanQuery`. It is built using one or more boolean clauses, each clause with a typed occurrence. The occurrence types are:

| Occur      | Description                                                  |
| ---------- | :----------------------------------------------------------- |
| `must`     | The clause (query) must appear in matching documents and will contribute to the score. |
| `filter`   | The clause (query) must appear in matching documents. However unlike `must` the score of the query will be ignored. Filter clauses are executed in [filter context](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-filter-context.html), meaning that scoring is ignored and clauses are considered for caching. |
| `should`   | The clause (query) should appear in the matching document.   |
| `must_not` | The clause (query) must not appear in the matching documents. Clauses are executed in [filter context](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-filter-context.html) meaning that scoring is ignored and clauses are considered for caching. Because scoring is ignored, a score of `0` for all documents is returned. |







(eg.)

```json
GET /hotel/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "city": {
              "value": "上海"
            }
          }
        }
      ],
      "should": [
        {
          "term": {
            "brand": {
              "value": "丽笙"
            }
          }
        },
        {
          "term": {
            "brand": {
              "value": "美丽"
            }
          }
        }
      ],
      "must_not": [
        {
          "range": {
            "price": {
              "lte": 500
            }
          }
        }
      ],
      "filter": [
        {
          "range": {
            "score": {
              "gte": 43
            }
          }
        }
      ]
    }
  }
}

```





```json
GET /hotel/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "丽"
          }
        }
      ],
      "must_not": [
        {
          "range": {
            "price": {
              "lte": 400
            }
          }
        }
      ],
      "filter": [
        {
          "geo_distance": {
            "distance": "10km",
            "location": {
              "lat": 31.21,
              "lon": 121.5
            }
          }
        }
      ]
    }
  }
}
```





## Search your data



### 1.Sort search results



默认根据 _score 来排序

可排序字段类型有：keyword、数值类型、地理坐标类型、日期类型



普通类型

(template)

```json
GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "FIELD": {
        "order": "desc"
      }
    }
  ]
}
```



地理坐标类型

(template)

```json
GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "_geo_distance": {
        "FIELD": {
          "lat": 40,
          "lon": -70
        },
        "order": "asc",
         "unit": "km"
      }
    }
  ]
}
```



(eg.)

对酒店数据按照用户评价降序，评价相同按照价格升序

```json
GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "score": "desc"
    },
    {
      "price": "asc"
    }
  ]
}
```



```json
GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "_geo_distance": {
        "location": {
          "lat":  31.034661,
          "lon": 121.612282
        },
        "order": "asc",
        "unit": "km"
      }
    }
  ]
}
```



### 2.Paginate search results

To paginate through a larger set of results, you can use the search API’s `size` and `from` parameters. The `size` parameter is the number of matching documents to return. The `from` parameter is a zero-indexed offset from the beginning of the complete result set that indicates the document you want to start with.



elasticsearch默认情况下只返回10条数据，如果要查询更多数据就需要修改分页参数。

```json
GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "price": {
        "order": "asc"
      }
    }
  ],
  "from": 0,
  "size": 3
}
```

 	

#### Deep pagination

[Paginate search results | Elasticsearch Guide [7.8\] | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/paginate-search-results.html#search-after)



Search after:

Pagination of results can be done by using the `from` and `size` but the cost becomes prohibitive when the deep pagination is reached. The `index.max_result_window` which defaults to 10,000 is a safeguard, search requests take heap memory and time proportional to `from + size`. The [scroll](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/paginate-search-results.html#scroll-search-results) API is recommended for efficient deep scrolling but scroll contexts are costly and it is not recommended to use it for real time user requests. The `search_after` parameter circumvents this problem by providing a live cursor. The idea is to use the results from the previous page to help the retrieval of the next page.





### 3.Highlighting



Highlighters enable you to get highlighted snippets from one or more fields in your search results so you can show users where the query matches are. When you request highlights, the response contains an additional `highlight` element for each search hit that includes the highlighted fields and the highlighted fragments.



(template)

```json
GET /hotel/_search
{
  "query": {
    "match": {
      "FIELD": "TEXT"
    }
  },
  "highlight": {
    "fields": {
      "FIELD": {
        "pre_tags": "<em>",
        "post_tags": "</em>"
      }
    }
  }
}
```



默认情况下，搜索字段必须与高亮字段一致，否则不会高亮

(eg.)

```json
GET /hotel/_search
{
  "query": {
    "match": {
      "all": "丽"
    }
  },
  "highlight": {
    "fields": {
      "name": {
        "pre_tags": "<em>",
        "post_tags": "</em>"
      }
    }
  }
}
```

(response)

```json
{
  "took" : 29,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.99134624,
    "hits" : [
      {
        "_index" : "hotel",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.99134624,
        "_source" : {
          "address" : "南京西路88号",
          "brand" : "丽笙",
          "business" : "人民广场",
          "city" : "上海",
          "id" : "1341",
          "location" : "31.23462,121.473227",
          "name" : "上海新世界丽笙大酒店",
          "pic" : "xxx",
          "price" : "1341",
          "score" : "46",
          "starName" : "五星级"
        }
      },
      {
        "_index" : "hotel",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 0.5623363,
        "_source" : {
          "address" : "邯郸路199号",
          "brand" : "皇冠假日丽笙",
          "business" : "江湾/五角场商业区",
          "city" : "上海",
          "id" : "1342",
          "location" : "30.23462,123.473227",
          "name" : "上海复旦皇冠假日酒店",
          "pic" : "xxx",
          "price" : "942",
          "score" : "47",
          "starName" : "五星级"
        }
      }
    ]
  }
}

```



```json
GET /hotel/_search
{
  "query": {
    "match": {
      "all": "丽"
    }
  },
  "highlight": {
    "fields": {
      "name": {
        "require_field_match": "false", 
        "pre_tags": "<em>",
        "post_tags": "</em>"
      }
    }
  }
}
```

(response)

```json
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
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.99134624,
    "hits" : [
      {
        "_index" : "hotel",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.99134624,
        "_source" : {
          "address" : "南京西路88号",
          "brand" : "丽笙",
          "business" : "人民广场",
          "city" : "上海",
          "id" : "1341",
          "location" : "31.23462,121.473227",
          "name" : "上海新世界丽笙大酒店",
          "pic" : "xxx",
          "price" : "1341",
          "score" : "46",
          "starName" : "五星级"
        },
        "highlight" : {
          "name" : [
            "上海新世界<em>丽</em>笙大酒店"
          ]
        }
      },
      {
        "_index" : "hotel",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 0.5623363,
        "_source" : {
          "address" : "邯郸路199号",
          "brand" : "皇冠假日丽笙",
          "business" : "江湾/五角场商业区",
          "city" : "上海",
          "id" : "1342",
          "location" : "30.23462,123.473227",
          "name" : "上海复旦皇冠假日酒店",
          "pic" : "xxx",
          "price" : "942",
          "score" : "47",
          "starName" : "五星级"
        }
      }
    ]
  }
}

```







## Aggregations



### 1. Metrics Aggregations



#### Avg aggregation



#### Max aggregation



#### Min aggregation



#### Stats aggregation



聚合嵌套和自定义排序

```json
GET /hotel/_search
{
  "size": 0,
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 10
      },
      "aggs": {
        "scoreAgg": {
          "stats": {
            "field": "score"
          }
        }
      }
    }
  }
}
```



```json
GET /hotel/_search
{
  "size": 0,
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 10,
        "order": {
          "scoreAgg.avg": "desc"
        }
      },
      "aggs": {
        "scoreAgg": {
          "stats": {
            "field": "score"
          }
        }
      }
    }
  }
}
```





### 2. Bucket Aggregations



对文档分组



#### Terms Aggregation



按文档字段值分组



(template)

```json
GET /hotel/_search
{
  "size": 0,	//设置size为0，结果中不包含文档，只包含聚合结果
  "aggs": {
    "NAME": {
      "AGG_TYPE": {}
    }
  }
}
```



(eg.)

```json
GET /hotel/_search
{
  "size": 0,
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 10
      }
    }
  }
}
```



(response)

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "brandAgg" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "丽笙",
          "doc_count" : 1
        },
        {
          "key" : "大厦冰",
          "doc_count" : 1
        },
        {
          "key" : "皇冠假日丽笙",
          "doc_count" : 1
        },
        {
          "key" : "美丽",
          "doc_count" : 1
        }
      ]
    }
  }
}

```





自定义排序规则

```json
GET /hotel/_search
{
  "size": 0,
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 10,
        "order": {
          "_count": "asc"
        }
      }
    }
  }
}
```





默认情况下，bucket聚合是对索引库的所有文档做聚合，可以限定要聚合的文档范围，只要添加query条件即可

```json
GET /hotel/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 500
      }
    }
  }, 
  "size": 0,
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 10,
        "order": {
          "_count": "asc"
        }
      }
    }
  }
}
```



#### Date histogram aggregation



按照日期阶梯分组。例如：一周为一组，或者一月为一组





### 3. Pipeline Aggregations



其他聚合的结果为基础做聚合











# Neverland





## Query DSL



### Compound queries



#### Boolean



boolean query

A query that matches documents matching boolean combinations of other queries. The bool query maps to Lucene `BooleanQuery`. It is built using one or more boolean clauses, each clause with a typed occurrence. The occurrence types are:

| Occur      | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `must`     | The clause (query) must appear in matching documents and will contribute to the score. |
| `filter`   | The clause (query) must appear in matching documents. However unlike `must` the score of the query will be ignored. Filter clauses are executed in [filter context](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-filter-context.html), meaning that scoring is ignored and clauses are considered for caching. |
| `should`   | The clause (query) should appear in the matching document.   |
| `must_not` | The clause (query) must not appear in the matching documents. Clauses are executed in [filter context](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-filter-context.html) meaning that scoring is ignored and clauses are considered for caching. Because scoring is ignored, a score of `0` for all documents is returned. |



The `bool` query takes a *more-matches-is-better* approach, so the score from each matching `must` or `should` clause will be added together to provide the final `_score` for each document.







### Joining queries



#### Nested



nestd query

Wraps another query to search [nested](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/nested.html) fields.



The `nested` query searches nested field objects as if they were indexed as separate documents. If an object matches the search, the `nested` query returns the root parent document.



```
GET /xxx/_search
{
  "query": {
    "nested": {
      "path": "path_to_nested_doc",
      "query": {}
    }
  }
}
```



REQUEST

Index setup

```json
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "doc1": {
        "type": "nested",
        "properties": {
          "name": {
            "type":"text"
          },
          "count":{
            "type":"keyword"
          }
        }
      }
    }
  }
}
```



update Index

```json
PUT /my-index-000001/_mapping
{
  "properties": {
    "doc2": {
      "type": "nested",
      "properties": {
        "name": {
          "type": "text"
        },
        "count": {
          "type": "keyword"
        }
      }
    }
  }
}
```



index some docs to the index

```json
PUT /my-index-000001/_doc/1
{ 
  "doc1":{
    "name":"blue",
    "count":6
  }
}
PUT /my-index-000001/_doc/2
{ 
  "doc1":{
    "name":"red",
    "count":3
  }
}
PUT /my-index-000001/_doc/3
{ 
  "doc2":{
    "name":"black",
    "count":2
  }
}
PUT /my-index-000001/_doc/4
{ 
  "doc1":{
    "name":"red",
    "count":2
  },
  "doc2":{
    "name":"green",
    "count":3
  }
}
```



QUERY

```json
GET /my-index-000001/_search
{
  "query": {
    "nested": {
      "path": "doc1",
      "query": {
        "bool": {
          "must": [
            { "match": { "doc1.name": "red" } },
            { "range": { "doc1.count": { "gte": 2 } } }
          ]
        }
      },
      "score_mode": "avg"
    }
  }
}
```





### Term-level queries



#### Exists



exists query

Returns documents that contain an indexed value for a field.



An indexed value may not exist for a document’s field due to a variety of reasons:

- The field in the source JSON is `null` or `[]`
- The field has `"index" : false` set in the mapping
- The length of the field value exceeded an `ignore_above` setting in the mapping
- The field value was malformed and `ignore_malformed` was defined in the mapping



```
GET /my-index-000001/_search
{
  "query": {
    "nested": {
      "path": "doc1",
      "query": {
        "exists": {
          "field": "doc1.name"
        }
      }
    }
  }
}
```





#### Range



range query

Returns documents that contain terms within a provided range.



By default, the `terms` aggregation will return the buckets for the top ten terms ordered by the `doc_count`. One can change this default behaviour by setting the `size` parameter.



The `size` parameter can be set to define how many term buckets should be returned out of the overall terms list. By default, the node coordinating the search process will request each shard to provide its own top `size` term buckets and once all shards respond, it will reduce the results to the final list that will then be returned to the client. This means that if the number of unique terms is greater than `size`, the returned list is slightly off and not accurate (it could be that the term counts are slightly off and it could even be that a term that should have been in the top size buckets was not returned).





## Aggregations



basis

```json
"aggs": {
    "NAME": {
      "AGG_TYPE": {}
    }
  }
```





### Metrics Aggregations



#### Cardinality Aggregation

A `single-value` metrics aggregation that calculates an approximate count of distinct values. Values can be extracted either from specific fields in the document or generated by a script.



put some docs...

```json
PUT /my-index-000001/_doc/5
{ 
  "doc1":{
    "name":"red",
    "count":3
  },
  "doc2":{
    "name":"green",
    "count":5
  }
}

PUT /my-index-000001/_doc/6
{ 
  "doc1":{
    "name":"red",
    "count":2
  },
  "doc2":{
    "name":"green",
    "count":8
  }
}
```



Assume you are indexing "my-index-000001" and would like to count the unique number of "doc1.count" that match a query:

```json
GET /my-index-000001/_search
{
  "query": {
    "nested": {
      "path": "doc1",
      "query": {
        "bool": {
          "must": [
            {"match": { "doc1.name": "red" }}
          ]
        }
      }
    }
  },
  "aggs": {
    "docs_count": {
      "nested": {
        "path": "doc1"
      },
      "aggs": {
        "(doc1)s": {
          "terms": {
            "field": "doc1.count"
          },
          "aggs": {
            "unique_(doc1)s": {
              "cardinality": {
                "field": "doc1.count"
              }
            }
          }
        }
      }
    }
  }
}
```



Response:

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "my-index-000001",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.2876821,
        "_source" : {
          "doc1" : {
            "name" : "red",
            "count" : 3
          }
        }
      },
      {
        "_index" : "my-index-000001",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 0.2876821,
        "_source" : {
          "doc1" : {
            "name" : "red",
            "count" : 2
          },
          "doc2" : {
            "name" : "green",
            "count" : 3
          }
        }
      },
      {
        "_index" : "my-index-000001",
        "_type" : "_doc",
        "_id" : "5",
        "_score" : 0.2876821,
        "_source" : {
          "doc1" : {
            "name" : "red",
            "count" : 3
          },
          "doc2" : {
            "name" : "green",
            "count" : 5
          }
        }
      },
      {
        "_index" : "my-index-000001",
        "_type" : "_doc",
        "_id" : "6",
        "_score" : 0.2876821,
        "_source" : {
          "doc1" : {
            "name" : "red",
            "count" : 2
          },
          "doc2" : {
            "name" : "green",
            "count" : 8
          }
        }
      }
    ]
  },
  "aggregations" : {
    "docs_count" : {
      "doc_count" : 4,
      "(doc1)s" : {
        "doc_count_error_upper_bound" : 0,
        "sum_other_doc_count" : 0,
        "buckets" : [
          {
            "key" : "2",
            "doc_count" : 2,
            "unique_(doc1)s" : {
              "value" : 1
            }
          },
          {
            "key" : "3",
            "doc_count" : 2,
            "unique_(doc1)s" : {
              "value" : 1
            }
          }
        ]
      }
    }
  }
}

```





### Bucket Aggregations



#### Terms Aggregation



terms aggs

A multi-bucket value source based aggregation where buckets are dynamically built - one per unique value.





#### Filter Aggregation



Defines a single bucket of all the documents in the current document set context that match a specified filter. Often this will be used to narrow down the current aggregation context to a specific set of documents.





#### Filters Aggregation



Defines a multi bucket aggregation where each bucket is associated with a filter. Each bucket will collect all documents that match its associated filter.





#### Nested Aggregation



A special single bucket aggregation that enables aggregating nested documents.







### Pipeline Aggregation



#### Sum Bucket Aggregation



A sibling pipeline aggregation which calculates the sum across all buckets of a specified metric in a sibling aggregation. The specified metric must be numeric and the sibling aggregation must be a multi-bucket aggregation.



Syntax:

A `sum_bucket` aggregation looks like this in isolation:

```json
{
  "sum_bucket": {
    "buckets_path": "the_sum"
  }
}
```





```json
GET /my-index-000001/_search
{
  "aggs": {
    "ips": {
      "terms": {
        "field": "destination.ip"
      },
      "aggs": {
        "unique_ips": {
          "cardinality": {
            "field": "destination.ip"
          }
        }
      }
    },
    "unique_ips_total": {
      "sum_bucket": {
        "buckets_path": "ips>unique_ips"
      }
    }
  }
}
```

`buckets_path` instructs this sum_bucket aggregation that we want the sum of the `unique_ips` aggregation in the `ips` aggregation.







Most pipeline aggregations require another aggregation as their input. The input aggregation is defined via the `buckets_path` parameter, which follows a specific format:

```ebnf
AGG_SEPARATOR       =  `>` ;
METRIC_SEPARATOR    =  `.` ;
AGG_NAME            =  <the name of the aggregation> ;
METRIC              =  <the name of the metric (in case of multi-value metrics aggregation)> ;
MULTIBUCKET_KEY     =  `[<KEY_NAME>]`
PATH                =  <AGG_NAME><MULTIBUCKET_KEY>? (<AGG_SEPARATOR>, <AGG_NAME> )* ( <METRIC_SEPARATOR>, <METRIC> ) ;
```

SEE MORE ON:

```http
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/search-aggregations-pipeline.html#buckets-path-syntax
```









### Returning the type of the aggregation



Sometimes you need to know the exact type of an aggregation in order to parse its results. The `typed_keys` parameter can be used to change the aggregation’s name in the response so that it will be prefixed by its internal type.



```
"aggs": {
    "NAME": {
      "AGG_TYPE": {}
    }
  }
```

