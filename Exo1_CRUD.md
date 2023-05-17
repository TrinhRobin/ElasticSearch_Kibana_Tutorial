# Tutorial First Exercice : CRUD Operations

## [Create Index](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)
- In the Console : Automatically create the index

- Syntaxe : index_name/endpoint/document_id
```
PUT my_blogs/_doc/1
{
"title": "fighting common sense with Elastic Search",
"category" : "Comedy",
  "author": {"first_name":"Robin","last_name" : "Trinh"}
}
#If Same index => reindex the document and overwrites the old one
PUT my_blogs/_doc/1
{
"title": "Elastic Search strikes again",
"category" : "Action",
"Date" : "2023-05-17",
  "author": {"first_name":"Robin","last_name" : "Trinh","company":"P"}
}
##RESPONSE:  "_version": 2,
 # "result": "updated",
 ```
 ## Create
 - `_Create` Endpoint so as not avoid overwriting

 ```
PUT my_blogs/_create/1
{
"title": "Elastic Search strikes again",
"category" : "Action",
"Date" : "2023-05-17",
  "author": {"first_name":"Robin","last_name" : "Trinh","company":"P"}
}
#Response : "status": 409 , "error":  "reason": "[1]: version conflict, document already exists (current version [2])",
```

- [Index vs Create Actions](https://stackoverflow.com/questions/34572878/elasticsearch-bulk-api-index-vs-create-update)

## [Update](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)

- `_Update` Endpoint : you have to add the `doc` context and precise the updated informations
- PUT method not allowed
```
POST my_blogs/_update/1
{
  "doc":{
    "title": "Elastic Search strikes again",
    "category" : "Action",
    "Date" : "2023-05-17"
  }
}
##Result : {
##  "_index": "my_blogs",
##  "_id": "1",
##  "_version": 2,
##  "result": "noop",
## "_shards": {
##    "total": 0,
##    "successful": 0,
##    "failed": 0
##  },
##  "_seq_no": 1,
##  "_primary_term": 1
##}
```

## [Delete](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete.html)
```
DELETE /my_blogs/_doc/1 
##Result:   "_version": 3,  "result": "deleted",
```
## [Bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html) 
- Many Operations with a single API Call (batch processing for log events)
- Chaining 4 Operations: create, index, update and delete
```
 POST _bulk
{ "index" : { "_index" : "comments", "_id" : "1" } }
{"title": "Why Bridges are inherently superior to crossroads", "category": "Engineering"}
{ "index" : { "_index" : "comments", "_id" : "2" } }
{"title": "the devil itself Elastic Stack  Released", "category": "Releases"}
{ "delete" : { "_index" : "comments", "_id" : "2" } }
{ "create" : { "_index" : "comments", "_id" : "3" } }
{"title": "Maths did nothing wrong", "category": "User Stories"}
{ "update" : {"_id" : "1", "_index" : "comments"} }
{ "doc" : {"title" : "Maybe Crossroads were fine after all"} }
#index action is followed by a document contrary to the delete action
 ```
