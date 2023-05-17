# Exo 3 Querying documents (2)
## Summary
-  Bool Query : combine multiple boolean conditions 
- Clauses : must, must_not,should...
- Good practices

## [Bool Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html)

- Pipelines that combines mutliple conditions
- Clauses :
- Must (hard condition)
- Must not (hard condition)
- Should (preferably from)
- Query (score : how good is the match?) vs Filter (Yes or NO : Does it match?) (Regression vs Classification)

 ```
GET comments/_search 
{"query":{
  "bool":{
    "must" :[
      {
        "match":{
          "title" : "Brown"
        }
      },
      {
        "range":{
          "date":{
            "lt":"now-2h"
          }
        }
      }
      ],
    "must_not":{
        "match":{
          "_id":5
        }
      },
      "should":{
        "match_phrase":{
          "title": "an Actor"
        }
        
      },
      "filter": {
        "range":{
        "date" : {
          "gte":"2000-06-12"
        }
        }
      }
    }
  }
}
 ```
## Good practices 
- `should` clauses precise the result (higher score) but do not reduce the scope of the search
- so we can pile them up to rise the quality of the result ranking

 ```
 GET comments/_search
{
  "query": {
    "bool": {
      "must": {
        "match":{
          "title":"the"
        }
      },
      "should":[
        {"match": {"title":"Brown"}},
        {"match": {"title":"new"}}
        ]
      }
    }
  }
 ```
- `minimum_should_match` to control the number of  should clauses must be met
- At least 1 of the conditions must be met

```
GET comments/_search
{
  "query": {
    "bool": {
      "must": {
        "match":{
          "title":"the"
        }
      },
      "should":[
        {"match": {"title":"Brown"}},
        {"match": {"title":"new"}},
         {"match": {"title":"Yellow"}}
        ],
        "minimum_should_match": 1
      }
    }
  }
 ```
 - Bool Query with only `Should` condition : at least one has to be met

```
## the same thing  as "minimum_should_match": 1
GET comments/_search
{
  "query": {
    "bool": {
      "should":[
        {"match": {"title":"Brown"}},
        {"match": {"title":"new"}},
         {"match": {"title":"Maths"}}
        ]
      }
    }
  }
```
- Should not = **nested** bool queries : bool :should : bool must_not
```
#comments with Brown but not Actor
GET comments/_search
{
  "query": {
    "bool": {
      "must": {"match": {"title": "Brown"}},
      "should": {
        "bool": {
          "must_not": {
            "match": {"title": "Actor"}
          }
        }
      }
    }
  }
}
 ```
- Combine `must` + `match` and `should` + `match_phrase` : Widen Result + Higher Precision
- Search Tip for Phrases (multiple words) : `should`+ `match_phrase`
 ```
GET comments/_search
{
  "query": {
    "bool": {
      "must": {"match": {"title": "Brown Red Maths"}
      },
      "should": {
        "match_phrase": {
          "title": {
            "query":"Brown famous Actor",
            "slop":4
          }
        }
      }
    }
  }
}
 ```
