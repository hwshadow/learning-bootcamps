# Query exampless

## match_all
The simplest query you can execute; it will match everything, in lucene you use a `wildcard` character to make this
> lucene style |
> `*` represents a wildcard, notice the field selector is absent

```bash
*
```

> dsl-style | `{}` a blank object also works or a blank payload

```json
{"query" : {"match_all" : {}}}
```

```bash
curl -H "Content-Type: application/x-ndjson" -XGET 'localhost:9200/_msearch?pretty' --data-binary '
{"index" : "shakespeare"}
{"size":0, "query" : {"match_all" : {}}}
'
#result - matched 112k documents, we omit them with size 0
{
  "responses" : [
    {
      "took" : 1,
      "timed_out" : false,
      "_shards" : {
        "total" : 5,
        "successful" : 5,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : 111396,
        "max_score" : 0.0,
        "hits" : [ ]
      },
      "status" : 200
    }
  ]
}
```
## field-based query
Getting a little more advanced, we can search specific fields

> lucene style |

```bash
#fieldName:desiredValue
text_entry:"cross"
```

> dsl-style |

```json
{ "term":  { "text_entry": "cross" }}
```

```bash
curl -H "Content-Type: application/x-ndjson" -XGET 'localhost:9200/_msearch?pretty' --data-binary '
{"index" : "shakespeare"}
{"size":1, "query" : { "term":  { "text_entry": "cross" }} }}
'
#result - matched 76 documents, we asked for 1 returned
{
  "responses" : [
    {
      "took" : 27,
      "timed_out" : false,
      "_shards" : {
        "total" : 5,
        "successful" : 5,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : 76,
        "max_score" : 10.55878,
        "hits" : [
          {
            "_index" : "shakespeare",
            "_type" : "doc",
            "_id" : "104512",
            "_score" : 10.55878,
            "_source" : {
              "type" : "line",
              "line_id" : 104513,
              "play_name" : "Twelfth Night",
              "speech_number" : 28,
              "line_number" : "3.4.48",
              "speaker" : "OLIVIA",
              "text_entry" : "Cross-gartered!"
            }
          }
        ]
      },
      "status" : 200
    }
  ]
}
```

## multiple conditions
We can chain together multiple search-criteria by

> lucene style |
>
```
(text_entry:"cross" AND NOT text_entry:who) OR text_entry:"never"
```

> dsl-style |

```json
{"query_string":{"query":"(text_entry:\"cross\" AND NOT text_entry:who) OR text_entry:\"never\""}}
```

```bash
curl -H "Content-Type: application/x-ndjson" -XGET 'localhost:9200/_msearch?pretty' --data-binary '
{"index" : "shakespeare"}
{"size":1, "query" : {"query_string":{"query":"(text_entry:\"cross\" AND NOT text_entry:who) OR text_entry:\"never\""} }}
'
#result - matched 1048 documents, we asked for 3 returned
{
  "responses" : [
    {
      "took" : 6,
      "timed_out" : false,
      "_shards" : {
        "total" : 5,
        "successful" : 5,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : 1048,
        "max_score" : 13.381853,
        "hits" : [
          {
            "_index" : "shakespeare",
            "_type" : "doc",
            "_id" : "66785",
            "_score" : 13.381853,
            "_source" : {
              "type" : "line",
              "line_id" : 66786,
              "play_name" : "Merry Wives of Windsor",
              "speech_number" : 11,
              "line_number" : "5.5.35",
              "speaker" : "FALSTAFF",
              "text_entry" : "never else cross me thus."
            }
          },
          {
            "_index" : "shakespeare",
            "_type" : "doc",
            "_id" : "62210",
            "_score" : 12.583748,
            "_source" : {
              "type" : "line",
              "line_id" : 62211,
              "play_name" : "Merchant of Venice",
              "speech_number" : 18,
              "line_number" : "2.4.38",
              "speaker" : "LORENZO",
              "text_entry" : "And never dare misfortune cross her foot,"
            }
          },
          {
            "_index" : "shakespeare",
            "_type" : "doc",
            "_id" : "104512",
            "_score" : 10.55878,
            "_source" : {
              "type" : "line",
              "line_id" : 104513,
              "play_name" : "Twelfth Night",
              "speech_number" : 28,
              "line_number" : "3.4.48",
              "speaker" : "OLIVIA",
              "text_entry" : "Cross-gartered!"
            }
          }
        ]
      },
      "status" : 200
    }
  ]
}
```

