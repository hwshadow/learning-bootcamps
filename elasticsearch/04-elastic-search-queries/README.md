# Interfaces avaliable for search
Elasticsearch is built on top of lucene.  In generally a simple lucene query looks like this
```
fieldName:"termToSearchFor"
```
We will explore the power of lucene a little later.

There are 4 common methods of executing queries against Elasticsearch (outside of GUIs like kibana and grafana):
## [URI search](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/search-uri-request.html) via Lucene query
 - `q` parameter is a URL-encoded Lucene query
 - `size` parameter sets the max documents returned (up to 10k)
 - `sort` parameter (not shown) is useful for sorting, in the form of `fieldName:asc/fieldName:desc`
 - good for quick and simple lucene queries

```bash
curl -XGET 'localhost:9200/shakespeare/_search?size=1&q=text_entry:"arms"&pretty'

#result
{
  "took" : 14,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 259,
    "max_score" : 8.286975,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_type" : "doc",
        "_id" : "51254",
        "_score" : 8.286975,
        "_source" : {
          "type" : "line",
          "line_id" : 51255,
          "play_name" : "King Lear",
          "speech_number" : 21,
          "line_number" : "3.6.55",
          "speaker" : "KING LEAR",
          "text_entry" : "Arms, arms, sword, fire! Corruption in the place!"
        }
      }
    ]
  }
}
```


## [Request Body search](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/search-request-body.html) via Lucene query or Query DSL
- great for complex and lengthy queries that would require extensive url-encoding/escaping

```bash
curl -XGET -H 'Content-type: application/json' 'localhost:9200/shakespeare/_search?pretty' -d '
{
  "size": 1,
    "query" : {
        "term" : {
          "text_entry": "arm"
        }
    }
}'
# query dsl style

#     "query" : {
#         "query_string" : {
#           "query": "text_entry:arm"
#         }
#     }
# }'
# lucene style (note the query_string block)

#result
{
  "took" : 9,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 132,
    "max_score" : 9.623348,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_type" : "doc",
        "_id" : "3914",
        "_score" : 9.623348,
        "_source" : {
          "type" : "line",
          "line_id" : 3915,
          "play_name" : "Henry VI Part 1",
          "speech_number" : 15,
          "line_number" : "2.1.41",
          "speaker" : "Sentinels",
          "text_entry" : "Arm! arm! the enemy doth make assault!"
        }
      }
    ]
  }
}
```

## [Multisearch API](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/search-multi-search.html) via Lucene query or Query DSL
- great for numberous queries, queries are executed in bulk


```bash
curl -H "Content-Type: application/x-ndjson" -XGET 'localhost:9200/_msearch?pretty' --data-binary '
{"index" : "shakespeare"}
{"size":1,"query":{"term":{"text_entry":"arm"}}}
{"index" : "shakespeare"}
{"size":1,"query":{"query_string":{"query":"text_entry:make"}}}
'

#results
{
  "responses" : [
    {
      "took" : 3,
      "timed_out" : false,
      "_shards" : {
        "total" : 5,
        "successful" : 5,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : 132,
        "max_score" : 9.623348,
        "hits" : [
          {
            "_index" : "shakespeare",
            "_type" : "doc",
            "_id" : "3914",
            "_score" : 9.623348,
            "_source" : {
              "type" : "line",
              "line_id" : 3915,
              "play_name" : "Henry VI Part 1",
              "speech_number" : 15,
              "line_number" : "2.1.41",
              "speaker" : "Sentinels",
              "text_entry" : "Arm! arm! the enemy doth make assault!"
            }
          }
        ]
      },
      "status" : 200
    },
    {
      "took" : 5,
      "timed_out" : false,
      "_shards" : {
        "total" : 5,
        "successful" : 5,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : 1575,
        "max_score" : 6.609525,
        "hits" : [
          {
            "_index" : "shakespeare",
            "_type" : "doc",
            "_id" : "87935",
            "_score" : 6.609525,
            "_source" : {
              "type" : "line",
              "line_id" : 87936,
              "play_name" : "Romeo and Juliet",
              "speech_number" : 9,
              "line_number" : "4.4.18",
              "speaker" : "CAPULET",
              "text_entry" : "Make haste, make haste."
            }
          }
        ]
      },
      "status" : 200
    }
  ]
}
```

## [Scroll query](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/search-request-scroll.html)
- used to return large data sets, over 10k documents
- kind of a pain to execute from bash
- 3rd-party library contain 'sugar' which make these type of queries a breeze (logstash can also do this)
- not going to elaborate on this (sorry, eventually, just not yet); beware of dragons

# Some practical application
Thoughout this section I will attempt to show both the lucene query and query dsl variants.  Keep in mind, you can  execute a lucene query from any JSON search method using the [query_string](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/query-dsl-query-string-query.html) block which takes a raw lucene query.

```json
{
  "query_string" : {
    "query": "text_entry:arm"
  }
}
```
Also keep in mine queries below are not optimized for performance, this will be covered in a later section.  Finally this is not an exhaustive list.

for complete dsl reference see [query dsl](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/query-dsl.html)

## match_all
The simplest query you can execute, it will match everything using a `wildcard` character in lucene, or a match
```bash
#lucene-style
*
# `*` represents a wildcard, notice the field selector is absent

#dsl-style
{"query" : {"match_all" : {}}}

# {}   a blank object also works
#      or a blank payload

curl -H "Content-Type: application/x-ndjson" -XGET 'localhost:9200/_msearch?pretty' --data-binary '
{"index" : "shakespeare"}
{"size":0, "query" : {"match_all" : {}}}
'

#result - matched 112k documents, we omit them with size 0
{
  "responses" : [
    {
      "took" : 21,
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
