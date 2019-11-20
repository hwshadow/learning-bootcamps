# https://stedolan.github.io/jq/ https://github.com/tomnomnom/gron https://github.com/kislyuk/yq

# Technologies
* jq (JSON processor)
* gron (JSON flattener)
* yq (YAML processor, fork of jq)
* xq (XML processor, fork of jq)

# About
jq is like sed for JSON data - you can use it to slice and filter and map and transform structured data with the same ease that sed, awk, grep and friends let you play with text.  yq & xq are variants built for YAML/XML processing.

gron transforms JSON into discrete assignments to make it easier to grep for what you want and see the absolute 'path' to it. It eases the exploration of APIs that return large blobs of JSON but have terrible documentation. (Make JSON greppable!)


## Learning
### Documentation
* jq official documentation https://stedolan.github.io/jq/manual/ `this is your best resource hands down`
* jq online playground https://jqplay.org/
* gron official documentation https://github.com/tomnomnom/gron/blob/master/README.mkd#usage
* yq&xq official documentation https://github.com/kislyuk/yq/blob/master/README.rst#synopsis

### Texts
* jq quickstart https://stedolan.github.io/jq/tutorial/
* jq equivalents to unix commands http://andrew.gibiansky.com/blog/command-line/jq-primer/

# Local Environment (hands-on)
To get started with a local hands on demo environment
```bash
# see repo_deps for pre-requisite software, also elasticsearch/01-sample-data-by-elastic
docker-compose run jqgron-learn
bash-3.2$ cd /project/jq+gron/01-jq-basics && \
jq '.' ./single.json


# to trash your environment
bash-3.2$ exit
docker-compose down
```
