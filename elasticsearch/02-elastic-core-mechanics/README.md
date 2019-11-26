> for further details see `https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_basic_concepts.html` or the video tutorials in the root README.md

# [Cluster](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_basic_concepts.html)
* a distributed, highly avaliable, and optionally redundant, datastore/search engine that scales horizontally with commodity nodes
* a quorum controls who is actively participating in the cluster and also the election of a cluster leader
* communication is achieved via both REST API and internal Transport Protocol.

return basic cluster health with

```bash
# Cluster health
curl -sk 'http://localhost:9200/_cluster/health?pretty'
{
  "cluster_name" : "docker-cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 26,
  "active_shards" : 26,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 25,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.98039215686274
}
```

check for bottlenecks

```bash
# Thread pools
curl -sk 'http://localhost:9200/_cat/thread_pool?v' #?v&format=json for json-format
node_name name                active queue rejected
dGc_XMg   analyze                  0     0        0
dGc_XMg   fetch_shard_started      0     0        0
dGc_XMg   fetch_shard_store        0     0        0
dGc_XMg   flush                    0     0        0
dGc_XMg   force_merge              0     0        0
dGc_XMg   generic                  0     0        0
dGc_XMg   get                      0     0        0
dGc_XMg   index                    0     0        0
dGc_XMg   listener                 0     0        0
dGc_XMg   management               1     0        0
dGc_XMg   refresh                  0     0        0
dGc_XMg   rollup_indexing          0     0        0
dGc_XMg   search                   0     0        0
dGc_XMg   snapshot                 0     0        0
dGc_XMg   warmer                   0     0        0
dGc_XMg   write                    0     0        0
```
#### status explained

_cluster/health "status"

|color|description|
|-|-|
|green|All primary and replica shards are avaliable|
|yellow|All primary are avaliable|
|red|Not all primary shards are avaliable|

* it is **normal** for elasticsearch to briefly cycle through periods of brief `yellow` and `red` status; generally occurs when
  - new indices are created
  - shards are being rebalanced
  - a node leaves the cluster and the cluster is under a "healing" workload
* **NOT** desired for elasticsearch to persist in a `red` state, indicating a subset of our data is unavaliable
* for "production" environments it is **NOT** desired for a cluster to persist in a `yellow` state, this means we have not our defined replication requirements.  In this state we are at increased risk for data loss, if corruption/loss of the primary shard was to occur
* our single-node test elasticsearch cluster is "yellow" which is normal, and we will learn why a little later

#### troubling signs
in addition to persistant non-green cluster status, the following are also signs of sickness:
* "number_of_nodes" not equal to the expected node count
* "unassigned_shards" greater than zero, persisting (normal for our single-node test cluster)
* "relocating_shards" persistantly non-zero
* increasing "rejected" counters within the thread_pool API

# [Nodes](elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)
* a compute host with disk and storage joined to the cluster
* can be assigbed one or more roles
* a cluster is made up of 1 or more nodes, generally 3 is the smallest production cluster you would make.

by default, every node in the cluster can handle HTTP and Transport traffic
* Transport-layer is used exclusively for communication between nodes and the Java TransportClient
* HTTP-layer is used only by external REST clients (kibana, 3rd party libraries, etc)

cluster membership and node status can be checked with
```bash
curl -sk 'http://localhost:9200/_cat/nodes?v' #?v&format=json for json-format
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           40          29   0    0.01    0.02     0.00 mdi       *      dGc_XMg
```
the master designation (signified by `*`) shows the currently "elected" cluster leader. if you have no elected master, that is bad.
the test cluster has only a single node, which handles three role: master, data, and ingest.

a slightly more complex cluster consisting of 3 masters and 3 data/ingest nodes.  demuxing roles from nodes, allowing them to specialize is a good pattern
```bash
curl -sk 'http://es-master2-test:9200/_cat/nodes?v' #?v&format=json for json-format
ip            heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
192.168.1.97           70         100   6    2.20    1.58     1.49 di        -      es-data3-test
192.168.1.94            7          98   0    0.01    0.05     0.05 m         -      es-master3-test
192.168.1.95           31         100   5    1.69    1.54     1.98 di        -      es-data1-test
192.168.1.92           42          98   5    0.05    0.13     0.14 m         *      es-master2-test
192.168.1.96           55          99   5    1.78    1.48     1.47 di        -      es-data2-test
192.168.1.91            5          98   0    0.12    0.07     0.06 m         -      es-master1-test
```


