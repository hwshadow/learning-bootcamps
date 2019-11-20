# feeding jq data
> assumes `cd /project/jq+gron/01-jq-basics`

unix pipes to jq
```bash
cat ./single.json | jq '.'
```
input file as argument
```bash
jq '.' ./single.json
```
```json
{
  "text": "this document showcases a user's purchases for 6 months",
  "items_per_month": [
    1,
    3,
    1,
    1,
    1,
    0
  ],
  "items_purchased": [
    {
      "item": "WOODEN_CHAIR_BROWN",
      "orderid": 13001,
      "sku": "F1040011",
      "wholesale_cost": 15.55,
      "msrp": 32.5,
      "end_price": 29.95,
      "returned": false
    },
    {"-----CONTENT_TRUNCATED-----":"-----CONTENT_TRUNCATED-----"}
  ],
"user.name": "jqlot1",
"user.email": "jqlot@example.com"
}
```

# filter to a specific
field, output json
```bash
jq '.text' ./single.json
```
```json
"this document showcases a user's purchases for 6 months"
```

field, output raw
```bash
jq -r '.text' ./single.json
```
```json
this document showcases a user's purchases for 6 months
```

nested field
```bash
jq '.items_purchased[3].sku' ./single.json
```
```json
"F200"
```

field with special character
```bash
jq '.["user.email"]' ./single.json
```
```json
"jqlot@example.com"
```

array, output compact
```bash
jq '.items_per_month' ./single.json
```
```json
[1,3,1,1,1,0]
```

array element
```bash
jq '.items_per_month[1]' ./single.json
```
```json
3
```


# slurping data
> assumes you have downloaded sample documents `cd /projectelasticsearch/01-sample-data-by-elastic`

slurpes in `ldjson/ndjson/jsonl` format and to an array
```bash
jq --slurp '.| .[0]' ./accounts.json
```
```json
{
  "index": {
    "_id": "1"
  }
}
```
