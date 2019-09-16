# Elastic Search
### Table of Contents
1. [Docker Commends](#docker-commends)
2. [ES Syntax](#es-syntax)
3. [Text Analysis](#text-analysis-tokenization--token-filters)
4. [Q & A](#q--a)


### Docker Commends
#### List images
```
docker image ls
```
#### Run ES in development mode
```
docker run -p 9200:9200 -e "discovery.type=single-node" -e "xpack.security.enabled=false" docker.elastic.co/elasticsearch/elasticsearch:5.5.3
```
#### List all containers (only IDs): 
```
docker ps -aq
```
#### List all running containers (only IDs): 
```
docker ps 
```
#### Stop all running containers: 
```
docker stop $(docker ps -aq)
```
#### Remove all containers: 
```
docker rm $(docker ps -aq)
```
#### Remove all images: 
```
docker rmi $(docker images -q)
```
#### Remove docker image by id
```
docker rmi 5857f98b5920(id of image)
```

### ES Syntax
GET: http://localhost:9200/inspections/_search \
GET: http://localhost:9200/_cluster/health \
POST: http://localhost:9200/inspections/doc (Content-Type:application/json)
```json
{
    "business_address": "10000 Lucy St",
	"business_city":"San Francisco",
	"business_location": {
		"type": "Point",
		"coordinates": [-99, 100]
	}
}
```

GET: http://localhost:9200/_search (get result from across all indexes)
```json
{
	"query": {
		"range": {
			"inspection_score": {"gte":0}
		}
	}
}

result shown below:
{
    //...
    "hits": {
        "total": 2, // which means 2 documents are found
        //...
    }
}
```

curl -XPOST "localhost:9200/shakespeare/_bulk?pretty" -H 'Content-Type: application/json' --data-binary @test.json
```save contents in the following format in a file, in this case, the file is called test.json
{"index":{"_index":"shakespeare_line","_type":"line","_id":10}}
{"line_id":142,"play_name":"Henry IV","speech_number":7,"line_number":"1.2.28","speaker":"FALSTAFF","text_entry":"being governed, as the sea is, by our noble and"}
```

GET: http://localhost:9200/inspections/_search
```json
{
	"query": {
		"match_phrase": {
			"business_address": "16161 hello"
		}
	}
}
```


GET: http://localhost:9200/inspections/_search (bool query -> conditions in must array must be all true)
```json
{
	"query": {
		"bool": {
			"must": [
				{
					"match": {"business_address": "Lucy"}
				},
				{
					"match_phrase": {"business_city": "San Francisco"}
				}
			]
		}
	}
}
```


GET: http://localhost:9200/inspections/_search (ranage query)
```json
{
	"query": {
		"range": {
			"inspection_score": {"gte":3}
		}
	},
	"sort": [{"inspection_score": "desc"}]
}
```


GET: http://localhost:9200/inspections/_search (ranges key 0-2 means from 0 up to 1, 2 is not included)
```json
{
	"query": {
		"range": {
			"inspection_score": {"gte":0}
		}
	},
	"sort": [{"inspection_score": "desc"}],
	"aggregations": {
		"inspection_score": {
			"range": {
				"field": "inspection_score",
				"ranges": [
					{
						"key": "0-2",
						"from": 0,
						"to": 2
					},
					{
						"key": "2-5",
						"from": 2,
						"to": 5
					}
				]
			}
		}
	}
}
```


Sort based on geo distance with the coordinates 
```json
DELETE: http://localhost:9200/inspections
PUT: http://localhost:9200/inspections
PUT: http://localhost:9200/inspections/doc/_mapping
{
    "properties": {
        "business_address": {
            "type": "text",
            "fields": {
                "keyword": {
                    "type": "keyword",
                    "ignore_above": 256
                }
            }
        },
        "business_location": {
            "properties": {
                "coordinates": {
                    "type": "geo_point"
                },
                "type": {
                    "type": "text",
                    "fields": {
                        "keyword": {
                            "type": "keyword",
                            "ignore_above": 256
                        }
                    }
                }
            }
        },
        "inspection_score": {
            "type": "long"
        }
    }
}
POST: http://localhost:9200/inspections/_bulk
{ "index" : { "_id" : 1, "_type":"doc"}}
{ "business_address": "999 Lucy St", "inspection_score": 1, "business_location": {"type":"point","coordinates": [-122.481299, 34.547228]}}
{ "index" : { "_id" : 2, "_type":"doc"}}
{ "business_address": "1818 Lucy St", "inspection_score": 2, "business_location": {"type":"point","coordinates": [-122.481299, 36.547228]}}
{ "index" : { "_id" : 3, "_type":"doc"}}
{ "business_address": "999 Lucy St", "inspection_score": 3, "business_location": {"type":"point","coordinates": [-122.481299, 38.547228]}}
{ "index" : { "_id" : 4, "_type":"doc"}}
{ "business_address": "1818 Lucy St", "inspection_score": 4, "business_location": {"type":"point","coordinates": [-122.481299, 40.547228]}}
GET: http://localhost:9200/inspections/_search
{
	"query": {
		"match": {"business_address": "lucy"}
	},
	"sort": [{
		"_geo_distance": {
			"business_location.coordinates": {
				"lat": 31.547228,
				"lon": -122.481299
			},
			"order": "asc",
			"unit": "km",
			"distance_type": "plane"
		}
	}]
}
```

Update one document by id
```json
POST: http://localhost:9200/inspections/doc/1/_update
{
	"doc": {
		"business_address": "88 Blah St",
		"inspection_score": 2,
		"has_pet": false
	}
}
```

Delete one document by id
```json
DELETE: http://localhost:9200/inspections/doc/1
{
	"doc": {
		"business_address": "88 Blah St",
		"inspection_score": 2,
		"has_pet": false
	}
}
```

### Text Analysis (tokenization + token filters)
You can use the analyze API to see how text is analyzed. Specify which analyzer to use in the query-string parameters, and the text to analyze in the body:

Standard
```json
GET: http://localhost:9200/inspections/_analyze
{
	"tokenizer": "standard",
	"text": "my email address test123@company.com"
}
```

Whitespace
```json
GET: http://localhost:9200/inspections/_analyze
{
	"tokenizer": "whitespace",
	"text": "my email address test123@company.com"
}
```

Applying filters
```json
GET: http://localhost:9200/inspections/_analyze
{
	"tokenizer": "standard",
    "filter": ["lowercase", "unique"],
	"text": "my email address Email@Address"
}
```

### Q & A
1. Why use ES?
    * It gives you the ability to not only save data, but also explore the data.
    * Document orientated.
    * Schema free.
    * DISTRIBUTED SYSTEM
        * __using nodes, each node is a single server, participate in the cluster's indexing and search capabilities__
        * __using clusters, a cluster is a collection of one or more nodes that hold the entire data together__
2. What is index and type in ES?
    * Index is the category of a collection of files.
    * Type is defined for documents that have a set of common fields. You can define more than one type in your index.
    * ex: you have index "Movies" and type could be movie description, movie crew info ...
3. What is Shard?
    * ES provides the ability to subdivide the index into multiple pieces called shards. Each shard is in itself in a fully-functional and independent "index" that can be hosted on any node within the cluster
4. What is replica?
    * ES allows you to make one or more copies of your index's shards which are called replica shards or shards.
5. __What is inverted index__?
    * Elasticsearch uses a structure called an inverted index, which is designed to allow very fast full-text searches. An inverted index consists of a list of all the unique words that appear in any document, and for each word, a list of the documents in which it appears.
6. __How to create an inverted index__?
    * let’s say we have two documents, each with a field containing the following:
        * The quick brown fox jumped over the lazy dog
        * Quick brown foxes leap over lazy dogs in summe
    * we split the content field of each document into separate words (which we call terms, or tokens), create a sorted list of all the unique terms, and then list in which document each term appears. The result looks something like this:
    ```
        Term      Doc_1  Doc_2
        -------------------------
        Quick   |       |  X
        The     |   X   |
        brown   |   X   |  X
        dog     |   X   |
        dogs    |       |  X
        fox     |   X   |
        foxes   |       |  X
        in      |       |  X
        jumped  |   X   |
        lazy    |   X   |  X
        leap    |       |  X
        over    |   X   |  X
        quick   |   X   |
        summer  |       |  X
        the     |   X   |
        ------------------------
    ```
    * we normalize the terms into a standard format, ie Quick can be lowercased to become quick, jumped and leap are __synonyms__ and can be indexed as just the single term jump, foxes can be stemmed (reduced to its root form "fox"). Similarly, dogs could be stemmed to dog. The result looks something like this: 
    ```
        Term      Doc_1  Doc_2
        -------------------------
        brown   |   X   |  X
        dog     |   X   |  X
        fox     |   X   |  X
        in      |       |  X
        jump    |   X   |  X
        lazy    |   X   |  X
        over    |   X   |  X
        quick   |   X   |  X
        summer  |       |  X
        the     |   X   |  X
        ------------------------
    ```
    * search for +quick +fox will match both documents!
    * __Notes__: Both the indexed text and the query string must be normalized into the same form.
7. __What is analysis__? 
    * Analysis is a process that consists of the following:
        * First, tokenizing a block of text into individual terms suitable for use in an inverted index,
        * Then normalizing these terms into a standard form to improve their “searchability,”, for example, word "foxes" is normalized to "fox", word "Quick" is normalized to "quick"
8. What is analyzer? 
    * An analyzer does the analysis job.
    * It contains charater filters, tokenizer, and token filters.
    * *__Charater filters__*: first, the string is passed through any character filters in turn. A character filter could be used to strip out HTML, or convert & characters to the word and
    * *__Tokenizer__*: next, the string is tokenized into individual terms by a tokenizer. A simple tokenizer might split the text into terms whenever it encounters whitespace or punctuation.
    * *__Token filters__*: Last, each term is passed through any token filters in turn, which can change terms (lowercase terms), remove terms (remove a/and/the), or add terms (synonyms like jump and leap)
    * The standard analyzer is the default analyzer that Elasticsearch uses.
    * You can use the analyze API to see how text is analyzed. Specify which analyzer to use in the query-string parameters, and the text to analyze in the body: 
        ```
        GET: http://localhost:9200/_analyze
        {
            "analyzer": "english",
            "text": "The quick brown fox jumped over the lazy dog leap buying"
        }
        ```
    * When ES detects a new string field in your documents, it automatically configures it as a full-text string field and analyzes it with the standard analyzer. If you do not want this to happen, you need to specify the mapping manually.
9. How does mapping work?
6. How is data stored?
5. Why is ES better than MongoDB?
6. How is score calculated?
7. What does "keyword" type mean and "text" type mean?
```json
{
    "inspections": {
        "mappings": {
            "dock": {
                "properties": {
                    "business_location": {
                        "properties": {
                            "coordinates": {
                                "type": "long"
                            },
                            "type": {
                                "type": "text",
                                "fields": {
                                    "keyword": {
                                        "type": "keyword",
                                        "ignore_above": 256
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```
8. How does search work based on different type? 
9. The query in syntax section, which is for sort based on geo distance with the coordinates, can only bulk edit 3 items at a time. WHY?
