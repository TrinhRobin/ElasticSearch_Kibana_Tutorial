# [Mappings](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)& [Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html)
- **Inverted** Index and Analysis
- [ Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html) a document's fields : data types
- Multi-fields for indexing a field in multpile ways
- Either **Dynamic** or **Custom** Mapping
- Create a custom Analyzer and use it with the `_analyze` API 
## Mappings

- **Mapping** is appling a Schema Definition => Data types, Names and Storage to the **Fields**
- Possible data types : text, keyword, date, int,float, boolean, IP address

```
GET my_blogs/_mapping
GET comments/_mapping
```

- the `properties` key contains the data types of a document's fields

## Dynamic mapping vs Explicit Mapping
- We can define our own mapping with the Index API
```
PUT /my-index-mapping
{
  "mappings": {
    "properties": {
      "year":    { "type": "integer" },  
      "country":  { "type": "keyword"  }, 
      "title":   { "type": "text"  }     
    }
  }
}
```
- Add a new field to a mapping

```
PUT /my-index-mapping/_mapping
{
  "properties": {
    "address-location": {
      "type": "geo_point",
      "index": false
    }
  }
}

GET /my-index-mapping/_mapping
```
- Get the mapping for a specific field 
```
GET /my-index-mapping/_mapping/field/address-location
```

## Inverted Index

- **Elastic Search** is not case sensitive upper/ sensible to lowercase
- Thanks to the **inverted index**
- Like in a book important word are paired with pages'numbers : 
- Long text is broken into tokens
‒ Indexed by a unique document id
- Same index for all tokens : lowercase, stemming
- We can remove stop words from the inverted index

## Analyzer

- Analysis is converting text into the inverted index
- There are already some Built-In Analyzers : 
- `standard` : no character filters
- `simple` : breaks the text whenever a word is not a letter
- `keyword``

```
GET _analyze
{
"analyzer": "simple",
"text": "How to configure ingest nodes?"
}

GET /_analyze
{
  "analyzer" : "standard",
  "text" : "How to hunt quick Brown Foxes?"
}
```
- `text` can be a list to get **Multi Value** Field

```
GET /_analyze
{
  "analyzer" : "standard",
  "text" : ["Foxes are generally smaller ", "Fox features typically include a triangular face"]
}
```
## Custom Analyzer 
- Based on the filter parameter 
- three levels of choice for customization :
- token filters,
- character filters
- tokenizer

```
GET /_analyze
{
  "tokenizer" : "whitespace",
  "filter" : ["lowercase", {"type": "stop", "stopwords": ["a", "is", "this"]}],
  "text" : "A fox's coat color and texture may vary due to the change in seasons"
}
```
- Build our own analyzer :

```
PUT create_my_analyzer
{
  "settings": {
    "analysis": {
      "filter": {
        "my_stopwords": {
          "type": "stop",
          "stopwords": ["a","and","or", "this", "that", "the", "is","are"]
        }
      },
      "analyzer": {
        "my_content_analyzer": {
          "type": "custom",
          "char_filter": [],
          "tokenizer": "standard",
          "filter": ["lowercase","my_stopwords"]   
          }
        }
      
    }
    
  }
}
```
- Testing the custom Analyzer :
```
POST create_my_analyzer/_analyze
{
"text": ["True hope is swift, and flies with swallow's wings",
"Kings it makes gods, and meaner creatures kings.",
"Conscience is but a word that cowards use", 
"Devised at first to keep the strong in awe."],
"analyzer": "my_content_analyzer"
}
```
- Be careful with the words' order
```
GET create_my_analyzer/_analyze
{
"tokenizer": "standard",
"filter": ["lowercase", "my_stopwords" ],
"text": ["This is fine", "what is that ?"]
}
```
```
{
  "tokens": [
    {
      "token": "fine",
      "start_offset": 8,
      "end_offset": 12,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "what",
      "start_offset": 13,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 103
    }
  ]
}
```
```
GET create_my_analyzer/_analyze
{
"tokenizer": "standard",
"filter": [ "my_stopwords", "lowercase" ],
"text":["This is fine", "what is that ?"]
}
```

```
{
  "tokens": [
    {
      "token": "this",
      "start_offset": 0,
      "end_offset": 4,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "fine",
      "start_offset": 8,
      "end_offset": 12,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "what",
      "start_offset": 13,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 103
    }
  ]
}
```
