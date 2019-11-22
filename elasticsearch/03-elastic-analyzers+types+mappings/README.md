# Mappings
Mappings in elasticsearch are very important, they *dictate how information is stored and searched*. Rounding back to the sample data training module, loading the shakespeare data into elastic, notice that we didn't provide a mapping? Helpful, but not always.

# Schema-less?
A selling point of elasticsearch is it's ability to coerce a schema/mappings on-demand. A misconception it is a completely schema-less datastore; this is 100% false (if you expect fields to be searchable).

For each new index created, without external input ([static mappings](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html) or [dynamic template](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-templates.html)) elastic will try and build a schema referencing the interpretted type for each field using it's first observation. This schema is considered LAW until the index is rotated. Documents which do not align with this schema are rejected, unless the [ignore_malformed](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/ignore-malformed.html) option is enabled, or the field is set for storage without indexing (won't cause typing conflicts, but also will not be searchable).
>

Due to elasticsearch's design, changing mappings almost always requires a reindex of all in-scope data; elasticsearch is blazing fast because it pays it's search cost mostly at index time, rather than search time.

# JIT schema generation, visual aid
|field|doc-1|realized-type|ingest-result-1||d2|rt|ir2||d3|rt|ir3|
|---|---|---|---|---|---|---|---|---|---|---|---|
|msg|null||success||string|string|success||object|string|`FAILURE`|
|error|object|object|success||object|object|success||string|object|`FAILURE`|
|count|string|string|success||int |string|success `(typecast)`||string|string|success|
|data_a|array[int,string,int,int]|array[int...]|`FAILURE`||array[int,int,int] |array[int...]|success||null|array[int...]|success|
|timing|int|int|success||int|int|success||array[int,int,int]|int|`FAILURE`|
|nitrox_code|null||success||null||success||int|int|success|
|  -- |  -- |  -- | `FAILURE`  | | --  | --  | `success` | | --  | --  |  `FAILURE` |

A document will index into elastic if it obeys the existing schema; or the ignore_malformed option is used (which discards the conflicting value). **It is imperative** that producers from elastic **Do NOT mingle conflicting types within the same field namespace**, with the expectation the document will successfully index.

# String type
The string type arguably the most confusing type, due to the many avaliable indexing permutations.

The default mapping for string data (as of >= 5.x) is:
```json
{
  "{FIELD_NAME}": {
    "type" : "text",
    "fields" : {
      "keyword" : {
        "type" : "keyword",
        "ignore_above" : 256
      }
    }
  }
}
```
This, by default, double indexes each field in roughly the following manner:
* **[keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)**:
  - given `The 'smart' fox cries Aloud!`
  - field is stored and searchable as an **exact match** `{FIELD_NAME}.keyword:"The 'smart' fox cries Aloud!"`
  - useful when the full context of a field matters.
  - *example: List top N HTTP URLs*
* **[text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)**:
  - given `The 'smart' fox cries Aloud!`
  - stored and searchable as a **collection of normalized terms** `["the","smart","fox","cries","aloud"]` -> `{FIELD_NAME}:"{TERM}"`
  - useful when you want to search for a piece of something.
  - *example: Find essay documents that contain a particular word from a set of 100k*

You can actually analyze text in many many other ways, the name of the game with elasticsearch is to index data as close to your search pattern as possible. [Customize string analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html) can help with this effort, such as n-grams and auto-completion analyzers; these are not the default however.

# What is an analyzer doing?
Let's walk through several analyzers below via the [Analyze API](https://www.elastic.co/guide/en/elasticsearch/reference/current/_testing_analyzers.html)


# More types!
Other [data types]( https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html) are pretty straight-forward with few indexing tweakables. The exception being elasticsearch's handling of arrays and objects.

In short (if you desire searchability):
* **[arrays(nested)](https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html)**:
  * may only contain a single type
  * index zero `array[0]` dictates the type of an array within the schema
  * a field with type array can ingest, via typecast their primitive counterpart on ingestion, allowing ingestion.  *example: field is `array[int]`, doc has `int` value, elastic indexs that int as `array[int]\(length 1\)`*
  * a field with type primitive cannot ingest an array of that primitive; ingestion will fail ingestion.  *example: field is `int`, doc has `array[int]` value, elastic says no*
* **[objects/sub objects](https://www.elastic.co/guide/en/elasticsearch/reference/current/object.html)**:
  * like the root JSON document, **MUST MAINTAIN CONSISTENT FIELD TYPING**
  * you cannot index a child object or child attribute, if you do not index the parent object. indexing/walking the parent object successfully requires consistent typing.
  * a string cannot be an object, an object cannot auto-marshal to a string.
