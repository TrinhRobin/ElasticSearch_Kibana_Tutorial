# Exo 2 Querying documents 
## Summary
- Get for Retrieving a Document
- Ranking the results according to their relevancy => score
- precision, recall, ranking
- Query DSL > Query 
- Match Query (equality test)
- Range Query :  documents with fields that fall in a given range
 
## GET
```
GET _search
GET my_blogs/_doc/1
GET comments/_doc/3
GET comments
```

## Multiple gets at once : [mget](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-multi-get.html)
```
GET /_mget
{
  "docs": [
    {
      "_index": "comments",
      "_id": "1"
    },
    {
      "_index": "comments",
      "_id": "2"
    },
     {
      "_index": "comments",
      "_id": "3"
    },
      {
      "_index": "my_blogs",
      "_id": "3"
    }
  ]
}
```
## `_Search` Endpoint : getting all the hits
```
GET my_blogs/_search
GET comments/_search
```

## Queries
- Relevancy of the results is based around three notions :
-  Precision (TP/(FP+TP)) : %/rate of irrelevant documents
-  Recall = (TP/(TP+FN))  (can we find all documents of the search?)
-  Ranking of the documents returns =>score

- Query String (URL, easy to use, sensible à la casse,) vs Query DSL (json, body, powerful)

## [Match Queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html)
### String Query : Simple and Brief
```
GET comments/_search?q=title:(Crossroads)
```
## DSL Query : Powerful but Complex
```
GET comments/_search
{
  "query" : {
    "match":{
      "title":"Crossroads" 
    }
  }
}
GET comments/_search
{
  "query" : {
    "match":{
      "title":"Crossroads Maths" 
    }
  }
}
##Response :hits -> total :1 , hits are returned by decreasing order
```
- Match with the `and` operator
- for String Query
```
GET comments/_search?q=title:(Crossroads AND fine )
```
- for DSL Query
```
GET comments/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Maybe Crossroads",
        "operator": "and"
      }
    }
  }
}

GET comments/_search
{
  "query": {
    "match": {
      "title": {
        "query": "God Crossroads",
        "operator": "and"
      }
    }
  }
}
```
- Match Options with `minimum_should_match` : Set the Minimum number of clauses that must match for a document to be returned
```
GET comments/_search
{
  "query": {
    "match": {
      "title": {
        "query": "God Crossroads Maths fine",
        "minimum_should_match": 2
      }
    }
  }
}
```
- Search for Synonyms :
```
GET /_search
{
   "query": {
       "match" : {
           "title": {
               "query" : "Math",
               "auto_generate_synonyms_phrase_query" : false
           }
       }
   }
}
```

## Match phrases
- `Match phrases`  selects documents with all the terms and in the same order
- Costly to use
```
GET comments/_search?q=title:"did maths"
```
```
GET comments/_search
{
  "query": {
    "match_phrase":{
    "title" : "did Maths" 
    }
  }
}
```
- `Slop` Parameter : maximal distance between the words 
- String Query
```
GET comments/_search?q=title:"maths wrong"~2
```
- DSL Query :
```
GET comments/_search
{
  "query": {
    "match_phrase":{
      "title" : {
        "query":"Maths wrong",
        "slop":1
      }
    }
  }
}

GET comments/_search
{
  "query": {
    "match_phrase":{
      "title" : {
        "query":"Maths wrong",
        "slop":2
      }
    }
  }
}
```

## Dateime functions
```
POST _bulk
{ "index" : { "_index" : "comment", "_id" : "10" } }
{ "title" : "Brown is the new Red","date":"2022-01-02" }
{ "index" : { "_index" : "comment", "_id" : "11" } }
{ "title" : "Brown is  becoming famous as an Actor","date":"2022-03-15" }
{ "index" : { "_index" : "comment", "_id" : "12" } }
{ "title" : "read the Docs you moron","date":"2019-11-22" }
{ "index" : { "_index" : "comment", "_id" : "13" } }
{ "title" : "The beginning of a new era","date":"1999-01-01" }
 ```
-  Dates can be specified as strings
- `Range` Query for continous fields : greater than `gte` or lower than `lt`
```
GET comments/_search
{
  "query": {
    "range":{
      "date":{
        "gte": "2022-01-01",
        "lt":"2023-04-05"
        
        }
      }
    }
  }
 ```
- Date math expressions for computations + or - 
- `y`(year) `M` (month) `w` (week) `h` (hour) `m` (minutes)

```
GET comments/_search
{
  "query": {
    "range":{
      "date":{
        "gte": "now-2y-2M+1w-1h"
        }
      }
    }
  }
  ```
