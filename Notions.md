# Nodes
## Nodes Role : 
- Master eligible ?
- Main Roles : Data, Ingest, Coordinating, Machine Learning
- Get the Cluster State : nodes in the cluster, indices, mappings, settings
```
GET _cluster/state
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

## Document Routing
 - Formula for determining which shard to index the document to :
```
PUT blogs/_doc/551
{
}
shard = hash(_routing) % number_of_primary_shards
```
