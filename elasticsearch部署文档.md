**官方文档**

https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html

### 1.安装

#### es版本

elasticsearch-7.8.0

国内镜像下载地址：https://mirrors.huaweicloud.com/elasticsearch/7.8.0/

#### 可视化工具

##### kibana

国内镜像下载地址：https://mirrors.huaweicloud.com/kibana/7.8.0/

##### ElasticHD

##### elasticsearch-head插件

https://github.com/mobz/elasticsearch-head



### 2.配置

#### elasticsearch.yml

指定名字、路径

```yml
#集群名
cluster.name: elasticsearch_production
#节点名
node.name: node-1
#对所有ip访问做出响应
network.host: 0.0.0.0
#端口默认9200
http.port: 9200
#集群master节点
cluster.initial_master_nodes: ["node-1"]
#数据
path.data: /path/to/data1,/path/to/data2 
#日志路径
path.logs: /path/to/logs
# 插件
path.plugins: /path/to/plugins
```





#### JVM

jvm.options

修改初始化、最大堆内存大小，默认1G

-Xms1g
-Xmx1g



### 3.数据容量



| doc数量 | 3分片  |
| ------- | ------ |
| 5W      | 5.4m   |
| 6W      | 6.4m   |
| 9W      | 9.9m   |
| 10w     | 11m    |
| 15W     | 16.2mb |



每1W个 doc大约1.1m, waybill 单个doc大约0.11k，按照公司理想的业务增长情况**每天100W个运单**，一年产成**3.6亿个waybill doc**约40G数据5年达到200G。每个分片容量30-50G，不超过50G，单个节点**5个分片**磁盘空间250G。

按照上述情况规划**每天100W个运单** 5个分片5年内不需要扩容，若每天10W个运单左右3个分片足够



### 4.监控

#### 健康

```bash
GET _cluster/health
```



和 Elasticsearch 里其他 API 一样，`cluster-health` 会返回一个 JSON 响应。这对自动化和告警系统来说，非常便于解析。响应中包含了和集群有关的一些关键信息：

```js
{
   "cluster_name": "elasticsearch_zach",
   "status": "green",
   "timed_out": false,
   "number_of_nodes": 1,
   "number_of_data_nodes": 1,
   "active_primary_shards": 10,
   "active_shards": 10,
   "relocating_shards": 0,
   "initializing_shards": 0,
   "unassigned_shards": 0
}
```



响应信息中最重要的一块就是 `status` 字段。状态可能是下列三个值之一：

- **`green`**

  所有的主分片和副本分片都已分配。你的集群是 100% 可用的。

- **`yellow`**

  所有的主分片已经分片了，但至少还有一个副本是缺失的。不会有数据丢失，所以搜索结果依然是完整的。不过，的高可用性在某种程度上被弱化。如果 *更多的* 分片消失，就会丢数据了。把 `yellow` 想象成一个需要及时调查的警告。

- **`red`**

  至少一个主分片（以及它的全部副本）都在缺失中。这意味着在缺少数据：搜索只能返回部分数据，而分配到这个分片上的写入请求会返回一个异常。

  

  `green`/`yellow`/`red` 状态是一个概览的集群并了解眼下正在发生什么的好办法。集群的状态概要：

  - `number_of_nodes` 和 `number_of_data_nodes` 这个命名完全是自描述的。
  - `active_primary_shards` 指出集群中的主分片数量。这是涵盖了所有索引的汇总值。
  - `active_shards` 是涵盖了所有索引的_所有_分片的汇总值，即包括副本分片。
  - `relocating_shards` 显示当前正在从一个节点迁往其他节点的分片的数量。通常来说应该是 0，不过在 Elasticsearch 发现集群不太均衡时，该值会上涨。比如说：添加了一个新节点，或者下线了一个节点。
  - `initializing_shards` 是刚刚创建的分片的个数。比如，当刚创建第一个索引，分片都会短暂的处于 `initializing` 状态。这通常会是一个临时事件，分片不应该长期停留在 `initializing` 状态。还可能在节点刚重启的时候看到 `initializing` 分片：当分片从磁盘上加载后，它们会从 `initializing` 状态开始。
  - `unassigned_shards` 是已经在集群状态中存在的分片，但是实际在集群里又找不着。通常未分配分片的来源是未分配的副本。比如，一个有 5 分片和 1 副本的索引，在单节点集群上，就会有 5 个未分配副本分片。如果集群是 `red` 状态，也会长期保有未分配分片（因为缺少主分片）。

### 5.部署



#### 内存

12G

单独针对waybill而言目前需求检索消耗内存， 没有分词、聚合操作，单个文档大小 只有0.1Kb左右。



##### 预留内存的（少于）一半给lucene

Lucene 的性能取决于和操作系统的相互作用。如果把所有的内存都分配给 Elasticsearch 的堆内存，将不会有剩余的内存交给 Lucene。 这将严重地影响全文检索的性能



##### 少于32G内存分配给Elasticsearch

JVM 在内存小于 32 GB 的时候会采用一个内存对象指针压缩技术。分配超过32G内存，每个对象的指针都变长了，就会使用更多的 CPU 内存带宽，实际上失去了更多的内存。当内存到达 40–50 GB 的时候，有效内存才相当于使用内存对象指针压缩技术时候的 32 GB 内存。



##### 禁用 swap

内存交换 到磁盘对服务器性能来说是 *致命* 的。一个内存操作必须能够被快速执行。

如果内存交换到磁盘上，一个 100 微秒的操作可能变成 10 毫秒。 再想想那么多 10 微秒的操作时延累加起来。 不难看出 swapping 对于性能是多么可怕。

禁用swap，需要打开配置文件中的 `mlockall` 开关。 它的作用就是允许 JVM 锁住内存，禁止操作系统交换出去。在 `elasticsearch.yml` 文件中，设置如下：

```yaml
bootstrap.mlockall: true
```



### 创建index

#### waybill

针对每个index做具体的分片规划

number_of_shards		分片数

number_of_replicas	  副本数

创建index的put请求

```json

PUT waybill
{
    "settings" : {
        "index" : {
            "number_of_shards" : 5, 
            "number_of_replicas" : 0
        }
    }
}
```