## boolean filters (implementation of kibana filter)
This is another way to do multiple conditions, offers better performance

> lucene style |  DOES NOT APPLY

> dsl-style |

```json
{
  "bool": {
    "must": [
      {
        "query_string": {
          "query": "*"
        }
      },
      {
        "match_phrase": {
          "text_entry": {
            "query": "never"
          }
        }
      },
      {
        "match_phrase": {
          "text_entry": {
            "query": "cross"
          }
        }
      }
    ],
    "filter": [],
    "should": [],
    "must_not": [
      {
        "match_phrase": {
          "text_entry": {
            "query": "dare"
          }
        }
      }
    ]
  }
}
```

```bash
curl -H "Content-Type: application/x-ndjson" -XGET 'localhost:9200/_msearch?pretty' --data-binary '
{"index" : "shakespeare"}
{"size":1, "query" : {"bool":{"must":[{"query_string":{"query":"*"}},{"match_phrase":{"text_entry":{"query":"never"}}},{"match_phrase":{"text_entry":{"query":"cross"}}}],"filter":[],"should":[],"must_not":[{"match_phrase":{"text_entry":{"query":"dare"}}}]}} }}
'
#result - matched 1 document, we asked for 1 returned
{
  "responses" : [
    {
      "took" : 7,
      "timed_out" : false,
      "_shards" : {
        "total" : 5,
        "successful" : 5,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : 1,
        "max_score" : 14.381853,
        "hits" : [
          {
            "_index" : "shakespeare",
            "_type" : "doc",
            "_id" : "66785",
            "_score" : 14.381853,
            "_source" : {
              "type" : "line",
              "line_id" : 66786,
              "play_name" : "Merry Wives of Windsor",
              "speech_number" : 11,
              "line_number" : "5.5.35",
              "speaker" : "FALSTAFF",
              "text_entry" : "never else cross me thus."
            }
          }
        ]
      },
      "status" : 200
    }
  ]
}
```

# Beware the mappings !!!
Take the `modsec` index for example, it has a somewhat complicated mapping.
```bash
#Display mapping
jq '.' ./01-elastic-sample-data/modsec_mapping.json
#Display sample document
jq '.' ./01-elastic-sample-data/modsec_normalized.json
```

Lets try and find the sample document in elasticsearch.  Say we are interested in the `request` field and we want to find the substring `%645&vars`, which clearly exists in this document.

## Attempt #1
> lucene style |

```bash
request:%645&vars
```

> dsl-style |

```json
{"bool":{"must":[{"query_string":{"query":"request:%645&vars","analyze_wildcard":true,"default_field":"*"}},{"range":{"@timestamp":{"gte":1565670037521,"lte":1574096829530,"format":"epoch_millis"}}}],"filter":[],"should":[],"must_not":[]}}
```

```bash
curl -H "Content-Type: application/x-ndjson" -XGET 'localhost:9200/_msearch?pretty' --data-binary '
{"index" : "shakespeare"}
{"size":1, "query":{ "bool":{"must":[{"query_string":{"query":"request:%645&vars","analyze_wildcard":true,"default_field":"*"}},{"range":{"@timestamp":{"gte":1565670037521,"lte":1574096829530,"format":"epoch_millis"}}}],"filter":[],"should":[],"must_not":[]} }}
'
#result - matched 0 documents, we asked for 1 returned
{
  "responses" : [
    {
      "took" : 1,
      "timed_out" : false,
      "_shards" : {
        "total" : 5,
        "successful" : 5,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : 0,
        "max_score" : null,
        "hits" : [ ]
      },
      "status" : 200
    }
  ]
}
```

Hmmmm why didn't that work?  Let's look at the mapping
```bash
curl -sk 'localhost:9200/modsec/_mapping?pretty' | jq '.modsec.mappings.doctype.properties | .request'
{
  "type": "text",
  "index": false
}
```
Ahhh, the field is not indexed which means it is not searchable...