# Udemy Course: Elasticsearch 7 and the Elastic Stack - In Depth & Hands On!

* [Course Link](https://www.udemy.com/course/elasticsearch-7-and-elastic-stack/)
* [Course Repo]()


## Section 1: Installing and Understanding Elasticsearch

### Lecture 3. Installing Elasticsearch [Step by Step]

* installation instructions on Ubuntu Linux machine or VM
* we will try to install it in Cloud9 Dev VM and see how it goes
* try disabling Hyper-V from BIOS for virtualization
* Win/Mac users get VirtualBox
* download OS iso file to run on VM (prefered Ubuntu Server x18.04 LTS)
* he is using 8GB of RA and 20GB of HDD (ouch.. $$$)
* an openSSH server is needed
* check [course website](https://sundog-education.com/elasticsearch/)
* [elasticsearch7 course material](http://media.sundog-soft.com/es7/ElasticStack7.pdf)
* get the binary `wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -`
* run `sudo apt-get install apt-transport-https`
* run
```
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" |
sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```
* install es7 `sudo apt-get update && sudo apt-get install elasticsearch`
* config it `sudo vi /etc/elasticsearch/elasticsearch.yml`
    * Uncomment the node.name line 
    * Change `network.host` to 0.0.0.0, `discovery.seed.hosts` to [“127.0.0.1”], and `cluster.initial_master_nodes` to [“node-1”] 
* run
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
sudo /bin/systemctl start elasticsearch.service
```
* srvice is started. it should auto start at boot time in future restarts
* we test server is live `curl -XGET 127.0.0.1:9200`
* to test the elasticsearch api we download the shakespear search index `wget http://media.sundog-soft.com/es7/shakes-mapping.json`
* we import it in elasticsearch `curl -H 'Content-Type: application/json' -XPUT 127.0.0.1:9200/shakespeare --data-binary @shakes-mapping.json`
* we get the actual shakespear works in json format `wget http://media.sundog-soft.com/es7/shakespeare_7.0.json`
* we load them in elasticsearch `curl -H 'Content-Type: application/json' -XPOST '127.0.0.1:9200/shakespeare/_bulk?pretty' --data-binary @shakespeare_7.0.json`
* this action loads the data in elastcsearch cluster and does indexing
* we test that data were loaded o issuing a query to the cluster
```
curl -H 'Content-Type: application/json' -XGET '127.0.0.1:9200/shakespeare/_search?pretty' -d '
{
"query" : {
"match_phrase" : {
"text_entry" : "to be or not to be"
}
}
}
'
```

### Lecture 4. Elasticsearch Overview

* started as scalable Apache Lucene Framework
* a horizontal scalable search engine
* a shard in es7 is a lucene inverted index of documents
* good for full text search but can handle structured data
* often faster than hadoop/spark/flink
* es7 serves json requests
* Kibana built on top of es7 offers WebUI for search and visualize, complex aggregations, graphs, charts
* Kibana is used for log analysis
* Logstash + Beats
*   * a way to feed data in es7 cluster
*   * Filebeat can monitor log files, parse them, import in ES7 in near realtime
*   * Logstaxh also pushes data in ES7 from various machines
*   * Not just log files
*   * X-Pack offers security,alerting,monitoring,reporting,ML and Graph Exploration for ES7 (Paid service)

### Lecture 5. Intro to HTTP and RESTful API's

* 