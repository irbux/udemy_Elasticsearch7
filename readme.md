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

* es7 uses  a RESTful API
* REST (Representational State Transfer)
* REST guiding constrains:
    * client-server architecture
    * statelessness
    * cacheability
    * layered system
    * code on demand (send JS)
    * uniform interface
* in the course we wil use curl (duh....).  `curl -H "Content-Type: application/json" <URL> -d <BODY>`
* we can use postman
* ?pretty prettifies response

### Lecture 6. Elasticsearch Basics: Logical Concepts

* the basic logical entity of elasticsearch is the document:
    * they are like a row in a DB. they have a unique ID and type
    * they are what we look for. text, JSON, any structured data
* index:
    * it powers search in documents in a collection
    * they contain inverted indices to search on anything at once
    * they contain mappings that define schemas for data within
    * like a DB table

### Lecture 7. Term Frequency / Inverse Document Frequency (TF/IDF)

* inverted index contains the document index where a term occurs
* it also stores the position in teh document
* TF-IDF: Term Frequency / Inverse Document Frequency
* TF is how often a term appears in a given document
* DF is how often a term appears in all documents
* TF/DF: measures the relevance of a term in a document

### Lecture 8. Using Elasticsearch

* We can use an index in elastisearch:
    * with RESTful API
    * client API (using specialized ES& libs) (wrapper of RESTful API)
    * analytic tools. (eb based GUIs like Kibana). no code involved

### Lecture 9. What's New in Elasticsearch 7

* ES7 vs ES6:
    * concept of document types is deprecated (only type = doc is used)
    * Elasticsearch SQL released to prod
    * lots of changes to default configurations (num of shards, replicattion etc)
    * Lucene 8 is used
    * Many X-Pack plugins included in ES7
    * no need to install java
    * Kibana still needs external JDK
    * Cross-cluster replication available
    * Index Lifecycle Management (ILM) available

### Lecture 10. How Elasticsearch Scales

* documents are hashed to a specific shard
* each shard may be in a different node of the cluster
* esch shard is a self-contained Lucene index
* nodes might fail
* our example index has 2 primary shards and 2 replicas
* our cluster has 3 nodes
* the app round-robin requests among nodes
* Write requests are routed to the primary shard => replicated to replicas
* Read requests are routed to the primary or replica shards
* if node with primary fails. a replica will take its place
* an odd number of nodes is prefered for resiliency
* round robin requests srpeads out the load
* reads is faster than write. write is limited by the number of piamry index shards
* the number of primary shards cannot change later on, unless we re-index our data. 
* number of shards can be set upfront with a PUT reqest
```
PUT /testindex
{
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 1
    }
}
```
* in our example we have 3 primary shards an 1 replica for each (6 shards it total)
* most usecases are read heavy

## Section 2: Mapping and Indexing Data

### Lecture 14. Connecting to your Cluster

* shows how to connect with ssh on the local machine (duh?)
* in vm do port forwarding in settings => network
* open port 22 for 127.0.0.1 as guest port 22 for SSH
* open port 9200 for 127.0.0.1 as guest port 9200 for elastic
* open port  5601 for 127.0.0.1 as guest port 5601 for kibana
* `ssh 127.0.0.1` to connect

### Lecture 15. Introducing the MovieLens Data Set

