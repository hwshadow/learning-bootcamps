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
Thoughout this section I will attempt to show both the lucene query and query dsl variants.  Keep in mind, you can execute a raw lucene query from any JSON search method using the [query_string](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/query-dsl-query-string-query.html) full-text search block.

```json
{
  "query_string" : {
    "query": "text_entry:arm"
  }
}
```
Keep in mind the queries below are not optimized for performance, this is not an exhaustive list, and we can't cover every permutation; just enough to get you started. For complete dsl reference see [query dsl](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/query-dsl.html).  High-level be aware of:
* [Full-text queries](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/full-text-queries.html)
* [Term-level queries](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/term-level-queries.html) - will analyze the query string before executing, the term-level queries operate on the exact terms that are stored in the inverted index
  - term query - Find documents which contain the exact term specified in the field specified.
  - terms query - Find documents which contain any of the exact terms specified in the field specified.
  - terms_set query - Find documents which match with one or more of the specified terms. The number of terms that must match depend on the specified minimum should match field or script.
  - range query - Find documents where the field specified contains values (dates, numbers, or strings) in the range specified.
  - exists query - Find documents where the field specified contains any non-null value.
  - prefix query -Find documents where the field specified contains terms which begin with the exact prefix specified.
  - wildcard query - Find documents where the field specified contains terms which match the pattern specified, where the pattern supports single character wildcards (?) and multi-character wildcards (*)
  - regexp query - Find documents where the field specified contains terms which match the regular expression specified.
  - fuzzy query - Find documents where the field specified contains terms which are fuzzily similar to the specified term. Fuzziness is measured as a Levenshtein edit distance of 1 or 2.
  - type query - Find documents of the specified type.
  - ids query - Find documents with the specified type and IDs.
* [Difference between filter and query context](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/query-filter-context.html)
