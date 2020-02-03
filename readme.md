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
            "autocomplete_filter":{
                "type": "edge_ngram",
                "min_gram": 1,
                "max_gram": 20
            }
        },
        "analyzer": {
            "autocomplete": {
                "type": "custom",
                "tokenizer": "standard",
                "filter": {
                    "lowercase",
                    "autocomplete_filter"
                }
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

* reindex

### Lecture 43. Importing Data with a Script

* we can import data to ES from almost anything
    * standalone scripts can submit bulk documents via RESTful API
    * logstash and beats can stream data from logs, S3, databases, and more
    * AWS systems can stream data via lambda or kinesis firehose
    * kafka,spark and more frameworks have ES integration add-ons
* importing data via script as json files
* a python script can be used to:
    * read in data from some distributed system (csv)
    * transform data to JSON bulk inserts
    * submit data to ES custer via HTTP/REST requests
* our python script
```
import csv
import re

csvfile = open('ml-latest-small/movies.csv', 'r')

reader = csv.DictReader( csvfile )
for movie in reader:
        print ("{ \"create\" : { \"_index\": \"movies\", \"_id\" : \"" , movie['movieId'], "\" } }", sep='')
        title = re.sub(" \(.*\)$", "", re.sub('"','', movie['title']))
        year = movie['title'][-5:-1]
        if (not year.isdigit()):
            year = "2016"
        genres = movie['genres'].split('|')
        print ("{ \"id\": \"", movie['movieId'], "\", \"title\": \"", title, "\", \"year\":", year, ", \"genre$
        for genre in genres[:-1]:
            print("\"", genre, "\",", end='', sep='')
        print("\"", genres[-1], "\"", end = '', sep='')
        print ("] }")
```
* we install unzip `sudo apt install unzip`
* we get the dataset `wget http://files.grouplens.org/datasets/movielens/ml-latest-small.zip`
* unzip the file `unzip ml-latest-small.zip`
* we dl the python script `wget media.sundog-soft.com/es7/MoviesToJson.py` which builds the JSon file for import
* we run it piping the print to a file `python3 MoviesToJson.py > moremovies.json`
* we remove the index `curl -XDELETE 127.0.0.1:9200/movies`
* we import the new dataset `curl -XPUT 127.0.0.1:9200/_bulk --data-binary @moremovies.json`
* we search for a movie `curl -XGET "127.0.0.1:9200/movies/_search?q=mary%20poppins&pretty"`

### Lecture 44. Importing with Client Libraries

* using custom scripting is cryptic and hard
* free ES client libs are available for any programming language
    * java has a clinet maintained by elastic.co
    * python has an ES package
    * elasticsearch-ruby for ruby
    * several choices for scalable
    * elascisearch.pm for pearl
* we will use again a python script but using the Python ES library
* in this script we dont just build JSON, we read from csv, and interact with ES cluster to insert the data and issue gueries
* python dictionaty is used.
* we also do denormalization of data NoSQL style
* in our ubuntu instance we have python3
* we need pip to install the elasticsearch python package
```
sudo apt install python3-pip
pip3 install elasticsearch
```
* we get the script `wget media.sundog-soft.com/es7/IndexRatings.py`
```
import csv
from collections import deque
import elasticsearch
from elasticsearch import helpers

def readMovies():
    csvfile = open('ml-latest-small/movies.csv', 'r')

    reader = csv.DictReader( csvfile )

    titleLookup = {}

    for movie in reader:
            titleLookup[movie['movieId']] = movie['title']

    return titleLookup

def readRatings():
    csvfile = open('ml-latest-small/ratings.csv', 'r')

    titleLookup = readMovies()

    reader = csv.DictReader( csvfile )
    for line in reader:
        rating = {}
        rating['user_id'] = int(line['userId'])
        rating['movie_id'] = int(line['movieId'])
        rating['title'] = titleLookup[line['movieId']]
        rating['rating'] = float(line['rating'])
        rating['timestamp'] = int(line['timestamp'])
        yield rating


es = elasticsearch.Elasticsearch()

es.indices.delete(index="ratings",ignore=404)
deque(helpers.parallel_bulk(es,readRatings(),index="ratings"), maxlen=0)
es.indices.refresh()
```
* we run the script `python3 IndexRatings.py`
* we fetch some docs `curl -XGET 127.0.0.1:9200/ratings/_search?pretty`

### Lecture 45. [Exercise] Importing with a Script

* write a python script to import the tags.csv data from ml-latest-small into a new "tags" index
```
import csv
from collections import deque
import elasticsearch
from elasticsearch import helpers

def readMovies():
    csvfile = open('ml-latest-small/movies.csv', 'r')

    reader = csv.DictReader( csvfile )

    titleLookup = {}

    for movie in reader:
            titleLookup[movie['movieId']] = movie['title']

    return titleLookup

def readTags():
    csvfile = open('ml-latest-small/tags.csv', 'r')

    titleLookup = readMovies()

    reader = csv.DictReader( csvfile )
    for line in reader:
        tag = {}
        tag['user_id'] = int(line['userId'])
        tag['movie_id'] = int(line['movieId'])
        tag['title'] = titleLookup[line['movieId']]
        tag['tag'] = line['tag']
        tag['timestamp'] = int(line['timestamp'])
        yield tag


es = elasticsearch.Elasticsearch()

es.indices.delete(index="tags",ignore=404)
deque(helpers.parallel_bulk(es,readTags(),index="tags"), maxlen=0)
es.indices.refresh()
```
* we save it as IndexTags.py and run it `python3 `
* we test `curl -XGET 127.0.0.1:9200/tags/_search?pretty`

### Lecture 46. Introducing Logstash

* logstash can get data from various sources: files, s3, beats, kafka
* logstash can insert data to: elasticsearch, aws, hadoop, mongodb
* it supports multiple inputs and multiple outputs
* logstash parses, transforms, and filters data as it passes them through
* it can derive structure from unstructured data
* it can anonymize personal data or exclude it entirely
* it can do geolocation lookups
* it can scale across many nodes
* it guarantees at-least-once delivery
* it absorbs throughput from load spikes
* can see the [logstash filter plugins list](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html)
* logstash can listen to a huge variety of input source events and a huge variety of stash destinations
* typical use case in ELK stack: web logs (beats) OR file => Logstash (parse data into structuered fields, geolocate) => elasticsearch

### Lecture 47. Installing Logstash

* we get the apache access.log `wget media.sundog-soft.com/es/access_log`
* we install the jdk `sudo apt install openjdk-8-jre-headless`
* assuming we have the ES repo added we install logstash `sudo apt-get update && sudo apt-get install logstash`
* we need to add a configuration file `sudo vi /etc/logstash/conf.d/logstash.conf` or `sudo nano /etc/logstash/conf.d/logstash.conf`
```
input {
    file {
        path => "/home/ubuntu/access_log"
        start_position => "beginning"
    }
}

filter {
    grok {
        match => {"message" => "%{COMBINEDAPACHELOG}"}
    }
    date {
        match => ["timestamp","dd/MMM/yyyy:HH:mm:ss Z"]
    }
}

output {
    elasticsearch {
        hosts => ["localhost:9200"]
    }
    stdout {
        codec => rubydebug
    }
}
```
* the above config file
    * it tells logstash where to find its input data
    * setting beginning it will process file from beginning. default behaviour is to tail it
    * filter block tells logstash how to extract structured data out of the log (uses a inbuild plugin for apache logs)
    * output sends it to elacticsearch and to stdout

### Lecture 48. Running Logstash

* we cd into the dir and run the binary using the config file
```
cd /usr/share/logstash/
sudo bin/logstash -f /etc/logstash/conf.d/logstash.conf
```
* we see the output of the data load
* once the log stops we ctrl+c to stop as it will wait for new lines in log file to parse
* we want to see the ES indices created by the log parsing `curl -XGET 127.0.0.1:9200/_cat/indices?v`
* logstash index is 'logstash-2020.01.31-000001' as it is date based...
* we get the docs `curl -XGET "127.0.0.1:9200/logstash-2020.01.31-000001/_search?pretty"`

### Lecture 49. Logstash and MySQL, Part 1

* we will use logstash to extract and feed data from a MySQL DB table
* first we update system `sudo apt-get update`
* then install mysql `sudo apt-get install mysql-server`
* we download movielens data. `wget http://files.grouplens.org/datasets/movielens/ml-100k.zip`
* unzip data `unzip ml-100k.zip`
* we connect to db `sudo mysql -u root -p`
* in mysqll console
    * we create a DB `CREATE DATABASE movielens;`
    * we create a table `CREATE TABLE movielens.movies ( movieID INT PRIMARY KEY NOT NULL, title TEXT, releaseDate DATE);`
    * we load data to the table `LOAD DATA LOCAL INFILE 'ml-100k/u.item' INTO TABLE movielens.movies FIELDS TERMINATED BY '|' (movieID, title, @var3) set releaseDate = STR_TO_DATE(@var3, '%d-%M-%Y');`
    * we go to db `use movielens;`
    * issue a simple query `SELECT * FROM movies WHERE title LIKE 'Star%';`
    * exit mysql `exit`

### Lecture 50. Logstash and MySQL, Part 2

* we need a JDBC connector for mySQL `wget https://dev.mysql.com/get/Doonloads/Connector-J/mysql-connector-java-8.0.15.zip`
* then unzip it `unzip mysql-connector-java-8.0.15.zip`
* we need to configure logstash to use jdbc adding the `sudo nano /etc/logstash/conf.d/mysql.conf` file adding the credentials to connect to DB
```
input {
    jdbc {
        jdbc_connection_string => "jdbc:mysql://localhost:3306/movielens"
        jdbc_user => "ubuntu"
        jdbc_password => "password"
        jdbc_driver_library => "/home/ubuntu/mysql-connector-java-8.0.15/mysql-connector-java-8.0.15.jar"
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        statement => "SELECT * FROM movies"
    }
}

output {
    stdout { codec => json_lines }
    elasticsearch {
        hosts => ["localhost:9200"]
        index => "movielens-sql"
    }
}
```
* we connect to the mysql to add the user we set in config `sudo mysql -u root -p`
* add user `CREATE USER 'ubuntu'@'localhost' IDENTIFIED BY 'password';`
* grant permissions `GRANT ALL PRIVILEGES ON *.* TO 'ubuntu'@'localhost';`
* apply privileges `FLUSH PRIVILEGES;` and `quit`
* we cd to logstash bin `cd /usr/share/logstash` 
* and run it using config `sudo bin/logstash -f /etc/logstash/conf.d/mysql.conf` 
* test the data entry searching in elasticsearch `curl -XGET '127.0.0.1:9200/movielens-sql/_search?q=title:Star&pretty'`

### Lecture 51. Logstash and S3

* AWS S3 (Amazon Web Services Simple STorage Service) is a cloud based distributed storage system
* to set logstash to use data from S3 we need to config its input as
```
input {
    s3 {
        bucket => 'sundog-es'
        access_key_id => "AKIAIS****C26Y***Q"
        secret_access_key => "d*****FENOXcCuNC4iTbSLbibA*****eyn****"
    }
}
```
* we need to spec the bucket name and the access keys
* we upload there the access-log as `acces-log.txt`
* we use IAM fr securing S3 bucket
* we set a user with AmazonS3ReadOnlyAccess rights
* we create security credentials for the user (access-key and secret access key)
* as we are using the same apache access log we mod the `/etc/logstash/conf.d/logstash.conf` file replacing the input part... filter and output are the same
* and the logstash run command

### Lecture 52. Elasticsearch and Kafka, Part 1

* apache kafka
    * an open source stream processing platorm
    * hight throughput - low latency
    * publish / subscribe
    * process streams
    * store streams

* many things in common with logstash
* getting data from kafka to ES with logstash is as simple as adding the right input config part in the config file used by logstash e.g
```
input {
    kafka {
        bootstrap_servers => "localhost:9092"
        topics => ["kafka-logs"]
    }
}
```
* a topic is a kafka channel
* we setup kafka on our host
* install zookeeper `sudo apt-get install zookeeperd`
* we get kafka installation file on home dir `wget http://www.trieuvan.com/apache/kafka/2.2.0/kafka_2.12-2.2.0.tgz`
* we uncompress `tar -xvf kafka_2.12-2.2.0.tgz`
* we cd in kafka dir `cd kafka_2.12-2.2.0` and start the server `sudo bin/kafka-server-start.sh config/server.properties`
with kafka running we open a new terminal and cd to kafka to create a new topic `sudo bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic kafka-logs`
* replication and partition is 1 as our kafka cluster has 1 node

### Lecture 53. Elasticsearch and Kafka, Part 2

* with the kafka stream ready we need to config logstash to listen for this stream `sudo nano /etc/logstash/conf.d/logstash.conf`
```
input {
    kafka {
        bootstrap_servers => "localhost:9092"
        topics => ["kafka-logs"]
    }
}

filter {
    grok {
        match => {"message" => "%{COMBINEDAPACHELOG}"}
    }
    date {
        match => ["timestamp","dd/MMM/yyyy:HH:mm:ss Z"]
    }
}

output {
    elasticsearch {
        hosts => ["localhost:9200"]
        index => "kafka-logs"
    }
    stdout {
        codec => rubydebug
    }
}
```

* we cd to logstash bin `cd /usr/share/logstash` 
* and run it using config `sudo bin/logstash -f /etc/logstash/conf.d/logstash.conf` 
* kafka server is ready and listening and logstash is listening to the stream to stream it to elasticsearch
* when we publish data to the kafka topic they will inserted to elasticsearch
* we open a 3rd terminal to publish to kafka
* we cd to kafka `cd kafka_2.12-2.2.0/`
* we publish the access-log file into kafka-logs topic using the console producer tool `sudo bin/kafka-console-producer.sh --broker-list localhost:9092 --topic kafka-logs < ../access_log`
* we test with `curl -XGET '127.0.0.1:9200/kafka-logs/_search?pretty'`

### Lecture 54. Elasticsearch and Apache Spark, Part 1

* Apache Spark
    * a fast and general engine for large scale data processing
    * a faster alternative to mapreduce
    * spark apps are written in java,scala,python or r
    * supports SQL, streaming, machine learning and graph processsing
    * usually runs on hadoop cluster
* we will install apache spark and write a script to show integration with elasticsearch
* the script will read a csv convert it to a dataframe and load it to ES
* to do it we need a spark addon package 'elasticsearch-spark-20' `./spark-2.4.2-bin-hadoop2.7/bin/spark-shell --packages org.elasticsearch:elasticsearch-spark-20_2.11:7.0.1`
*  first we need to install [spark](https://spark.apache.org) and download spark.
* dont get latest version. it must support the scala version used in the package `wget https://archive.apache.org/dist/spark/spark-2.3.3/spark-2.3.3-bin-hadoop2.7.tgz`
* decompress `tar -xvf spark-2.3.3-bin-hadoop2.7.tgz`
* next we need to install the spark-es integration package. go to [maven repo](https://mvnrepository.com/search?q=elasticsearch)
* we need 'Elasticsearch Spark (for Spark 2.X)' see the compatibility notes
* first we get a fake data csv `wget https://raw.githubusercontent.com/PacktPublishing/Frank-Kanes-Taming-Big-Data-with-Apache-Spark-and-Python/master/fakefriends.csv`
* we run spark passing in the es package `./spark-2.3.3-bin-hadoop2.7/bin/spark-shell --packages org.elasticsearch:elasticsearch-spark-20_2.11:7.0.1`

### Lecture 55. Elasticsearch and Apache Spark, Part 2

* spark is up and running and we see the scala prompt
* our commands are in scala
* in the command promt we explicitly import the package `import org.elasticsearch.spark.sql._`
* we create a Person class to store the data for each user `case class Person(ID:Int,name:String,age:Int,numFriends:Int)`
* we add a mapper scala function to take a line from the csv file and convert it to Person object
```
def mapper(line:String): Person = {
    val fields = line.split(',')
    val person:Person = Person(fields(0).toInt,fields(1),fields(2).toInt,fields(3).toInt)
    return person
}
```
* load csv file 
* we `import spark.implicits._` to use  alib to convert data to dataframe objects
* we load the csv to a lines obj `val lines = spark.sparkContext.textFile("fakefriends.csv")`
* not that our code is not executed... we are merely building a graph for later execution..
* we convert lines to people `val people = lines.map(mapper).toDF()`
* we load the data to ES `people.saveToEs("spark-friends")`
* exit shell `:quit` and test `curl -XGET 127.0.0.1:9200/spark-friends/_search?pretty`
* IT WORKSSS

### Lecture 56. [Exercise] Importing Data with Spark

* write spark code that imports movie ratings from ml-latest-small into a "ratings" index in ES
* note that we have a header Row we need to delete
```
val header = lines.first()
val data = lines.filter(row => row != header)
```
* fire up spark with the pluginm
```
import org.elasticsearch.spark.sql._
case class Rating(userI:Int,movieId:Int,rating:Float,timestamp:Int)
def mapper(line:String): Rating = {
    val fields = line.split(',')
    val rating:Rating = Rating(fields(0).toInt,fields(1).toInt,fields(2).toFloat,fields(3).toInt)
    return rating
}
import spark.implicits._
val lines = spark.sparkContext.textFile("ml-latest-small/ratings.csv")
val header = lines.first()
val data = lines.filter(row => row != header)
val ratings = data.map(mapper).toDF()
ratings.saveToEs("ratings")
:quit
```

* test `curl -XGET 127.0.0.1:9200/ratings/_search?pretty`

## Section 5: Aggregation

### Lecture 58. Section 5 Intro

* ES is very powerful in analyzing big data... much more than hadoop
* the key to analyze data in ES is aggregations

### Lecture 59. Aggregations, Buckets, and Metrics

* in most orgs ES is used more for aggregations than searching in big data
* ES cluster can be used to do data analytics
    * metrics (average, stats, min/max, percentiles. etc)
    * buckets (histograms, ranges, distances, significant terms)
    * pipelines (moving average, average bucket, cumulative sum)
    * matrix (matrix stats)
* ES aggregations can sometimes take the place of hadoop / spark / etc because it returns results much faster
* we can even nest aggregations together or pair them with search queries
* say we want to bucket the movielens docs by rating value
* in the example
```
curl -XGET '127.0.0.1:9200/ratings/_search?size=0&pretty' -d '
{
    "aggs": {
        "ratings": {
            "terms": {
                "field": "rating"
            }
        }
    }
}'
```
* in this example we bucket all the movies in movielens dataset by the rating value
* size=0 means we dont want back the query results only the aggregations
* we create an aggregation called ratings that aggregates on the rating field term
* in a nutshell it sums up all documents that have the same rating value
* it can be used then to build a histogram
* in the following example
```
curl -XGET '127.0.0.1:9200/ratings/_search?size=0&pretty' -d '
{
    "query": {
        "match": {
            "rating": 5.0
        }
    },
    "aggs": {
        "ratings": {
            "terms": {
                "field": "rating"
            }
        }
    }
}'
```
* here we count only the 5 star rated ratings
* in the example below we get the average rating for the movie that matches the phrase Star Wars Episode IV
```
curl -XGET '127.0.0.1:9200/ratings/_search?size=0&pretty' -d '
{
    "query": {
        "match_phrase": {
            "title": "Star Wars Episode IV"
        }
    },
    "aggs": {
        "avg_rating": {
            "avg": {
                "field": "rating"
            }
        }
    }
}'
```
* we will use the ratings index we imported from spark
* we try first example we get back JSON with doc counts for each rating in a bucket array

```
{
  "took" : 76,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "ratings" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 4.0,
          "doc_count" : 53636
        },
        {
          "key" : 3.0,
          "doc_count" : 40094
        },
        {
          "key" : 5.0,
          "doc_count" : 26422
        },
....................
```
* in the second example we get as reply
```
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "ratings" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 5.0,
          "doc_count" : 26422
        }
      ]
    }
  }
}
```
* it takes 3ms.....
* the reply for the 3rd example is
```
{
  "took" : 52,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 251,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "avg_rating" : {
      "value" : 4.231075697211155
    }
  }
}
```

### Lecture 60. Histograms

* with aggregations we can very quickly create histograms
* histograms: display totals of documents bucketed by some internal range
* in example if we want to display ratings totals bucketed by 1.0-rating intervals
```
curl -XGET '127.0.0.1:9200/ratings/_search?size=0&pretty' -d'
{
    "aggs": {
        "whole_ratings": {
            "histogram": {
                "field": "rating",
                "interval": 1.0
            }
        }
    }
}'
```
* histogram is the type of aggregation
* another example is crating a histogram with movies released in each decade
```
curl -XGET '127.0.0.1:9200/movies/_search?size=0&pretty' -d '
{
    "aggs": {
        "release": {
            "histogram": {
                "field": "year",
                "interval": 10

            }
        }
    }
}'
```
* we get back same style replies with buckets with doc counts

### Lecture 61. Time Series

* a server log is a time series data as they are timestamped
* ES can bucket and aggregate fields that contain time and dates properly. We can aggregate by month or year and it knows about calendar rules
* an example of how to breakdown website hits by hour (access logs we streamed with kafka)
```
curl -XGET '127.0.0.1:9200/kafka-logs/_search?size=0&pretty' -d '
{
    "aggs": {
        "timestamp": {
            "date_histogram": {
                "field": "@timestamp",
                "interval": "hour"
            }
        }
    }
}'
```
* the type is date-histogram by hour
* what we get is a doc count of events per hour
```
{
  "took" : 42,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "timestamp" : {
      "buckets" : [
        {
          "key_as_string" : "2017-04-30T04:00:00.000Z",
          "key" : 1493524800000,
          "doc_count" : 225
        },
        {
          "key_as_string" : "2017-04-30T05:00:00.000Z",
          "key" : 1493528400000,
          "doc_count" : 375
        },
...............
```

*  another example is the same aggregation but with a query before to filter events. is the traffic that Googlebots creates to our site when it does scraping....
```
curl -XGET '127.0.0.1:9200/kafka-logs/_search?size=0&pretty' -d '
{
    "query": {
        "match": {
            "agent": "Googlebot"
        }
    },
    "aggs": {
        "timestamp": {
            "date_histogram": {
                "field": "@timestamp",
                "interval": "hour"
            }
        }
    }
}'
```

### Lecture 62. [Exercise] Generating Histogram Data

* when did my site went down on May 5 2017??? 
* bucket 500 status codes by the minute in kafka logs
```
curl -XGET '127.0.0.1:9200/kafka-logs/_search?size=0&pretty' -d '
{
    "query": {
        "match": {
            "response": "500"
        }
    },
    "aggs": {
        "timestamp": {
            "date_histogram": {
                "field": "@timestamp",
                "interval": "minute"
            }
        }
    }
}'
```
* we see it in reply
```
        {
          "key_as_string" : "2017-05-05T11:19:00.000Z",
          "key" : 1493983140000,
          "doc_count" : 55
        },

```

### Lecture 63. Nested Aggregations, Part 1

* aggregations can be nested for more powerful queries
* eg whats the average rating for each Star Wars movie?
we will undertake this task as an acttivity to show what can go wrong along the way
* when we aggregate on text fields many things can go wrong
* we want to get the average rating of each individual star wars film
* the complete query is
```
curl -XGET '127.0.0.1:9200/ratings/_search?size=0&pretty' -d '
{
    "query": {
        "match_phrase": {
            "title": "Star Wars"
        }
    },
    "aggs": {
        "titles": {
            "terms": {
                "field": "title"
            },
            "aggs": {
                "avg_rating": {
                    "avg": {
                        "field": "rating"
                    }
                }
            }
        }
    }
}'
```
* first we get all star wars movies with the query
* we will aggregate on the query results based on raw title
* next we will nest an aggregation to get the average ratings for each
* note that the query result is all ratings for movieswith star wars in title
* the first aggregation buckets these ratings per raw title.
* the on those buckets we chain the second aggregation to get the avg ratings

### Lecture 64. Nested Aggregations, Part 2

* we run it and it does not work. what we get back is 
```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "illegal_argument_exception",
        "reason" : "Fielddata is disabled on text fields by default. Set fielddata=true on [title] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
      }
    ],
..............
```
* we cannot aggregate on a stragit analyzed text field
* we follow advice and enable fielddata on title field... we dont even have to do reindexing for this
```
curl -XPUT '127.0.0.1:9200/ratings/_mapping?pretty' -d '
{
    "properties": {
        "title": {
            "type":"text",
            "fielddata": true
        }
    }
}'
```
* we rerun the query but the results are wrong...
* we get avg ratings for docs that contain a term of the title...
* if we had type keyword... it would work but we need to reindex and reload
* we aso do it on a subfield..
```
curl -XDELETE 127.0.0.1:9200/ratings
curl -XPUT 127.0.0.1:9200/ratings/ -d '
{
    "mappings": {
        "properties": {
            "title": {
                "type": "text",
                "fielddata": true,
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
* we mod the IndexRating.py xcript as we dont need to delete the index `#es.indices.delete(index="ratings",ignore=404)`
* we load data `python3 IndexRatings.py`
* we rerun the query chanding the filed from title to title.raw as it is keyword type
```
curl -XGET '127.0.0.1:9200/ratings/_search?size=0&pretty' -d '
{
    "query": {
        "match_phrase": {
            "title": "Star Wars"
        }
    },
    "aggs": {
        "titles": {
            "terms": {
                "field": "title.raw"
            },
            "aggs": {
                "avg_rating": {
                    "avg": {
                        "field": "rating"
                    }
                }
            }
        }
    }
}'
```
* 18 F...... ms to run...

## Section 6: Using Kibana

### Lecture 67. Installing Kibana

* Kibana is a web ui that uses ES aggregations agains the ES cluster indices to show nice visualizations
* to install it on the host
```
sudo apt-get install kibana
sudo nano /etc/kibana/kibana.yml
```
* change server host to 0.0.0.0
* run it
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable kibana.service
sudo /bin/systemctl start kibana.service
```
* Kibana is now available on port 5601
* service now will start when we start our vm
* we hot it externaly on <vmip>:5601 in a browser

### Lecture 68. Playing with Kibana

* we will analyze the works of Shakespeare.. why??? because we can...
* we click the management icon in left side menu
* we need to setup an index pattern that will allow us to specify the index we want to use
* index patterns allow us to group indexes
* we select 'Create Index Pattern' and insert 'shakespeare' => Next Step => Create index Pattern
* we see the fields available and their properties
* we click the compass to go to Discover mode
* we see raw query response
* in filter we can enter search words and see the filtered results
* if we click 'play_name' we see the plays where the keyword appears the most (with hits and %)
* clicking visualize we get a visualization
* clicking + on a lsted play we get the occurences of keyword for this play in the list...
* we click on visuzalize => create new visualization and select a type => tag cloud => index=shakespeare 
    * tag size count: aggregation => count
    * buckets => ADD BUCKET (TAGS) => aggregation => terms => filed: textentry.keyword. order=> descentding 50 (top 50 results)
* we see Dev Tools there we can execute raw JSON request to the ES cluster and avoid curl (more like postman)

### Lecture 69. [Exercise] Exploring Data with Kibana

* task: find the longest shakespeare play - create a vertical bar chart that aggregates the count of documents by play name in dscending order
* visualize => create visualization => vertical barchart => shakespeare indx
    * y axis : count
    * x axis: buckets => xaxis, aggregation: terms, field: play_name, order: descent 50

## Section 7: Analyzing Log Data with the Elastic Stack

### Lecture 72. FileBeat and the Elastic Stack Architecture

* Filebeat is a lightweight shipper for logs
* it stands between logstash and files
* it is installed on the source machine offering fault tolerance by buffering in case of loss of command
* filebeat can optionally talk directly to elasticsearch. when using logstash ES is one of the many possible destinations for filebeat
* logstash and filebeat communicate to maintain "backpressure" when things  back up
* filebeat maintains a read pointer on the logs. every log line acts like a queue
* logs can be from apache, nginx, auditd or mysql
* its very light on servers
* before beats we talked about the ELK stack
* why use filebeat+logstash?
    * it wont let us overload the pipelines
    * we get more flexibility on scaling our cluster
* backpressure: 
    * logstash might not be able to cope with all logs coming in. 
    * filebeat does not move the read pointer faster than logstash can handle
    * logstash comms with filebeats on a backpressure sensitive protocol. logstash gives the rate to filebeat
* we dont have to overscale to handle transient peaks (less money)
* tipical cluster: beats on each source +  logstash modes (wit persistent queues) + elasticsearch cluster (master nodes, hot data nodes, warm data nodes) + kibana frontend
* persistent queues guarantee at least once delivery and fast recovery from nodes failures. logstash will try to push from persistent quess untill delivery succeeds at least once

### Lecture 73. X-Pack Security

* Security: access control, data integrity, audit trails
* Licenced version of X-pack offers many security feats (RBAC, IP filtering, Controlled access)
* Security/Auth => users, groups, and roles
    * RBAC
    * Assign privileges to roles
    * Assign roles to users and groups
    * Control access to indices, aliases, documents or fields
    * Can use Active Dir or LDAP realms via Distinguished names (dn's) for users
* example of enforcing rules via x-pack
```
PUT /_security/role_mapping/admins
{
    "roles": ["monitoring", "user"],
    "rules": { "field": { "groups": "cn=admins,dc=example,dc=com" } },
    "enabled": true
}

PUT /_security/role_mapping/basic_users
{
    "roles": ["user"],
    "rules": { "any": [
        {"field": { "dn": "cn=John Doe,cn=contractors,dc=example,dc=com"}}.
        {"field": { "groups": "cn=users,dc=example,dc=com"}}
    ]},
    "enabled": true
}
```
* define a role that willl be used to users and groups
```
POST /_security/role/users
{
    "run_as": ["user_impersonator"],
    "cluster": ["movielens"],
    "indices": [
        {
            "names": ["movies"],
            "privileges": ["read"],
            "field_security": {
                "grant": ["title","year","genres"]
            }
        }
    ]
}
```
* this creates a user giving him rights to access the cluster for specific data on his behalf. the permissions are very fine grained
* permissions on RBAC basis are available in Kibana but on a paid x-pack scheme

### Lecture 74. Installing FileBeat

* installing and testing filebeat (assuming elastic search repo in the package repo)
```
cd ~
sudo apt-get update && sudo apt-get install filebeat
sudo /bin/systemctl stop elasticsearch.service
sudo /bin/systemctl start elasticsearch.service

mkdir ~/logs
cd /logs
wget http://media.sundog-soft.com/es7/access_log

cd /etc/filebeat/modules.d
sudo mv apache.yml.disabled apache.yml
sudo nano apahe.yml
```
* add in file
```
["home/ubuntu/logs/access"]
["home/ubuntu/logs/error"]
```
* run flebeats
```
sudo /bin/systemctl start filebeat.service
```
* the apache access_log will be consumed by filebeat and passed in elasticsearch (no logstash in between)