* movielens is like netflix free db with data from movielens.org
* we will load them to our single node elastic search cluster
* we grab the dataset from [Grouplens](https://grouplens.org/)
* we get it [](http://files.grouplens.org/datasets/movielens/ml-latest-small.zip) and extract the data to the folder where we run the ssh
* its a buch of csv datasets related to movie ratings from users

### Lecture 16. Analyzers

* mappings tell elasticsheard how to store the data. 
* its like a schema definition
* it tells ES7 how to store the data and how to index them
* ES7 has reasonable defaults. but sometimes we need to do customization
```
curl -XPUT 127.0.0.1:9200/movies -d
'{
    "mappings": {
        "properties": {
            "year": {"type":"date"}
        }
    }
}'
```
* with mappings we can set: 
* Field Types (sting,byte,integer,long,float,double,boolean,date)
```
"properties": {
    "user_id" : {
        "type": "long"
    }
}
```
* Field Index (do you want this field incexed for full text search?analyzed/not_analyzed)
```
"properties": {
    "genre": {
        "index": "not_analyzed"
    }
}
```
* Field Analyzer (define your tokenizer and token filter, standard/whitespace/simple/english etc)
```
"properties": {
    "description": {
        "analyzer": "english"
    }
}
```
* An analyzer can: 
    * apply character filter: remove HTML encodings, convert chars
    * tokenizer: split strings on whitespace/punctuation/non;letters
    * token filter: lowercasing, stemming, synonyms, stopwords
    * standard: splits on word boundaries, removes punctuation, lowercases, good if languae is unknown
    * simple: split on anything that not a letter and lowercase
    * whitespace: splits on whitespace but not lowercase
    * language (english)

### Lecture 17. Import a Single Movie via JSON / REST

* we create our own curl script
```
mkdir bin
cd bin
nano curl
```
* in the nano editor we add
```
#!/bin/bash
/usr/bin/curl/ -H "Content-Type: application/json" "$@"
```
* we make script executable `chmod a+x curl`
* we go to home dir and `source .profile`
* then we check which command we use for default `which curl`
* we create a new mapping for our movie index setting year type to date
```
curl -XPUT 127.0.0.1:9200/movies -d '
{
    "mappings": {
        "properties": {
            "year": {
                "type":"date"
            }
        }
    }
}'
```
* we get reply `{"acknowledged":true,"shards_acknowledged":true,"index":"movies"}`
* we get the mapping to confirm insertion `curl -XGET 127.0.0.1/movies/_mapping`
* import a doc
```
curl -XPOST 127.0.0.1:9200/movies/_doc/109487 -d
'{
    "genre": ["IMAX","Sci-Fi"],
    "title": "Interstellar",
    "year": 2014
}'
```
* we pass in the doc using the actual index from dataset
* the reply is
```
{
    "_index": "movies",
    "_type": "_doc",
    "_id": "109487",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```
* we can do a general search with `curl -XGET 127.0.0.1:9200/movies/_search?pretty`

### Lecture 18. Insert Many Movies at Once with the Bulk API

* the bulk info format the ES7 uses is 
```
curl -XPUT 127.0.0.1:9200/_bulk -d 
'{ "create": { 
    "_index": "movies",
    "_id": "124343"
    }
}
{
    "id": "124343",
    "title": "Star Trek Beyond",
    "year": 2016,
    "genre": ["Action"."Adventure","Sci-Fi"]
}
....
'
```

* first we create an index and then we pass the doc
* ES7 will index each doc in a shard so it needs data entry one by one even in bulk data entry
* we get data from course material `wget http://media.sundog-soft.com/es7/movies.json`
* we pass in the json in curl to save time as `curl -XPUT 127.0.0.1:9200/_bulk?pretty --data-binary @movies.json`
* we check what docs are in movies index with `curl -XGET 127.0.0.1:9200/movies/_search?pretty`

### Lecture 19. Updating Data in Elasticsearch

* Every document has a _version field
* ES documents are immutable
* When we update an existing document   
    * a new document is created with an incremented _version
    * the old document is marked for deletion
* we can have multiple versions of a doc. ES uses the latest one, it cleans up old versions at will
* actual command we use in our cluster to do a full update (pass in all fields)
```
curl -XPUT 127.0.0.1:9200/movies/_doc/109487?pretty -d '
{
  "genres": ["IMAX","Sci-Fi"],
  "title": "Interstellar foo",
  "year": 2014
}'
```
* reply is
```
{
  "_index" : "movies",
  "_type" : "_doc",
  "_id" : "109487",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 5,
  "_primary_term" : 2
}
```
* we get the doc to confirm `curl -XGET 127.0.0.1:9200/movies/_doc/109487?pretty`
* to do a PARTIAL update on a doc we use
```
curl -XPOST 127.0.0.1:9200/movies/_doc/109487/_update -d '
{
  "doc": {
    "title": "Interstellar"
  }
}'
```
* so PUT commands on existing docs are treated as full updates. POST requsts with partial fields are treated as partial updates

### Lecture 20. Deleting Data in Elasticsearch

* command to delete an individual doc is `curl -XDELETE 127.0.0.1:9200/movies/_doc/58559`
* real world scenario. search and delete (to get id) and delete 
```
curl -XGET 127.0.0.1:9200/movies/_search?q=Dark
curl -XDELETE 127.0.0.1:9200/movies/_doc/58559?pretty
```

### Lecture 21. [Exercise] Insert, Update and Delete a Movie

* Insert fake movie
```
curl -XPUT 127.0.0.1:9200/movies/_doc/20000?pretty -d '
{
    "title": "Sakis Adventures in Elasticsearch",
    "genres": ["Documentary"],
    "year": 2020
}'
```
* search movie by id `curl -XGET 127.0.0.1:9200/movies/_doc/20000?pretty`
* update
```
curl -XPOST 127.0.0.1:9200/movies/_doc/20000/_update -d '
{
  "doc": {
    "genres": ["Documentary","Comedy"]
  }
}'
```
* delete `curl -XDELETE 127.0.0.1:9200/movies/_doc/20000?pretty`

### Lecture 22. Dealing with Concurrency

* concurrency is paramaount when multple clients hit the elasticsearch cluster moding data
* ES handles it with optimistic concurrency control. much like the document version a sequence num is used
* `_seq_no` is associated to the value and `_primary_term` to the sequence number
* if both clients try to update a doc with the same sequence num only one will succeed
* not that write requests go to the primary shards
* client might silently retry if retry on conflict param is used. then he will get the doc with an incremented seq num 
* we test
```
curl -XGET 127.0.0.1:9200/movies/_doc/109487?pretty
```
* we see that `"_seq_no" : 6`, and `"_primary_term" : 2,`
* we will try to write in the same seq num to trigger the conflict
```
curl -XPUT "127.0.0.1:9200/movies/_doc/109487?if_seq_no=6&if_primary_term=2" -d '
{
    "genres": ["IMAX","Sci-Fi"],
    "title":"Interstellar foo",
    "year": 2014
}'
```
* if it fails. get the seq number returned and try again
* we can send a request with retry_on_conflict option so that ES handles retries on its own (we spec max retries)
```
curl -XPOST 127.0.0.1:9200/movies/_doc/109487/_update?retry_on_conflict=5 -d '
{
    "doc": {
        "title": "Interstellar"
    }
}'
```
* so when we expect concurrency issues we should take 1 of the 2 approaches presented here

### Lecture 23. Using Analyzers and Tokenizers

* every time we have a textual field in a document we can choose between 2 definitions
    * search as an exact-match: we use a "keyword" mapping instad of "text". this will suppress analyzing entirely
    * search on analyzed text fields: this will return partial matches. depending on the analyzer chosen results will be case-insensitive, stemmed etc. need not match all terms
* lets search for movies with star trek on title
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "match": {
            "title": "Star Trek"
        }
    }
}'
```
* we get back star trek and star wars movie with a smaller score
* so title is indexed as text
* we do a query based on genre
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "match_phrase": {
            "genre": "sci"
        }
    }
}'
```
* again we get partial results as title is analyzed
* we want exact matches... we delete the movies index `curl -XDELETE 127.0.0.1:9200/movies`
* we reindex
```
curl -XPUT 127.0.0.1:9200/movies -d '
{
    "mappings": {
        "properties": {
            "id": {"type":"integer"},
            "year": {"type": "date"},
            "genre": {"type": "keyword"},
            "title": {"type":"text", "analyzer": "english"}
        }
    }
}'
```
* we reinsert docs in bult from json `curl -XPUT 127.0.0.1:9200/_bulk?pretty --data-binary @movies.json`
* we retry the genre search we dit before we get no results now

