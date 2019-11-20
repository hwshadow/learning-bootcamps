FROM ubuntu:20.04

RUN apt-get update && apt-get upgrade -y
RUN apt-get install -y jq gron
RUN echo "cd /project/elasticsearch/01-sample-data-by-elastic" >> ~/.bash_history
RUN echo "jq '.' ./accounts.json" >> ~/.bash_history
RUN echo "cd /project/jq+gron/01-jq-basics" >> ~/.bash_history
RUN echo "jq '.' ./single.json" >> ~/.bash_history
