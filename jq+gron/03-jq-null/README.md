> assumes `cd /project/jq+gron/03-jq-null`

# jq with null
you ca start jq with a null object
```bash
jq -n '.'
```
```json
null
```

# arguments + null
create well-formed JSON
```bash
jq --arg t "bumblebee sightings by day" --argjson o '[1,2,10,24,1,5]' -n '{"text":$t,"occurences": $o}'
```
```json
{
  "text": "bumblebee sightings by day",
  "occurences": [
    1,
    2,
    10,
    24,
    1,
    5
  ]
}
```