### Lecture 24. Data Modeling and Parent/Child Relationships, Part 1

* Strategies for relational data
* in ES we can normalize or denormalize the data
* normalize the data:
    * look up rating => Rating[userID,movieID,rating] => Look up Title => Movie[movieID,title,genres]
    * SQL relational DB style, 
    * minimizes storage, easy to change data, but requires 2 queries
    * transactions matter in response time
* denormalized data:
    * look up rating => Rating[userID,rating,title]
    * titles are duplicated but we got only one query
    * data integrity/consistency issue
* Parent/Child relationships:
    * Star wars => [n new hope,empire strikes back...]

### Lecture 25. Data Modeling and Parent/Child Relationships, Part 2

* we will create an index called seies with a mapping implementing the parent child relationship
```
curl -XPUT 127.0.0.1:9200/series -d '
{
    "mappings": {
        "properties": {
            "film_to_franchise":{
                "type": "join",
                "relations": {
                    "franchise": "film"
                }
            }
        }
    }
}'
```
* we get sample data to test this relationship `wget http://media.sundog-soft.com/es7/series.json`
* data in json look like
```
{ "create" : { "_index" : "series", "_id" : "1", "routing" : 1} }
{ "id": "1", "film_to_franchise": {"name": "franchise"}, "title" : "Star Wars" }
{ "create" : { "_index" : "series", "_id" : "260", "routing" : 1} }
{ "id": "260", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode IV 
- A New Hope", "year":"1977" , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "series", "_id" : "1196", "routing" : 1} }

```
* we see that we add a franchise doc and film docs related to franchise
* we insert the data `curl -XPUT 127.0.0.1:9200/_bulk?pretty --data-binary @series.json`
* say we want to get all films related to star wars franchise 
```
curl -XGET 127.0.0.1:9200/series/_search?pretty -d '
{
    "query": {
        "has_parent": {
            "parent_type": "franchise",
            "query": {
                "match": {
                    "title": "Star Wars"
                }
            }
        }
    }
}'
```
* we will now query for the parent (franchise) based on the child (film) in the relationship
```
curl -XGET 127.0.0.1:9200/series/_search?pretty -d '
{
    "query": {
        "has_child": {
            "type": "film",
            "query": {
                "match": {
                    "title": "The Force Awakens"
                }
            }
        }
    }
}'
```
* So essentialy ES has NoSQL DB schema with Relational capabilities based on the relationship we saw

