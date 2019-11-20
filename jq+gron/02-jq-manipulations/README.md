> assumes ```
cd /project/jq+gron/01-jq-basics
```

# restructure an object
to an `array`
```bash
jq -c '. | [.["user.name"], .["user.email"]]' ./single.json
```
> ```json
["jqlot1","jqlot@example.com"]
```

to an `object`
```bash
jq -c '. | {"user": .["user.name"], "email": .["user.email"]}' ./single.json
```
> ```json
{"user":"jqlot1","email":"jqlot@example.com"}
```

to a `string`
```bash
jq -c '. | "\(.["user.name"])'"'"'s email is \(.["user.email"])" ' ./single.json
```
> ```json
"jqlot1's email is jqlot@example.com"
```

# add field(s)
adds `region`
```bash
jq '. += {"region":"USA"} ' ./single.json
```
> ```json
{
...
"user.name": "jqlot1",
"user.email": "jqlot@example.com",
"region": "USA"
}
```

# remove a field(s)
removes `items_purchased` and `text`
```bash
jq -c '. | del(.["items_purchased"],.text)' ./single.json
```
>```json
{"items_per_month":[1,3,1,1,1,0],"user.name":"jqlot1","user.email":"jqlot@example.com"}
```

# select based on criteria
contains the substring `WOOD`
```bash
jq -c '.items_purchased[] | select(.item | contains("WOOD"))' ./single.json
```
> ```json
{"item":"WOODEN_CHAIR_BROWN","orderid":13001,"sku":"F1040011","wholesale_cost":15.55,"msrp":32.5,"end_price":29.95,"returned":false}
```

matches regexp `(RED|BLACK|BROWN)`
```bash
jq -c '.items_purchased[] | select(.item | test("(RED|BLACK|BROWN)")) | .item' ./single.json
```
> ```json
"WOODEN_CHAIR_BROWN"
"VIDEO_GAME_RED_DEAD_REDEMPTION"
"BLACK_GARBAGE_CAN"
```

matches items sold higher than msrp
```bash
jq '.items_purchased | ( . | map(select(.end_price-.msrp >0))) ' ./single.json
```
> ```json
[
  {
    "item": "4PORT_SWITCH",
    "sku": "E9993",
    "orderid": 12554,
    "wholesale_cost": 20,
    "msrp": 25,
    "end_price": 26,
    "returned": false
  },
  {
    "item": "GOLD_PICTURE_FRAME",
    "orderid": 15178,
    "sku": "F35091",
    "wholesale_cost": 10,
    "msrp": 12,
    "end_price": 12.99,
    "returned": false
  }
]
```

place matches `back into` array
```bash
jq -c '[ .items_purchased[] | select(.item | test("(RED|BLACK|BROWN)")) | .item ]' ./single.json
```
> ```json
["WOODEN_CHAIR_BROWN","VIDEO_GAME_RED_DEAD_REDEMPTION","BLACK_GARBAGE_CAN"]
```



# functions (sum, length, etc)
