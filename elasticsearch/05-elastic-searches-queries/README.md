# Query examples
A showcase both the lucene query and query dsl variants.  Keep in mind, you can execute a raw lucene query from any JSON search method using the [query_string](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/query-dsl-query-string-query.html) full-text search block.

## match_all
The simplest query you can execute; it will match everything, in lucene you use a `wildcard` character to make this
> lucene style |
> `*` represents a wildcard, notice the field selector is allowed to be absent

```bash
*
```

> dsl-style | `{}` a blank object or a blank payload

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
Take the `modsec` index for example, it has a complicated mapping.
```bash
#Display mapping
jq '.' ./01-elastic-sample-data/modsec_mapping.json
#Display sample document
jq '.' ./01-elastic-sample-data/modsec_normalized.json
```

Lets try and find a sample document in elasticsearch.  Say we are interested in the `request` field and we want to find the substring `%645&vars`, which clearly exists in this document.

```
	GET /elrekt.php?s=%2f%69%6e%64%65%78%2f%5c%74%68%69%6e%6b%5c%61%70%70%2f%69%6e%76%6f%6b%65%66%75%6e%63%74%69%6f%6e&function=%63%61%6c%6c%5f%75%73%65%72%5f%66%75%6e%63%5f%61%72%72%61%79&vars[0]=%6d%645&vars[1][]=%48%65%6c%6c%6f%54%68%69%6e%6b%50%48%50 HTTP/1.1\r\nUser-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0\r\nHost: 10.1.1.100\r\nConnection: Keep-Alive\r\nCache-Control: no-cache\r\nX-Forwarded-For: 139.155.127.88\r\nVia: 1.1 localhost
```

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
{"index" : "modsec"}
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

## Attempt #2
Lets try the `@request` field
```bash
curl -sk 'localhost:9200/modsec/_mapping?pretty' | jq '.modsec.mappings.doctype.properties | .["@request"] | .index != false'
true
```

Yep it is indexed, let's try the same thing.

> lucene style |

```bash
@request:%645&vars
```

> dsl-style |

```json
{"bool":{"must":[{"query_string":{"query":"@request:%645&vars","analyze_wildcard":true,"default_field":"*"}},{"range":{"@timestamp":{"gte":1565670037521,"lte":1574096829530,"format":"epoch_millis"}}}],"filter":[],"should":[],"must_not":[]}}
```

