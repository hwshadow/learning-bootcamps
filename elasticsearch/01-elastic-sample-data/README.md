# loading sample data

1) download samples

```bash
#!/bin/bash
wget -O ./shakespeare_6.0.json https://download.elastic.co/demos/kibana/gettingstarted/shakespeare_6.0.json
wget -O ./accounts.zip https://download.elastic.co/demos/kibana/gettingstarted/accounts.zip && unzip accounts.zip
wget -O ./logs.jsonl.gz https://download.elastic.co/demos/kibana/gettingstarted/logs.jsonl.gz && gunzip logs.jsonl.gz
```
> the following assumes you have the local docker environment up and running

> also being familar with JQ is helpful, see the module on that.

2) insert samples into elasticsearch (_without mappings_)
```bash
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/shakespeare/doc/_bulk?pretty' --data-binary @shakespeare_6.0.json
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/_bulk?pretty' --data-binary @logs.jsonl
```

3) verify data in elasticsearch
```bash
curl 'localhost:9200/_cat/indices?v'
```
should look like this
```bash
bash-3.2$ curl 'localhost:9200/_cat/indices?v'
health status index               uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   shakespeare         4QYCPm9zSWegvRv4aB1mWA   5   1     111392            0     22.5mb         22.5mb
yellow open   logstash-2015.05.18 GjM8gZQdRaOUO3A1TcAsFw   5   1       4631            0     22.2mb         22.2mb
yellow open   logstash-2015.05.19 hg1fVRMyTJufuK3ia15tCw   5   1       4624            0     21.5mb         21.5mb
yellow open   bank                91Xgpu8TQhS96rGhIrtzeg   5   1       1000            0    475.1kb        475.1kb
yellow open   logstash-2015.05.20 -6JV0MfsS8CZtLHKrz8lqw   5   1       4750            0     23.6mb         23.6mb
```

# inspecting mappings
> because we did not specify an explicit mapping, it was introspected from the first occurrence of each field indexed to the 'shakespeare' index.  documents not matching this mapping will fail to index.


```bash
curl 'localhost:9200/shakespeare/_mapping?pretty'
```
```json
{
  "shakespeare" : {
    "mappings" : {
      "doc" : {
        "properties" : {
          "line_id" : {
            "type" : "long"
          },
          "line_number" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "play_name" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "speaker" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "speech_number" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "text_entry" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "type" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          }
        }
      }
    }
  }
}
```

# searching an index
```bash
curl 'localhost:9200/shakespeare/_search?size=3&pretty'
```
```json
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
    "total" : 111392,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_type" : "doc",
        "_id" : "0",
        "_score" : 1.0,
        "_source" : {
          "type" : "act",
          "line_id" : 1,
          "play_name" : "Henry IV",
          "speech_number" : "",
          "line_number" : "",
          "speaker" : "",
          "text_entry" : "ACT I"
        }
      },
      {
        "_index" : "shakespeare",
        "_type" : "doc",
        "_id" : "14",
        "_score" : 1.0,
        "_source" : {
          "type" : "line",
          "line_id" : 15,
          "play_name" : "Henry IV",
          "speech_number" : 1,
          "line_number" : "1.1.12",
          "speaker" : "KING HENRY IV",
          "text_entry" : "Did lately meet in the intestine shock"
        }
      },
      {
        "_index" : "shakespeare",
        "_type" : "doc",
        "_id" : "19",
        "_score" : 1.0,
        "_source" : {
          "type" : "line",
          "line_id" : 20,
          "play_name" : "Henry IV",
          "speech_number" : 1,
          "line_number" : "1.1.17",
          "speaker" : "KING HENRY IV",
          "text_entry" : "The edge of war, like an ill-sheathed knife,"
        }
      }
    ]
  }
}
```

source
`https://www.elastic.co/guide/en/kibana/6.4/tutorial-load-dataset.html`
