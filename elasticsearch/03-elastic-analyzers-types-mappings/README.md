# Mappings
Mappings in elasticsearch are very important, they *dictate how information is stored and searched*. Rounding back to the sample data training module, loading the shakespeare data into elastic, remember we didn't provide a mapping? Helpful, but not always.

# Schema-less?
A selling point of elasticsearch is it's ability to introspect a schema/mappings on-demand. A misconception it is a completely schema-less datastore; this is 100% false (if you are here for search).

For every new index created, without external inputs like [static mappings](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html) or [dynamic template](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-templates.html), elastic will try and build a schema containing interpretted types for each field; using it's first observation. This schema is considered LAW until the index is rotated. Documents which do not align with this schema are rejected, unless the [ignore_malformed](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/ignore-malformed.html) option is enabled, or the field is set for storage without indexing (won't cause type conflicts, but also makes the field unsearchable).
>

Due to elasticsearch's design, changing mappings almost always requires a reindex of all in-scope data; elasticsearch is blazing fast because it pays the cost of search mostly at index time, rather than at search time.

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

A document will index into elastic if it obeys the existing schema; or the ignore_malformed option is used (which discards the conflicting value). **It is imperative** that producers from elastic **Do NOT mingle conflicting types within the same field namespace** and expect  documents to successfully index.

# String type
The string type is arguably the most confusing type, due to the many avaliable indexing permutations.

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
  - *example: Find essay documents that contain a particular word from a set of 100k documents or find documents with urls that contain the word `shop`*

You can analyze text in many other ways, it is encourage to index data as close to your search pattern as possible. [Customize string analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html) can help with this effort, such as n-grams and auto-complete analyzers, these are not applied by default.

Your index mapping may differ environment to environment. Be sure when yous are unfamilar with an index to check it's mapping before issuing a query, will save you a lot of headache.

```bash
curl -sk 'http://localhost:9200/${INDEX}/_mapping?pretty'
```

# What is the (string) analyzer doing?
Search is all about tokens.  Tokens are produced by analyzers, in order to understand search you need to understand the tokenization process.

Let's walk through a couple analyzers below using the [Analyze API](https://www.elastic.co/guide/en/elasticsearch/reference/current/_testing_analyzers.html), which can be used anytime.

> the jq filter below is omitting information, remove it to see ordinal and other extended information

#### [standard (text)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html); the default analyzer
input is tokenized based on grammar (using the Unicode Text Segmentation algorithm, as specified in Unicode Standard Annex #29). works with most languages. special characters and whitespace are often removed.
```bash
curl -sk 'http://localhost:9200/_analyze?pretty' -H 'Content-type: application/json' -d '
{
 "analyzer": "standard",
 "text":     "The 2 QUICK Brown-Foxes jumped over the lazy dog'"'"'s bone."
}' | jq '[.tokens[].token]'

#resulting tokens
[
  "the",
  "2",
  "quick",
  "brown",
  "foxes",
  "jumped",
  "over",
  "the",
  "lazy",
  "dog's",
  "bone"
]
```

#### [keyword (keyword)](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-keyword-analyzer.html)
input is tokenized as a single, unmodified term
```bash
curl -sk 'http://localhost:9200/_analyze?pretty' -H 'Content-type: application/json' -d '
{
 "analyzer": "keyword",
 "text":     "The 2 QUICK Brown-Foxes jumped over the lazy dog'"'"'s bone."
}' | jq '[.tokens[].token]'

#resulting tokens
[
  "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
]
```

#### [whitespace](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-whitespace-analyzer.html)
input is tokenized on whitespace
```bash
curl -sk 'http://localhost:9200/_analyze?pretty' -H 'Content-type: application/json' -d '
{
 "analyzer": "whitespace",
 "text":     "The 2 QUICK Brown-Foxes jumped over the lazy dog'"'"'s bone."
}' | jq '[.tokens[].token]'

#resulting tokens
[
  "The",
  "2",
  "QUICK",
  "Brown-Foxes",
  "jumped",
  "over",
  "the",
  "lazy",
  "dog's",
  "bone."
]
```

#### custom
if you are a Database admin responsible for storing and indexing data you can create your own anaylzer using a combination of
* [tokenizers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html) - receives a stream of characters, breaks it up into individual tokens (usually individual words), outputs a stream of tokens
* [token filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html) - accept a stream of tokens from a tokenizer and can modify tokens
* [character filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-charfilters.html) - used to preprocess a stream of characters before it is passed to the tokenizer
* [normalizers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-normalizers.html) - similar to an analyzer, but lacking a tokenizer. solely comprised of token filters and character filters. they may only emit a single tokens.

```bash
curl -sk 'http://localhost:9200/_analyze?pretty' -H 'Content-type: application/json' -d '
{
  "tokenizer": "standard",
  "filter":  [ "lowercase", "asciifolding" ],
  "text":      "Is this déja vu?"
}' | jq '[.tokens[].token]'

#resulting tokens
[
  "is",
  "this",
  "deja",
  "vu"
]
```
be sure to see the other built-in [analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html) and experiment


# More types!
Other [data types]( https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html) are pretty straight-forward with generally few indexing tweakables. Two types, arrays(nested) and objects, have some caveats...


* **[arrays(nested)](https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html)**:
  * may only contain a single type
  * the first value, AKA `array[0]`, dictates the type of an array within the schema
  * a field defined as an array can index a primitive of the same type  *example: field is `array[int]`, doc has `int` value, elastic indexs that int as `array[int]\(length 1\)`*
  * a non-array field cannot ingest an array of the same type.  *example: field is `int`, doc has `array[int]` value, elastic says no*
* **[objects/sub objects](https://www.elastic.co/guide/en/elasticsearch/reference/current/object.html)**:
  * fields inside the object **MUST MAINTAIN CONSISTENT FIELD TYPING**, just like fields at the root level
  * you cannot index a child object or child attribute, if you do not index it's parent. indexing/walking the parent object successfully requires consistent typing.
  * a string cannot be an object, an object cannot auto-marshal to a string