## Section 3: Searching with Elasticsearch

### Lecture 28. "Query Lite" interface

* up to now we used full body JSON requests to elasticsearch backend
* query lite is good for testing and does not use body. the query is complete in URL
* eg if we want all movies with star in their titles `curl -XGET "127.0.0.1:9200/movies/_search?q=title:star&pretty"`
* eg we want to search movies after 2010 and title containing the word trek `curl -XGET "127.0.0.1:9200/movies/_search?q=+year:>2010+title:trek&pretty"`
* in query like syntax spaces etc need to be URL encoded, eg the above URL does not work because of + and > if we dont use quotes "" (eg in a browser) we need to URL encode them `curl -XGET 127.0.0.1:9200/movies/_search?q=%2Byear%3A%3E2010+%2Btitle%3Atrek`
* Query lite should not be used in production. it can be dagerous
    * cryptic and tough to debug
    * can be a security issue if exposed to users
    * fragile. even one wrong character can break it
* Query lite is formally called URI search

### Lecture 29. JSON Search In-Depth

* Query DSL is the request body as JSON. we have seen that
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "match": {
            "title": "Star Trek"
        }
    }
}'
```
* filters ask a yes/no question of our data
* queries return data in terms of relevance
* filters are faster and cacheable
* example of boolean query with a filter
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "bool": {
            "must": { "term": { "title": "trek"}},
            "filter": {"range": { "year": {"gte": 2010}}}
        }
    }
}'
```
* the equivalnet od binary AND is "must" , "gte" is >
* some types of filters all return boolean:
    * term: filter by exact values `{"term": {"year":2014}}`
    * terms: match if any exact values in a list match `{"terms":{"genre": ["Sci-Fi","Adventure"]}}`
    * range: find numbers or dates in a given range (gt,gte,lt,lte) `{"range": { "year": {"gte": 2010}}}`
    * exists: find documents wher a field exists `{"exists": {"field": "tags"}}`
    * missing: find docs where a filed is missing `{"missing": {"field": "tags"}}`
    * bool: combine filters with boolean logiv (must,must_not,should)
* some types of queries:
    * match_all: returns all documents and is the default. normally used with a filter `{"match_all": {}}`
    * match: searches analyzed results such as full text search `{"match": { "title": "star"}}`
    * multi_match: run the same query on multiple fileds `{"multi_match": {"query": "star","fileds": ["title","synopsis"]}}`
    * bool: works like a boolean filter but results are scored by relevance
* queries are wrapped in a "query": {} block
* filters are wrapped in a "filter": {} block
* we can combine them (see prev example)

