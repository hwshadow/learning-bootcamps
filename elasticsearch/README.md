# About
Elasticsearch is a distributed, open source search and analytics engine for all types of data, including textual, numerical, geospatial, structured, and unstructured. Elasticsearch is built on Apache Lucene and was first released in 2010 by Elasticsearch N.V. (now known as Elastic). Known for its simple REST APIs, distributed nature, speed, and scalability, Elasticsearch is the central component of the Elastic Stack, a set of open source tools for data ingestion, enrichment, storage, analysis, and visualization. Commonly referred to as the ELK Stack (after Elasticsearch, Logstash, and Kibana), the Elastic Stack now includes a rich collection of lightweight shipping agents known as Beats for sending data to Elasticsearch.



## Technologies
* Elasticsearch (JSON search engine)
* Kibana (GUI for elasticsearch)


## Course-work
### Documentation
* Elasticsearch's official documentation is quite complete.  It is however more of a reference rather than a complete guide.
https://www.elastic.co/guide/en/elasticsearch/reference/6.4/index.html

*  The technology is built on Apache Lucene.  The query bar in Kibana expects lucene queries as input. Using the query_string query or HTTP GET _search API you can also execute raw Lucene queries against data in Elastic
    *  Lucene 5-minute Primer http://www.lucenetutorial.com/lucene-query-syntax.html
    *  Lucene Official http://lucene.apache.org/core/3_5_0/queryparsersyntax.html
    *  IBM Lucene Query terminology https://www.ibm.com/support/knowledgecenter/SSSH5A_8.0.1/com.ibm.rational.clearquest.user_web.doc/topics/c_fts_term_syntax.htm

### Video
#### Core
* [7m] Elasticsearch Architecture (via Youtube, Sundog Education with Frank Kane) https://youtu.be/YsYUgZu9-Y4
* [1hr 31m] Elasticsearch Course via Linkedin Learning https://www.linkedin.com/learning/elasticsearch-essential-training/using-the-exercise-files?u=2284609
* [31m] Text Search with Lucene (via Yotube, Brian Will)
    *  [12m] Part 1 https://youtu.be/x37B_lCi_gc
    *  [19m] Part 2 https://youtu.be/fCK9U3L7c8U


#### Extended
* [1hr 4m] What is in a Lucene index? (via Youtube, LuceneSolrRevolution) https://youtu.be/T5RmMNDR5XI
* [38m] ElasticSearch in action - Thijs Feryn (via Youtube, Codemotion) https://youtu.be/oPObRc8tHgQ
* [36m] Elasticsearch from the bottom up (via Youtube, EuroPython 2014) https://youtu.be/PpX7J-G2PEo
* [44m] Getting Down and Dirty with ElasticSearch by Clinton Gormley (via Youtube, NoSQL matters Conference) https://youtu.be/7FLXjgB0PQI

## Books
* Elasticsearch: The Definitive Guide (via Safari Books, O'Reilly Media, Inc.) https://learning.oreilly.com/library/view/elasticsearch-the-definitive/9781449358532/

# Local Environment (hands-on)
To get started with a local hands on demo environment
```bash
# see repo_deps for pre-requisite softwar
docker-compose up -d

# to access your local elasticsearch API
curl -sk localhost:9200

# to access your local kibana GUI
http://localhost:5601/app/kibana#/

# to trash your environment
docker-compose down
```

Sample data set can be found here
```
https://www.elastic.co/guide/en/kibana/6.4/tutorial-load-dataset.html
```