# [Roles](elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)
* [master eligible](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#master-node) -
  - a node which controls the cluster
  - responsible for cluster state
  - only one master is authoritative at any point in time
  - all masters contribute to cluster quorum
* [data](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#data-node) - nodes that are responsible for persisting our data
* [ingest](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html) - nodes which can apply transformations and enrichment to documents before indexing (slimmed down, in-cluster logstash). good for ingest heavy workloads.
* [cordinating node](elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#coordinating-node) - a node which handles client requests. the first responser when it comes to things like searches, aggregations, etc.  the cordinating node terminates a client request: performing/coordinating the needed operation, appling any reductions/transformation/cleanup, and returning a response. by default every node is implicitly a coordinating node.
* [query](elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#coordinating-node) - a dedicated cordinating node.  serves to shield the load barring Nodes from large and intensive queries which could incapacitate them. when nodes drop out of the cluster you can have a cascading failure, beginning with increased pressure to remaining nodes, which keel over, rinse and repeat until you are very sad.
* [machine learning](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#ml-node) - a node dedicated to execution of machine learning jobs avaliable as a premium feature in x-pack (not covered here)


# [Index](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_basic_concepts.html#_indexs)
* a logical bucket of data

# [Index Alias](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/indices-aliases.html)
* an abstraction layer in front of an index or group indices
* can apply pre-flight query logic to index/indices which are nested by an alias
* can make OPS life easier

# [Document](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_basic_concepts.html#_document)
* a basic unit of information
* tied to an index
* stored in a shard
* structured in a JSON format
* indexed by a unique ID "_id"

a sample document from the bank index
```json
{
      "_index" : "bank",
      "_type" : "account",
      "_id" : "25",
      "_score" : 1.0,
      "_source" : {
        "account_number" : 25,
        "balance" : 40540,
        "firstname" : "Virginia",
        "lastname" : "Ayala",
        "age" : 39,
        "gender" : "F",
        "address" : "171 Putnam Avenue",
        "employer" : "Filodyne",
        "email" : "virginiaayala@filodyne.com",
        "city" : "Nicholson",
        "state" : "PA"
      }
}
```

> we have ignored addressing "_type" because it is deprecated.

# [Shards](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_basic_concepts.html#getting-started-shards-and-replicas)
* a physical bucket of data
* each index is made up of one or more shards
* shards partition data and workload within the distributed cluster
* a shards gets associated with single node
* two types: primary and replica

#### example index shard allocation
set to 3 shards and 2 replicas

|||||
|---|---|---|---|
| s1p  | s2p  | s3p   | the authorative copy of data |
| s1r1  | s2r1  | r3r1   | a secondary copy of data |
| s1r2  | s2r2  | s3r2   | a tertiary copy of data |
||||||

check out the shard layout for the index `bank`
```bash
curl -sk 'http://localhost:9200/_cat/shards/bank?v' | sort #?v&format=json for json-format
bank  0     p      STARTED     197  102kb 127.0.0.1 dGc_XMg
bank  0     r      UNASSIGNED
bank  1     p      STARTED     191   91kb 127.0.0.1 dGc_XMg
bank  1     r      UNASSIGNED
bank  2     p      STARTED     211 99.2kb 127.0.0.1 dGc_XMg
bank  2     r      UNASSIGNED
bank  3     p      STARTED     200 94.8kb 127.0.0.1 dGc_XMg
bank  3     r      UNASSIGNED
bank  4     p      STARTED     201 95.2kb 127.0.0.1 dGc_XMg
bank  4     r      UNASSIGNED
index shard prirep state      docs  store ip        node
```
notice all the `r` replica shards are mark as `UNASSIGNED`, this is because the index has a replication factor of 1, but the cluster only has a single member.  Shard1Replica1 can't live on the same node as Shard1Primary; since we have no second node we can't place (allocate) the replicas.  This is why the test cluster is perpetually in a "yellow" status.  Maximum replica count for an index is `Data node count - 1`.

#### shard types
* **primary shard**
  - N shards in the cluster per configurations; at minimum one
  - multiple primary shards for the same index can exist on the same node; though not optimal as they will compete for resources
  - authorative for data
  - when a primary is lost a replica is promote to primary status, then a new replica is seeded elsewhere
  - writes are funneled through primaries, replicas are updated as needed from primaries
  - primaries can also serve read operations

* **replica shard**
  - a set of replicas are generated and distributed across the cluster according to the replication configuration specified
  - a replica of the same shard cannot exist on the same node as the primary shard
  - a replica cannot exist on the same node as another replica of the same shard
  - each replica set increases reliablity, but decrease avaliable storage. more sets equal increased ingestion and recovery workload
  - having more replicas generally increases search performance
  - replicas serve read/recovery operations
  - an index can have zero replicas if desired

view shard and replication settings for the `bank` index
```bash
curl -sk 'http://localhost:9200/bank/_settings?include_defaults=true&pretty' | jq '[. | to_entries[].value.settings.index | to_entries[] | (select(.key | contains ("number_of")))] | from_entries'
{
  "number_of_shards": "5",
  "number_of_replicas": "1"
}
```
the settings above requires a minimum of 5 nodes, to maintain green healthy cluster.

# Fields
* are namespaces inside a JSON document
* have an associated type
* can be ingested as index-only, index+store, store-only
* can be searched if indexing allows (index-only, index+store)