### Lecture 30. Phrase Matching

* in ES to search for multiple terms placed in order "phrase" is easy using match_phrase
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "match_phrase":{
            "title": "star wars"
        }
    }
}'
```
* IDF in ES store also the order of terms thus enabling  phrase search
* if we care about terms order but not if they are attached to each other we can use 'slop'
* the slop represents how far we are willing to let a term  move to satisfy a phrase (in either direction). e.g 'quick brown fox' will match 'quick fox' with a slop of 1
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "match_phrase":{
            "title": {"query": "star beyond", "slop": 1}
        }
    }
}'
```
* our queries are sorted by relevance score.
* just use a really high slop if you want to get any documents that contain the words in your phrase, but want docs with the eords closer together score higher

### Lecture 31. [Exercise] Querying in Different Ways

* search for Star Wars movies released after 1980
    * URI search (query lite)
```
curl -XGET "127.0.0.1:9200/movies/_search?q=+year:>1980+title:star%20wars&pretty"
```
    * we dont get correct results. we need to URL encode all special chars `curl -XGET "127.0.0.1:9200/movies/_search?q=%2Byear%3A%3E1980+%2Btitle%3Astar%20wars&pretty"`
    * Query JSON body
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "bool": {
            "must": {"match_phrase":{"title": "star wars"}},
            "filter": {"range": { "year": {"gte": 1980}}}
        }
    }
}'
```

### Lecture 32. Pagination

* specify "from" and "size" to retrieve paginated results..
* from is 0 based
* URL style
```
curl -XGET "127.0.0.1:9200/movies/_search?size=2&from=2&pretty"
```
* or JSON body style w/ query
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "from": 2,
    "size": 2,
    "query": {"match":{"genre": "Sci-Fi"}}
}'
```
* deep pagination can kill performance
* every result must be retrieved,collated and sorted
* enforce an uppoer bound on how many results will return to users
* ommiting from implies 0

### Lecture 33. Sorting

* quite simple `curl -XGET '127.0.0.1:9200/movies/_search?sort=year&pretty'`
* a string field that is analyzed for full-text search cant be used to sort documents
* this is because it exists in the inverted index as individual terms, not as entire string
* if we want to sort on an analyzed text field, we need to map a keyword copy, 
* in the example below we have 2 copies of title field index one analyzed and one not for sorting
* raw is a subfield like title.raw
```
curl -XPUT 127.0.0.1:9200/movies/ -d '
{
    "mappings": {
        "properties": {
            "title": {
                "type": "text",
                "fields": {
                    "raw": {
                        "type": "keyword"
                    }
                }
            }
        }
    }
}'
```
* with the new maping to sort on title we use `curl -XGET '127.0.0.1:9200/movies/_search?sort=title.raw&pretty'`
* we cannot change the mapping on an existing index. we need to reindex

### Lecture 34. More with Filters

* a comlex filter query example (must be sci-fi, must not contain trek in title, must be between 2010 and 2015)
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d'
{
  "query": {
      "bool": {
          "must": {"match": {"genre": "Sci-Fi"}},
          "must_not": {"match": {"title": "trek"}},
          "filter": {"range": {"year": {"gte": 2010, "lt": 2015}}}
      }
  }
}'
```

### Lecture 35. [Exercise] Using Filters

* search for sci-fi movies before 1960, sorted by title
```
curl -XGET '127.0.0.1:9200/movies/_search?sort=title.raw&pretty' -d'
{
    "query": {
        "bool": {
            "must": {"match":{"genre": "Sci-Fi"}},
            "filter": {"range": {"year": {"lt": 1960}}}
        }
    }
}
```

### Lecture 36. Fuzzy Queries

* Fuzzy matches is a way to account for typos and misspellings
* the 'levenstein edit distance' accounts for:
    * substitutions of characers (interstellar => instersteller)
    * insertions of characters (intertellar => intersstellar)
    * deletion of characters (interstellar => insterstelar)
* all the above examples have an edit distance of 1
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d'
{
    "query": {
        "fuzzy": {
            "title": {
                "value": "intrsteller","fuzziness": 2
            }
        }
    }
}'
```
* fuzziness: AUTO 
    * 0 for 1-2 character strings
    * 1 for 3-5 char strings
    * 2 for anything else

