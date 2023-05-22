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
-
