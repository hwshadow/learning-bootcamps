# Common operations

## match_all
The simplest query you can execute, it will match everything using a `wildcard` character in lucene, or a match
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

## boolean conditons

## invertion / exclusion
