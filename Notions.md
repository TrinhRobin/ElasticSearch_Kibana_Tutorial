# Summary :
- Elastic Search has an distributed architecture based on cluster/slave-master nodes
- Elastic Search uses **shards** to divide and scale the data of an **index**
- Each shard consists of each of one primary and many replicas
- Query consists of a **query** phase (operated on each shard separatly) and a **fetch** phase (aggregation of local results)

# Nodes
## Nodes Role : 
- Master eligible ?
- Main Roles : Data, Ingest, Coordinating, Machine Learning
- Get the Cluster **State** : nodes in the cluster, indices, mappings, settings
```
GET _cluster/state
```
- Get the Cluster **Health**

```
GET _cluster/health
```
Output :
```
{
  "cluster_name": "docker-cluster",
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 34,
  "active_shards": 34,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 16,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 68
}
```
- `status` : green (all shards allocated) , yellow (all primaries allocated but >1 replica isn't) or red (>1 primary shard not allocated), 
- exists at shard, index and cluster level
- Shards Allocation : shards can have several states : unassigned, initializing, active, relocating

- Get The Health of a specific index:
```
GET _cluster/health/{index_name}
```

## Configuring Node Role
- In the `elasticsearch.yml` file
- Parameters : `node.master`, `node.ingest`, `node.data`
- Master Nodes : avoid split brain

-  Recommandation for productions:
- Setting the parameters `master-eligible` to 3 and `minimum_master_nodes`=2
## Data Nodes
- Contains Shards for storing documents
- Execute CRUD, Search and aggregation operations on the data
- All nodes are data nodes by default
## Coordinating Nodes
- receives and handles a specific client request
- forwards the request to other relevant nodes, then combines those results into a single result
 
# Shard
- A [shard](https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html) is a worker unit that disitributes data to the cluster's nodes
- An index can be seen as a pointer that refers multiple shards 
-  An indes is split into shards before indexing of any document
-  Documents are replicated thanks to 2 types of shards :
-  **Primary** (original shards of an index)
-  **Replicas** (copies)
-  A Primary and its Replicas are stored on different nodes
-  The node master decides on the distribution of shards 
-  For instance it can redistribute the shards if new nodes are added to the cluster
-  For an index, the number primary shards is by default set to 5
-  Number of shards can be changed but costly
-  Number of replicas can be adapt at any given time :
```
PUT my_index
{
 "settings": {
  "number_of_shards": 3,
  "number_of_replicas": 2
  }
}
```
- Replicas advantages :
- High availability (Promotion of Replica if needed)
- Better Scalability : Read throughput (queries can be performed on both primary and replicas shard

## How to determine the number of shards ?
- heavy indexing calls for higher number of primary shards
- heavy search traffic = increase the number of replicas
- large dataset = more primary shards (each shard around 10-40 GB)
- small dataset = one shard index

## Document Routing
 - Formula for determining which shard to index the document to :
```
PUT blogs/_doc/551
{
}
shard = hash(_routing) % number_of_primary_shards
```
# Elastic Search Response :
- **CRUD** operations return `shard` information :
```
"_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  }
```
- `total` : this operation will be executed on this amout of shards copies (both primary & replicas)
-  `successful` : number of shard copies with sucessful index operations
-  `failed` : idem with failed operations
-  `failures` : array with description of the error's root

- Same response body for the `_search` API:
```
 "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
```
- `skipped` : number of shard that avoided the search execution because they the data stored inside them is not relevant for this query

# Search & Scoring

- Search = Query then fetch :
- doc identifiers and local results are performed for each shard
- and then aggregated/merged to get global/final result

## Computing the `_score` :
- Term Frequency (TF) is a document-level information, so it's a local value
-  whereas IDF is a global value
- by default `_score` computes a local IDF (more efficient)
- it's hardly a problem on large datasets
- or if the documents are well distributed acroos the multiple shards
- To get the Global IDF (costly) you may use the `search_type=dfs_query_then_fetch` parameter :
```
GET comments/_search?search_type=dfs_query_then_fetch
```

## Cluster Allocation Explain API :
- Show informations of the first unnassigned shard (status=yelllow) found 
```
GET _cluster/allocation/explain
```