### Lecture 37. Partial Matching

* Prefix queries
    * it works with prefix queries on strings
    * if we remapped 'year' as string we could query for 2010 - 2019 with
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "prefix": {
            "year": "201"
        }
    }
}'
```

* Wildcard queries
    * use * as windcard
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "wildcard": {
            "year": "1*"
        }
    }
}'
```
* Regexp queries
    * use "regexp" and a regular expression 

* we delete and remap index to test
```
curl -XDELTE 127.0.0.1:9200/movies
curl -XPUT 127.0.0.1:9200/movies -d '
{
      "mappings": {
          "properties": {
              "year": {
                  "type": "text"
              }
          }
      }
}'
curl -XPUT 127.0.0.1:9200/_bulk --data-binary @movies.json
```

### Lecture 38. Query-time Search As You Type

* cool feat google search style
* an easy way to do it is to abuse sloppines with prefix
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "match_phrase_prefix":{
            "title": {"query": "star", "slop": 10}
        }
    }
}'
```

### Lecture 39. N-Grams, Part 1

* prefix is prefeed to be done at index time by building n-grams
* this speeds up queries
* "star":
    * unigram: [s,t,a,r]
    * bigram: [st,ta,ar]
    * trigram: [sta,tar]
    * 4-gram: [star]
* edge n-grams are built only on the beginning of each term. they are very useful as we type start to end
* * edge n-grams for star s,st,sta,star
* if we treat our input in the search text box as n-gram we can match it agains our index
* even 1 letter gets a match from unigram of star
* create an autocomplete analyzer
```
curl -XPUT 127.0.0.1:9200/movies/?pretty -d '
{
  "settings": {
    "analysis": {
        "filter": {
            "autocomplete_filter": {
                "type": "edge_ngram",
                "min_gram": 1,
                "max_gram": 20
            }
        },
        "analyzer": {
            "autocomplete": {
                "type": "custom",
                "tokenizer": "standard",
                "filter": [
                    "lowercase",
                    "autocomplete_filter"
                ]
            }
        }
    }
  }
}'
```
* we use it in our index mapping at index time (we need to reindex)
```
curl -XPUT 127.0.0.1:9200/movies/_mapping?pretty -d '
{
    "properties": {
        "title": {
            "type": "text",
            "analyzer": "autocomplete"
        }
    }
}'
```
* our queries must be properly formated to use a standard analyzer NOT the autocomplete
* we should use the n-grams only at index size otherwise tour query will also get split into ngrams and we ll get results for everything tha matches s,t,a,st,ta etc
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "match": {
            "title": {
                "query": "sta",
                "analyzer": "standard"
            }
        }
    }
}'
```
* we can also upload a list of all possible completions ahead of time using 'completion suggesters'
* this is also google style

### Lecture 40. N-Grams, Part 2

* delete index add analyzer
```
curl -XDELETE 127.0.0.1:9200/movies
curl -XPUT 127.0.0.1:9200/movies/?pretty -d '
{
  "settings": {
    "analysis": {
        "filter": {
            "autocomplete_filter": {
                "type": "edge_ngram",
                "min_gram": 1,
                "max_gram": 20
            }
        },
        "analyzer": {
            "autocomplete": {
                "type": "custom",
                "tokenizer": "standard",
                "filter": [
                    "lowercase",
                    "autocomplete_filter"
                ]
            }
        }
    }
  }
}'
```
* test the analyzer
```
curl -XGET 127.0.0.1:9200/movies/_analyze?pretty -d '
{
    "analyzer": "autocomplete",
    "text": "Sta"
}'
```
* we map the analyzer to the title field
```
curl -XPUT 127.0.0.1:9200/movies/_mapping?pretty -d '
{
    "properties": {
        "title": {
            "type": "text",
            "analyzer": "autocomplete"
        }
    }
}'
```
* load in the data `curl -XPUT 127.0.0.1:9200/_bulk --data-binary @movies.json`
* do a query with the analyzer
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "match": {
            "title": {
                "query": "sta",
                "analyzer": "standard"
            }
        }
    }
}'
```

## Section 4: Importing Data into your Index - Big or Small

### Lecture 43. Importing Data with a Script

8 