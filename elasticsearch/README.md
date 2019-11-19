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
idx | Title | Medium | Time | isFree | Link
-|--------|------|---|--|---------
1 | Elasticsearch Architecture | Youtube, Sundog Education with Frank Kane | 7m | y | https://youtu.be/YsYUgZu9-Y4 
2 | Elasticsearch Course | LinkedIn Learning, Ben Sullins | 91m | n | https://www.linkedin.com/learning/elasticsearch-essential-training/using-the-exercise-files 
3 | Text Search with Lucene | Youtube, Brian Will | 31m | y | https://youtu.be/x37B_lCi_gc part 1 https://youtu.be/fCK9U3L7c8U part 2

#### Extended
idx | Title | Medium | Time | isFree | Link
-|--------|-----|---|--|---------
0.1 | What is in a Lucene index? | Youtube, LuceneSolrRevolution | 64m | y | https://youtu.be/T5RmMNDR5XI 
0.2 | ElasticSearch in action - Thijs Feryn | Youtube, Codemotion | 38m | y | https://youtu.be/oPObRc8tHgQ
0.3 | Elasticsearch from the bottom up | Youtube, EuroPython 2014 | 36m | y | https://youtu.be/PpX7J-G2PEo
0.4 | Getting Down and Dirty with ElasticSearch by Clinton Gormley | Youtube, NoSQL matters Conference | 44m | y | https://youtu.be/7FLXjgB0PQI


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
