# https://rmoff.net/2017/08/08/simple-export/import-of-data-sources-in-grafana/
how to export and import config


## export
```bash
#datasources
cd ./export
mkdir -p data_sources
curl -s "http://localhost:3000/api/datasources"  -u admin:admin|jq -c -M '.[]'|split -l 1 - data_sources/

#dashboards
mkdir -p dashboards && for dash in $(curl -sk -u admin:admin http://localhost:3000/api/search\?query\=\& | jq -r '.[].uri' | awk -F '/' '{print $2}'); do
  curl -sk -u admin:admin http://localhost:3000/api/dashboards/db/$dash > dashboards/$dash.json
done
```

## import
```bash
#data_sources
for i in data_sources/*; do \
    curl -X "POST" "http://localhost:3000/api/datasources" \
    -H "Content-Type: application/json" \
     --user admin:admin \
     --data-binary @$i
done

#dashboards
for i in dashboards/*; do \
    r=$(jq '. += {"overwrite": true}' "$i")
    curl -X "POST" "http://localhost:3000/api/dashboards/db" \
    -H "Content-Type: application/json" \
     --user admin:admin \
     -d "$r"             #overwrite insert
     #--data-binary @$i  #(if you don't want an overwrite insert)
done
```