```bash
curl -H "Content-Type: application/x-ndjson" -XGET 'localhost:9200/_msearch?pretty' --data-binary '
{"index" : "modsec"}
{"size":1, "query":{ "bool":{"must":[{"query_string":{"query":"@request:%645&vars","analyze_wildcard":true,"default_field":"*"}},{"range":{"@timestamp":{"gte":1565670037521,"lte":1574096829530,"format":"epoch_millis"}}}],"filter":[],"should":[],"must_not":[]} }}
'
#result - matched 0 documents, we asked for 1 returned
{
  "responses" : [
    {
      "took" : 10,
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

Still nothing... why...
```bash
curl -sk 'localhost:9200/modsec/_mapping?pretty' | jq '.modsec.mappings.doctype.properties | .["@request"]'
{
  "type": "keyword",
  "ignore_above": 4096
}
```

Ah this is a keyword index field, we will need wildcards

## Attempt #3
Lets try that one more time!

> lucene style |

```bash
@request:*%645&vars*
```

> dsl-style |

```json
{"bool":{"must":[{"query_string":{"query":"@request:*%645&vars*","analyze_wildcard":true,"default_field":"*"}},{"range":{"@timestamp":{"gte":1565670037521,"lte":1574096829530,"format":"epoch_millis"}}}],"filter":[],"should":[],"must_not":[]}}
```
s
```bash
curl -H "Content-Type: application/x-ndjson" -XGET 'localhost:9200/_msearch?pretty' --data-binary '
{"index" : "modsec"}
{"size":1, "query":{ "bool":{"must":[{"query_string":{"query":"@request:*%645&vars*","analyze_wildcard":true,"default_field":"*"}},{"range":{"@timestamp":{"gte":1565670037521,"lte":1574096829530,"format":"epoch_millis"}}}],"filter":[],"should":[],"must_not":[]} }}
'
#result - matched 1 documents, we asked for 1 returned
{
  "responses" : [
    {
      "took" : 4,
      "timed_out" : false,
      "_shards" : {
        "total" : 5,
        "successful" : 5,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : 1,
        "max_score" : 2.0,
        "hits" : [
          {
            "_index" : "modsec",
            "_type" : "doctype",
            "_id" : "1",
            "_score" : 2.0,
            "_source" : {
              "@attack_type" : [
                "Overflow",
                "code injection"
              ],
              "@attack_type_len" : 2,
              "@headers" : {
                "cache-control" : "no-cache",
                "connection" : "Keep-Alive",
                "host" : "10.1.1.100",
                "user-agent" : "Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0",
                "via" : "1.1 localhost",
                "x-forwarded-for" : "139.155.127.88"
              },
              "@parser_timestamp" : "2019-09-19T05:12:21.326Z",
              "@request" : "GET /elrekt.php?s=%2f%69%6e%64%65%78%2f%5c%74%68%69%6e%6b%5c%61%70%70%2f%69%6e%76%6f%6b%65%66%75%6e%63%74%69%6f%6e&function=%63%61%6c%6c%5f%75%73%65%72%5f%66%75%6e%63%5f%61%72%72%61%79&vars[0]=%6d%645&vars[1][]=%48%65%6c%6c%6f%54%68%69%6e%6b%50%48%50 HTTP/1.1\r\nUser-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0\r\nHost: 10.1.1.100\r\nConnection: Keep-Alive\r\nCache-Control: no-cache\r\nX-Forwarded-For: 139.155.127.88\r\nVia: 1.1 localhost",
              "@signature_names_len" : 1,
              "@signature_names" : [
                "Remote Code Execution"
              ],
              "@timestamp" : "2019-09-19T05:12:08.000Z",
              "attack_type" : "Overflow,code injection",
              "date_time" : "2019-09-19 05:12:08",
              "dest_ip" : "10.1.1.100",
              "dest_port" : "80",
              "geo_location" : "CN",
              "ip_client" : "139.155.127.88",
              "method" : "GET",
              "parser_delta" : 13.326910018920898,
              "parser_host" : "txt1.localhost",
              "parser_id" : "36b2b036d2cc",
              "policy" : "web_protect_modsec",
              "protocol" : "HTTP",
              "query_string" : "s=%2f%69%6e%64%65%78%2f%5c%74%68%69%6e%6b%5c%61%70%70%2f%69%6e%76%6f%6b%65%66%75%6e%63%74%69%6f%6e&function=%63%61%6c%6c%5f%75%73%65%72%5f%66%75%6e%63%5f%61%72%72%61%79&vars[0]=%6d%645&vars[1][]=%48%65%6c%6c%6f%54%68%69%6e%6b%50%48%50",
              "request" : "GET /elrekt.php?s=%2f%69%6e%64%65%78%2f%5c%74%68%69%6e%6b%5c%61%70%70%2f%69%6e%76%6f%6b%65%66%75%6e%63%74%69%6f%6e&function=%63%61%6c%6c%5f%75%73%65%72%5f%66%75%6e%63%5f%61%72%72%61%79&vars[0]=%6d%645&vars[1][]=%48%65%6c%6c%6f%54%68%69%6e%6b%50%48%50 HTTP/1.1\\r\\nUser-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0\\r\\nHost: 10.1.1.100\\r\\nConnection: Keep-Alive\\r\\nCache-Control: no-cache\\r\\nX-Forwarded-For: 139.155.127.88\\r\\nVia: 1.1 localhost",
              "request_status" : "alerted",
              "response_code" : "0",
              "signature_names" : "Remote Code Execution",
              "source_ip_geo" : {
                "city_name" : "Beijing",
                "country_code3" : "CN",
                "location" : {
                  "lat" : 39.9289,
                  "lon" : 116.3883
                },
                "region_name" : "Beijing"
              },
              "src_port" : "64210",
              "alert_id" : "90960",
              "uri" : "/elrekt.php",
              "user_agent" : {
                "build" : "",
                "device" : "Other",
                "major" : "52",
                "minor" : "0",
                "name" : "Firefox",
                "os" : "Linux",
                "os_name" : "Linux"
              },
              "violations" : "query string,file type,signature detected",
              "x_forwarded_for_header_value" : "139.155.127.88"
            }
          }
        ]
      },
      "status" : 200
    }
  ]
}
```
